---
layout: post
title: Bulk Create Active Directory Users with PowerShell
published: true
tags: ['Active Directory',PowerShell]
youtubeId: r1LN7B6ObN0
---

Need to create a metric boat load of Active Directory users quickly?
PowerShell is your best tool for the job.

{% include youtubePlayer.html id=page.youtubeId %}

The four parts to the script that I'll be breaking down are:
1. [Importing a CSV file into PowerShell to be used as an object](#Importing-a-CSV-file-into-PowerShell)
2. [Using a foreach loop to iterate through the imported CSV and perform a single action to each item in the CSV](#foreach-Loop-to-Iterate-Through-the-Users-Object)
3. [Create an Active Directory user with the New-ADUser cmdlet](#Create-an-Active-Directory-User-with-the-New-ADUser-Cmdlet)
4. [Using Splatting with New-ADUser](#Using-Splatting)

## Importing a CSV file into PowerShell

This is an example of a CSV file with users and the values of the Active Directory account attributes

````csv
GivenName,SurName,EmployeeID,Department                 
James,Smith,123001,Accounting
Michael,Miller,123007,Management
Barbara,Lopez,123012,Research and Development
Daniel,Thompson,123023,Analyst                    
Michelle,Adams,123042,Sales
Deborah,Roberts,123050,Human Resources
````
The Import-Csv[^1] cmdlet imports the CSV file into PowerShell as a PSCustomObject:

````powershell
Import-Csv -Path C:\users.csv
````

I'd like to use that object for things later in the script, so I'll assign that object to the variable `$usersCsv` and that users CSV object is stored in memory (RAM).

````powershell
$usersCsv = Import-Csv -Path C:\users.csv
````

Now I can recall that object by typing the variable `$usersCsv` and

Output:
````
GivenName   SurName   EmployeeID   Role                    
---------   -------   ----         ----                    
James       Smith     123001       Accounting              
Michael     Miller    123007       Management                     
Barbara     Lopez     123012       Research and Development
Daniel      Thompson  123023       Analyst                                                             
Michelle    Adams     123042       Sales
Deborah     Roberts   123050       Human Resources         
````

I can also view a single property and it's values by using `$usersCsv.GivenName`

Output:
````
James
Michael
Barbara
Daniel                    
Michelle
Deborah
````

## foreach Loop to Iterate Through the Users Object

The basic form of a foreach loop[^2] is:
````powershell
foreach ($user in $userCsv) { 
	"some action"
 }
````

The `$user` variable, by itself, does not have a value assigned to it. This  variable is given a value each time the loop goes through a row in the `$userCsv` object and assigns the row to the `$user` variable.

For every iteration, the CSV column headers `GivenName,SurName,Role` are the variable property.
The first iteration of the loop, the first row of data that contains `James,Smith,Accounting` are values assigned to the `$user` variable.

So if we add a Write-Host to the loop using the $user variable followed by a Read-Host (to act as a pause), we can see the values changing after each iteration.
*Parenthesis and extra dollar sign are inserted because of mixing text and variable properties within the same double quotation marks*

````powershell  
foreach ($user in $userCsv) {
	Write-Host "$($user.GivenName) $($user.SurName) works in the $($user.Role) department."
	Read-Host
}
````


## Create an Active Directory User with the New-ADUser Cmdlet

New-ADUser[^3] is fairly straight forward, but there are a few things that should be known:

`-AccountPassword`
*  If $null or no password is specified then no password is set and the account is disabled unless it is requested to be enabled.
*   If the user password is specified, then the password is set and the account is disabled unless it is requested to be enabled.
* If the password specified does not meet the requirements of the password policy, the account will be created and Set-ADAccountPassword will be needed to set the password.
* Input should be a secure string.

* `-PasswordNotRequired` is an option, but terrible security practice and should never be used.

* Accounts created are disabled by default unless `-Enabled $true` is used.

* If `-Path` (Organizational Unit) is not specified, the user account will be created in the default Users container for the domain in which the computer that is running PowerShell is joined.

* SamAccountName attribute will be truncated (maybe) if the characters are over the 20 character limit
*one technique to get around having to deal with the character limit is to use the Employee ID as the SAM account name*

I'll use the distinguished name of the Organizational Unit that the user accounts will be created.

````powershell
$UsersOU  =  "OU=Users,OU=LAB,DC=breakdown,DC=lab"

New-ADUser -Name ($user.SurName)+', '+($user.GivenName) -SamAccountName $user.EmployeeID -Path $UsersOu -GivenName $user.GivenName -Surname $user.Surname -Description $user.Role -Enabled $true -AccountPassword  ('P@$$w0rd123!@#' | ConvertTo-SecureString -AsPlainText -Force)
````

That's quite long and probably difficult to read. Unless you're reading from an ultrawide monitor the above command is likely viewed only by scrolling far over.
This is not great for scripting. Splatting is a much better form to use.

### Using Splatting
Splatting[^4] is a way to consolidate all parameters and their values into key pairs. It's a lot easier when scripting because it resembles more of a column format rather than a long horizontally run-on cmdlet.

Start with a regular variable assignment, then begin the key pairs with a hash table `@{ }`

````powershell  
$UsersOU = "OU=Users,OU=LAB,DC=karl,DC=lab"  

$params = @{
	Name = ($user.SurName)+', '+($user.GivenName)
	SamAccountName = $user.EmployeeID
	Path = $UsersOU
	GivenName = $user.GivenName
	Surname = $user.Surname
	Description = $user.Role
	Enabled = $true
	AccountPassword = ('P@$$w0rd123!@#' | ConvertTo-SecureString -AsPlainText -Force)
	ChangePasswordAtLogon = $true
}
````

Once the table is complete, type the `New-ADUser` cmdlet and the variable, but substitute the dollar sign `$` with an at symbol `@`:

````powershell
New-ADUser @params
````

### Complete Script

````powershell
$usersCsv = Import-Csv -Path C:\users.csv

$UsersOU = "OU=Users,OU=LAB,DC=karl,DC=lab"

foreach ($user in $userCsv) {  
	$params = @{  
		Name = ($user.SurName)+', '+($user.GivenName)  
		SamAccountName = $user.EmployeeID  
		Path = $UsersOU  
		GivenName = $user.GivenName  
		Surname = $user.Surname  
		Description = $user.Role  
		Enabled = $true  
		AccountPassword = ('P@$$w0rd123!@#' | ConvertTo-SecureString -AsPlainText -Force)  
		ChangePasswordAtLogon = $true  
	}
	New-ADUser @params
}

```

## Post Thoughts

This is a basic script for user creation.
There are a number of different things that can be added or expanded to automate other aspects of the new user provisioning process such as security group membership or Exchange mailbox creation as well as adding error handling, but this is a good foundation in which to start.


[^1]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-csv
[^2]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_foreach
[^3]: https://docs.microsoft.com/en-us/powershell/module/activedirectory/new-aduser
[^4]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting
