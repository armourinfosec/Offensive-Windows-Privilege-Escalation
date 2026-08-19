# Skills Assessment Part 1

A self-check on the **enumeration and misconfiguration** half of this module, run against the practice box built by [Windows Privilege Escalation Lab Setup](Windows-Privilege-Escalation-Lab-Setup.md). Provision that VM, log in as the low-privileged `lowpriv` account, and work the objectives below without looking at the technique notes first — reach for them only as hints. The goal is to internalise the enumerate → identify → exploit loop, not to memorise commands.

> [!warning]
> Run only against the disposable lab VM you provisioned and control. Snapshot first.

## Setup

On the target VM (as administrator), run the provisioning script from [Windows Privilege Escalation Lab Setup](Windows-Privilege-Escalation-Lab-Setup.md), then log in / `runas` as `lowpriv` (`Lab_P@ss123`).

## Objectives

Reach `nt authority\system` **three different ways** using only misconfiguration vectors, and answer the checks:

1. **Enumerate first.** Run [PowerShell Privilege Escalation Enumeration](../PowerShell-Privilege-Escalation-Enumeration.md) (or enumerate by hand). How many distinct escalation vectors does it flag? List them.
2. **Unquoted service path.** Identify the vulnerable service and its writable intermediate directory. What file path would you drop a payload at, and which command starts the service? → hint: [Unquoted Service Path Vulnerability](../Services-Exploitation/Unquoted-Service-Path-Vulnerability.md).
3. **Weak service configuration.** Find the service whose DACL lets `Users` change its configuration. Reconfigure its `binPath` to add your account to `Administrators` and confirm. → hint: [Insecure Service Permissions(binPath)](../Services-Exploitation/Insecure-Service-Permissions(binPath).md).
4. **Writable service binary.** Locate the service whose executable you can overwrite. → hint: [Insecure File Permissions Service Executable Files Path](../Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md).
5. **AlwaysInstallElevated.** Confirm the policy, build an MSI, and install it as SYSTEM. What are the two registry values that must both be set? → hint: [AlwaysInstallElevated Exploitation](../Registry-Exploitation/AlwaysInstallElevated-Exploitation.md).
6. **Scheduled task.** Which task runs as SYSTEM against a writable binary, and how often does it fire? → hint: [Privilege Escalation via Scheduled Tasks](../Privilege-Escalation-via-Scheduled-Tasks.md).

## Verify

For each successful path:

```cmd
whoami
:: -> nt authority\system
```

You have passed Part 1 when you have reached SYSTEM via **at least three** of objectives 2–6 and can explain, for each, the single misconfiguration that made it possible and the one-line hardening that would close it (cross-check against [Windows Hardening](../Windows-Hardening.md)).

## Related
- [Windows Privilege Escalation Lab Setup](Windows-Privilege-Escalation-Lab-Setup.md) — provisions the target for this assessment
- [Skills Assessment Part 2](Skills-Assessment-Part-2.md) — the token-privilege and credential half
- [Escalate My Privilege Windows](../Escalate-My-Privilege-Windows.md) — the methodology being tested
- [PowerShell Privilege Escalation Enumeration](../PowerShell-Privilege-Escalation-Enumeration.md) — enumerate the vectors
- [Services Exploitation](../Services-Exploitation/Services-Exploitation.md) · [Registry Exploitation Techniques](../Registry-Exploitation/Registry-Exploitation-Techniques.md) — the vector families
