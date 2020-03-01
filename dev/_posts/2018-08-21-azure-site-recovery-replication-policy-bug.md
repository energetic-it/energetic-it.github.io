---
layout: post
title: "Azure Site Recovery Replication Policy Bug"
subtitle: "There is a bug in ASR, when you create a new Replication Policy the Initial replication start time is created wrong"
tags: [ASR, AzureSiteRecovery, SCVMM]
redirect_from: 
  - https://energetic-it.github.io/asr/powershell/2018/08/21/azure-site-recovery-replication-policy-bug.html
  - https://energetic-it.github.io/dev/asr/powershell/2018/08/21/azure-site-recovery-replication-policy-bug.html
  - /asr/powershell/2018/08/21/azure-site-recovery-replication-policy-bug.html
  - /dev/asr/powershell/2018/08/21/azure-site-recovery-replication-policy-bug.html
---

{: .box-error}
**Update:** 01-11-2018 - This bug seems to be fixed!
When creating a new replication policy, the time is correct.

---

At work we use [Azure Site Recovery](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-overview) (ASR) for a while now. We use it primarily to synchronize the VM's between datacenters and use ASR for the orchestration.

![arch-onprem-onprem](/dev/img/arch-onprem-onprem.png)  
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

{: .box-error}
**Note:** You don't need to have anything connected to the recovery vault. Create a new Azure subscription and a Recovery Vault and click along with the instructions below.

<tr>
<td> <img src="/img/asr-recovery-services.png" alt="asr-recovery-services" style="width: 350px;"/> </td>
<td> <img src="/img/asr-vault.png" alt="asr-vault" style="width: 350px;"/> </td>
</tr>

Find the Recovery Services Vault, and click on it.

<tr>
<td> <img src="/img/asr-vault-manage.png" alt="asr-vault-manage" style="width: 350px;"/> </td>
<td> <img src="/img/asr-vault-manage-replication-policy.png" alt="asr-vault-manage-replication-policy" style="width: 350px;"/> </td>
</tr>

Go down to __Manage__ and select _Site Recovery Infrastructure_ scroll down to __For System Center VMM__ and select the Replication Policies

<tr>
<td> <img src="/img/asr-vault-manage-replication-policy-add.png" alt="asr-vault-manage-replication-policy-add" style="width: 350px;"/> </td>
<td> <img src="/img/asr-vault-manage-replication-policy-create-1.png" alt="asr-vault-manage-replication-policy-create-1" style="width: 350px;"/> </td>
</tr>

Create a new Replication Policy and select Hyper-V as a source and target.

![asr-vault-manage-replication-policy-create-2.png](/dev/img/asr-vault-manage-replication-policy-create-2.png)

Choose for the Initial replication start time something else then Immediately or 07:45PM.  
Keep the rest default values (or not, doesn't really matter).

## Result

Once the replication policy has been created, and you click back into the newly created policy, it shows 07:45 PM. Even though in this example we put in 10:00PM.  
![asr-vault-manage-replication-policy-init-time.png](/dev/img/asr-vault-manage-replication-policy-init-time.png)

Looking at it with `Powershell` we see the following:

```powershell
# Get all the policies
Get-AzureRmRecoveryServicesAsrPolicy  | Select-Object Name -ExpandProperty ReplicationProviderSettings

```

![asr-powershell-policy-initialtime](/dev/img/asr-powershell-policy-initialtime.png)

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

![asr-powershell-policy-result](/dev/img/asr-powershell-policy-result.png)

## The Script

{: .box-error}
**Note:** Don't forget to update the `Azure.RM modules`, this bit me in the beginning, my old scripts did not work anymore because of some changes.

<script src="https://gist.github.com/energetic-it/34a948153cd7b3ed1f89d4d68969976b.js"></script>

Hopefully this helps someone or creates enough awareness that Microsoft will fix it.
