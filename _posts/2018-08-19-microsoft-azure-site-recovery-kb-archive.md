---
layout: post
title: "Microsoft Azure Site Recovery - KB Archive"
subtitle: "Since the KB Articles of Microsoft Azure Site Recovery are so hard to keep track of, I'm keeping an overview for me and my co-workers."
tags: [ASR, AzureSiteRecovery]
redirect_from: 
  - https://energetic-it.github.io/post/2018/08/19/microsoft-azure-site-recovery-kb-archive.html
---

# Microsoft Azure Site Recovery - KB Archive

_[updated: 19-10-2018]_

Since the KB Articles of Microsoft Azure Site Recovery are so hard to keep track of, I'm keeping an overview for me and my co-workers.

{: .box-note}
**Update:** I just found this [page](https://azure.microsoft.com/en-us/updates/?product=site-recovery). It seems Microsoft does post (sometimes) links to the new ASR updates!

{: .box-error}
**Note:** When you install: Update Rollup 5 for System Center 2016 Virtual Machine Manager, the ASR client needs to be upgraded to version 5.1.3100 or later. Older versions are not supported!

To upgrade Microsoft Azure Site Recovery Provider (also known as DRA), follow these steps:

- Uninstall the existing version of DRA.
- Install Update Rollup 5.
- Install version 5.1.3100 or a later version of DRA.

An easy [Google Search](https://www.google.com/search?q=Update+Rollup+*+for+Azure+Site+Recovery+site:https://support.microsoft.com/en-us/help&lr=&hl=en&source=lnt&tbs=sbd:1,qdr:y&sa=X&ved=0ahUKEwiT7fKn9qrbAhVRr6QKHeXNCc0QpwUIIA&biw=1920&bih=974) to check if new releases have been made.

# Microsoft Azure Site Recovery Updates
- [Update Rollup 30 for Azure Site Recovery __(version 5.1.3650.0)__](https://support.microsoft.com/en-us/help/4468181/azure-site-recovery-update-rollup-30) [No agent update]
- [Update Rollup 29 for Azure Site Recovery __(version 5.1.3650.0)__](https://support.microsoft.com/en-us/help/4466466/update-rollup-29-for-azure-site-recovery)
- [Update Rollup 28 for Azure Site Recovery (version 5.1.3600.0)](https://support.microsoft.com/en-us/help/4460079/update-rollup-28-for-azure-site-recovery)
- [Update Rollup 27 for Azure Site Recovery (version 5.1.3550.0)](https://support.microsoft.com/en-us/help/4055712/update-rollup-27-for-azure-site-recovery)
- [Update Rollup 26 for Azure Site Recovery (version 5.1.3400.0)](https://support.microsoft.com/en-us/help/4344054/update-rollup-26-for-azure-site-recovery)
- [Update Rollup 25 for Azure Site Recovery (version 5.1.3300.0)](https://support.microsoft.com/en-us/help/4278275/update-rollup-25-for-azure-site-recovery)
- (what happend to 24?)
- [Update Rollup 23 for Azure Site Recovery (version 5.1.3100.0)](https://support.microsoft.com/en-us/help/4091311/update-rollup-23-for-azure-site-recovery)
- [Update Rollup 22 for Azure Site Recovery (version 5.1.2900.0)](https://support.microsoft.com/en-us/help/4072852/update-rollup-22-for-azure-site-recovery)
- [Update Rollup 21 for Azure Site Recovery (version 5.1.2700.1)](https://support.microsoft.com/en-us/help/4051380/update-rollup-21-for-azure-site-recovery)

# Microsoft Azure Site Recovery Changes

{: .box-note} 
Here I'll only highlight the changes that are odd of noteworthy:

There is no agent update from Update 29 to 30. Most of the changes are made server side.