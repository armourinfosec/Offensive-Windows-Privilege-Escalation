# Communication with Processes

Windows processes rarely run in isolation — they talk to one another through named pipes, RPC/DCOM, COM, and ALPC. Each of those inter-process channels is also an escalation surface: if a **privileged** process exposes an endpoint that a low-privileged user can reach, you may be able to impersonate it, feed it malicious input, or hijack what it loads. Understanding these channels is what turns "there's a SYSTEM service running" into a concrete escalation path — it is the mechanism behind the potato family and named-pipe impersonation.

## Named pipes

Named pipes are the most directly abusable IPC channel. Enumerate the pipes a host exposes:

```powershell
Get-ChildItem \\.\pipe\
[System.IO.Directory]::GetFiles("\\.\pipe\")
```

A server that calls `ImpersonateNamedPipeClient` adopts the token of whoever connects — so if *you* can make a SYSTEM service connect to *your* pipe, you capture its token. That is exactly the primitive [PrintSpoofer](Impersonation-and-Potato-Attacks/PrintSpoofer.md) and [RoguePotato](Impersonation-and-Potato-Attacks/RoguePotato.md) weaponise. The reverse also matters: a privileged pipe **server** you can talk to may trust your input more than it should. Full walkthrough: [Named Pipes](Services-Exploitation/Named-Pipes.md).

## RPC and DCOM

Remote Procedure Call (and its object-oriented cousin DCOM) let one process invoke functions in another, often across a SYSTEM boundary. The potato attacks abuse DCOM activation to coerce a SYSTEM COM server into authenticating to a resolver you control ([Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md)). Enumerate DCOM apps:

```powershell
Get-CimInstance Win32_DCOMApplication | Select-Object Name, AppID
```

Misconfigured DCOM launch/access permissions (via `dcomcnfg`) can let a low-priv user activate a privileged object.

## COM hijacking

If a privileged process loads a COM object by CLSID and the per-user (`HKCU`) registration is checked first, you can register a malicious in-process server for that CLSID and have your DLL loaded in the privileged context — a cousin of DLL hijacking. Hunt abandoned/hijackable CLSIDs under `HKCU\Software\Classes\CLSID`.

## ALPC / LPC

Advanced Local Procedure Call is the low-level channel underneath RPC and much of the Windows subsystem. It is rarely abused by hand but underpins several kernel/privilege exploits (e.g. the ALPC/Task Scheduler bug) — relevant when chasing [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md).

## Detection and defenses

- **Detection:** anomalous named-pipe creation/connection (Sysmon Event ID 17/18), unexpected DCOM activation, `HKCU\...\CLSID` writes.
- **Defenses:** least privilege on service accounts; tighten DCOM launch/access ACLs; monitor pipe and COM registration activity.

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Named Pipes](Services-Exploitation/Named-Pipes.md) — hands-on named-pipe impersonation
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) — DCOM/pipe coercion in practice
- [Token Impersonation](Impersonation-and-Potato-Attacks/Token-Impersonation.md) — how the captured tokens are used
- [Dynamic Link Library Hijacking(DLL Hijacking)](Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md) — COM hijacking's sibling
