# DLL Injection

DLL injection forces a **running process** to load an attacker-supplied DLL, executing code inside that process's security context. In a privilege-escalation setting the target is a process owned by SYSTEM or a higher-privileged user: inject your DLL and you inherit its token. It differs from [DLL hijacking](Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md) — hijacking waits for a process to load a DLL from a writable path, while injection actively pushes a DLL into an already-running process (and usually needs a privilege such as `SeDebugPrivilege` to open the target).

## Prerequisites

- The ability to open the target process — typically `SeDebugPrivilege` (see [SeDebug Abuse](Token-Privilege-Abuse/SeDebug-Abuse.md)) or being same-or-higher integrity than the target.

```cmd
whoami /priv | findstr /i "SeDebug"
```

## Classic injection flow

The traditional `CreateRemoteThread` + `LoadLibrary` technique:

1. `OpenProcess(PROCESS_ALL_ACCESS, pid)` on the SYSTEM process.
2. `VirtualAllocEx` a buffer in it and `WriteProcessMemory` the DLL path.
3. `CreateRemoteThread` pointing at `LoadLibraryA` with that buffer → the DLL's `DllMain` runs inside the target.

Tooling makes this a one-liner:

```powershell
# PowerSploit
Invoke-DllInjection -ProcessID <SYSTEM_PID> -Dll C:\Windows\Temp\evil.dll
```

```cmd
:: standalone injector
injector.exe <SYSTEM_PID> C:\Windows\Temp\evil.dll
```

Build the DLL with msfvenom (`-f dll`) or a small `DllMain` that spawns a shell.

## Detection and defenses

- **Detection:** `CreateRemoteThread` into another process (Sysmon Event ID 8), remote memory writes, a SYSTEM process loading an unsigned DLL from a temp path (Event ID 7), handle opens to SYSTEM processes.
- **Defenses:** restrict `SeDebugPrivilege`; enable protected processes (RunAsPPL) for sensitive targets; EDR that monitors remote-thread creation.

## Related
- [Dynamic Link Library Hijacking(DLL Hijacking)](Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md) — the passive load-path counterpart
- [SeDebug Abuse](Token-Privilege-Abuse/SeDebug-Abuse.md) — the privilege that lets you open a SYSTEM process
- [LSASS Credential Dumping](Password-Mining/LSASS-Credential-Dumping.md) — a common post-injection objective
- [Windows Privilege Escalation](README.md) — category MOC
