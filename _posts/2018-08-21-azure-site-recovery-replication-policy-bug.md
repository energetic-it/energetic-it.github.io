---
title: "Azure Site Recovery Replication Policy Bug"
excerpt: ""
categories:
  - ASR
tags: 
  - ASR
  - Azure
last_modified_at: 2018-08-20T20:00:16-05:00
---

![asr-recovery-services](/assets/images/asr-recovery-services.png)

![asr-vault](/assets/images/asr-vault.png)

![asr-vault-manage](/assets/images/asr-vault-manage.png)

![asr-vault-manage-replication-policy](/assets/images/asr-vault-manage-replication-policy.png)

![asr-vault-manage-replication-policy-add](/assets/images/asr-vault-manage-replication-policy-add.png)

![asr-vault-manage-replication-policy-create-1](/assets/images/asr-vault-manage-replication-policy-create-1.png)

![asr-vault-manage-replication-policy-create-2.png](/assets/images/asr-vault-manage-replication-policy-create-2.png)

![asr-vault-manage-replication-policy-init-time.png](/assets/images/asr-vault-manage-replication-policy-init-time.png)

```ps
# Get all the policies
Get-AzureRmRecoveryServicesAsrPolicy  | Select-Object Name -ExpandProperty ReplicationProviderSettings

```

![asr-powershell-policy-initialtime](/assets/images/asr-powershell-policy-initialtime.png)

```ps
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

![asr-powershell-policy-result](/assets/images/asr-powershell-policy-result.png)

## Extra info

Login to Azure with powershell.

```ps
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


# $vault = Get-AzureRmRecoveryServicesVault -Name $rName.Name -ResourceGroupName $rgName.ResourceGroupName
$vault = Get-AzureRmRecoveryServicesVault -Name $SiteRecovery -ResourceGroupName $RGSiteRecovery

# Creating a temp directory to store the VaultSettingsFile
$tempFileName = "c:\data\"
$FilePath = Get-AzureRmRecoveryServicesVaultSettingsFile -Vault $vault -Path $tempFileName | Where-Object -Property FilePath
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