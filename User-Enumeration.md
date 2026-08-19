# User Enumeration Cheatsheet

This document provides essential commands for enumerating users, privileges, groups, and logon requirements in a Windows environment. Proper user enumeration is critical for privilege escalation and lateral movement.

- Displays the current logged-in username.

> Using `echo` or `whoami`:

```bash
echo %USERNAME%
```

```bash
whoami
```

> Using PowerShell:

```powershell
$env:username
```


- Displays the privileges of the current user.

```bash
whoami /priv
```


- Displays all groups the current user belongs to.

```bash
whoami /groups
```


- Provides detailed information about the current user, including security identifiers (SIDs), groups, privileges, and logon ID.

```bash
whoami /all
```


- If `whoami.exe` is missing or deleted, you can copy it from a network share:

```bash
copy \\192.168.1.5\Public\whoami.exe whoami.exe
```

- Lists all user accounts on the system.

> Using `net user`:

```bash
net user
```

> Using PowerShell:

```powershell
Get-LocalUser | ft Name,Enabled,LastLogon
```

> Directly from the `Users` directory:

```powershell
Get-ChildItem C:\Users -Force | select Name
```

- Displays logon policies like password expiration, lockout duration, and complexity requirements — useful for password brute-forcing strategies.

```bash
net accounts
```


- Retrieve detailed information about a specific user.

> Administrator:

```bash
net user administrator
```

> Admin:

```bash
net user admin
```

> Current User:

```bash
net user %USERNAME%
```

- Displays a list of all local groups on the machine.

> Using `net localgroup`:

```bash
net localgroup
```

> Using PowerShell:

```powershell
Get-LocalGroup | ft Name
```

- Displays details about a specific local group.

> Administrators:

```bash
net localgroup administrators
```

> Domain Admins:

```bash
net group "Domain Admins"
```

> Using PowerShell:

```powershell
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```


## Usage Tips:

- `net user` and `net localgroup` work on most Windows versions.

- PowerShell offers more advanced filtering and formatting options.

- Use `whoami /priv` to quickly identify if the user has administrative or elevated privileges.

- `net accounts` helps identify if weak password policies are in place — potential for brute force!

## Related
- [Network Enumeration](Network-Enumeration.md) — companion network enumeration
- [Windows Version and Configuration](Windows-Version-and-Configuration.md) — companion system enumeration
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) — whoami /priv reveals SeImpersonate
- [Token Impersonation](Impersonation-and-Potato-Attacks/Token-Impersonation.md) — exploit discovered privileges
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — automate these checks
