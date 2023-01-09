---
layout: post
title:  "Migrating existing Terraform-managed Azure Virtual Machines that aren’t in an availability zone to an availability zone"
date:   2022-06-30 08:47:29 +0000
---
Hi team, I wanted to write about a particular script I write over Christmas that helped to cut down the manual work required to move a whole raft of virtual machines to availability zones.

## Situation

We had multiple frontend VMs in a load-balanced configuration but not in availability zones.

## Task

Migrate these VMs across the Availability Zones in the Azure Region to take advantage of the (basically free - since we're already paying for the VMs) zonal redundancy

## The problem

The Azure Resource Mover only supports cross-regional resource movement. Within a particular Azure region, only AZ-AZ regional migration is supported via Site Recovery. There is no native support for non-AZ to AZ VM migrations.

Your company also uses basic Terraform modules to manage the configuration of the VMs, which means a lot of post-migration Terraform changes if we follow the idea of snapshotting and recreating the resource as detailed in [kpantos's] blog post.

Your company also uses Terraform, specifically the azurerm_linux_virtual_machine module which does not support attaching OS disks.
To minimize the amount of variable refactoring in Terraform, I've written a PowerShell script that will perform the following tasks:

- Stop the VM
- Detach, snapshot, and restore all data disks to the desired zone (As data disks need to be in the same zone as the VM object to be eligible to attach)
- Create a new image with Azure Compute Gallery (Set this up beforehand)
- Delete the original VM
- Create the new VM based on the image, in the specified zone, having the same configuration as the original
- Attaches NIC and restored disks.

After running the script across the machines (one at a time under strict change control of course. I'm not a cowboy - yee haw!), on the Terraform side the following need to be changed so Terraform doesn't delete your resource - The below is pretty generic advice that applied to our Terraform module which is quite basic - so while this was all I required, your mileage may vary:

- Any availability sets are not compatible with availability zones, so remove this resource type from the resource module
- A `zone` variable will need to be added for the VM and the data disk modules
- A `"source_image_id"` will need to be added for the VM module (obtainable from the resource "JSON view" on the "Overview" page - under `"imageReference"`)
- The `"source_resource_id"` variable will need to be added for the data disk module
- The `createOption` of the OsDisk will need to be `"FromImage"`
- The `createOption` of the data disk will need to be `"Copy"`

After the above changes, the Terraform plan running on this resource should only plan changes related to extensions, tags, and diagnostics settings, applying these will bring your VM back to a managed state.
The average runtime of the script is around 35 minutes per machine from start to finish - though you may want to run it in sections
The script is available at my GitHub page linked [azmigration-script]

[kpantos's]: https://blog.pantos.name/2019/10/15/move-an-azure-vm-to-an-availability-zone/
[azmigration-script]: https://github.com/RawPatty/Azure-Scripts/blob/main/AZ%20Migration/azmigration.ps1