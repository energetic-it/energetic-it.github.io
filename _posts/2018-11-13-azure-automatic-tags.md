---
layout: post
title: "Automatic tagging within Azure subscription"
subtitle: "An automatic way to tag any resource created."
tags: [Azure, Tags, Powershell]
---

After realising the importance of tagging within Azure, I went searching for solutions to help out.
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
<td> <img src="/img/azure-modules-update.png" alt="azure-modules-update" style="width: 500px;"/> </td>
</tr>

Add the AzureRM.Insights Module from the Gallery
<tr>
<td> <img src="/img/azure-modules-gallery.png" alt="azure-modules-gallery" style="width: 350px;"/> </td>
<td> <img src="/img/azure-modules-import.png" alt="azure-modules-import" style="width: 350px;"/> </td>
</tr>

Inspired by [Jason Spoon](http://jasonpoon.ca/tagging-azure-resource-group-with-owners/) I ended up with this script:

{: .box-error}
**Note:** Change the tag you want added, in this script the tag "CreatedBy" is used.  Also adjust the "@microsoft.com" with your own domain name to only get the username.

<script src="https://gist.github.com/energetic-it/87ecbd1ffa428aed7abadc0d6d74b62d.js"></script>