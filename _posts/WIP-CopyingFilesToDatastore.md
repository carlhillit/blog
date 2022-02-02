---
layout: post
title: Copying Files to Remote VMware Host
published: false
tags: [PowerCLI,VMware]
---

How to copy files to ESXi host with scp and to a remote datastore with PowerCLI.

## Copying Files to/between ESXi host(s) with SCP





## Copying Files to Remote VMFS Datastore with PowerCLI

Since the datastores are viewable from sn SSH session by way of the `/vmfs/volumes` directory, it's easy to think that you can SCP a file to the datastore.

However, SCP cannot be used to copy a file to a datastore. If you were to try, the file would be corrupted on the datastore. It's not noticeable at first glance, but a mismatch of file size may clue you in.

A file hash shows the true story: that the copied file is not the same source file :( <!-- Initial testing shows this is not the case. The shs256sum both match after scp. Check to see if this is true only for vsan datastores and not with vmfs datastores -->







![_config.yml]({{ site.baseurl }}/images/deployova/invaliduri.jpg)

The only supported option is to [connect to the vCenter server](https://developer.vmware.com/docs/powercli/latest/vmware.vimautomation.core/commands/connect-viserver/#Default).

