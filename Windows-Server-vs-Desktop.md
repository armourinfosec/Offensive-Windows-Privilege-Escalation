# Windows Server vs Desktop

The same enumeration finds different escalation paths depending on whether you have landed on a Windows **Server** or a **Desktop** (client) edition — and, for servers, on which roles it runs. Orienting to the target type early tells you which vectors are worth your time: a domain controller's built-in groups and AD roles, versus a workstation's stored user credentials and browser loot. This is a context note; each vector links to its technique.

## Tell them apart

```cmd
systeminfo | findstr /B /C:"OS Name"
wmic os get Caption, ProductType
```

`ProductType` = 1 is a workstation; 2 is a domain controller; 3 is a member/standalone server. On a domain, confirm DC status:

```cmd
nltest /dclist:%USERDNSDOMAIN%
```

## Servers — role-driven paths

- **Domain Controllers** elevate the stakes: privileged **built-in groups** become domain-compromise paths — [Server Operators](Windows-Built-in-Groups/Server-Operators.md) (reconfigure services on the DC), [DnsAdmins](Windows-Built-in-Groups/DnsAdmins.md) (SYSTEM via a DNS plugin DLL), [Backup Operators](Windows-Built-in-Groups/Backup-Operators.md) (`NTDS.dit` via [SeBackup and SeRestore Abuse](Token-Privilege-Abuse/SeBackup-and-SeRestore-Abuse.md)). See [Windows Built in Groups](Windows-Built-in-Groups/Windows-Built-in-Groups.md).
- **More services, more surface.** Servers run databases, web servers, and backup agents — richer [service](Services-Exploitation/Services-Exploitation.md) and [GPP](Password-Mining/Group-Policy-Preferences-cpassword.md) targets, and service accounts holding `SeImpersonatePrivilege` for [potato](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) attacks.
- **Hyper-V hosts** expose [Hyper V Administrators](Windows-Built-in-Groups/Hyper-V-Administrators.md) escalation.
- **AD context** turns local SYSTEM into a foothold for domain attacks (out of scope here; hand off to AD tooling).

## Desktops — user-centric paths

- **Stored user credentials** dominate: autologon registry, saved RDP/VPN, browser passwords, DPAPI — [Password Mining](Password-Mining/Password-Mining.md), [Pillaging](Pillaging.md).
- **Interactive users** to snoop on ([Interacting with Users](Interacting-with-Users.md)), and **UAC bypass** when you are an unelevated admin ([UAC Bypass](UAC-Bypass/UAC-Bypass.md)).
- Fewer services, but common third-party apps with [DLL-hijack](Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md) and [unquoted-path](Services-Exploitation/Unquoted-Service-Path-Vulnerability.md) flaws.

## Detection and defenses

- **Defenses:** tier your administration (separate DC/server/workstation admin), treat DC built-in groups as tier-0, and apply the consolidated checklist in [Windows Hardening](Windows-Hardening.md).

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Windows Built in Groups](Windows-Built-in-Groups/Windows-Built-in-Groups.md) — the server/DC group-based paths
- [Password Mining](Password-Mining/Password-Mining.md) · [Pillaging](Pillaging.md) — the desktop credential paths
- [Legacy Operating Systems](Legacy-Operating-Systems.md) — how EOL editions shift the approach
- [Windows Hardening](Windows-Hardening.md) — defending both target types
