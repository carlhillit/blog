---
layout: post
title: Tips for Optimizing SQL Server 
published: false
tags: [SQL,VMware]
---

These tips will help you easily improve performance for Microsoft SQL Server workloads on ESXi virtualization hosts.

## First Header

### Right-Size the VM [^1]

Allocating too many vCPUs or RAM can, surprisingly, lead to increase overhead on the ESXi host, negatively impacting the performance of the VMs on the host.
*Not to mention the wasted licenses for unused CPU cores.*
For critical SQL Server VMs, take the peak utilization that is consistent and that becomes the baseline for MSSQL workloads.
For non-critical VMs, use the average. 

### ESXi Power Management [1^]
For high-performance workloads like MSSQL, the virtualization administrator can change the Power Management policy settings to the "High Performance" profile.

To view the power policy with PowerCLI:
````powershell
$vmHosts = Get-VMHost
foreach ($vmHost in $vmHosts) {
 
$vmhostesxcli = Get-EsxCli -VMHost $vmHost -V2
$vmhostesxcli.hardware.Power.policy.get.invoke()

}
````

Replace the last line in the loop with the following:
````powershell
$vmhostesxcli.hardware.Power.policy.set.Invoke(@{id="1"})

````

The number at the end, i.e. `id="1"` determines the power policy.
1. HighPerformance
2. Balanced
3. LowPower

An easy to use script can be found at the Sleep Admin GitHub.[^2]

### Disable CPU Hot Plug

affect the vNUMA topology and might have degraded performance because the NUMA architecture does not reflect that of the underlying physical server.
Therefore, VMware recommends to not enable CPU hot plug by default, especially for VMs that require vNUMA. Rightsizing the VM’s CPU is always a better choice than relying on CPU hot plug. The decision whether to use this feature should be made on a case-by-case basis and not implemented in the VM template used to deploy SQL.

````powershell
Function Disable-vCpuHotAdd($vm){
    $vmview = Get-vm $vm | Get-View
    $vmConfigSpec = New-Object VMware.Vim.VirtualMachineConfigSpec
    $extra = New-Object VMware.Vim.optionvalue
    $extra.Key="vcpu.hotadd"
    $extra.Value="false"
    $vmConfigSpec.extraconfig += $extra
    $vmview.ReconfigVM($vmConfigSpec)
}

````
Script sourced from David Stamen blog.[^3]

### Use separate VMDKs for SQL Server workloads connected via PVSCSI adapter.
Once VM tools are installed onthe MSSQL VM, add additional VMDKs for the following

* SQL Server data (system and user)
* transaction log
* backup files into separate VMDKs



[^1]: https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/solutions/sql-server-on-vmware-best-practices-guide.pdf
[^2]: https://github.com/TheSleepyAdmin/Scripts/blob/master/VMware/EsxCli/Get-ESXiPowerPolicy.ps1]
[^3]: https://davidstamen.com/2015/01/14/powercli-enable-cpu-and-memory-hotadd/
