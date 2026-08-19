# Privilege Escalation Tools

List of some popular tools for **privilege escalation** in Windows environments:

---

## Tools

1. **[WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)**
   - A powerful Windows privilege escalation script that searches for misconfigurations, weak permissions, and common pitfalls.

2. **[Windows-PrivEsc-Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)**
   - A comprehensive checklist for Windows privilege escalation, covering various techniques and tools.

3. **[Sherlock](https://github.com/rasta-mouse/Sherlock)**
   - A tool that searches for common misconfigurations and weak permissions in a Windows environment, aiding privilege escalation efforts.

4. **[Watson](https://github.com/rasta-mouse/Watson)**
   - A Windows post-exploitation tool that identifies potential privilege escalation vectors.

5. **[PowerUp](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)**
   - A PowerShell script designed to assist with privilege escalation on Windows.

6. **[JAWS](https://github.com/411Hall/JAWS)**
   - A tool for auditing Windows Active Directory environments, focused on finding privilege escalation opportunities.

7. **[Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)**
   - A tool that helps identify missing security patches and potential vulnerabilities for privilege escalation.

8. **[Metasploit-Local-Exploit-Suggester](https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/)**
   - A Metasploit module that suggests local exploits based on system architecture and patch levels.

9. **[Seatbelt](https://github.com/GhostPack/Seatbelt)**
   - A C# tool for performing offensive security assessments, focusing on Windows environments.

10. **[SharpUp](https://github.com/GhostPack/SharpUp)**
    - A C# tool used for privilege escalation by identifying misconfigurations and vulnerabilities in Windows systems.

11. **[Rotten-Potato](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/)**
    - A privilege escalation exploit that targets Windows services running with low-privileged accounts.

12. **[Juicy-Potato-Github](https://github.com/ohpe/juicy-potato)**
    - A tool that exploits Windows Service Accounts to escalate privileges, leveraging the "Juicy Potato" technique.

13. **[Groovy-Reverse-Shell](https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76)**
    - A Groovy-based reverse shell for use in penetration testing.

14. **[Alternate-Data-Streams](https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/)**
    - An article explaining Alternate Data Streams (ADS) and how they can be used for privilege escalation or persistence.

---

## Useful Links

Here are some helpful links for Windows privilege escalation techniques and resources:

1. [FuzzySecurity Tutorial - Windows Privilege Escalation](https://www.fuzzysecurity.com/tutorials/16.html)
2. [PayloadsAllTheThings - Windows Privilege Escalation Methods](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
3. [Absolomb's Windows Privilege Escalation Guide](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)
4. [Sushant747's OSCP Guide - Privilege Escalation (Windows)](https://sushant747.gitbooks.io/total-oscp-guide/privilege_escalation_windows.html)
5. [GUIF - Windows Privilege Escalation Resources](https://guif.re/)

---

These tools and resources are essential for conducting **privilege escalation** on Windows environments, particularly in penetration testing or post-exploitation scenarios.

## Related
- [Windows Version and Configuration](Windows-Version-and-Configuration.md) — feed systeminfo to exploit suggesters
- [Services Exploitation](Services-Exploitation/Services-Exploitation.md) — misconfigurations these tools surface
- [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — patch-level exploit suggestions
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks/Impersonation-and-Potato-Attacks.md) — token privilege checks
- [User Enumeration](User-Enumeration.md) — manual enumeration complement
