# Windows Hardening

Every escalation technique in this module has a corresponding defense; this note consolidates them into a single hardening checklist. Privilege escalation is almost always the exploitation of an avoidable misconfiguration — an over-broad privilege, a writable service binary, a cached credential — so hardening is mostly about **removing** those conditions rather than adding controls. Work it top to bottom on a host you are defending, and each item names the offensive note it closes.

## Identity and privileges

- **Least privilege.** Do not run services or users as administrator/SYSTEM when a scoped account will do. Audit membership of privileged **built-in groups** (Backup Operators, Server/Print Operators, DnsAdmins, Hyper-V Admins, Event Log Readers) — treat them as tier-0. Closes [Windows Built in Groups](Windows-Built-in-Groups/Windows-Built-in-Groups.md).
- **Strip dangerous token privileges.** Review "User Rights Assignment" (`secpol.msc`) and remove `SeImpersonate`, `SeAssignPrimaryToken`, `SeBackup`, `SeRestore`, `SeDebug`, `SeTakeOwnership`, `SeLoadDriver` from accounts that do not need them. Closes [Token Privilege Abuse](Token-Privilege-Abuse/Token-Privilege-Abuse.md).
- **Tier administration.** Separate DC / server / workstation admin accounts; never log an admin interactively into a lower-trust host. Closes [Interacting with Users](Interacting-with-Users.md).

## Services, tasks, and autoruns

- **Quote service paths** and ensure service **binaries and their directories** are writable only by administrators. Audit service DACLs (no `SERVICE_CHANGE_CONFIG` for Users). Closes [Unquoted Service Path Vulnerability](Services-Exploitation/Unquoted-Service-Path-Vulnerability.md), [Insecure Service Permissions(binPath)](Services-Exploitation/Insecure-Service-Permissions(binPath).md), [Insecure File Permissions Service Executable Files Path](Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md).
- **Scheduled-task and autorun binaries** writable only by admins. Closes [Privilege Escalation via Scheduled Tasks](Privilege-Escalation-via-Scheduled-Tasks.md), [Autorun Registry Persistence](Registry-Exploitation/Autorun-Registry-Persistence.md).
- **Disable the Print Spooler** where not needed, and keep `%PATH%` directories non-writable. Closes [PrintSpoofer](Impersonation-and-Potato-Attacks/PrintSpoofer.md), DLL-hijack paths ([Dynamic Link Library Hijacking(DLL Hijacking)](Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md)).

## Registry and UAC

- **Disable AlwaysInstallElevated** (ensure both hive values are 0 / unset). Closes [AlwaysInstallElevated Exploitation](Registry-Exploitation/AlwaysInstallElevated-Exploitation.md).
- **Set UAC to "Always notify."** Removes the auto-elevate assumption behind [UAC Bypass](UAC-Bypass/UAC-Bypass.md); remove standing local-admin rights from day-to-day accounts.

## Credentials

- **LSASS protection:** enable **RunAsPPL** and **Credential Guard**; apply the ASR rule "Block credential stealing from lsass.exe." Closes [LSASS Credential Dumping](Password-Mining/LSASS-Credential-Dumping.md).
- **Eliminate cleartext secrets:** remove GPP `cpassword` (and MS14-025), clear autologon registry passwords, delete `unattend`/`sysprep` files, and never pass secrets on the command line. Closes [Password Mining](Password-Mining/Password-Mining.md), [Group Policy Preferences cpassword](Password-Mining/Group-Policy-Preferences-cpassword.md).
- **LAPS** for unique, rotated local-admin passwords; clear saved RDP credentials.

## Platform and monitoring

- **Patch** the OS and third-party software; enable the **vulnerable-driver blocklist** and HVCI. Closes [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md), [SeLoadDriver Abuse](Token-Privilege-Abuse/SeLoadDriver-Abuse.md), and third-party [CVEs](Miscellaneous-Techniques.md).
- **Application control** (AppLocker/WDAC) to block dropped tooling and constrain LOLBins.
- **Logging:** command-line process auditing (4688), PowerShell ScriptBlock/Module logging, and **Sysmon** — the telemetry that catches everything above. See [Situational Awareness](Situational-Awareness.md) for what this looks like from the attacker's side.
- **Decommission or isolate** EOL systems. Closes [Legacy Operating Systems](Legacy-Operating-Systems.md).

## Verify your hardening

Run the module's own enumeration against the host — a hardened box should light up **no** `[+]` findings:

```powershell
IEX (Get-Content .\privesc-enum.ps1 -Raw)   # from [[PowerShell-Privilege-Escalation-Enumeration]]
```

## Related
- [Windows Privilege Escalation](README.md) — category MOC (each technique's own note has a detailed "Defenses" section)
- [PowerShell Privilege Escalation Enumeration](PowerShell-Privilege-Escalation-Enumeration.md) — use it to verify the hardening holds
- [Situational Awareness](Situational-Awareness.md) — the detection surface, from the attacker's view
- [Escalate My Privilege Windows](Escalate-My-Privilege-Windows.md) — the offensive methodology this defends against
