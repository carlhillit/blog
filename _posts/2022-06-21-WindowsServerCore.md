---
layout: post
title: Windows Server Core
published: true
tags: [PowerShell,'Server Core]'
youtubeId: _dZAneAmRU
---

How to install and provision Windows Server Core 2022.

{% include youtubePlayer.html id=page.youtubeId %}

# Contents

1. [Why Windows Server Core?](#why-windows-server-core)
2. [Creating the VM in vCenter](#creating-the-vm)
3. [Installing Windows Server Core](#installing-windows-server-core)
4. [Initial Configuration](#initial-configuration)
5. [Formatting Disks](#formatting-disks)
6. [Conclusion](#conclusion)

## Why Windows Server Core

Microsoft marketing states that Server Core is preferred over the GUI version because it has a smaller footprint[^1].
While this is correct, I've had a difficult time convincing others that those benefits are actually *worth* the steeper learning curve and added time it will take to troubleshoot a server without a GUI, especially for someone that is not very confident with their PowerShell skills.

While I haven't seen much research or real-world data, the smaller footprint being a reduced attack surface theoretically checks out.
Windows Server Core does save a bit of disk space, but those small savings can be overpowered by whatever else the server is running, so noticing the resource savings can be hard to do sometimes.

With these benefits, Server Core is a great option for servers that are normally managed by an MMC snap-in or a web console.
Examples include:

* Domain Controllers
* DNS servers
* DHCP servers
* DFS servers

### Large Environment

At scale, the reduced risk because of a smaller footprint is multiplied hundreds of times, so there are measurable benefits.
With hundreds of servers, the person(s) managing those will likely be well versed in PowerShell and how to manage servers at scale, so Server Core is an obvious choice.

### Medium Environment

This is is where the comfort level of the administrator really determines which version to use.
At this scale, the performance difference isn't worth it if the admin feels more comfortable with the GUI.

### Lab Environment

I **highly recommend** using Server Core for lab environments.
These are usually very minimal in terms of compute power and will benefit from a smaller footprint.

The bigger benefit, in my opinion, is that it forces you to learn how to manage a Windows server with remote management tools like Server Manager or [Windows Admin Center](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/overview).
Without a GUI, you don't have the temptation to use the VM console or RDP when you need to accomplish some random task.
This is particularly true when troubleshooting or provisioning a new machine.

TLDR; Server Core is great for learning PowerShell.
Without the GUI it keeps you from using the path of least resistance and teaches you to treat your servers like [cattle, not pets](http://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/).



## Creating the VM

I'll briefly go over the steps to create the VM in vCenter to show the VM's configuration. Chapters are available to skip to the desired portion of the video.

### Select a compute resource & Select storage

I'll creating a new Virtual Machine on the NVME datastore on host #3.

### Select a guest OS

Select **Microsoft Windows Server 2022 (64-bit)** for the Guest OS Version and check **Enable Windows Virtualization Based Security**.

### Customize hardware

Click the drop-down on the hard disk and change the Disk Provisioning to **Thin Provision**.

Then add two more thin provisioned Hard Disks of 10GB each for Logs and other data as it is a good security practice to keep those separate from the OS disk.

Change the Network Adapter Type form E1000E to **VMXNET3**.

New CD/DVD Drive should be changed from "Client Device" to **Datastore ISO File** and select the Windows Server 2022 ISO.


Click **Next**, **Finish**, and then power on the VM.

## Install Windows Server Core

Choose the appropriate Language and input, click **Next**

Click **Install Now**.

Select the option **Windows Server 2022 Standard Evaluation**, click **Next**

Check the box to accept the Microsoft Software License Terms, click **Next**

Choose **Custom: Install Microsoft Server Operating System only (advanced)**

Select **Drive 0 Unallocated Space**, click **Next**

Wait until the install is complete. The server will restart automatically.

Select **OK** and enter a password that meets the complexity requirements to set the password for the built-in Administrator account.
Select **OK** when finished.


## Initial Configuration

I need to install VM tools before I do anything else.

### Install VM Tools

In the vSphere Client, the Domain Controller 1 VM is selected, so I can simply click on **Install VMware Tools**. 
I'm then prompted to **Mount** the VM tools ISO.

In SConfig type option **15** to exit the SConfig menu and return to PowerShell.

The command `Get-Volume` will list the volumes with drive letters which I will use to determine the drive letter of the mounted ISO.

The VMware Tools CD-ROM ISO is mounted on the D: drive.

Change directory to the D: drive and enter `Get-ChildItem` to list the directory contents.

Run `.\setup64.exe` to start the installer.

The installer window flashes then disappears. **Minimize** the cmd.exe window to find the installer window again.

Select the appropriate options and navigate through the wizard and reboot when prompted.

### SConfig

The Server Configuration tool, called [SConfig](https://docs.microsoft.com/en-us/windows-server/administration/server-core/server-core-sconfig), loads automatically upon login.

I'll choose the corresponding number and follow the on-screen directions to change the following:

* Set Hostname (opt #2)
* Set static IP address (opt #8)
* Set Timezone (opt #9)
* Disable Telemetry (opt #10)
* Install Updates (opt #6)

Once those are completed, reboot the server (opt #13).

## Format Disks

Type 15 to close SConfig and exit to PowerShell.

`Get-Volume` does not show the two additional virtual hard disks that I added to the VM, so I need to format them.

`Get-Disk` shows that the OS recognizes the two 10GB disks.

I'll specify disk number 1 to select the first 10GB disk with `Get-Disk -Number 1`.
I'll then pipe that into `Initialize-Disk` and select a partition style of GPT.
The `-PassThru` parameter will allow me to pass the object through the pipeline to create an NTFS volume and assign a drive letter with the `New-Volume` command.
In order to see the command parts better, I'll put the parts on a new line.

The FriendlyName is optional but assigning a label to the volume will help identify its contents.
This drive will host the NTDS database of a domain controller, so I'll name it "NTDS".

````posh
Get-Disk -Number 1 | Initialize-Disk -PartitionStyle GPT -PassThru | New-Volume -FileSystem NTFS -DriveLetter E -FriendlyName NTDS 
````

Repeat the command with Disk #2, assign a different drive letter, and name it "Logs".

````posh
Get-Disk -Number 2 | Initialize-Disk -PartitionStyle GPT -PassThru | New-Volume -FileSystem NTFS -DriveLetter F -FriendlyName Logs 
````

## Conclusion

Now that the extra drives are formatted, and everything is set, I can use this machine as a reference VM to [Export to an OVA template]({% link _posts/_posts/2021-12-3-ExportingVMtoOVA.md %}), but I recommend clearing network config and the [Windows Security Identifier](https://techcommunity.microsoft.com/t5/windows-blog-archive/the-machine-sid-duplication-myth-and-why-sysprep-matters/ba-p/723859).



[^1]: https://docs.microsoft.com/en-us/windows-server/administration/server-core/what-is-server-core
[^2]: https://www.stigviewer.com/stig/microsoft_windows_server_2019/2021-03-05/finding/V-205723
