# Token Privilege Abuse

Windows attaches a set of **privileges** to every access token — rights like "back up any file" or "debug any process" that bypass normal ACLs. Service accounts and some admin-adjacent users hold privileges that, while not "administrator", are individually enough to reach SYSTEM. The first command after landing any Windows shell should therefore be `whoami /priv`: a single enabled privilege in the table below is often a complete escalation path. This hub maps each dangerous privilege to its exploit; follow the links into the technique notes.

## Enumerate your privileges

```cmd
whoami /priv
```

A privilege listed as **Disabled** is still exploitable — a process can enable any privilege already present in its token. What matters is that the privilege is *held*, not its current state.

## Privilege → exploit map

| Privilege | What it grants | Path to SYSTEM |
|-----------|----------------|----------------|
| `SeImpersonatePrivilege` | Impersonate a client after authentication | Potato attacks → [PrintSpoofer](../Impersonation-and-Potato-Attacks/PrintSpoofer.md), [RoguePotato](../Impersonation-and-Potato-Attacks/RoguePotato.md), [God Potato](../Impersonation-and-Potato-Attacks/God-Potato.md) |
| `SeAssignPrimaryTokenPrivilege` | Assign a primary token to a new process | Same potato family as SeImpersonate |
| `SeBackupPrivilege` / `SeRestorePrivilege` | Read/write **any** file, ignoring ACLs | [SeBackup and SeRestore Abuse](SeBackup-and-SeRestore-Abuse.md) — steal SAM/SYSTEM or overwrite files |
| `SeTakeOwnershipPrivilege` | Take ownership of any securable object | [SeTakeOwnership Abuse](SeTakeOwnership-Abuse.md) — own then rewrite a SYSTEM binary/DLL |
| `SeLoadDriverPrivilege` | Load a kernel driver | [SeLoadDriver Abuse](SeLoadDriver-Abuse.md) — load a vulnerable driver, run code in the kernel |
| `SeDebugPrivilege` | Open **any** process | [SeDebug Abuse](SeDebug-Abuse.md) — inject into / dump a SYSTEM process (e.g. LSASS) |
| `SeManageVolumePrivilege` | Full volume access | Gain write to `C:\` root → DLL hijack a privileged service |
| `SeTcbPrivilege` | Act as part of the OS | Craft a token with SYSTEM group membership |

## General approach

1. `whoami /priv` — identify held privileges.
2. Match against the table; pick the note for the strongest one.
3. Execute, get SYSTEM, verify with `whoami`, and clean up any dropped tooling.

## Detection and defenses

- **Detection:** Event ID 4672 (special privileges assigned at logon), 4673/4674 (privileged service/object use), and process-creation telemetry showing a service account spawning a shell.
- **Defenses:** grant these privileges only to accounts that genuinely require them (audit via `secpol.msc` / group policy "User Rights Assignment"); prefer virtual/service accounts scoped tightly; monitor privilege-use auditing.

## Related
- [Windows Privilege Escalation](../README.md) — category MOC
- [Impersonation and Potato Attacks](../Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) — the SeImpersonate/SeAssignPrimaryToken branch
- [Password Mining](../Password-Mining/Password-Mining.md) — what to do with SeBackup once you can read SAM/SYSTEM
- [Escalate My Privilege Windows](../Escalate-My-Privilege-Windows.md) — the methodology checklist that starts here
