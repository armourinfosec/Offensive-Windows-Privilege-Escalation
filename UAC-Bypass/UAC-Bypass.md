# UAC Bypass

User Account Control (UAC) is not a security boundary — Microsoft says so explicitly — but it is the barrier between a **medium-integrity** admin shell and a **high-integrity** (elevated) one. If you already have a user in the local Administrators group but your process is unelevated (no consent prompt was accepted), a UAC bypass silently promotes you to a full administrative token without a prompt. The common family abuses **auto-elevating** Microsoft binaries: signed executables that elevate themselves without a prompt and can be hijacked into running your code via a writable registry key they read.

## When this applies

You need a UAC bypass (not a full privilege-escalation exploit) when **both** are true:

```cmd
whoami /groups | findstr /i "S-1-5-32-544"   :: member of Administrators
whoami /groups | findstr /i "Medium"          :: but running at Medium integrity
```

If you are only a standard user, UAC bypass does not apply — use a real vector from [Windows Privilege Escalation](../README.md) instead.

## The auto-elevate + registry-hijack pattern

Most fileless UAC bypasses share one shape:

1. An auto-elevating signed binary reads a **per-user (HKCU)** registry value to decide what to run.
2. HKCU is writable by the standard user, so you point that value at your payload.
3. You launch the binary; it auto-elevates and runs your payload at high integrity.
4. You delete the key to clean up.

## Techniques in this folder

- [UAC Bypass via fodhelper](UAC-Bypass-via-fodhelper.md) — `fodhelper.exe` + `HKCU\...\ms-settings` (the most common).
- [UAC Bypass via eventvwr](UAC-Bypass-via-eventvwr.md) — `eventvwr.exe` + `HKCU\...\mscfile` (the classic).
- [UAC Bypass via computerdefaults](UAC-Bypass-via-computerdefaults.md) — `computerdefaults.exe`, an `ms-settings` sibling of fodhelper.
- [UAC Bypass via sdclt.exe and App Paths Hijack](../Registry-Exploitation/UAC-Bypass-via-sdclt.exe-and-App-Paths-Hijack.md) — `sdclt.exe` via App Paths / `IsolatedCommand`.

The **UACME** project (hfiref0x) catalogues 70+ such methods; the notes above cover the ones you will reach for by hand.

## Detection and defenses

- **Detection:** writes to `HKCU\Software\Classes\ms-settings\...\command` or `mscfile\...\command`, an auto-elevating binary spawning `cmd`/`powershell`, Sysmon Event ID 12/13 (registry) + 1 (process create).
- **Defenses:** set UAC to **"Always notify"** (defeats the auto-elevate assumption), remove admin rights from day-to-day accounts, monitor the known hijack keys.

## Related
- [Windows Privilege Escalation](../README.md) — category MOC
- [Registry Exploitation Techniques](../Registry-Exploitation/Registry-Exploitation-Techniques.md) — the registry-hijack mechanics these rely on
- [Autorun Registry Persistence](../Registry-Exploitation/Autorun-Registry-Persistence.md) — related HKCU registry abuse for persistence
- [Escalate My Privilege Windows](../Escalate-My-Privilege-Windows.md) — where UAC bypass sits in the methodology
