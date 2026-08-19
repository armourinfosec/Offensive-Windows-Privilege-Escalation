# Share Kit — Offensive Windows Privilege Escalation

**Repo:** https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation
**License:** CC BY 4.0 · **By:** Armour Infosec

Copy-paste blocks per platform. Facts used: 60+ hands-on notes on escalating from a low-privileged Windows
foothold to Administrator or `NT AUTHORITY\SYSTEM`, organized by vector — automated enumeration
(WinPEAS/PowerUp/SharpUp/Seatbelt/Watson), service misconfigurations (unquoted service paths, weak `binPath`
and file permissions, DLL hijacking, service-via-registry, named pipes), scheduled tasks and startup apps,
registry exploitation (`AlwaysInstallElevated`, autoruns), UAC bypass (fodhelper, eventvwr, computerdefaults,
sdclt), token-privilege abuse (SeImpersonate, SeBackup/SeRestore, SeTakeOwnership, SeLoadDriver, SeDebug),
the impersonation/potato family (Juicy, Rogue, Print, God, Rotten), Windows kernel exploits, and credential
mining (SAM/SYSTEM, LSASS, NTDS.dit, GPP `cpassword`, registry, ADS, unattend files). Includes a methodology
checklist that indexes every vector and two full hands-on labs (SeImpersonate→SYSTEM, unquoted service
path→SYSTEM). Each note pairs exploitation with detection and defensive guidance. Supports OSCP, PNPT, CEH,
CRTP, and eJPT preparation. Every technique is documented for authorized testing only — your own lab, a CTF,
or an engagement you have permission to run.

---

## LinkedIn

🪟🔒 Offensive Windows Privilege Escalation — open, hands-on study notes

We've published a free, practical knowledge base for **escalating from a low-privileged Windows foothold to SYSTEM** — the part of every engagement that starts the moment you land a shell.

60+ notes covering:
• Enumeration & methodology — WinPEAS, PowerUp, and a top-down escalation checklist
• Service & scheduled-task misconfigs — unquoted paths, weak `binPath`/file permissions, DLL hijacking
• Registry abuse & UAC bypass — AlwaysInstallElevated, autoruns, fodhelper/eventvwr/computerdefaults
• Token-privilege abuse — SeImpersonate, SeBackup/SeRestore, SeTakeOwnership, SeLoadDriver, SeDebug
• The potato family — Juicy, Rogue, Print, God, Rotten
• Kernel exploits and credential mining — SAM/SYSTEM, LSASS, NTDS.dit, GPP cpassword
• A methodology checklist + two full hands-on labs

Every technique pairs exploitation with detection and defensive guidance, and targets a host you own or are authorized to test. Maps to OSCP, PNPT, CEH, and CRTP. Licensed CC BY 4.0 — free to use and share with attribution.

🔗 https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

⭐ Star it, share it, and open an issue if you spot something to improve.

By Armour Infosec — offensive security, done responsibly.

#WindowsSecurity #PrivilegeEscalation #PenetrationTesting #RedTeam #OSCP #InfoSec #CyberSecurity #PostExploitation #EthicalHacking

---

## X / Twitter

🪟🔒 Free & open: Offensive Windows Privilege Escalation.

60+ hands-on notes — enumeration, service & scheduled-task misconfigs, UAC bypass, token privileges (SeImpersonate/SeBackup/SeDebug…), the potato family, kernel exploits, credential mining. Methodology + 2 labs. CC BY 4.0.

👉 https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

⭐ if it's useful.
#WindowsSecurity #PrivEsc #OSCP #RedTeam

### X thread (optional)
1/ We open-sourced our Offensive Windows Privilege Escalation notes — 60+ notes, every vector, free under CC BY 4.0. 🧵
2/ It starts with a methodology: land a shell, run `whoami /priv` + WinPEAS/PowerUp, and match findings to vectors.
3/ Token privileges are the fast win: SeImpersonate → potato to SYSTEM; SeBackup steals SAM/SYSTEM; SeDebug dumps LSASS.
4/ Service & scheduled-task misconfigs: unquoted paths, weak `binPath`/file perms, DLL hijacking, writable task binaries.
5/ Already admin but unelevated? UAC bypass via fodhelper/eventvwr/computerdefaults — fileless, no prompt.
6/ The potato family, explained end to end: Rotten → Juicy → Rogue → PrintSpoofer → God, and which works on which build.
7/ Then credential mining — LSASS, SAM/SYSTEM, NTDS.dit, GPP cpassword — plus two full hands-on labs.
8/ Every note has detection + defenses too. ⭐ Star + share: https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

---

## Facebook

🪟🔒 New & free: Offensive Windows Privilege Escalation

A hands-on study guide for escalating from a low-privileged Windows foothold to SYSTEM — 60+ notes spanning enumeration, service and scheduled-task misconfigurations, UAC bypass, token-privilege abuse, the potato family, kernel exploits, and credential mining, each with detection and defensive guidance.

Open to everyone under CC BY 4.0. Clone it, learn from it, contribute to it.

👉 https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

— Armour Infosec

---

## Instagram (caption)

🪟🔒 Free & open-source: Offensive Windows Privilege Escalation

60+ hands-on notes on getting from a low-priv Windows shell to SYSTEM — enumeration, service misconfigs, UAC bypass, token privileges, the potato family, kernel exploits, credential mining. Methodology + 2 labs.

Link in bio 👉 grab it on GitHub (armourinfosec). CC BY 4.0.

.
.
#windows #privesc #pentesting #redteam #oscp #cybersecurity #infosec #kalilinux #postexploitation #armourinfosec

---

## YouTube (community post / video description)

🪟🔒 Offensive Windows Privilege Escalation — free, open study notes

We've published our full 60+ note reference on escalating from a low-privileged Windows foothold to Administrator or SYSTEM: enumeration and methodology, service and scheduled-task misconfigurations, registry abuse and UAC bypass, token-privilege abuse (SeImpersonate, SeBackup/SeRestore, SeTakeOwnership, SeLoadDriver, SeDebug), the impersonation/potato family (Juicy, Rogue, Print, God, Rotten), kernel exploits, and credential mining (SAM/SYSTEM, LSASS, NTDS.dit, GPP cpassword) — plus a methodology checklist and two full hands-on labs.

Every technique pairs exploitation with detection and defenses. Licensed CC BY 4.0, free to use and share. For authorized testing only.

🔗 GitHub: https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation
🌐 Website: https://www.armourinfosec.com

---

## Discord / Telegram / Slack

📢 **Offensive Windows Privilege Escalation** — free & open

60+ hands-on notes covering every Windows privesc vector: enumeration (WinPEAS/PowerUp), service & scheduled-task misconfigs, UAC bypass (fodhelper/eventvwr/computerdefaults), token privileges (SeImpersonate/SeBackup/SeDebug…), the potato family (Juicy/Rogue/Print/God/Rotten), kernel exploits, and credential mining (SAM/LSASS/NTDS.dit/GPP). Methodology checklist + 2 labs, each with detection & defenses. CC BY 4.0.

🔗 https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

Contributions & feedback welcome — open an issue or PR. ⭐ the repo if it helps!

---

## Reddit (e.g. r/oscp, r/netsec, r/HowToHack, r/AskNetsec, r/cybersecurity)

**Title:** Free, open study notes: Offensive Windows Privilege Escalation (60+ notes, every vector, CC BY 4.0)

**Body:**
Sharing an open knowledge base we put together for Windows privilege escalation — the part of every engagement that starts the moment you land a low-priv shell.

It's 60+ hands-on notes, organized by vector: enumeration and a top-down methodology checklist (WinPEAS/PowerUp/Seatbelt/Watson), service and scheduled-task misconfigurations (unquoted paths, weak `binPath`/file permissions, DLL hijacking, service-via-registry, named pipes), registry exploitation (`AlwaysInstallElevated`, autoruns), UAC bypass (fodhelper/eventvwr/computerdefaults/sdclt), token-privilege abuse (SeImpersonate, SeBackup/SeRestore, SeTakeOwnership, SeLoadDriver, SeDebug), the impersonation/potato family (Juicy/Rogue/Print/God/Rotten), Windows kernel exploits, and credential mining (SAM/SYSTEM, LSASS, NTDS.dit, GPP `cpassword`, registry, ADS, unattend files).

Every note pairs exploitation with detection and defensive guidance, there's a methodology checklist that indexes everything, and two full hands-on labs (SeImpersonate→SYSTEM, unquoted service path→SYSTEM). Useful for OSCP/PNPT/CEH/CRTP prep. Licensed CC BY 4.0, free to use, adapt, and share. Documented for authorized testing only.

Repo: https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

Feedback, issues, and PRs welcome.

---

## WhatsApp / SMS

🪟 Free open-source repo: *Offensive Windows Privilege Escalation* — 60+ hands-on notes on going from a low-priv Windows shell to SYSTEM (enumeration, service misconfigs, UAC bypass, token privileges, potato family, kernel exploits, credential mining). Methodology + 2 labs. CC BY 4.0.
👉 https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

---

## Email / Newsletter

**Subject:** Free & open: Offensive Windows Privilege Escalation (60+ hands-on notes)

Hi there,

We've just published a free, open-source knowledge base: **Offensive Windows Privilege Escalation**.

It's a hands-on reference for escalating from a low-privileged Windows foothold to Administrator or SYSTEM — the problem that surfaces the moment you land a shell. It spans enumeration and methodology, service and scheduled-task misconfigurations, registry abuse and UAC bypass, token-privilege abuse (SeImpersonate, SeBackup/SeRestore, SeTakeOwnership, SeLoadDriver, SeDebug), the impersonation/potato family (Juicy, Rogue, Print, God, Rotten), kernel exploits, and credential mining (SAM/SYSTEM, LSASS, NTDS.dit, GPP cpassword) — plus a methodology checklist and two full hands-on labs.

Every technique pairs exploitation with detection and defenses, maps well to OSCP/PNPT/CEH/CRTP preparation, and the whole thing is licensed CC BY 4.0 — free to use, adapt, and share with attribution. It is documented for authorized testing only.

Explore it here: https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

If it's useful, a ⭐ on GitHub helps others find it. Feedback, issues, and pull requests are always welcome.

— The Armour Infosec Team
https://www.armourinfosec.com

---

## One-liner (chat / DM)

Sharing a free, open study repo on Windows privilege escalation — 60+ hands-on notes from low-priv shell to SYSTEM (enumeration, service misconfigs, UAC bypass, token privileges, potato family, kernel exploits, credential mining), CC BY 4.0: https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation

---

## Official links (for reference)

- Repo: https://github.com/armourinfosec/Offensive-Windows-Privilege-Escalation
- Website: https://www.armourinfosec.com
- LinkedIn: https://www.linkedin.com/company/armourinfosec
- X: https://x.com/ArmourInfosec
- YouTube: https://www.youtube.com/c/TheHackersWorld
- Facebook: https://www.facebook.com/armourinfosec
- Email: info@armourinfosec.com
