# Print Operators

**Print Operators** is deceptively powerful: members hold **`SeLoadDriverPrivilege`**, the right to load kernel-mode drivers. Loading a *vulnerable* signed driver (a bring-your-own-vulnerable-driver attack) yields a kernel read/write primitive that escalates to SYSTEM — so Print Operators membership is effectively SYSTEM, even though the group's stated purpose is only managing printers. The privilege abuse itself is detailed in [SeLoadDriver Abuse](../Token-Privilege-Abuse/SeLoadDriver-Abuse.md).

## Confirm membership and privilege

```cmd
whoami /groups | findstr /i "Print Operators"
whoami /priv   | findstr /i "SeLoadDriver"
```

## Exploitation

1. Register a driver service under a user-writable registry path (no admin needed with `SeLoadDriverPrivilege`) pointing at a vulnerable `.sys`, and load it:

```cmd
EoPLoadDriver.exe System\CurrentControlSet\MyService C:\Windows\Temp\Capcom.sys
```

2. Trigger the vulnerable driver's IOCTL to execute code in the kernel and spawn a SYSTEM shell (e.g. `ExploitCapcom.exe`).

> [!warning]
> Loading a kernel driver can BSOD the host. Only do this on a system you own or are explicitly authorized to test.

## Detection and defenses

- **Detection:** driver-load events (Sysmon Event ID 6) for non-standard/known-vulnerable `.sys`, new per-user driver-service registry entries, Event ID 4674.
- **Defenses:** minimise Print Operators membership; enable the **Microsoft vulnerable-driver blocklist** and HVCI/Memory Integrity; remove `SeLoadDriverPrivilege` where not required.

## Related
- [Windows Built in Groups](Windows-Built-in-Groups.md) — group overview and escalation map
- [SeLoadDriver Abuse](../Token-Privilege-Abuse/SeLoadDriver-Abuse.md) — the underlying privilege abuse in detail
- [Windows Kernel Exploits](../Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — kernel code execution once the driver loads
