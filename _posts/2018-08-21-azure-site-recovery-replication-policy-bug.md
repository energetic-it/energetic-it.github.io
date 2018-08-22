---
title: "Azure Site Recovery Replication Policy Bug"
excerpt: "There is a bug in ASR, when you create a new Replication Policy the Initial replication start time is created wrong"
comments: true
categories:
  - ASR
  - Powershell
tags: 
  - ASR
  - Azure
  - Powershell
last_modified_at: 2018-08-20T20:00:16-05:00
---

At work we use [Azure Site Recovery](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-overview) (ASR) for a while now. We use it primarily to synchronize the VM's between datacenters and use ASR for the orchestration.

![arch-onprem-onprem](/assets/images/arch-onprem-onprem.png)  
[Image source](https://docs.microsoft.com/en-us/azure/site-recovery/hyper-v-vmm-architecture)

Not that long ago I rebuild our test ASR environment after some updates and came across a weird issue.

After creating a new replication policy, __nothing__ worked anymore. At first I figured it was due to the update of the agents on the System Center Virtual Machine Manager (SCVMM) servers.

In comes `Powershell` to the rescue :)

## TL;DR

- This issue is as of this date still unresolved by Microsoft.
- When a new Replication Policy is created (through the portal), the Initial replication start time is created wrong.
- As far as I know the only way to fix this, is to create the replication policy through `Powershell`.
- __The impact__ is that the Replication Policy breaks everything if the initial time to synchronize isn't correct.
  - It won't synchronize and it will give errors that don't point towards the replication policy.
  - VM's will not replicate, ASR will not work.

## The Setup

I'll go into the details and show you the screenshots on how to create a replication policy.

> Note: You don't need to have anything connected to the recovery vault.  
> Create a new Azure subscription and a Recovery Vault and click along with the instructions below.

| ||
:-------------------------:|:-------------------------:
![asr-recovery-services](/assets/images/asr-recovery-services.png) | ![asr-vault](/assets/images/asr-vault.png)

Find the Recovery Services Vault, and click on it.

| ||
:-------------------------:|:-------------------------:
![asr-vault-manage](/assets/images/asr-vault-manage.png) | ![asr-vault-manage-replication-policy](/assets/images/asr-vault-manage-replication-policy.png)

Go down to __Manage__ and select _Site Recovery Infrastructure_ scroll down to __For System Center VMM__ and select the Replication Policies

| ||
:-------------------------:|:-------------------------:
![asr-vault-manage-replication-policy-add](/assets/images/asr-vault-manage-replication-policy-add.png) | ![asr-vault-manage-replication-policy-create-1](/assets/images/asr-vault-manage-replication-policy-create-1.png)

Create a new Replication Policy and select Hyper-V as a source and target.

| ||
:-------------------------:|:-------------------------:
![asr-vault-manage-replication-policy-create-2.png](/assets/images/asr-vault-manage-replication-policy-create-2.png) |

Choose for the Initial replication start time something else then Immediately or 07:45PM.  
Keep the rest default values (or not, doesn't really matter).

## Result

Once the replication policy has been created, and you click back into the newly created policy, it shows 07:45 PM. Even though in this example we put in 10:00PM.  
![asr-vault-manage-replication-policy-init-time.png](/assets/images/asr-vault-manage-replication-policy-init-time.png)

Looking at it with `Powershell` we see the following:

```powershell
# Get all the policies
Get-AzureRmRecoveryServicesAsrPolicy  | Select-Object Name -ExpandProperty ReplicationProviderSettings

```

![asr-powershell-policy-initialtime](/assets/images/asr-powershell-policy-initialtime.png)

The time is set to something, except the time what we want.

I tested these replication policies and when in use the VM do __not__ replicate from one Hyper-V Host to the other Hyper-V Host.

## The Solution

The only way I found on how to fix this, is to create a Replication Policy through `Powershell`.
At the bottom of the blog you can see the script I used.

The thing that took me a little bit was to find out how to use the ReplicationStartTime.
When you check out the [docs page](https://docs.microsoft.com/en-us/powershell/module/azurerm.recoveryservices.siterecovery/new-azurermrecoveryservicesasrpolicy) it will tell you this:

### -ReplicationStartTime

Specifies the replication start time.
It must be no later than 24-hours from the start of the job.

```yaml
Type: [System.TimeSpan]
Parameter Sets: HyperVToAzure, EnterpriseToEnterprise
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

Some searching around I found the `New-TimeSpan` with an Output of `System.TimeSpan`

This resulted in the following splat:

```powershell
$NamePolicy = Read-Host -Prompt 'Give the name of the new policy'

$Time = New-TimeSpan -Hour 22 -Minute 00
$PolicySplat = @{
    VmmToVmm                        = $true
    ReplicationProvider             = 'HyperVReplica2012R2'
    Name                            = $NamePolicy
    ReplicationFrequencyInSeconds   = '900'
    NumberOfRecoveryPointsToRetain  = '1'
    ApplicationConsistentSnapshotFrequencyInHours = '1'
    Authentication                  = 'Certificate'
    ReplicationPort                 = '8083'
    Compression                     = 'Enable'
    ReplicaDeletion                 = 'Required'
    ReplicationMethod               = 'Online'
    ReplicationStartTime            = $Time
}

New-AzureRmRecoveryServicesAsrPolicy @PolicySplat
```

Looking at both of the created Replication Policies, you can see clearly the differences.

![asr-powershell-policy-result](/assets/images/asr-powershell-policy-result.png)

## The Script

> Note: don't forget to update the `Azure.RM modules`, this bit me in the beginning, my old scripts did not work anymore because of some changes.

```powershell
#Requires -Modules AzureRM.profile, AzureRM.RecoveryServices, AzureRM.RecoveryServices.SiteRecovery

# Update the modules to the latest version
Update-Module AzureRM.profile, AzureRM.RecoveryServices, AzureRM.RecoveryServices.SiteRecovery -Force

# Load the modules
Import-Module AzureRM.profile, AzureRM.RecoveryServices, AzureRM.RecoveryServices.SiteRecovery

# Login to Azure
Connect-AzureRmAccount

# Select an Azure Subscription
$SubscriptionID =  (Get-AzureRmSubscription | Select-Object Name, SubscriptionID | Out-GridView -PassThru).SubscriptionID
Select-AzureRmSubscription -SubscriptionId $SubscriptionID

# Give the ASR Vault name and the Resource Group Name of the Vault
$SiteRecovery = "Test-Vault"
$RGSiteRecovery = "Test-Vault"

# Store the vault settings in the variable
$vault = Get-AzureRmRecoveryServicesVault -Name $SiteRecovery -ResourceGroupName $RGSiteRecovery

# Creating a temp directory to store the VaultSettingsFile
$tempFileName = "c:\data\"
$FilePath = Get-AzureRmRecoveryServicesVaultSettingsFile -Vault $vault -Path $tempFileName | Where-Object -Property FilePath

# Importing the vault settingFile
Import-AzureRmRecoveryServicesAsrVaultSettingsFile -Path $FilePath.FilePath

# Get all the policies
Get-AzureRmRecoveryServicesAsrPolicy  | Select-Object Name -ExpandProperty ReplicationProviderSettings

# Create a new one
$NamePolicy = Read-Host -Prompt 'Give the name of the new policy'

$Time = New-TimeSpan -Hour 22 -Minute 00
$PolicySplat = @{
    VmmToVmm                        = $true
    ReplicationProvider             = 'HyperVReplica2012R2'
    Name                            = $NamePolicy
    ReplicationFrequencyInSeconds   = '900'
    NumberOfRecoveryPointsToRetain  = '1'
    ApplicationConsistentSnapshotFrequencyInHours = '1'
    Authentication                  = 'Certificate'
    ReplicationPort                 = '8083'
    Compression                     = 'Enable'
    ReplicaDeletion                 = 'Required'
    ReplicationMethod               = 'Online'
    ReplicationStartTime            = $Time
}

New-AzureRmRecoveryServicesAsrPolicy @PolicySplat
```

Hopefully this helps someone or creates enough awareness that Microsoft will fix it.
