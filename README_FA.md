[English](./README_EN.md)
# Fixing-Nextcloud-Users-Login-Issue-with-Restricted-Log-On-To-in-AD
Fixes Nextcloud login issues for AD users with restricted Log On To; includes PowerShell scripts and best practices.


## مقدمه

در محیطی که سرور Nextcloud روی اوبونتو نصب شده و به دامین Active Directory متصل است، کاربران از طریق LDAP authenticate می‌شوند.  
مشکل این بود که با محدود کردن Log On To، کاربران قادر به لاگین نبودند مگر اینکه گزینه All Computers فعال شود.

---

## مشکل

با استفاده از Log On To محدود، کاربران نمی‌توانستند authenticate شوند.  
تست‌های مختلفی با PowerShell و GUI انجام شد (Get-ADUser، Set-ADUser، UserWorkstations).  
همچنین مشاهده شد که تغییرات با `Set-ADUser -LogonWorkstations` روی GUI اثر نمی‌گذارد و برای درست نمایش دادن باید از `UserWorkstations` استفاده شود.

---

## راه حل

راه حل عملی این بود که در Log On To **ابتدا نام سرور Nextcloud و سپس سرور AD وارد شود**.  
این کار باعث می‌شود کاربران Nextcloud بتوانند authenticate شوند بدون اینکه دسترسی به سایر سیستم‌ها آزاد شود.

### نمونه PowerShell برای یک کاربر:

```powershell
Set-ADUser -Identity iman -Clear UserWorkstations
Set-ADUser -Identity iman -Add @{UserWorkstations = "NEXTCLOUDSRV,ADSRV"}

نمونه PowerShell برای گروه کاربران Nextcloud:

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
        Write-Host "❌ خطا برای $($user.SamAccountName): $_"
    }
}


---

نکات امنیتی

اضافه کردن سرور AD به Log On To باعث کاهش امنیت نمی‌شود، چون کاربران پیش‌تر امکان لاگین مستقیم به AD داشتند.

با این کار کنترل دقیق‌تر روی سیستم‌ها اعمال می‌شود.

توصیه می‌شود تنها اعضای گروه Domain Admins بتوانند مستقیماً به DC لاگین کنند.



---

نتیجه گیری

این روش عملی و قابل اعتماد برای محیط‌های مشابه است و اهمیت تست مرحله‌ای و استفاده همزمان از PowerShell و GUI را نشان می‌دهد.


---

نویسنده:Iman Mehregan Rad
تاریخ: 2025-08-21
