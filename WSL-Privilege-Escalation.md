# WSL Privilege Escalation

**The Windows Subsystem for Linux (WSL) is both a privilege-escalation target and a tool: a root shell inside a WSL distro can read and write the Windows host filesystem via `/mnt/c`, and WSL binaries (`wsl.exe`, `bash.exe`) that run as a privileged user can be abused to execute commands in that user's context.** WSL blurs the trust boundary — files created from Linux land on NTFS, and a user who is root in their WSL distro (the default) can touch host files subject to their Windows token. Where a privileged user or scheduled task invokes WSL, or where WSL runs elevated, that becomes a Windows escalation path.

## Enumerate WSL

```powershell
wsl --list --verbose             # installed distros and versions
wsl --status
Get-ChildItem "$env:LOCALAPPDATA\Packages" -Filter '*Linux*' -Directory 2>$null
```

Inside a distro:

```bash
id                                # root by default in many distros
ls -la /mnt/c/Users/              # the Windows host filesystem, subject to your Windows token
cat /etc/passwd                   # WSL's own users
```

## Escalation angles

- **Host file access from WSL root:** as root in WSL you can read/modify host files your Windows user can access — harvest credentials, drop a payload into a Startup folder, or overwrite a script a privileged Windows task runs (`/mnt/c/...`). Elevation depends on the *Windows* token WSL runs under.
- **Credential harvesting:** WSL stores Linux creds/keys (`~/.ssh`, history) on NTFS under the distro package dir — readable from Windows too ([Password Mining](Password-Mining/Password-Mining.md)).
- **`sudo`/`wsl.exe` as a privileged invoker:** if a Windows admin or task runs `wsl.exe <cmd>`, and you control the distro's default user or a script it calls, your Linux payload runs with that invoker's Windows privileges.
- **Interop abuse:** WSL interop lets Linux launch Windows binaries (`cmd.exe`), and vice-versa — useful for pivoting between the two worlds once you hold one.

## Detection and defenses

- **Detection:** `wsl.exe`/`bash.exe` spawning Windows processes, WSL access to sensitive `/mnt/c` paths, distro installs by non-admins.
- **Defenses:** restrict who may install/enable WSL; do not run WSL or `wsl.exe` from privileged accounts/tasks; treat the WSL distro filesystem as untrusted; apply least privilege to the Windows accounts WSL runs under.

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Escalation Path via Windows Subsystem for Linux(WSL)](Escalation-Path-via-Windows-Subsystem-for-Linux(WSL).md) — the existing WSL escalation-path note
- [Password Mining](Password-Mining/Password-Mining.md) — harvesting credentials stored by WSL
- [Miscellaneous Techniques](Miscellaneous-Techniques.md) — WSL in the wider "also try" set
