# Impersonation and Potato Attacks

[Jeeves HTB](https://app.hackthebox.com/machines/Jeeves)

Potato Attacks are a family of Windows privilege escalation techniques that abuse impersonation tokens and misconfigured services. These attacks typically escalate a low-privileged user to SYSTEM by tricking privileged services into running attacker-controlled payloads.

## What is Token Impersonation?

Windows allows services with certain privileges (like `SeImpersonatePrivilege`) to impersonate tokens of other users. If an attacker can trick a privileged service into impersonating them, they can capture that token and spawn a privileged process.

### Token Impersonation Flow:

1. Trigger a privileged service or COM object.

2. Force or wait for impersonation.

3. Hijack the impersonated token.

4. Execute a payload (e.g., shell) as SYSTEM.

## Hot Potato (Legacy NTLM Relay Exploit)

- [Foxglove Original Article](https://foxglovesecurity.com/2016/01/16/hot-potato/)

- [PentestLab](https://pentestlab.blog/2017/04/13/hot-potato/)

- [J. Lajara’s Guide](https://jlajara.gitlab.io/Potatoes_Windows_Privesc#hotPotato)


### Requirements:

- Windows 7/8/Server 2008–2012

- `SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege`

- SYSTEM service using NTLM


### Technique:

Combines NBNS spoofing, NTLM relay, and token impersonation to hijack a privileged token.

> Deprecated on modern Windows (patched).


## Rotten Potato (Named Pipe / DCOM Abuse)

-  [Rotten Potato on J. Lajara](https://jlajara.gitlab.io/Potatoes_Windows_Privesc#rottenPotato)


### Requirements:

- `SeImpersonatePrivilege`

- SYSTEM COM server (e.g., BITS)


> Patched in Windows 10 1709+. Still works on older systems.


## JuicyPotato (Standalone COM Token Hijack)

- [GitHub](https://github.com/ohpe/juicy-potato)

- [Juicy Potato Reference](https://jlajara.gitlab.io/Potatoes_Windows_Privesc#juicyPotato)


### Requirements:

- `SeImpersonatePrivilege`

- Pre-Windows 10 1809 / Server 2019

- Vulnerable CLSID


### Example:

```cmd
JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -t *
```


## RoguePotato (SMB Capture)

- [GitHub](https://github.com/antonioCoco/RoguePotato)

- [Rogue Potato Guide](https://jlajara.gitlab.io/Potatoes_Windows_Privesc#roguePotato)


### Requirements:

- `SeImpersonatePrivilege`

- Works on Windows 10 1809+


### Example:

```cmd
RoguePotato.exe -e "cmd.exe" -l 9999
```


## PrintSpoofer (Spooler Exploit)

-  [GitHub](https://github.com/itm4n/PrintSpoofer)


### Requirements:

- `SeImpersonatePrivilege`

- Spooler service running


### Example:

```cmd
PrintSpoofer.exe -i -c cmd
```

>  Patched by Microsoft (2021). Effective on older builds.


## SweetPotato (COM + Print Spooler Hybrid)

-  [GitHub](https://github.com/CCob/SweetPotato)

-  [Sweet Potato Guide](https://jlajara.gitlab.io/Potatoes_Windows_Privesc#sweetPotato)


### Requirements:

- `SeImpersonatePrivilege`

- Print Spooler enabled


>  Slightly more stable than PrintSpoofer. COM-based.


## Checking Privileges

Before attempting Potato attacks, verify privileges:

```powershell
whoami /priv
```

Look for:

```text
SeImpersonatePrivilege   Enabled
```


## Defenses and Detection

### Defenses:

- Patch Windows regularly

- Disable Print Spooler if unused

- Restrict `SeImpersonatePrivilege` to trusted services


### Detection Tools:

- Sysmon (with custom rules)

- EDR solutions (e.g., Defender, CrowdStrike)

- Event Logs – `Microsoft-Windows-Security-Auditing`



## Summary Table

|Attack Tool|Privileges Needed|Works On|Notes|
|---|---|---|---|
|HotPotato|SeImpersonate|Legacy systems|NBNS spoof + NTLM relay|
|RottenPotato|SeImpersonate|Pre-Win10 1709|COM + named pipe abuse|
|JuicyPotato|SeImpersonate|Pre-Win10 1809|COM-based standalone|
|RoguePotato|SeImpersonate|Win10 1809+|SMB impersonation technique|
|PrintSpoofer|SeImpersonate|Pre-2021 patch|Spooler token abuse|
|SweetPotato|SeImpersonate|Mixed|Print Spooler + COM exploitation|


## Additional Resources

- [Tokenvator (for token manipulation)](https://github.com/hatRiot/tokenvator)

- [J. Lajara’s Full Potato Collection](https://jlajara.gitlab.io/Potatoes_Windows_Privesc)

- [PrintSpoofer](https://github.com/itm4n/PrintSpoofer)

- [RoguePotato](https://github.com/antonioCoco/RoguePotato)

- [JuicyPotato](https://github.com/ohpe/juicy-potato)

- [SweetPotato](https://github.com/CCob/SweetPotato)
## Related
- [Windows Privilege Escalation](../README.md) — parent MOC for all Windows privesc techniques
- Privilege Escalation — top-level MOC
- [Token Impersonation](Token-Impersonation.md) — foundational sibling: how Windows access tokens work before potato exploits can leverage them
- [Juicy Potato](Juicy-Potato.md) — sibling tool: classic potato for SeImpersonatePrivilege abuse on older Windows builds
- [God Potato](God-Potato.md) — sibling tool: modern universal potato that works across Windows 7–Server 2022
- [PrintSpoofer](PrintSpoofer.md) — spooler-based SeImpersonate→SYSTEM on Win10/Server 2016–2019
- [RoguePotato](RoguePotato.md) — DCOM/OXID technique for Windows 10 1809+ / Server 2019
- [RottenPotato](RottenPotato.md) — the original NTLM-relay potato (historical; ancestor of the family)
- [Services Exploitation](../Services-Exploitation/Services-Exploitation.md) — service account misconfigurations often grant the SeImpersonatePrivilege needed for potato attacks
