# Token Impersonation

## Access Tokens

Access tokens are the foundation of all authorization decisions for securable resources hosted on the Windows operating system. They are granted to authorized users by the **Local Security Authority (LSA)**. An access token contains:

- The user's **Security Identifier (SID)**

- Group SIDs

- Assigned privileges

- Integrity level

- And other security-related metadata

 [Microsoft Docs – Access Tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)

Tokens are analogous to **web cookies**. They serve as temporary credentials that allow a user or service to access system resources without re-entering credentials for each access request.

## Types of Tokens

There are two primary types of access tokens:

> **Delegate Tokens**
>
> - Created for **interactive logons**
>
> - Examples: local login or Remote Desktop sessions
>
> **Impersonate Tokens**
>
> - Used in **non-interactive sessions**
>
> - Examples: network drive access, domain logon scripts
>

[Offensive Security – Incognito in Metasploit](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/)

## Checking for SeImpersonatePrivilege

To exploit impersonation-based privilege escalation techniques like **Juicy Potato** or **Rogue Potato**, the user must possess the `SeImpersonatePrivilege`.

### CMD:

```cmd
whoami /priv
```

Look for:

```text
SeImpersonatePrivilege			Enabled
```

### Meterpreter:

```text
meterpreter > getprivs
```

Result will include:

```text
SeImpersonatePrivilege
```


## Tools and References

- [Priv2Admin by gtworek (Token Priv Esc Tool)](https://github.com/gtworek/Priv2Admin)

- [Metasploit Incognito Module (Legacy Reference)](https://www.offensive-security.com/metasploit-unleashed/fun-incognito/)


Understanding how tokens work and how they can be impersonated is critical for both attackers and defenders. Attackers use impersonation to elevate privileges, while defenders must monitor and restrict token usage to reduce the attack surface.

## Related
- [Impersonation and Potato Attacks](Impersonation-and-Potato-Attacks.md) — parent hub
- [Juicy Potato](Juicy-Potato.md) — potato exploiting impersonation tokens
- [God Potato](God-Potato.md) — potato exploiting impersonation tokens
- [Windows Privilege Escalation](../README.md) — escalation context
- Offensive Active Directory — token theft for lateral movement
