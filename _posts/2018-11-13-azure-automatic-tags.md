---
layout: post
title: "Automatic tagging within Azure subscription"
subtitle: "An automatic way to tag any resource created."
tags: [Azure, Tags, Powershell]
---

After realising the importanct of tagging within Azure, I went searching for solutions to help out.
The one thing that annoyed me most is that my collegues did not use Tags and after a few months we had a hard time finding out who created what and if we could turn it off. 

So first thing:
- Create an Automation Account
- Update Azure Modules
- Add the AzureRM.Insights Module
- Add Runbook
- (copy mine of create your own script)
- Import an existing runbook
  - Import your powershell script
- Run a Test
- Save and Publish 
- Run the script to see if it works

Some screenshots:

Update Azure Modules
<tr>
<td> <img src="/dev/img/azure-modules-update.png" alt="azure-modules-update" style="width: 500px;"/> </td>
</tr>

Add the AzureRM.Insights Module from the Gallery
<tr>
<td> <img src="/dev/img/azure-modules-gallery.png" alt="azure-modules-gallery" style="width: 350px;"/> </td>
<td> <img src="/dev/img/azure-modules-import.png" alt="azure-modules-import" style="width: 350px;"/> </td>
</tr>

Inspired by [Jason Spoon](http://jasonpoon.ca/tagging-azure-resource-group-with-owners/) I ended up with this script:

{: .box-error}
**Note:** Change the tag you want added, in this script the tag "CreatedBy" is used.  Also adjust the "@microsoft.com" with your own domain name to only get the username.

```ps
<#
    .DESCRIPTION
        Gets all the ARM resourcesGroups using the Run As Account (Service Principal) and add the CreatedBy tag
		Removes "@microsoft.com" and adds the remaining name to the CreatedBy tag
    .NOTES
        AUTHOR: Ralf
        LASTEDIT: Nov 13, 2018
#>

$connectionName = "AzureRunAsConnection"
try {
    # Get the connection "AzureRunAsConnection "
    $servicePrincipalConnection = Get-AutomationConnection -Name $connectionName         

    "Logging in to Azure..."
    Add-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
}
catch {
    if (!$servicePrincipalConnection) {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    }
    else {
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

#Get all ARM resources from all resource groups
$rgs = Get-AzureRmResourceGroup

# Make an array of resources with and without the CreatedBy Tag
$nocreatedbyrg = @()
$createdbyrg = @()
foreach ($rg in $rgs) {
    
    foreach ($tag in $rg.Tags) {
        if ($tag.Keys -eq "CreatedBy") {
            $createdbyrg += $rg.ResourceGroupName
        }
        else {
            $nocreatedbyrg += $rg.ResourceGroupName
        }

    }
}

# Check the owner through the Logs and set the Tag
$notAliasedRGs = $nocreatedbyrg

foreach ($rg in $notAliasedRGs) {
    $currentTime = Get-Date

    $endTime = $currentTime.AddDays(-7 * $cnt)
    $startTime = $endTime.AddDays(-40)
        
    $callers = Get-AzureRmLog -ResourceGroup $rg -StartTime $startTime -EndTime $endTime -WarningAction SilentlyContinue |
        Where {$_.Authorization.Action -eq "Microsoft.Resources/deployments/write" -or $_.Authorization.Action -eq "Microsoft.Resources/subscriptions/resourcegroups/write" } | 
            Select-Object -ExpandProperty Caller | 
                Group-Object |  
                    Sort-Object  | 
                        Select-Object -ExpandProperty Name

    if ($callers) {
        $owner = $callers | Select-Object -First 1
        $alias = $owner -replace "@microsoft.com", ""
            
        $tags = (Get-AzureRmResourceGroup -Name $rg).Tags
        $tags += @{ "CreatedBy" = $alias }

        $rg + ", " + $alias
        if (-not $dryRun) {
            Set-AzureRmResourceGroup -Name $rg -Tag $tags
            # Write-Host $rg "using this tag:" 
        }
    } 
    else {
        $rg + ", Unknown"
    }   
}
```

[Github link](https://gist.github.com/energetic-it/87ecbd1ffa428aed7abadc0d6d74b62d)

<script src="https://gist.github.com/energetic-it/87ecbd1ffa428aed7abadc0d6d74b62d.js"></script>