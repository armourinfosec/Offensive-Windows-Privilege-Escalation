# SeDebug Abuse

`SeDebugPrivilege` lets a token **open a handle to any process on the system**, including those owned by SYSTEM — it exists so debuggers can attach to arbitrary processes, but it is effectively a master key. With it you can inject code into a SYSTEM process, duplicate a SYSTEM process's token, or dump **LSASS** for credentials. It is held by local administrators by default, but also appears on some service accounts, where it becomes a direct escalation path.

## Confirm the privilege

```cmd
whoami /priv | findstr /i "SeDebug"
```

## Technique 1 — steal a SYSTEM token

Open a SYSTEM-owned process, duplicate its token, and spawn a new process with it. Tools like `psgetsystem`, `Invoke-TokenManipulation`, or a small C# stub do this:

```powershell
Import-Module .\Invoke-TokenManipulation.ps1
Invoke-TokenManipulation -CreateProcess "cmd.exe" -Username "NT AUTHORITY\SYSTEM"
```

## Technique 2 — dump LSASS for credentials

`SeDebugPrivilege` is what lets a process read LSASS memory:

```cmd
procdump.exe -accepteula -ma lsass.exe C:\Windows\Temp\lsass.dmp
```

Then parse offline for hashes/tickets → [LSASS Credential Dumping](../Password-Mining/LSASS-Credential-Dumping.md), Mimikatz Usage and Execution.

## Technique 3 — meterpreter

```text
meterpreter > getprivs        # confirms SeDebugPrivilege
meterpreter > migrate <SYSTEM-PID>
```

## Detection and defenses

- **Detection:** processes opening LSASS with `PROCESS_VM_READ` (Sysmon Event ID 10), `procdump`/comsvcs `MiniDump` of lsass, token-manipulation patterns, Event ID 4674.
- **Defenses:** restrict `SeDebugPrivilege`; enable **LSASS protection (RunAsPPL)** and Credential Guard; alert on LSASS handle access.

## Related
- [Token Privilege Abuse](Token-Privilege-Abuse.md) — hub mapping every dangerous privilege
- [LSASS Credential Dumping](../Password-Mining/LSASS-Credential-Dumping.md) — the credential-theft payoff of this privilege
- Mimikatz Usage and Execution — parse the LSASS dump for secrets
- [Windows Kernel Exploits](../Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — alternative SYSTEM routes
