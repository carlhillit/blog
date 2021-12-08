---
layout: post
title: Exporting VMs to OVA
published: true
tags: [PowerCLI,VMware]
youtubeId: jqM3fNx0py0
---

In this video, I'll demonstrate an easy way to export a VMware virtual machine to an OVA template file using PowerCLI.

{% include youtubePlayer.html id=page.youtubeId %}

## OVF vs OVA

First, a little background on OVF and OVA templates:

### OVF

When exporting VM templates from the vCenter web GUI, a "Folder of files (OVF)" is the default chosen option.

![_config.yml]({{ site.baseurl }}/images/exportova/exportovf.png)

The .ovf file is like an index configuration file in XML format that tells vCenter all of the specifications and component files of the VM template.

![_config.yml]({{ site.baseurl }}/images/exportova/ovfvscode.png)

### OVA

OVAs are more like zip files for VM templates in that it is a single file that contains other files.
This makes it a bit easier to transfer and store the VM template outside of vCenter. As a matter of fact, you can even extract the .ova file with 7zip in Windows, or with `tar xzf` in Linux/MacOS.

![_config.yml]({{ site.baseurl }}/images/exportova/7zipextractova.png)

[From vSphere 6.5 and newer](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vm_admin.doc/GUID-AFEDC48B-C96F-4088-9C1F-4F0A30E965DE.html), VM templates can only be exported in OVF format from the vCenter web GUI, which is why I'm using PowerCLI for this demonstration.

### Prerequisites

[Install the PowerCLI Module]({% link _posts/2021-12-2-GettingStartedwithVMwarePowerCLI.md %})

## Retrieving VM Object

The command that I'll be using to export the VM to an OVA template is `Export-vApp`.
Before I use the command, I'll take a look at the [documentation](https://powercli-core.readthedocs.io/en/latest/cmd_export.html) for a quick reference on how to use it.

I can see that either the `VApp` or `VM` parameter is required. For this demo, I'm using `VM`.

Since the VM parameter type requires a VM object type, we need to use `Get-VM` before I can export.

The two VMs that I want to export to a template are named "WS19C", which is Windows Server 2019 Core and "WS19D", with Desktop Experience.

````posh
Get-VM -Name WS19C
````

*if the VM is powered on, be sure to power it off at this time*

To export a single VM, I can simply assign a single VM to a variable, then use the variable with the Export-VApp command

````posh
$VM = Get-VM -Name WS19C
````

Verify the `$VM` variable is the correct type

````posh
$VM.GetType()
````

## Exporting the Template

Now everything is in place, I can now export the VM to the OVA template:

````posh
Export-VApp -VM $VM -Description "Windows Server 2019 Core" -Format Ova -Destination C:\templates
````

Since the VM parameter allows objects to be passed to it from the pipeline, I could combine these two commands into a one-liner:

````posh
Get-VM -Name WS19C | Export-VApp -Format Ova -Destination C:\templates
````

*PowerCLI will assume the name of the VM as the ova file name*

Either the command documentation, or a quick Ctrl+Space keyboard shortcut, will show me that the VM parameter can take an array of objects as input, as indicated by the double brackets ( `[]` ).
This means that I can export multiple VMs to templates with a single command.

````posh
Get-VM WS19* | Export-VApp -Format Ova -Destination C:\templates
````

## Conclusion

That concludes the post on how to use PowerCLI to export VMware virtual machines to OVA templates.
