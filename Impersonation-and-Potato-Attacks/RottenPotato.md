# RottenPotato

RottenPotato is the original token-impersonation potato and the conceptual ancestor of the whole family. It abuses Windows' NTLM local negotiation: it tricks a SYSTEM COM server (historically **BITS**) into authenticating to a local relay, captures the SYSTEM token during the NTLM exchange, and impersonates it. It is largely of historical and educational value now — patched on Windows 10 1809+ — but understanding it explains why JuicyPotato, RoguePotato, and PrintSpoofer exist. On modern targets, reach for those; `RottenPotatoNG` still works on older builds.

## Requirements

- `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege`.
- A reachable SYSTEM COM server that performs NTLM (e.g. BITS).
- Pre-Windows 10 1809 / Server 2019 (patched after).

```powershell
whoami /priv | findstr /i "SeImpersonate SeAssignPrimaryToken"
```

## How it works

1. Trigger a SYSTEM COM object that initiates NTLM authentication.
2. Perform a **local NTLM relay**: RottenPotato sits between the client and server halves of the handshake.
3. During the exchange it obtains a SYSTEM impersonation token via `CoGetInstanceFromIStorage` / the RPC marshalling trick.
4. `ImpersonateLoggedOnUser` → spawn a process as SYSTEM.

## Exploitation

RottenPotato was most often used through Meterpreter's `incognito` after loading the token, or via the standalone `RottenPotatoNG` binary:

```cmd
MSFRottenPotato.exe t c:\windows\system32\cmd.exe
```

In Meterpreter (token-impersonation workflow):

```text
meterpreter > getprivs
meterpreter > load incognito
meterpreter > list_tokens -u
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"
```

For anything current, use [PrintSpoofer](PrintSpoofer.md) or [RoguePotato](RoguePotato.md) instead.

## Detection and defenses

- **Detection:** local NTLM relay patterns, BITS/COM anomalies, Sysmon token-manipulation and process-creation-as-SYSTEM events.
- **Defenses:** patch (the technique is fixed on modern builds), remove `SeImpersonatePrivilege` where unneeded, monitor for `incognito`-style token theft.

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — folder hub tracing the potato family evolution
- [Token Impersonation](Token-Impersonation.md) — how Windows access tokens and impersonation work
- [Juicy Potato](Juicy-Potato.md) — RottenPotato's standalone COM successor
- [RoguePotato](RoguePotato.md) · [PrintSpoofer](PrintSpoofer.md) — the modern replacements
