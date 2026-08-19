# Password Mining with LaZagne

[LaZagne](https://github.com/AlessandroZ/LaZagne)

## Tool Overview

- LaZagne is an open-source tool designed for recovering stored passwords from various applications on Windows, Linux, and macOS.
- It targets browsers, databases, mail clients, network configurations, and more.


## Installation

-  Clone the Repository

```bash
git clone https://github.com/AlessandroZ/LaZagne.git
```

```bash
cd LaZagne
```

-  Install Dependencies

> For Python 3:

```bash
pip install -r requirements.txt
```


## Usage

- General Help

```bash
python3 laZagne.py -h
```

- Recover All Passwords

```bash
python3 laZagne.py all
```


- Browser Passwords

|Target|Command|
|---|---|
|Brave|`python3 laZagne.py browsers`|
|Chromium|`python3 laZagne.py browsers`|
|Dissenter-Browser|`python3 laZagne.py browsers`|
|Google Chrome|`python3 laZagne.py browsers`|
|IceCat|`python3 laZagne.py browsers`|
|Firefox|`python3 laZagne.py browsers`|
|Opera|`python3 laZagne.py browsers`|
|SlimJet|`python3 laZagne.py browsers`|
|Vivaldi|`python3 laZagne.py browsers`|
|WaterFox|`python3 laZagne.py browsers`|

```bash
python3 laZagne.py browsers
```

- Messaging Clients

|Target|Command|
|---|---|
|Pidgin|`python3 laZagne.py chats`|
|Psi|`python3 laZagne.py chats`|


```bash
python3 laZagne.py chats
```

- Database Credentials

|Target|Command|
|---|---|
|DBVisualizer|`python3 laZagne.py databases`|
|Squirrel|`python3 laZagne.py databases`|
|SQLdeveloper|`python3 laZagne.py databases`|


```bash
python3 laZagne.py databases
```

- Email Clients

|Target|Command|
|---|---|
|Clawsmail|`python3 laZagne.py mail`|
|Thunderbird|`python3 laZagne.py mail`|

```bash
python3 laZagne.py mail
```

- System Passwords

|Target|Command|
|---|---|
|System Password|`python3 laZagne.py system`|
|Apache Directory Studio|`python3 laZagne.py system`|
|AWS|`python3 laZagne.py system`|
|Docker|`python3 laZagne.py system`|
|Environment Variables|`python3 laZagne.py system`|

```bash
python3 laZagne.py system
```

- FTP Clients

|Target|Command|
|---|---|
|FileZilla|`python3 laZagne.py ftp`|
|gFTP|`python3 laZagne.py ftp`|


```bash
python3 laZagne.py ftp
```

- Miscellaneous

|Target|Command|
|---|---|
|History Files|`python3 laZagne.py keychains`|
|Shares|`python3 laZagne.py keychains`|
|SSH Private Keys|`python3 laZagne.py keychains`|
|KeePass Configuration Files|`python3 laZagne.py keychains`|
|Grub|`python3 laZagne.py keychains`|
|Network Manager|`python3 laZagne.py wifi`|
|WPA Supplicant|`python3 laZagne.py wifi`|
|GNOME Keyring|`python3 laZagne.py keychains`|
|Kwallet|`python3 laZagne.py keychains`|
|Hashdump|`python3 laZagne.py hashdump`|


- Recover Chrome passwords:

```bash
python3 laZagne.py browsers -v
```

- Recover SSH private keys:

```bash
python3 laZagne.py keychains
```

- Dump all passwords:

```bash
python3 laZagne.py all
```

## Related
- [Windows Privilege Escalation](../README.md) — parent hub
- [SAM and SYSTEM files](SAM-and-SYSTEM-files.md) — dumping local hives
- [LSASS Credential Dumping](LSASS-Credential-Dumping.md) — dump live credentials from LSASS memory
- [Group Policy Preferences cpassword](Group-Policy-Preferences-cpassword.md) — decrypt GPP passwords from SYSVOL
- [Search for file contents](Search-for-file-contents/Search-for-file-contents.md) — searching the filesystem for secrets
- [Password in Windows Registry](Password-in-Windows-Registry.md) — registry-stored credentials
- Password Cracking — crack harvested hashes
