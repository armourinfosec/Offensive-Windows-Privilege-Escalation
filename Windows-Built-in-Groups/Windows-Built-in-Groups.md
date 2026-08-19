# Windows Built-in Groups

Windows ships with a set of privileged built-in groups whose membership grants powerful rights short of full administrator — and several of them are a direct path to SYSTEM or Domain Admin. Attackers frequently find a compromised account added to one of these groups "for convenience" by an administrator who did not realise the escalation it enables. After landing a foothold, always check group membership; a single line in `whoami /groups` can be the whole escalation.

## Enumerate your groups

```cmd
whoami /groups
net user %USERNAME%
net localgroup
```

On a domain, check domain groups too:

```powershell
Get-ADPrincipalGroupMembership $env:USERNAME | Select-Object name
```

## Dangerous groups → escalation

| Group | Why it's dangerous | Note |
|-------|--------------------|------|
| **Backup Operators** | Holds `SeBackup`/`SeRestore` — read/write any file | [Backup Operators](Backup-Operators.md) |
| **DnsAdmins** | Loads an arbitrary DLL into the SYSTEM `dns.exe` service | [DnsAdmins](DnsAdmins.md) |
| **Server Operators** | Can reconfigure and restart services on a DC | [Server Operators](Server-Operators.md) |
| **Print Operators** | Holds `SeLoadDriver` — load a vulnerable kernel driver | [Print Operators](Print-Operators.md) |
| **Hyper-V Administrators** | Full control of the hypervisor and VM files | [Hyper V Administrators](Hyper-V-Administrators.md) |
| **Event Log Readers** | Reads the Security log — credentials in process command lines | [Event Log Readers](Event-Log-Readers.md) |

## Detection and defenses

- **Detection:** unexpected membership changes to these groups (Event ID 4728/4732/4756), privileged group members performing service/driver/DLL operations.
- **Defenses:** treat these groups as tier-0; audit membership regularly; grant them only to accounts that genuinely require the function, and prefer scoped delegation over broad group membership.

## Related
- [Windows Privilege Escalation](../README.md) — category MOC
- [Token Privilege Abuse](../Token-Privilege-Abuse/Token-Privilege-Abuse.md) — the privileges several of these groups confer
- [Escalate My Privilege Windows](../Escalate-My-Privilege-Windows.md) — where group membership sits in the methodology
