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
