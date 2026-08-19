# Skills Assessment Part 2

A self-check on the **token-privilege, group-membership, and credential** half of this module, against the same [Windows Privilege Escalation Lab Setup](Windows-Privilege-Escalation-Lab-Setup.md) box. Where Part 1 exercised service and registry misconfigurations, Part 2 tests the privileges and secrets a low-privileged account can already leverage. Work the objectives as `lowpriv` before consulting the technique notes.

> [!warning]
> Run only against the disposable lab VM you provisioned and control. Snapshot first.

## Setup

Provision the box with [Windows Privilege Escalation Lab Setup](Windows-Privilege-Escalation-Lab-Setup.md) and log in as `lowpriv` (`Lab_P@ss123`). Objective 1 requires re-logging in after provisioning so the granted privilege and group appear in your token.

## Objectives

1. **Read your token.** Run `whoami /priv` and `whoami /groups`. Which dangerous privilege and which privileged built-in group has `lowpriv` been granted? → hints: [Token Privilege Abuse](../Token-Privilege-Abuse/Token-Privilege-Abuse.md), [Windows Built in Groups](../Windows-Built-in-Groups/Windows-Built-in-Groups.md).
2. **Potato to SYSTEM.** Using the granted `SeImpersonatePrivilege`, escalate to SYSTEM with a potato. Which potato is appropriate for the box's Windows build, and why? → hints: [PrintSpoofer](../Impersonation-and-Potato-Attacks/PrintSpoofer.md), [God Potato](../Impersonation-and-Potato-Attacks/God-Potato.md), [Impersonation and Potato Attacks](../Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md).
3. **Backup Operators → hive theft.** As a member of Backup Operators, save the `SAM` and `SYSTEM` hives and dump the local Administrator hash offline. What privilege makes this possible even though the hives are ACL-protected? → hints: [Backup Operators](../Windows-Built-in-Groups/Backup-Operators.md), [SeBackup and SeRestore Abuse](../Token-Privilege-Abuse/SeBackup-and-SeRestore-Abuse.md), [SAM and SYSTEM files](../Password-Mining/SAM-and-SYSTEM-files.md).
4. **Credential mining.** Recover **four** distinct stored credentials seeded on the box (registry autologon, unattend file, Credential Manager, PowerShell history). Where does each live? → hints: [Password in Windows Registry](../Password-Mining/Password-in-Windows-Registry.md), [Unattended Install Files(Cleartext Passwords)](../Password-Mining/Unattended-Install-Files(Cleartext-Passwords).md), [PowerShell Command History](../Password-Mining/PowerShell-Command-History.md), [Password Mining](../Password-Mining/Password-Mining.md).
5. **Pass the loot.** Take the Administrator hash from objective 3 and authenticate with it (no cracking). → hint: Pass The Hash Attack.

## Verify

- Objective 2: `whoami` → `nt authority\system`.
- Objective 3: `impacket-secretsdump -sam SAM -system SYSTEM LOCAL` yields the Administrator NT hash.
- Objectives 4–5: you can enumerate all four credentials and authenticate with the recovered hash.

You have passed Part 2 when you have reached SYSTEM via the token/potato path, dumped the hashes via Backup Operators, and recovered every seeded credential — and can name the [Windows Hardening](../Windows-Hardening.md) control that closes each.

## Related
- [Windows Privilege Escalation Lab Setup](Windows-Privilege-Escalation-Lab-Setup.md) — provisions the target for this assessment
- [Skills Assessment Part 1](Skills-Assessment-Part-1.md) — the service/registry half
- [Token Privilege Abuse](../Token-Privilege-Abuse/Token-Privilege-Abuse.md) · [Windows Built in Groups](../Windows-Built-in-Groups/Windows-Built-in-Groups.md) · [Password Mining](../Password-Mining/Password-Mining.md) — the vector families tested
- [SeImpersonate Potato to SYSTEM](SeImpersonate-Potato-to-SYSTEM.md) — a worked example of the potato objective
