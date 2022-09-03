---
layout: post
title: Create a Domain Controller on Windows Server Core 2022
published: false
tags: [PowerShell,Windows Server Core,Active Directory]
youtubeId: 8kHbFiHTb4Q
---

How to provision a Domain Controller on Server Core with PowerShell.

{% include youtubePlayer.html id=page.youtubeId %} <!-- embedded youtube player, remove if no yt video accompanies the post -->


## Why Windows Server Core

1. 

2. Reduced resource consumption, particularly disk space.

3. Allows you to stop using the GUI as a crutch and "remoting" or RDP'ing into each server. Once you stop relying on the Windows GUI, you're free to utilize server management tools such as WAC, Server Manager, or PowerShell (yay!). Free yourself of those "button and menu" chains and you'll find it much easier to scale management. You'll be able to manage servers in large quantities. Active Directory is 99% managed via the AD PowerShell module, various Active Directory management consoles, [Windows Admin Center](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/overview), or other tools. All on these can be used remotely, so there isn't really a need to have a Graphical User Interface (GUI) on the domain controller itself, only from a management computer, like a desktop PC.

## Prerequisites

Before we start with the Active Directory server roles and configuration, we must first prepare the server which will become our first domain controller.


As a domain controller is essential infrastructure and a building block of a domain, the first step is to set a static IP address:

````posh
$IPAddress = '10.0.0.10'
$SubnetLength = 24
$DefaultGateway = '10.0.0.1'
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress $IPAddress -PrefixLength $SubnetLength -DefaultGateway $DefaultGateway
````

Next, set the DNS client address to itself, as this domain controller will also be a DNS server.
````posh
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses $IPAddress
````

I like to place the NTDS database and log files on separate disks, so I'll format those at this time:

````posh
Get-Disk -Number 1 | Initialize-Disk -PartitionStyle GPT -PassThru | New-Volume -FileSystem NTFS -DriveLetter E -FriendlyName 'NTDS'
Get-Disk -Number 2 | Initialize-Disk -PartitionStyle GPT -PassThru | New-Volume -FileSystem NTFS -DriveLetter F -FriendlyName 'LOGS'
````

## Install Server Roles





## Conclusion





![_config.yml]({{ site.baseurl }}/images/deployova/invaliduri.jpg) <!-- embedded image url -->


[^1]: https://www.stigviewer.com/stig/microsoft_windows_server_2019/2021-03-05/finding/V-205723