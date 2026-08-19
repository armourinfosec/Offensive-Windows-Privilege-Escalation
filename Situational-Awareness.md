# Situational Awareness

Before hunting for an escalation vector, understand the machine you have landed on: its network position, what protections are active, and what might detect you. Situational awareness turns a blind foothold into a targeted one — it tells you which vectors are even viable (is there a domain? a second NIC to pivot through? is Defender watching?) and shapes how loud you can afford to be. It complements the account-level checks in [Network Enumeration](Network-Enumeration.md) and [User Enumeration](User-Enumeration.md).

## Network position

```cmd
ipconfig /all
route print
arp -a
netstat -ano
```

Look for a **second interface** (a pivot into another subnet), internal DNS/domain servers, and established connections that reveal internal hosts.

## Domain vs local

```cmd
systeminfo | findstr /i "Domain"
net config workstation
nltest /dclist:%USERDNSDOMAIN% 2>nul
```

A domain membership changes everything — group-based escalations ([Windows Built in Groups](Windows-Built-in-Groups/Windows-Built-in-Groups.md)) and AD attack paths open up.

## Defenses and detection surface

Know what is watching before you act:

```powershell
Get-MpComputerStatus | Select RealTimeProtectionEnabled, AntivirusEnabled   # Defender
Get-MpPreference | Select DisableRealtimeMonitoring, ExclusionPath           # AV exclusions = safe drop zones
```

```cmd
netsh advfirewall show allprofiles state          :: firewall
sc query windefend & sc query Sysmon              :: Defender / Sysmon present?
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell" 2>nul   :: PS logging/constrained mode
```

AV **exclusion paths** are ideal locations to drop tooling; the presence of Sysmon or PowerShell logging tells you how carefully to tread (Detecting File Transfers logic applies to privesc too).

## Installed software & patch level

```cmd
wmic product get name,version
systeminfo
```

Third-party software is a common source of [vulnerable services](Services-Exploitation/Services-Exploitation.md) and [DLL hijacks](Services-Exploitation/Dynamic-Link-Library-Hijacking(DLL-Hijacking).md); the patch level drives [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md).

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Network Enumeration](Network-Enumeration.md) · [User Enumeration](User-Enumeration.md) — account and network detail
- [Windows Built in Groups](Windows-Built-in-Groups/Windows-Built-in-Groups.md) — domain groups worth checking once you know it's a domain
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — automate this triage with WinPEAS/Seatbelt
