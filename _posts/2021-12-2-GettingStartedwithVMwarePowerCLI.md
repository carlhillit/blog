---
layout: post
title: Getting Started with VMware PowerCLI
published: true
tags: [PowerCLI,VMware]
---


### Install the PowerCLI Module
Before I can begin working with PowerCLI, I'll first need to install the module.

From my computer, I'll open PowerShell as an Administrator and install the VMware PowerCLI module from the PS Gallery by typing:
````posh
Install-Module VMware.PowerCLI -Force
````

![_config.yml]({{ site.baseurl }}/images/installpowercli/InstallPowerCLI.gif)

### Initial Configuration of PowerCLI

Because this is my first time using a PowerShell module on this computer, I need to allow the execution of the module I've just installed.
For lab purposes, I'll set the policy to `Unrestricted`.
````posh
Set-ExecutionPolicy Unrestricted
````

I'll then opt out of the Customer Experience Improvement Program for all users:
````posh
Set-PowerCLIConfiguration -ParticipateInCeip $false -Scope AllUsers
````

And since my vCenter lab has a non-trusted SSL certificate, I'll also need to set PowerCLI to ignore invalid certificates:
````posh
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore
````

### Connect to vCenter
Now all the prerequisites are filled, I can connect to vCenter.

I find it useful to store the credentials in a variable in case I need to later reconnect or connect to a different vCenter.

Store the vCenter credentials in a variable using the `Get-Credential` command.
````posh
$vscreds = Get-Credential
````

Then connect to the vCenter server[^1] using the credentials variable[^2].
````posh
Connect-VIServer -Server vcsa.breakdown.lab -Credential $vscreds
````


[^1]: You can also connect to stand-alone ESXi hosts with `Connect-VIServer`, but some commands only work with when connected to a vCenter server.

[^2]: If you'd like, you can add the `-SaveCredentials` parameter to save the credentials to the credentials store when using Windows PowerShell.