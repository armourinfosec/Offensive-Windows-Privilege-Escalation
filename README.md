# Offensive Windows Privilege Escalation

A structured, hands-on reference on **escalating from a low-privileged Windows foothold to Administrator or `NT AUTHORITY\SYSTEM`** — enumeration and methodology, service and scheduled-task misconfigurations, registry abuse and UAC bypass, token-privilege abuse, the impersonation/potato family, kernel exploits, and credential mining — written from an offensive-security perspective (escalate, then detect and defend).

> [!WARNING]
> **Educational use only**
> These are personal study notes. Every technique here is documented for use **only against systems you own or are explicitly authorized to test** (your own lab, a CTF, an authorized engagement). Escalating privileges on a host you do not have written permission to assess is unlawful in most jurisdictions. All examples use lab placeholders (`10.10.14.7` attacker, `10.10.10.5`/`192.168.x` target) — substitute your own.

## What's inside

- **60+ notes** organized by escalation vector, each in a consistent shape: concept → hands-on enumeration and exploitation (`whoami /priv`, `cmd`/`powershell`) → detection → defenses → related notes.
- **Full vector coverage** — automated enumeration (WinPEAS/PowerUp), service misconfigurations (unquoted paths, weak `binPath`/file permissions, DLL hijacking, service-via-registry, named pipes), scheduled tasks and startup apps, registry exploitation (`AlwaysInstallElevated`, autoruns), **UAC bypass** (fodhelper/eventvwr/computerdefaults/sdclt), **token-privilege abuse** (SeImpersonate, SeBackup/SeRestore, SeTakeOwnership, SeLoadDriver, SeDebug), the **potato family** (Juicy/Rogue/Print/God/Rotten), kernel exploits, and credential mining (SAM/SYSTEM, LSASS, NTDS.dit, GPP `cpassword`, registry, ADS, unattend files).
- A **methodology checklist** that indexes every vector, plus two full **hands-on labs** (SeImpersonate→SYSTEM, unquoted service path→SYSTEM).
- Copy-ready, language-tagged commands throughout, each with detection and defensive guidance.

## Map of Content

### Methodology & enumeration

| Note | Covers |
|------|--------|
| [Escalation Methodology & Checklist](Escalate-My-Privilege-Windows.md) | The step-by-step checklist that indexes every vector below |
| [Network Enumeration](Network-Enumeration.md) · [User Enumeration](User-Enumeration.md) · [Version & Configuration](Windows-Version-and-Configuration.md) | Situational awareness after landing a shell |
| [Privilege Escalation Tools](Privilege-Escalation-Tools.md) | WinPEAS, PowerUp, SharpUp, Seatbelt, Watson |

### Token privileges

| Note | Covers |
|------|--------|
| [Token Privilege Abuse (index)](Token-Privilege-Abuse/Token-Privilege-Abuse.md) | `whoami /priv` → exploit map |
| [SeBackup / SeRestore](Token-Privilege-Abuse/SeBackup-and-SeRestore-Abuse.md) · [SeTakeOwnership](Token-Privilege-Abuse/SeTakeOwnership-Abuse.md) | Read/write any file; take ownership |
| [SeLoadDriver](Token-Privilege-Abuse/SeLoadDriver-Abuse.md) · [SeDebug](Token-Privilege-Abuse/SeDebug-Abuse.md) | BYOVD kernel load; open any process |

### Impersonation & potato attacks

| Note | Covers |
|------|--------|
| [Impersonation & Potato Attacks (index)](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) | Token impersonation and the potato family |
| [Token Impersonation](Impersonation-and-Potato-Attacks/Token-Impersonation.md) · [Juicy Potato](Impersonation-and-Potato-Attacks/Juicy-Potato.md) · [JuicyPotatoNG](Impersonation-and-Potato-Attacks/JuicyPotatoNG.md) | Foundations and classic COM potatoes |
| [PrintSpoofer](Impersonation-and-Potato-Attacks/PrintSpoofer.md) · [RoguePotato](Impersonation-and-Potato-Attacks/RoguePotato.md) · [God Potato](Impersonation-and-Potato-Attacks/God-Potato.md) · [RottenPotato](Impersonation-and-Potato-Attacks/RottenPotato.md) | Modern SeImpersonate→SYSTEM |

### Service & scheduled-task misconfigurations

| Note | Covers |
|------|--------|
| [Services Exploitation (index)](Services-Exploitation/Services-Exploitation.md) | Enumeration, control, and abuse of services |
| [Unquoted Service Path](Services-Exploitation/Unquoted-Service-Path-Vulnerability.md) · [Insecure Permissions (binPath)](Services-Exploitation/Insecure-Service-Permissions(binPath).md) · [Insecure File Permissions](Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md) | Service misconfiguration classes |
| [DLL Hijacking](Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md) · [Service via Registry](Services-Exploitation/Service-Escalation-via-Registry.md) · [Named Pipes](Services-Exploitation/Named-Pipes.md) | Load-path and registry abuse |
| [Scheduled Tasks](Privilege-Escalation-via-Scheduled-Tasks.md) · [Startup Applications](Privilege-Escalation-via-Startup-Applications.md) · [RunAs](Escalation-via-RunAs.md) | Autostart vectors |

### Registry & UAC bypass

| Note | Covers |
|------|--------|
| [Registry Exploitation (index)](Registry-Exploitation/Registry-Exploitation-Techniques.md) | Registry-based escalation |
| [AlwaysInstallElevated](Registry-Exploitation/AlwaysInstallElevated-Exploitation.md) · [Autorun Persistence](Registry-Exploitation/Autorun-Registry-Persistence.md) | Registry escalation & persistence |
| [UAC Bypass (index)](UAC-Bypass/UAC-Bypass.md) | Medium→High integrity elevation |
| [fodhelper](UAC-Bypass/UAC-Bypass-via-fodhelper.md) · [eventvwr](UAC-Bypass/UAC-Bypass-via-eventvwr.md) · [computerdefaults](UAC-Bypass/UAC-Bypass-via-computerdefaults.md) · [sdclt](Registry-Exploitation/UAC-Bypass-via-sdclt.exe-and-App-Paths-Hijack.md) | Auto-elevate bypass techniques |

### Kernel exploits

| Note | Covers |
|------|--------|
| [Windows Kernel Exploits (index)](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) | Kernel/driver escalation |
| [MS10-015](Windows-Kernel-Exploits/MS10-015.md) · [MS10-059](Windows-Kernel-Exploits/MS10-059.md) · [MS14-058](Windows-Kernel-Exploits/MS14-058.md) · [HTB workflow](Windows-Kernel-Exploits/HTB-Kernel-Exploits.md) | Concrete kernel exploits |

### Credential & password mining

| Note | Covers |
|------|--------|
| [Password Mining (index)](Password-Mining/Password-Mining.md) | Credential discovery on Windows |
| [LSASS Dumping](Password-Mining/LSASS-Credential-Dumping.md) · [GPP cpassword](Password-Mining/Group-Policy-Preferences-cpassword.md) · [SAM & SYSTEM](Password-Mining/SAM-and-SYSTEM-files.md) · [NTDS.dit](Password-Mining/NTDS.DIT-Active-Directory-Domain.md) | Live memory, SYSVOL, hives, DC database |

### Hands-on labs

| Lab | Covers |
|-----|--------|
| [SeImpersonate → SYSTEM](Labs/SeImpersonate-Potato-to-SYSTEM.md) | Full potato escalation walkthrough |
| [Unquoted Service Path → SYSTEM](Labs/Windows-Unquoted-Service-Path-to-SYSTEM.md) | Service misconfiguration end to end |

## How to read

- **On GitHub** — every note is fully readable and its cross-references are **relative Markdown links** clickable directly in the GitHub web UI; tables and alert callouts render inline. Start here and follow the Map of Content.
- **Also great in [Obsidian](https://obsidian.md/)** — clone the repo and open the folder as a vault. The same relative links resolve, so click-through navigation, backlinks, and the graph view all work.

## Conventions

- Commands are written for a **Windows target** with a **Kali Linux** attacker unless noted; adapt IPs, paths, and account names to your environment.
- IP addresses (`10.10.14.7` attacker, `10.10.10.5` target), ports, and account/file names are **lab placeholders** — replace them with your own.
- Callouts use GitHub alert syntax (`> [!NOTE]`, `> [!WARNING]`, `> [!TIP]`) and bold-label blockquotes; both render on GitHub and in Obsidian.
- Every technique note pairs **detection** and **defense** guidance — these are documented to be understood and defended against, not just executed.

## License

Content is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — you may share and adapt it with attribution. All techniques are documented for **authorized testing and education only**; verify every command in an isolated lab before use.
