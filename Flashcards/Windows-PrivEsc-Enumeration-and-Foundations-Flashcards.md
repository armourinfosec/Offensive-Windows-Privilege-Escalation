# Flashcards — Windows PrivEsc Enumeration and Foundations

Spaced-repetition deck covering the enumeration and foundation half of the module. Uses the Obsidian spaced-repetition `Question::Answer` format.

## First steps

What is the single highest-value command after landing a Windows shell?::`whoami /priv` — an enabled `SeImpersonate`, `SeBackup`, or `SeDebug` is often an instant escalation.
What does `whoami /groups` tell you that matters for privesc?::Membership in privileged built-in groups (Backup/Server/Print Operators, DnsAdmins, Hyper-V Admins, Event Log Readers) and your integrity level.
When is a UAC bypass the right move rather than a full exploit?::When you are in the Administrators group but running at **Medium** integrity (unelevated) — you just need to reach High integrity.
Why run an automated scanner first, then verify by hand?::WinPEAS/PowerUp triage the host fast, but false positives happen — confirm the strongest finding manually before committing.

## Privileges and groups

What can `SeImpersonatePrivilege` be turned into?::SYSTEM, via a potato attack (PrintSpoofer/RoguePotato/God Potato) that impersonates a coerced SYSTEM token.
What does `SeBackupPrivilege` let you do?::Read any file regardless of ACL — e.g. `reg save HKLM\SAM` / `SYSTEM` (or NTDS.dit on a DC), then dump hashes offline.
Why is `SeDebugPrivilege` powerful?::It opens any process — inject into or dump a SYSTEM process (e.g. LSASS) or duplicate its token.
Why is the `Backup Operators` group root-equivalent?::Members hold SeBackup/SeRestore, so they can copy the SAM/SYSTEM hives and recover the local admin hash.
What does `DnsAdmins` membership allow on a DC?::Loading an arbitrary DLL into the SYSTEM `dns.exe` service via the server-level plugin DLL setting → SYSTEM/Domain Admin.

## Services and registry

How do you find an unquoted service path?::`Get-CimInstance Win32_Service` where `PathName` has spaces, is not quoted, and is outside `C:\Windows`.
What makes a service's binPath exploitable?::A weak service DACL granting `SERVICE_CHANGE_CONFIG` (WP) to Users/Authenticated Users, so you can `sc config <svc> binpath=`.
What are the two registry values required for AlwaysInstallElevated?::`HKLM` and `HKCU` `...\Installer\AlwaysInstallElevated = 1` — both must be set; then any `.msi` installs as SYSTEM.

## Tools and enumeration

Name the go-to automated Windows privesc enumerators.::WinPEAS, PowerUp/SharpUp, Seatbelt, and Watson (missing-patch suggester).
What does the in-repo PowerShell enumeration script give you over WinPEAS?::A dependency-free option that runs where you cannot drop tooling, mapping each finding to its technique note.
How do you find the patch level to pick a kernel exploit?::`systeminfo` (+ `wmic qfe`) fed to Watson / Windows-Exploit-Suggester.
