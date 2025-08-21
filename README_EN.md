# Fixing-Nextcloud-Users-Login-Issue-with-Restricted-Log-On-To-in-AD
Fixes Nextcloud login issues for AD users with restricted Log On To; includes PowerShell scripts and best practices.

---

## Introduction

In an environment where Nextcloud server is installed on Ubuntu and joined to Active Directory, users authenticate via LDAP.  
The issue was that with a restricted Log On To, users could not log in unless the All Computers option was enabled.

---

## Problem

With restricted Log On To, users could not authenticate.  
Various tests were performed using PowerShell and GUI (Get-ADUser, Set-ADUser, UserWorkstations).  
It was also observed that `Set-ADUser -LogonWorkstations` does not reflect in GUI; to display correctly, `UserWorkstations` must be used.

---

## Solution

The practical solution was to enter **the Nextcloud server name first, followed by the AD server name** in Log On To.  
This allows Nextcloud users to authenticate without granting access to other systems.

### PowerShell example for a single user:

```powershell
Set-ADUser -Identity iman -Clear UserWorkstations
Set-ADUser -Identity iman -Add @{UserWorkstations = "NEXTCLOUDSRV,ADSRV"}

PowerShell example for a group of Nextcloud users:

$groupName = "NextcloudUsers"
$newWorkstations = @("NEXTCLOUDSRV", "ADSRV")

$users = Get-ADGroupMember -Identity $groupName -Recursive | Where-Object { $_.objectClass -eq "user" }

foreach ($user in $users) {
    try {
        $adUser = Get-ADUser -Identity $user.SamAccountName -Properties UserWorkstations
        $current = @()
        if ($adUser.UserWorkstations) { $current = $adUser.UserWorkstations -split "," }

        $finalList = ($current + $newWorkstations) | Where-Object { $_ -ne "" } | Select-Object -Unique
        $finalString = $finalList -join ","

        Set-ADUser -Identity $user.SamAccountName -Replace @{UserWorkstations = $finalString}
        Write-Host "✅ Log On To for user '$($user.SamAccountName)' updated to: $finalString"
    }
    catch {
        Write-Host "❌ Error for $($user.SamAccountName): $_"
    }
}


---

Security Notes

Adding the AD server to Log On To does not reduce security since users previously had direct access to log on to AD.

This method provides more precise control over systems.

It is recommended that only members of the Domain Admins group log in directly to DC.



---

Conclusion

This method is practical and reliable for similar environments and highlights the importance of step-by-step testing and using both PowerShell and GUI.


---

Author: Iman Mehregan Rad
Date: 2025-08-21
