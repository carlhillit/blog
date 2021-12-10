---
layout: post
title: Unlock Locked Out Active Directory Accounts
published: true
tags: [PowerShell,Active Directory]
youtubeId: R40Xt96zMIg
---

A PowerShell one-liner that finds all locked out Active Directory accounts and unlocks them.

{% include youtubePlayer.html id=page.youtubeId %}

This really comes in handy when a recently identified issue causes large numbers of users to have their accounts become locked.

**_WARNING: Account lockouts can also be an indicator of malicious activity. Assess the risks involved with unlocking all accounts before doing so_.**

## Overview

1. [Unlock-ADAccount](#unlock-adaccount)
2. [Search-ADAccount](#search-adaccount)
3. [Combining Commands with the Pipeline](#combining-commands-with-the-pipeline)

## Unlock-ADAccount

The command that unlocks accounts is called `Unlock-ADAccount`[^1] and it is fairly simple to use and understand.

We only need to be concerned with the `-Identity` parameter, which identifies the account to unlock.

I typically use the SAM account name to identify a user.

It can be found in ADUC by double-clicking on the user, selecting the **Account** tab, and looking under **User Logon Name (Pre-Windows 2000)**.

![_config.yml]({{ site.baseurl }}/images/unlockalladaccounts/samaccountname.jpg:width=250)

I'll go ahead and copy the SAM account name to the clipboard.

You can also use any of these, which are found in the **Attribute Editor** tab:

* distinguished name
* GUID (objectGUID)
* security identifier (objectSid)

![_config.yml]({{ site.baseurl }}/images/unlockalladaccounts/dn.jpg)

We can see that Michelle's account is locked out by this message here:

![_config.yml]({{ site.baseurl }}/images/unlockalladaccounts/aduclocked.jpg)

Now that we've got the SAMAccountName, we can paste the SAM account name into PowerShell by right-clicking. The completed command should be:

````posh
Unlock-ADAccount -Identity michelle.adams
````

Hit ENTER and verify the account is unlocked in ADUC.

The message that was there before is now gone, so the account is now unlocked.

![_config.yml]({{ site.baseurl }}/images/unlockalladaccounts/unlocked.jpg)

That's how you unlock an account. Now I'll show you how to search for all of the accounts that are currently locked out.

## Search-ADAccount

The command we're using this time is `Search-ADAccount`[^2].

This searches Active Directory for various things like expired or inactive accounts, expired passwords,
and what we're using it for: _locked out accounts_.

We'll use two parameters with this command:

The first is `-LockedOut`. This will return the accounts that are currently locked out.

The second is `-SearchBase`. This tells PowerShell _where_ to search. Without it, PowerShell will search the entire domain.

Since we want to search only in a specific Organizational Unit, we need to input the OU here in Distinguished Name format.

To find the Distinguished Name of an Organizational Unit in ADUC, right-click on the OU, select **Properties**, and it will be listed in the **Attribute Editor** tab.

You can simply copy the Distinguished Name from the Attribute Editor and right-click to paste into PowerShell.

![_config.yml]({{ site.baseurl }}/images/unlockalladaccounts/copydn.jpg)

Add quotes around the Distinguished Name and we're good to go.

````posh
Search-ADAccount -LockedOut -SearchBase 'OU=Users,OU=LAB,DC=breakdown,DC=lab'
````

The result will show which accounts' `LockedOut` attribute is set to `True`.

![_config.yml]({{ site.baseurl }}/images/unlockalladaccounts/pslocked.jpg)

## Combining Commands with the Pipeline

We can combine `Search-ADAccount` and `Unlock-ADAccount` together with the pipe (` | `) to unlock all accounts that are currently locked out.[^3]

Since the command on the left (`Search-ADAccount`) outputs the identities of accounts, we don't need to use the `-Identity` parameter for the command on the right of the pipe (`Unlock-Account`).

The combined command will be:

````posh
Search-ADAccount -LockedOut -SearchBase 'OU=Users,OU=LAB,DC=breakdown,DC=lab' | Unlock-ADAccount
````

Verify that the account are unlocked by spot-checking in ADUC or re-run the `Search-ADAccount` command.

No results are returned, so that tells us that no accounts are currently locked out.

[^1]: https://docs.microsoft.com/en-us/powershell/module/addsadministration/unlock-adaccount
[^2]: https://docs.microsoft.com/en-us/powershell/module/addsadministration/search-adaccount
[^3]: https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/04-pipelines
