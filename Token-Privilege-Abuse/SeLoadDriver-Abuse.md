# SeLoadDriver Abuse

`SeLoadDriverPrivilege` lets a token **load a kernel-mode driver**. Because kernel code runs with the highest privilege on the system, loading a *vulnerable* signed driver (a "bring your own vulnerable driver" / BYOVD attack) hands you an arbitrary kernel read/write primitive that trivially escalates to SYSTEM. The classic proof-of-concept loads the vulnerable **Capcom.sys**, whose `IOCTL` executes an attacker-supplied function pointer in ring 0.

## Confirm the privilege

```cmd
whoami /priv | findstr /i "SeLoadDriver"
```

## How it works

1. Register a driver service pointing at the vulnerable `.sys` under an `HKCU` (user-writable) path — `NTLoadDriver` accepts a per-user registry path, so admin rights to `HKLM\SYSTEM\...\Services` are not required.
2. Call `NTLoadDriver` to load it with `SeLoadDriverPrivilege`.
3. Interact with the driver's IOCTL to run code in the kernel → escalate to SYSTEM.

## Exploitation

Point a registry key at the vulnerable driver and load it with a helper such as `EoPLoadDriver`:

```cmd
EoPLoadDriver.exe System\CurrentControlSet\MyService C:\Windows\Temp\Capcom.sys
```

Then trigger the vulnerable IOCTL with the matching exploit (e.g. `ExploitCapcom.exe`) to spawn a SYSTEM shell.

> [!warning]
> Loading a kernel driver can crash (BSOD) the target. Only do this on a host you own or are explicitly authorized to test, and expect potential instability.

## Detection and defenses

- **Detection:** driver-load events (Sysmon Event ID 6), new `HKCU`/service registry entries pointing to a non-standard `.sys`, known-vulnerable-driver hashes (Microsoft's blocklist), Event ID 4674.
- **Defenses:** remove `SeLoadDriverPrivilege` from non-admins; enable **Microsoft vulnerable-driver blocklist** and HVCI/Memory Integrity; require WHQL-signed drivers.

## Related
- [Token Privilege Abuse](Token-Privilege-Abuse.md) — hub mapping every dangerous privilege
- [Windows Kernel Exploits](../Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — kernel-level code execution once the driver is loaded
- [Windows Version and Configuration](../Windows-Version-and-Configuration.md) — determine whether HVCI/blocklist is enabled
