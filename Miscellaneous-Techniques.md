# Miscellaneous Techniques

A curated set of smaller Windows privilege-escalation and credential-access techniques that do not warrant a full note of their own — or that mostly point back to techniques covered elsewhere in the module. Treat this as a checklist of "also try" items once the primary vectors ([the methodology](Escalate-My-Privilege-Windows.md)) come up empty; several of these are the difference between a stuck engagement and a foothold.

## Saved RDP and remote-access credentials

Windows caches RDP session credentials; saved `.rdp` files and Credential Manager entries (`TERMSRV/*`) frequently hold admin passwords:

```cmd
cmdkey /list
dir /s /b %USERPROFILE%\*.rdp
```

DPAPI-protected RDP passwords can be recovered in the user's context — see [Pillaging](Pillaging.md).

## Sticky Keys / Utilman (physical or RDP)

With write access to `System32` (or via an offline attack), replacing an accessibility binary yields a SYSTEM shell at the logon screen:

```cmd
copy C:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
:: press SHIFT x5 at the lock screen  ->  SYSTEM cmd
```

Utilman.exe (Win+U) works the same way. Primarily a persistence/backdoor and offline-attack technique.

## WSL as a pivot

The Windows Subsystem for Linux can host tooling and, if a root WSL shell exists, read Windows files across the `/mnt/c` boundary — see [Escalation Path via Windows Subsystem for Linux(WSL)](Escalation-Path-via-Windows-Subsystem-for-Linux(WSL).md).

## Backup and third-party software

Backup agents and updaters run as SYSTEM and are a recurring privesc source (e.g. [Iperius Backup 6.1.0 Privilege Escalation](Iperius-Backup-6.1.0-Privilege-Escalation.md)). Enumerate installed software ([Windows Version and Configuration](Windows-Version-and-Configuration.md)) and check each against known local-privesc CVEs.

## COM / TypeLib hijacking

Abandoned per-user COM registrations let you load a DLL into a privileged process — introduced in [Communication with Processes](Communication-with-Processes.md).

## Clipboard and screen capture

A logged-on admin's clipboard or screen may leak secrets — [Interacting with Users](Interacting-with-Users.md).

## Detection and defenses

- **Detection:** accessibility-binary replacement (`sethc.exe`/`utilman.exe` hash change), `cmdkey`/`.rdp` access, SYSTEM shells spawned from the logon UI.
- **Defenses:** covered centrally in [Windows Hardening](Windows-Hardening.md) — patch third-party software, restrict `System32` write access, and clear saved RDP credentials.

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Pillaging](Pillaging.md) — saved credentials and DPAPI recovery
- [Interacting with Users](Interacting-with-Users.md) — clipboard/screen capture
- [Communication with Processes](Communication-with-Processes.md) — COM hijacking
- [Windows Hardening](Windows-Hardening.md) — defenses for these techniques
