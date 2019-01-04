---
layout: post
title: "Azure VM costs in-depth"
subtitle: "What factores contribute the Azure VM Costs?"
tags: [Azure]
redirect_from:
  - https://energetic-it.github.io/2018-12-07-azure-vm-costs-in-depth/
---

# Azure VM Costs Explained

How much does (something in) Azure cost? Is a question we get a lot from customers.
Of course this is not an easy question and not an easy answer.

I'm going to post a few blogs about it hoping it will make more sense. Starting with an example.

## How much does a VM cost in Azure

In an other post we'll dive into how to make it cheaper.

{: .box-error}
**Note:** Reading this post will give you insight on how Microsoft charges in Azure. For specific information, see the [Azure Pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/)

So lets break things down in what we already know. The following make up an Azure VM:

- Compute costs
- OS use costs
- Storage costs
- Application costs
- Other consumption costs

# Lets dive into the list

Most charges related with Azure VMs are billed under an hourly pay-as-you-go model.
The examples I'm giving are based upon a 730-hour month. These prices do not have any additional discounts (EA customers etc)

## Compute

All VMs are assessed an hourly compute charge to cover the use of the hardware in Microsoft's datacenter.
Azure offers multiple series of VMs available in a range of predefined 'sizes'.
Each size consists of specific number of virtual cores running on a specific class of hardware, memory (RAM) and temporary storage.

For example, a DS16s V3 VM is a size within the DS version 3 series.
The full compute price for this VM, running continuously for a full month is $561 (dollar).
When you see a VM price, sometimes the Windows license is already included, the real price of compute can be hidden.

The DS v3 series run servers with Intel Xeon E5-2673 v3 2.4GHz. The DS16s v3 VM size has 16 virtual cores, 64GB RAM and 400gb of temporary storage (used to save the system paging file)

Within any VM series the compute fee is pretty linear; a VM with twice the size of virtual cores as another within the same series usually costs twice as much.

If you want to reduce your compute resource rates, check out Azure Reserved VM Instances (more on that in another post)

[Link](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes)

## OS use

Azure VMs can run Windows Server (Standard or Datacenter edition) as well as Linux distributions.
Use of Windows Server or certain Linux distributions (Red Hat enterprise) adds to the cost of the VM, while free distributions (like CentOSor Ubuntu Linux) do not.

For example, use of Windows Server (either edition) within a D16s v3 VM running for a month adds $537 (dollar) on top of the compute costs, that brings it $1.098 per month.

Within the same VM series, the Windows rental fee is usually based on the VM size. However, OS renatal fees for VMs with the same numbers of virtual cores but in a different VM series can vary widely, with per-virtual-core OS rental fees for the high-end VM series costing as much as 6 times more than for a low-end VM series!

Customers with a Software Assurance (SA) on Windows Server can use a benefit called "Azure Hybrid Benefit for Windows Server".
With this, you can use these licenses to avoid having to pay for Azure fees for Windows Server OS rental, resulting in an overal cost saving. (more on that in another post).

[Link](https://azure.microsoft.com/en-us/pricing/hybrid-benefit/)

## Storage

A VM must include an OS drive so that the VM can have data through reboots, start/stop or other life cycle events.

For example, a single OS drive of 128GB, defined as a Premium (SSD) P10 disk, has a monthly cost of $17,92, which you have to pay whether or not the associated VM is on or not. Many VMs need additional data disks for their workloads.

For example, If SQL Server VM has two additional Premium (SSD) P30 managed disks of 1 TB each, used for SQL data and SQL logs, there would be an additional monthly charge of $246. You can not have any discounts for these charges

## Application

Customers can run Microsoft server applications within an Azure VM, Microsoft SQL Server is one of the most common used.

For example, renting the use of SQL Server Standard or Datacenter within a "D16s v3" VM running through the month adds $1.168 or $4.917 to the overal monthly VM cost. You also have these costs if you use a free Linux distribution.

Customers with a Software Assurance (SA) on Windows Server can use a benefit called "Azure Hybrid Benefit for SQL Server".
With this, you can use these licenses to avoid having to pay for Azure fees for SQL Server rental, resulting in an overal cost saving. License Mobility through SA has a similar effect. (more on that in another post).

## Other consumption

Small incremental charges are also assessed for each IP address used by each Azure VM.
Additional charges may also be assessed for data egress (data going from one point going to another) from each Azure subscription after the first 5GB per month, as well as other Azure services, or use of static IP addresses that may be required. 

For such consumption charges there are no discounts.