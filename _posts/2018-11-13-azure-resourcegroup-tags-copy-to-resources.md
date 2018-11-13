---
layout: post
title: "Azure ResourceGroup tags copy to resources"
subtitle: "A script to copy all the tags from the ResourceGroup down to the resources"
tags: [Azure, Tags, Powershell]
---

The following script is an example on how you can manage your Tags in Azure.  
The tags on the Resource Group (RG) are leading for all the other resources inside the RG.

So what it does is copy all the Tags on the RG down to all the resources.  
I decided not to append the tags, because then the old tags are not removed.  

This[ Microsoft doc page](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags) gave me the idea.

{: .box-error}
**Note:** Not all resources are able to hold tags. So in the script I added some types that gave me errors.

Like my other blog post about tags, I used an automation account to schedule it. See that [post](/posts/2018-11-13-azure-automatic-tags.md) for more info.

See that [post](/_posts/2018-11-13-azure-automatic-tags.md) for more info.

## The Script

<script src="https://gist.github.com/energetic-it/c48e146521f5e47da927ceef9ad34945.js"></script>