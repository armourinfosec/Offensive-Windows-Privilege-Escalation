# Windows File Search for Credentials and Sensitive Data

These commands help search for sensitive information such as passwords and credentials within files on a Windows system. The goal is to locate hardcoded passwords, API keys, and other sensitive data that might be stored in configuration files or plaintext files.

## Search for Passwords in Files

- Use `findstr` to search for the keyword `password` in common configuration files:

- Search within `.xml`, `.ini`, and `.txt` files:

```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
````

```bash
cd "C:\Program Files (x86)" & findstr /SI /M "pass" *.xml *.ini *.txt
````

> /SI – Searches for the string in subdirectories.

> /M – Displays only the file names that match the search pattern.

- Search for `password` in additional file types (`*.config`):

```bash
findstr /si password *.xml *.ini *.txt *.config
```

> Searches all matching files recursively for the word `password`.


- Search for `password` in all files (case-insensitive):

```bash
findstr /spin "password" *.*
```

> /S – Searches in subdirectories.  

> /P – Skips files with non-printable characters.  

> /I – Case-insensitive search.  

> /N – Displays line numbers in the output.


## Search for Specific File Names

- Use `dir` to find files with specific names or patterns related to credentials:

- Search for files containing `pass`, `cred`, `vnc`, or `.config`:

```bash
dir /S /B *pass*.txt *pass*.xml *pass*.ini *cred* *vnc* *.config*
```

> /S – Search all subdirectories.  

> /B – Output file path without additional details.  

> Helps locate files containing sensitive information.


## Locate Specific Files Using `where`

- Use `where` to locate files on the system:

- Find `user.txt` recursively:

```bash
where /R C:\ user.txt
```

> /R – Recursive search through subdirectories.

- Find `.ini` files recursively:

```bash
where /R C:\ *.ini
```

> Useful for identifying configuration files that might store sensitive data.


## Examples of Useful Targets

- `.ini` – Configuration files that may store plaintext passwords.

- `.xml` – Configuration files for applications and services.

- `.txt` – May contain saved credentials or key information.

- `.config` – Application settings, including API keys and credentials.

- `cred` – Often used in naming for credential files.

- `vnc` – May contain VNC session or authentication data.


## Pro Tips

- Use `findstr` with a case-insensitive (`/I`) flag to capture variations like `Password`, `PASSWORD`, or `PaSsWoRd`.  

- Redirect output to a file for easier processing:

```bash
findstr /si password *.* > results.txt
```

- Combine `dir` and `findstr` to locate and search for content in one command:

```bash
dir /S /B *pass* | findstr /si password
```

- Use PowerShell for more advanced search and pattern matching:

```powershell
Get-ChildItem -Recurse -Include *.xml,*.ini,*.txt | Select-String -Pattern "password"
```


## Use Cases

- Find hardcoded credentials for privilege escalation.  

- Identify misconfigured systems storing passwords in plaintext.  

- Expose potential security weaknesses in config files.  

- Automate post-exploitation enumeration in penetration tests.

## Related
- Password Mining — parent hub
- [Filezilla Server Password](Filezilla-Server-Password.md) — example app credential found by searching
- [Web Configuration Files and Sensitive Data Discovery](../Web-Configuration-Files-and-Sensitive-Data-Discovery.md) — config files with secrets
- [Windows Privilege Escalation](../../README.md) — escalation context
