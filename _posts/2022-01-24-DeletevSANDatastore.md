---
layout: post
title: Delete vSAN Datastore without vCenter
published: true
tags: [VMware,vSAN]
youtubeId: BC6PHOpqvDs
---

How to delete a vSAN datastore and clear vSAN partitions from disks when no vCenter Server is available.

{% include youtubePlayer.html id=page.youtubeId %}

## Recreating the Problem

Scenario: an ESXi host is a member of a vSAN cluster, but the vCenter Server is no longer available.

Right now, the vsanDatastore is unable to be deleted. If I try to delete it, the option is greyed out.

![_config.yml]({{ site.baseurl }}/images/deletevsandatastore/datastorenodelete.jpeg)

This disk on the host is currently formatted for vSAN. If I attempt to clear the partition, it fails with the error "Failed - Cannot change the host configuration."

![_config.yml]({{ site.baseurl }}/images/deletevsandatastore/clearparterror.jpeg)

The same happens on the other disk.

To delete the datastore and clear the partitions of both disks, I'll first SSH into the host:

````bash
ssh root@nuc1.breakdown.lab
````

## Leave vSAN Cluster

The command to use for most ESXi management tasks is `esxcli`.  If you need help figuring out which options are available, you can type `--help` or simply hit Enter.

![_config.yml]({{ site.baseurl }}/images/deletevsandatastore/esxclihelp.jpg)

Before I can clear the partitions, I'll first need to leave the vSAN cluster. To list the cluster configuration, I'll use:

````bash
esxcli vsan cluster list
````

I'll then copy the Sub-Cluster Master UUID. Since it's the same as other UUIDs, you may not need this specific one.

![_config.yml]({{ site.baseurl }}/images/deletevsandatastore/copyuuid.jpeg)

To leave the cluster the command is:

````bash
esxcli vsan cluster leave -u <UUID>
````

The ESXi web console shows that the datastore is now gone.

## Removing the vSAN Partitions from Disks

Now that the host has left the cluster and the datastore has been deleted, I can remove the vSAN partitions.

Similarly to the previous commands I'll list the vSAN storage configuration first.

````bash
esxcli vsan storage list
````

This shows both UUIDs for the device and the disk group. Since both disks are in the same group, I can clear the partitions of both disks by copying the VSAN Disk Group UUID.

![_config.yml]({{ site.baseurl }}/images/deletevsandatastore/diskgroupuuid.jpeg)

The the command to remove:

````bash
esxcli vsan storage remove -u <VSANdiskgroupUUID>
````

A `esxcli vsan storage list` returns nothing, so I'll double check the web console.  I'll hit refresh to show that the partitions have now been cleared.

![_config.yml]({{ site.baseurl }}/images/deletevsandatastore/clearedpartition.jpeg)

## Conclusion

Now that the VSAN datastore is deleted and the disk partitions have been cleared, the host storage is ready to be reused.
