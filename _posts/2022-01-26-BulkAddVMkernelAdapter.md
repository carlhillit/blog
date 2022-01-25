---
layout: post
title: Bulk Add VMKernel Adapters to ESXi Hosts
published: true
tags: [PowerCLI,VMware]
---

Short, simple way to bulk add VMkernel adapters to ESXi hosts with PowerCLI

## Scenario

I was rebuilding my home lab cluster and I figured I'd research a way to bulk add a vMotion VMkernel adapter to my hosts instead of doing it manually. Three hosts fit in that goldilocks range of small enough that manual configuration is fairly trivial, but still enough to benefit from scripting/automating.

Since I forsee a need to this to many more hosts in a different environment, I figured I'd do the research now, write it down, and share so everyone can benefit.

## The One-Liner

For those that do not have a complicated environment, this can be achieved in a one-line script, once connected to vCenter.

As always, connect to the vCenter server first:

````posh
$vcreds = Get-Credential

Connect-VIServer vcsa.breakdown.lab -Credential $vcreds
````

Assuming the IP address are sequential:

````posh
Get-VMHost | ForEach-Object { $ip=1;New-VMHostNetworkAdapter -VMHost $_ -PortGroup vMotion -VirtualSwitch DPortGroup -IP "10.10.10.$ip" -SubnetMask 255.255.255.0 -VMotionEnabled:$true ;$ip++  }
````

### The Breakdown

[`New-VMHostNetworkAdapter`](https://developer.vmware.com/docs/powercli/latest/vmware.vimautomation.core/commands/new-vmhostnetworkadapter/#PARAMETER_SET_SERVICES) requires a host object type for `-VMHost` so a `Get-VMHost` piped to a `ForEach-Object` is an easy method to loop through all VM hosts and sends the correct object type by way of the `$_` variable.

I have a sequential IP scheme, so I started the loop by assigning a variable with the last octet value of 1.

A semicolon ( `;` ) takes the place of a carriage return (*enter*) and that lets me combine commands together on one line, so I'm sort of cheating here by calling it a one-liner.

The PortGroup, VirtualSwitch, SubnetMask parameters accepts a string for the input, so simply typing in the information is all that's needed.

I've added the `$ip` variable to take the place of the last octet of the IP address to make it 10.10.10.1

VMotionEnabled is a switch parameter (true/false)

`$ip++` adds value of 1 increment to the `$ip` variable, which means after the first pass the value would be 2, then 3, then 4, and so on.



## The Script

The above does make a few assumptions, primarily that I would run the command against all of the VM hosts and secondarily, that the IP addresses would be sequential.

Since NUC #2 is being downgraded because there are numerous issues with ESXi 7.0.3[^1], this is a perfect scenario to showcase a script that uses a spreadsheet or CSV to specify which IP goes to what on which host. Of course you can customize the script and CSV as your needs require.

Example spreadsheet:

| Host | PortGroup | VDS | IPaddress | SubnetMask |
|---|---|---|---|---|
| nuc1.breakdown.lab | DPortGroup | DSwitch | 10.10.10.1 | 255.255.255.0 |
| nuc3.breakdown.lab | DPortGroup | DSwitch | 10.10.10.3 | 255.255.255.0 |


*The hosts need to be added to the virtual distributed switch before you can run the below script*

````posh
$csv = Import-Csv -Path /Users/karl/homelab/vmotionadapters.csv

foreach ($adapter in $csv) {

    $splat = @{
        PortGroup = $adapter.PortGroup
        VirtualSwitch = $adapter.VDS
        IP = $adapter.IPaddress
        SubnetMask = $adapter.SubnetMask
        VMotionEnabled = $true
    }

    Get-VMHost $adapter.Host | New-VMHostNetworkAdapter @splat

}
````

The output:

````posh
Name       Mac               DhcpEnabled IP              SubnetMask      DeviceName
----       ---               ----------- --              ----------      ----------
vmk1       00:50:56:6c:55:00 False       10.10.10.1      255.255.255.0         vmk1
vmk1       00:50:56:6d:dc:0d False       10.10.10.3      255.255.255.0         vmk1
````

## Conclusion

I can use the below command to confirm the configuration:
````posh
Get-VMHostNetworkAdapter | Select-Object VMHost,Name,IP,SubnetMask,PortGroupName,VMotionEnabled
````

Everything looks good! Hope this helps automate your VMware environment and save some clicks.

[^1]: https://kb.vmware.com/s/article/86398