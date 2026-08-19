# Token Privilege Abuse

Windows attaches a set of **privileges** to every access token — rights like "back up any file" or "debug any process" that bypass normal…

## Notes
- [Token Privilege Abuse](Token-Privilege-Abuse.md) — folder overview
- [SeBackup and SeRestore Abuse](SeBackup-and-SeRestore-Abuse.md) — SeBackupPrivilege lets a token **read any file on the system regardless of its ACL**, and SeRestorePrivilege lets it **write any file** the…
- [SeDebug Abuse](SeDebug-Abuse.md) — SeDebugPrivilege lets a token **open a handle to any process on the system**, including those owned by SYSTEM — it exists so debuggers can…
- [SeLoadDriver Abuse](SeLoadDriver-Abuse.md) — SeLoadDriverPrivilege lets a token **load a kernel-mode driver**.
- [SeTakeOwnership Abuse](SeTakeOwnership-Abuse.md) — SeTakeOwnershipPrivilege allows a token to **take ownership of any securable object** without being granted access first.

## Navigation
- [Windows Privilege Escalation (module root)](../README.md)
