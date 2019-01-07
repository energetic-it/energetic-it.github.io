---
layout: post
title: "Automatic tagging within Azure subscription"
subtitle: "An 'automagic' way to tag any created resource."
tags: [Azure, Powershell]
---

After working with Azure for some time and realizing the importance of tagging within Azure, I went searching for solutions to help out.

The one thing that drove me to concentrate on automatic tags were my colleagues that did not use Tags. So after a few months we had a hard time finding out who created what.

That drove me to a blog post by [Jason Spoon](http://jasonpoon.ca/tagging-azure-resource-group-with-owners/). Soon after I created my own version of the script and implemented the following:

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
- Set up the scheduling

Some screenshots:

Update Azure Modules
<tr>
<td> <img src="/img/azure-modules-update.png" alt="azure-modules-update"/> </td>
</tr>

Add the AzureRM.Insights Module from the Gallery
<tr>
<td> <img src="/img/azure-modules-gallery.png" alt="azure-modules-gallery" style="width: 350px;"/> </td>
<td> <img src="/img/azure-modules-import.png" alt="azure-modules-import" style="width: 350px;"/> </td>
</tr>

{: .box-error}
**Note:** Change the tag you want added, in this script the tag "CreatedBy" is used.  Also adjust the "@microsoft.com" with your own domain name to only get the username.

{: .box-error}
**Note:** The logs can only go back max 90 days, so it won't find older resources :(. Adjust the "startTime" in the script to get more results.

<script src="https://gist.github.com/energetic-it/87ecbd1ffa428aed7abadc0d6d74b62d.js"></script>