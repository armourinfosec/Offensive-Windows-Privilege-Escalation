# Legacy Operating Systems

End-of-life Windows — XP, Server 2003, Vista, 7, Server 2008/R2 — still turns up on internal networks, in OT/ICS environments, and on forgotten appliances, and it escalates very differently from a patched Windows 11/Server 2022 host. Legacy systems miss years of security hardening: kernel exploits land reliably, UAC is weak or absent, SMBv1 is on, and service/registry misconfigurations are rampant. When you fingerprint an old build ([Windows Version and Configuration](Windows-Version-and-Configuration.md)), the playbook shifts toward the techniques Microsoft has since mitigated.

## What changes on legacy targets

- **Kernel exploits are first-class.** Unpatched legacy hosts fall to public MS-series exploits that are dead on modern builds — see [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) ([MS10 015](Windows-Kernel-Exploits/MS10-015.md), [MS10 059](Windows-Kernel-Exploits/MS10-059.md), [MS14 058](Windows-Kernel-Exploits/MS14-058.md)). Feed `systeminfo` to Windows Exploit Suggester.
- **UAC is weak or missing.** XP/2003 have no UAC at all; Vista/7 UAC is bypassable with older, simpler techniques. An admin token is often already high-integrity.
- **The potato matrix opens up.** Hot Potato and Rotten Potato work on pre-2016 builds where they are patched on modern Windows — see the table in [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md).
- **Legacy credential stores.** Cleartext GPP `cpassword`, `sysprep`/`unattend` files, and SAM with LM hashes are common ([Password Mining](Password-Mining/Password-Mining.md), [Group Policy Preferences cpassword](Password-Mining/Group-Policy-Preferences-cpassword.md)).
- **SMBv1 / weak services.** MS17-010 (EternalBlue) and unquoted/weak service configs abound.

## Enumerate the build first

```cmd
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
wmic os get Caption,Version,BuildNumber,OSArchitecture
wmic qfe get HotFixID,InstalledOn
```

An OS with almost no hotfixes and an old build number is a signal to reach straight for a kernel exploit or MS17-010 rather than hunting misconfigurations.

## Constraints

- Modern tooling may not run: `Get-CimInstance`/newer PowerShell cmdlets can be absent on XP/2003 (PowerShell 2.0 or none). Fall back to `wmic`, `sc`, `reg`, and native binaries.
- Some modern potatoes require Windows 10 1809+; use the *older* family (Hot/Rotten) on legacy.

## Detection and defenses

- **Detection:** legacy hosts are hard to instrument — network monitoring (SMBv1, exploit traffic) is often the only signal.
- **Defenses:** the only real fix is to **decommission or isolate** EOL systems; where impossible, network-segment them, disable SMBv1, and restrict inbound access.

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — the primary legacy escalation route
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) — the potato version matrix
- [Windows Version and Configuration](Windows-Version-and-Configuration.md) — fingerprint the build and patch level
- [Windows Server vs Desktop](Windows-Server-vs-Desktop.md) — how role and edition shift the approach
