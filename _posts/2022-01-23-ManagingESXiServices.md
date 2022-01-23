---
layout: post
title: Managing ESXi Services with PowerCLI
published: false
tags: [PowerCLI,VMware]
youtubeId: twDJu5wSPUw
---

In this post, I'll show how to manage ESXi services with [PowerCLI]({% post_url 2021-12-2-GettingStartedwithVMwarePowerCLI %}).

{% include youtubePlayer.html id=page.youtubeId %}

## Managing Services in the Web Interface

In the web management interface, one can manage services by clicking on *Manage* in the side navigator and selecting the *Services* tab.

Then I'll select the service I want to manage and click the Start or Stop button to start or stop the service.

If you want to have the service start automatically upon system bootup, click the Actions button, go down to Policy, and check " Start and Stop with Host"

![_config.yml]({{ site.baseurl }}/images/managingesxiservices/servicepolicyweb.jpg)

## Starting PowerShell

To manage the host services in PowerCLI, I'll first start PowerShell core by typing pwsh into the terminal

In this case, I'm using PowerShell Core 7.2.1 on MacOS.

![_config.yml]({{ site.baseurl }}/images/managingesxiservices/pwsh.jpg)

## Connecting to the ESXi Host

First, I need to connect to the ESXi host by typing `Connect-VIServer` and then the IP address or hostname of the ESXi host.

````posh
Connect-VIServer 192.168.178.92
````

Since, I didn't supply the credentials already, I'll need to enter the username and password.

![_config.yml]({{ site.baseurl }}/images/managingesxiservices/connectviserver.jpg)

## VMHostService Commands

Managing host services are done through five commands. Get-VMHostService outputs general information about host services, Start,Stop, and Restart-VMHostService will start,stop, or restart the host service, and `Set-VMHostService` will configure the service policy.

I first want to find out which services are running on the host.

````posh
Get-VMHostService
````

I can see the the SSH service is not running and its policy is "off", which means I must start and stop the service manually.

![_config.yml]({{ site.baseurl }}/images/managingesxiservices/getvmhostservice.jpg)

I'll be using SSH for future configurations, so I'm going to start the service and set the policy to start with the host on boot.

## Object Types

If I try to simply use `Start-VMHostService -HostService TSM-SSH` to start the service, I'll get an error.

The error tells me that I've supplied the service as a *string* and not the host service object type that is required for the `Start-VMHostService` input.

Think of strings as plain ol' text, which is not the object type that I need. So how to I get the correct object type?

I need to retrieve it with `Get-VMHostService`, but filtered to only the SSH service by using `Where-Object`.

The property will be "Key" and Key will be equal to "TSM-SSH".

````posh
Get-VMHostService | Where-Object -Property Key -eq TSM-SSH
````

By placing the whole line in parenthesis, I can then use `.GetType()` to show that the object type is the kind I'm searching for.

````posh
(Get-VMHostService | Where-Object -Property Key -eq TSM-SSH).GetType()
````

![_config.yml]({{ site.baseurl }}/images/managingesxiservices/objecterror.jpg)

## Starting and Stopping Services 

I'll add a pipe and the `Start-VMHostService` to start the service. The output shows it's running and the web interface shows it's running as well.

````posh
Get-VMHostService | Where-Object -Property Key -eq TSM-SSH | Start-VMHostService
````

I can confirm by ssh'ing into the host.

````bash
ssh root@192.168.178.92
````

![_config.yml]({{ site.baseurl }}/images/managingesxiservices/startssh.jpg)

If I stop the service, I can no longer connect via SSH.

````posh
Get-VMHostService | Where-Object -Property Key -eq TSM-SSH | Stop-VMHostService
````

![_config.yml]({{ site.baseurl }}/images/managingesxiservices/stopssh.jpg)

## Setting Service Policy

Piping the object to `Set-VMHostService` will allow me to set the policy.

I have three options: Automatic, On, and Off.

These are labeled a bit different in the web interface, but correspond directly.

| Web Interface | PowerCLI |
--- | ---
| Start and stop with firewall ports | Automatic |
| Start and stop with host | On |
| Start and stop manually | Off |

If I set the policy to On, I can see that the web interface shows "Start and stop with host"

````posh
Get-VMHostService | Where-Object -Property Key -eq TSM-SSH | Set-VMHostService -Policy On
````

## Conclusion

I'll start the service for use in the next phase of my configuration.

Thanks for watching, I hope you learned something.


