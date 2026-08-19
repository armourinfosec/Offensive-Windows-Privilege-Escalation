# Citrix Breakout

Published-application and kiosk environments — Citrix, RDS RemoteApp, and locked-down VDI — expose a single application rather than a full desktop, on the assumption the user cannot reach the underlying OS. A **breakout** defeats that assumption: you escape the published app to a shell on the host, from where normal Windows privilege escalation applies. Breakouts are about creative navigation, not exploits — anywhere the app can open a file dialog, a browser, or a help window is a potential door to `cmd.exe`.

## Common breakout primitives

**File dialogs** (Open/Save/Print-to-file). Type a path or UNC into the filename box to browse the filesystem; right-click a file → **Open with** or drop to a command prompt via the address bar:

```text
Address bar:  cmd.exe
Address bar:  \\127.0.0.1\c$\Windows\System32\cmd.exe
Filename box: %SystemRoot%\System32\cmd.exe
```

**Help / hyperlinks.** An in-app Help window or any clickable link can launch a browser; from the browser, File → Save As reaches a file dialog (as above). This is the same GUI-walk used in [Windows Certificate Dialog Elevation of Privilege](Windows-Certificate-Dialog-Elevation-of-Privilege.md).

**Keyboard shortcuts & dialogs.** `Ctrl+O`, `Ctrl+S`, `Ctrl+P` (print → print to file → dialog), `Win+E`, `Shift+F10` (context menu), and the Windows/Start key if not blocked.

**Environment-variable and UNC tricks.** `%COMSPEC%`, `%WINDIR%\System32\cmd.exe`, and UNC paths often bypass path-based restrictions.

## After the breakout

Once you have `cmd`/`explorer` on the host, run the standard triage — [Situational Awareness](Situational-Awareness.md), [Escalate My Privilege Windows](Escalate-My-Privilege-Windows.md) — and escalate from your new foothold.

## Detection and defenses

- **Detection:** the published app spawning `cmd.exe`/`powershell.exe`/`explorer.exe` (Sysmon Event ID 1 with an unusual parent), file-dialog navigation to system paths.
- **Defenses:** app-locking (AppLocker/WDAC) to block child processes and unauthorized binaries, disable file-dialog navigation, remove Help/hyperlinks, restrict shortcut keys, and enforce least privilege inside the session.

## Related
- [Windows Privilege Escalation](README.md) — category MOC
- [Situational Awareness](Situational-Awareness.md) — first triage after breaking out
- [Escalate My Privilege Windows](Escalate-My-Privilege-Windows.md) — escalate from the host foothold
- [Windows Certificate Dialog Elevation of Privilege](Windows-Certificate-Dialog-Elevation-of-Privilege.md) — the same GUI-walk technique
