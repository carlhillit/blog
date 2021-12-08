---
layout: post
title: Restart Computers Remotely with PowerShell
published: true
tags: [PowerShell]
---

Learn how to restart a Windows computer remotely using PowerShell, even if the remote computer is BitLocker enabled.

[![Watch the video](https://img.youtube.com/vi/czDLFWBZ-JQ/hqdefault.jpg)](https://youtu.be/czDLFWBZ-JQ)

Virtual Machines Used in Demo
| Description | Operating System | ComputerName |
| --- | :---: | ---: |
| Local Computer | Windows 10 w/ BitLocker | WIN101 |
| Remote Computer | Windows 10 w/ BitLocker | WIN102 |
| Domain Controller | Windows Server 2019 | DC |

## Restart-Computer[^1]

I'll begin by opening PowerShell as an administrator on my local machine named "WIN101".

If I want to restart my local computer, I'll type:

````posh
Restart-Computer
````

I am required to enter the BitLocker PIN before the Operating System will boot.

After I login, I'll open PowerShell once again by right-clicking on the Start Menu and selecting "Windows PowerShell (Admin)"

Next, I will attempt to restart another domain-joined computer named WIN102 remotely by using:

````posh
Restart-Computer -ComputerName WIN102
````

After waiting a few seconds, I receive the error: `The RPC server is unavailable`.

![_config.yml]({{ site.baseurl }}/images/restartcomputerremotely/rpcunavailable.jpg)

## Create Firewall Rule for WMI

After some [research](https://community.spiceworks.com/topic/1640318-need-help-with-powershell-error-output), I determined the Windows Firewall was blocking WMI.

I'll hop on over to the domain controller and edit the empty "Firewall" Group Policy Object.

Under Computer Configuration, I'll then navigate to the Windows firewall Inbound rules.

`Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security > Windows Defender Firewall with Advanced Security > Inbound`

Right-clicking in the blank space will present the menu to create a **New Rule**.

![_config.yml]({{ site.baseurl }}/images/restartcomputerremotely/newfirewallrule.jpg)

Select the **Predefined** radio button and click on the drop-down menu to select `Windows Management Instrumentation (WMI)`.

I'll click Next, review the settings and click Next.

The default **Allow the connection** is good for me, so I'll click **Finish** to create the inbound firewall rule that will apply to all of my Windows 10 computers.

![_config.yml]({{ site.baseurl }}/images/restartcomputerremotely/fwrule.jpg)

If you do not want to wait until the remote computer's group policy updates, run the command [gpupdate](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/gpupdate) on the remote computer to force an immediate update.

Instead of the GUI, you can enable the firewall rule on the local computer with PowerShell by tying:

````posh
Set-NetFirewallRule -Name WMI-WINMGMT-In-TCP-NoScope -Enabled True
````

## Restart Remote Computer

When I retry the `Restart-Computer` command it fails again, but this time with a different error: someone is logged into WIN102

Add `-Force` to end to force the restart anyway.

## Suspend BitLocker for Reboot[^2]

The remote computer (WIN102) still requires the BitLocker PIN and that can be a problem because the computer will not start without it.

We can, however, suspend BitLocker on the remote computer for exactly one reboot to avoid that issue.

Since the Suspend-BitLocker command only works locally, we need to use it with `Invoke-Command`[^3] to run it against the remote computer.

````posh
Invoke-Command -ComputerName WIN102 -ScriptBlock { Suspend-BitLocker -Mountpoint C: -RebootCount 1 }
````

I'll view the encryption status[^4] of the local computer for comparison.

````posh
Get-BitLockerVolume -MountPoint C:
````

The Protection Status of the local drive is "ON" while the drive on the remote computer is "OFF".

![_config.yml]({{ site.baseurl }}/images/restartcomputerremotely/blcompare.jpg)

In File Explorer on the remote computer (WIN102) the C: drive shows a yellow triangle indicating that BitLocker protection is suspended.

![_config.yml]({{ site.baseurl }}/images/restartcomputerremotely/blsuspendc.jpg)

I can now proceed with restarting the remote computer (WIN102).

The remote computer boots without requiring a PIN, the C: drive icon is back to normal, and the BitLocker status now shows "ON".

![_config.yml]({{ site.baseurl }}/images/restartcomputerremotely/blstatuson.jpg)

## Conclusion

That's it! That's how to restart a BitLocker-enabled computer remotely with PowerShell.

[^1]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/restart-computer
[^2]: https://docs.microsoft.com/en-us/powershell/module/bitlocker/suspend-bitlocker?view=win10-ps
[^3]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/invoke-command
[^4]: https://docs.microsoft.com/en-us/powershell/module/bitlocker/get-bitlockervolume
