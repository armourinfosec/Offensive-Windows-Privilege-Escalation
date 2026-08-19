# FileZilla Server Password Recovery

FileZilla Server stores user passwords in a configuration file (`FileZilla Server.xml`) located on the system. The passwords are stored in a hashed format, which can be extracted and cracked using tools like `hashcat`.

## FileZilla Server Download Links

> You can download FileZilla Server from the official website:

- General download link:  [Filezilla Server](https://download.filezilla-project.org/server/)

- Version 0.9.27:  - [filezilla-server-0.9.27.exe](https://filezilla-server.en.uptodown.com/windows/download/5501)


## Location of FileZilla Server Configuration File

- The configuration file that stores user credentials is located here:

```text
C:\Program Files (x86)\FileZilla Server\FileZilla Server.xml
````

> This file contains hashed passwords and other server configuration details.  
> The passwords are typically hashed using SHA-512.


## Extracting and Cracking the Password Hash

### Extract the hash from `FileZilla Server.xml`

1. Open the configuration file using `vim` or `notepad`:

```bash
vim FileZilla Server.xml
```

2. Copy the extracted hash into a separate file (`FileZilla-hash.txt`):

```bash
vim FileZilla-hash.txt
```

> Example content of `FileZilla-hash.txt`:

```text
2CFB9BEE9259E3827C1515BF133F0B3970121CF05D99989CA56D8D049D92C9EDF542919506893681DCC3DD4E66FA38853F17025FD96ABD0D5F8234251143F69B:e{Nr^I:@W"T4*BbW]fVx#,uc>Y]EVL(UWl}+x2JP,cXANjwf-nls4CL`$D>yvX|V
53054CE77B7FCDD70578B7D932CB89977FA73923EEBA87E5DE9628D036CE8735D53D055AEB3B85772AAB0C47DAF1604A81E27AC0E8BB8ABFE5E166E07B498F69:IYDNe`(.>;.HEwj}DTYveBBmx{Ds!3yXN244^Y\S!,C)8Ohz7nJub](Q<=iB:V@_
5C8CE1DB7BBA7DE8930CFEA278F5A571D5CCD3CE5F3D90CA501CBEC910775BA7A3154B6825B9A31DDBB4DB601B3AA6F641CFF37C66D56D3E2CD4ECC5617CBE3B:Jfyn,n`ID}$KbI]I8D\AHyo/y4Rv4uSn\,.8,0?z9$kRLrOu~{+o;rC27tc0hR\-
5C01C1715D3EB64C7493EDEAD6089DFF26501A4F3B5DCDFBEBB77A7201DF27A9F3C8A56CC823F17847F909B4B77A9D9911CC65EEADB8B27D35EAC5E94C056772:r-w}m<{BR!2>Uw1sfu<CR7C?h,Jk]B[X-;~vE*W|uR\7!nK%1/""$#HlEC[#?vA~
```


### Crack the hash using `hashcat`

> Use `hashcat` to crack the password hash:

- -m 1710 – Mode for `sha512($pass.$salt)`

- -a 0 – Attack mode for dictionary-based attacks


```bash
hashcat -m 1710 -a 0 FileZilla-hash.txt /opt/password.txt
```

> `1710` – Hash type for SHA-512 with salt  
> `/opt/password.txt` – Wordlist for cracking


### Example for Cracking Hashes from FileZilla:

- For unsalted md5 hashes:

```bash
hashcat -m 0 -a 0 filezila.txt /opt/rockyou.txt
```

> `-m 0` – Hash type for raw SHA-512  
> `/opt/rockyou.txt` – Common password list


## Understanding Hashcat Output

- If successful, `hashcat` will output the cracked hash in the format:

```text
<hash>:<password>
```

> Example:

```text
2CFB9BEE9259E3827C1515BF133F0B3970121CF05D99989CA56D8D049D92C9EDF542919506893681DCC3DD4E66FA38853F17025FD96ABD0D5F8234251143F69B:password123
```


## Use Cases

- Cracking FileZilla server admin passwords.

- Extracting stored credentials for lateral movement.

- Recovering lost or forgotten FileZilla passwords.

## Security Recommendations

- Ensure that the `FileZilla Server.xml` file is not accessible by non-administrators.  

- Use strong, complex passwords for FileZilla Server accounts.  

- Regularly rotate passwords and monitor for suspicious activity.

## Related
- [Search for file contents](Search-for-file-contents.md) — parent technique
- Password Mining — Windows credential-harvesting hub
- [Windows Privilege Escalation](../../README.md) — escalation context
- Password Cracking — reuse harvested credentials
