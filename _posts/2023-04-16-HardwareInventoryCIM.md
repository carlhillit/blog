---
layout: post
title: Create Hardware Inventory with PowerShell
published: true
tags: [Powershell]
youtubeId: KLJBNFwSaZc
---

How to gather basic hardware information such as manufacturer, model, and serial number remotely via PowerShell.

{% include youtubePlayer.html id=page.youtubeId %}

# Contents
1. [Common Information Model](#common-information-model)
2. [Get CIM Instance](#get-cim-instance)
3. [Get Hardware Info From Remote Computer](#get-hardware-info-from-remote-computer)
4. [PSCustomObject](#pscustomobject)
5. [Usage in Loops](#usage-in-loops)
6. [Export to CSV](#export-to-csv)
7. [Set Active Directory Computer Info](#set-active-directory-computer-info)

## Common Information Model

Definition: [^1]

    The Common Information Model (CIM) is an open standard that defines how managed elements in an IT environment are represented as a common set of objects and relationships between them.



## Get CIM Instance
`Get-CimInstance` is the command to use that will get the instance, but it's the CIM class that'll give us the information we're seeking.

The class name that will give us the manufacturer and model of a computer is Win32_ComputerSystem.

    Get-CimInstance -ClassName Win32_ComputerSystem
    Get-CimInstance -ClassName Win32_Bios

If we're just mucking around in PowerShell, we can use the abbreviated form:

    gcim win32_computersystem

If we pipe the command to `Get-Member` we can see all of the members the command's output object has. Boy! that's a lot of information, but for now, we'll use Manufacturer and Model from `Win32_ComputerSystem` and SerialNumber form `Win32_Bios`.

If you're interested in why the Common Information Model cmdlet, Get-CimInstance, is recommended over the Windows Management Instrumentation cmdlet, Get-WmiObject, check out the article below:

https://devblogs.microsoft.com/scripting/should-i-use-cim-or-wmi-with-windows-powershell/

## Get Hardware Info From Remote Computer
To get CIM info from a remote computer in the same Active Directory domain, you can simply use the -ComputerName parameter with the previous command.

    Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName COMPUTER001

This is fine for one-off information retrieval, but if you wanted to do more frequent interaction, create a CIM session and then use the -CimSession parameter to refer to the CIM session of the remote computer.

```powershell
$cimSesh = New-CimSession -ComputerName COMPUTER001

Get-CimInstance -ClassName Win32_ComputerSystem -CimSession $cimSesh
```

## PSCustomObject

Creating a PowerShell custom object will allow us flexibility to use the output in different ways. For example we can then export the inventory results to a CSV/Excel spreadsheet or upload the

PSCustomObjects are simply key/value pairs that are converted to a PowerShell object and are easily created from hash tables by sticking `[PSCustomObject]` at the beginning.

The column headers (object member name) are on the left of the equals sign ( = ) and the value is on the right.

I prefer to assign a variable to the object so it's easier to recall later.

```powershell
$hardwareInventory = [PSCustomObject]@{
    Manufacturer = (Get-CimInstance -ClassName Win32_ComputerSystem).Manufacturer
    Model = (Get-CimInstance -ClassName Win32_ComputerSystem).Model
    SerialNumber = (Get-CimInstance -ClassName Win32_Bios).SerialNumber
}
```
Output of $hardwareInventory :

    Manufacturer    Model         SerialNumber
    ----            -----         ------------
    Dell Inc.       Latitude 9000 1234ABC


## Usage in Loops
To gather CIM information from a number of remote computers, we can easily place our previous scriptlet in a foreach loop:

```powershell
# list of computers in array format
$computerList = @(
    "COMPUTER001"
    "COMPUTER002"
    "COMPUTER003"
    "COMPUTER004"
    "COMPUTER005"
)

# loop through the array
foreach ($computer in $computerList) {

    [PSCustomObject]@{
        Manufacturer = (Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName $computer).Manufacturer
        Model = (Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName $computer).Model
        SerialNumber = (Get-CimInstance -ClassName Win32_Bios -ComputerName $computer).SerialNumber
    }

}
```

Note that the loop does not create a new PSCustomObject during each iteration.

## Export to CSV
To export the custom object to a CSV, simply pipe it to Export-Csv.

Use the `-NoTypeINformation` parameter to remove unnecessary type information.

Here, I'm assigning the loop output to a variable and then piping it

```powershell
$hardwareInventory = foreach ($computer in $computerList) {

    [PSCustomObject]@{
        'Manufacturer' = (Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName $computer).Manufacturer
        'Model' = (Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName $computer).Model
        'SerialNumber' = (Get-CimInstance -ClassName Win32_Bios -ComputerName $computer).SerialNumber
    }

}

$hardwareInventory | Export-Csv -Path "C:\hardware.csv" -NoTypeInformation
```

Result:

    "Manufacturer","Model","SerialNumber"
    "Dell Inc.","Latitude 9000","1234ABC"

## Set Active Directory Computer Info

We can get a list of computers from Active Directory, get the CIM info from those computers, and set the AD computer description all in the same script:

```powershell
# Get list of computer names from Active Directory
$computerList = Get-ADComputer -Filter * -SearchBase "OU=Computers,OU=TEST,DC=karl,DC=lab"


# get hardware info for each computer in the loop
foreach ($computer in $computerList.Name) {

    $hardwareInfo = [PSCustomObject]@{
        'Manufacturer' = (Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName $computer).Manufacturer
        'Model' = (Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName $computer).Model
        'SerialNumber' = (Get-CimInstance -ClassName Win32_Bios -ComputerName $computer).SerialNumber
    }

    # create the description string
    $ADdescription = "$($hardwareInfo.Manufacturer) - $($hardwareInfo.Model) - $($hardwareInfo.SerialNumber)"

    # set description with hardware info from CIM instance
    Set-ADComputer -Identity $computer -Description $ADdescription

}
```


Or alternatively, here is a more compact version:

```powershell
(Get-ADComputer -Filter * -SearchBase "OU=Computers,OU=TEST,DC=karl,DC=lab").Name | ForEach-Object {

    $compSys = Get-CimInstance -ClassName Win32_ComputerSystem -ComputerName $_

    $SerialNumber = (Get-CimInstance -ClassName Win32_Bios -ComputerName $_).SerialNumber

    Set-ADComputer -Identity $computer -Description "$($compSys.Manufacturer) - $($compSys.Model) - $SerialNumber"

}
```

## Common CIM Classes

| Class | Description |
| --- | --- |
| Win32_CompterSystem | Manufacturer, Model |
| Win32_BIOS | Bios Version, Serial Number |
| Win32_QuickFixEngineering | List of Installed Updates |
| Win32_LoggedOnUser |
| Win32_OperatingSystem | OS version, LastBootUpTime |



[^1]: https://en.wikipedia.org/wiki/Common_Information_Model_(computing)