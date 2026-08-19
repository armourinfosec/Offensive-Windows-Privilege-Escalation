# Web Configuration Files and Sensitive Data Discovery

## Common Web Configuration Files

Web configuration files often store sensitive information such as database connection strings, credentials, and API keys. These files are valuable targets for privilege escalation or lateral movement.

## Typical Locations of Web Configuration Files

Common locations for `web.config` files on a Windows system:

```cmd
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
```

```cmd
C:\inetpub\wwwroot\web.config
```

## Finding Web Configuration Files

- You can use PowerShell to recursively search for `web.config` files:

```powershell
Get-ChildItem -Path C:\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
Get-ChildItem -Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

>This command:

- Searches within `C:\inetpub\` (default IIS root)

- Filters for files named `web.config`

- Recursively checks subdirectories

- Suppresses error messages

## Other Potentially Sensitive Configuration Files

- Files containing database credentials, API keys, or other secrets:

|File Name|Description|
|---|---|
|`db_cont.php`|Database connection details (often includes username and password)|
|`db.php`|PHP file storing database credentials|
|`wp-config.php`|WordPress configuration file (contains database credentials, salts, and keys)|

- Example search for these files using PowerShell:

```powershell
Get-ChildItem -Path C:\ -Include db_cont.php,db.php,wp-config.php -File -Recurse -ErrorAction SilentlyContinue
```

- Example search using `findstr` in the Command Prompt:

```cmd
findstr /si "password" *.php *.config *.xml *.ini
```

> This command:

- Recursively searches files for the term `password`

- Checks common configuration file extensions (`.php`, `.config`, `.xml`, `.ini`)


## Direct File Discovery with `dir`

- You can also list matching files directly with `dir`:

```cmd
dir /S /B *pass*.txt *pass*.xml *pass*.ini *cred* *vnc* *.config*
```

This command:

- `/S` — Search subdirectories

- `/B` — Display bare format (only file paths)

- Looks for patterns like `pass`, `cred`, `vnc`, or `.config`


### Example: Extracting Credentials from `web.config`

- Example PowerShell command to extract sensitive values from `web.config`:

```powershell
Select-String -Path "C:\inetpub\wwwroot\web.config" -Pattern "connectionString"
```

> Example output:

```text
<connectionString>Data Source=localhost;Initial Catalog=mydb;User ID=admin;Password=P@ssw0rd!</connectionString>
```


- If a `.NET` web application uses a `web.config` file to store credentials, they can often be found in the `<connectionStrings>` or `<appSettings>` section. Here's an example:

### Example `web.config` with plaintext credentials:

```xml
<configuration>
  <connectionStrings>
    <add name="MyDB"
         connectionString="Data Source=localhost;Initial Catalog=mydb;User ID=admin;Password=SuperSecret123!" />
  </connectionStrings>
  
  <appSettings>
    <add key="APIKey" value="12345-abcdef-67890-ghijk" />
    <add key="SMTPUser" value="smtpuser@example.com" />
    <add key="SMTPPass" value="smtpsecret" />
  </appSettings>
</configuration>
```

### Where to Look:

1. Connection Strings – Often hold database credentials (SQL, MySQL, etc.).

2. App Settings – Can store API keys, SMTP credentials, or other sensitive info.

3. Identity Configuration – Sometimes holds OAuth tokens, client secrets, or encryption keys.

### Summary

- Search for sensitive configuration files using `Get-ChildItem`, `findstr`, and `dir`  

- Target common configuration files like `web.config`, `db.php`, and `wp-config.php`  

- Extract sensitive credentials directly from config files

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Unattended Install Files(Cleartext Passwords)](Unattended-Install-Files(Cleartext-Passwords).md) — sibling cleartext-credential source
- [Search for file contents](Search-for-file-contents/Search-for-file-contents.md) — searching the filesystem for secrets
- [Windows Privilege Escalation](../README.md) — escalation context
