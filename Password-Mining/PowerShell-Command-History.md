# PowerShell Command History

- PowerShell maintains a history of the commands you execute, but how it stores and retrieves this history depends on the version and execution context.

## Session History (Current Session)

PowerShell maintains an in-memory history of commands executed in the current session.

### View Command History

- This command lists all commands executed in the current session.

```powershell
Get-History
````

### Rerun a Command from History

- Replace `<ID>` with the command's history ID to execute it again.

```powershell
Invoke-History <ID>
```


### Clear the Session History

- This clears the command history of the current session.

```powershell
Clear-History
```

## Persistent History (Across Sessions)

- Starting from **PowerShell 5.0**, command history is saved persistently in a text file.

```powershell
Get-Host | Select-Object Version
```

### Default Location of Persistent History

#### Windows:

```cmd
%userprofile%\AppData\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

### Persistent History

```powershell
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath
```

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath -Tail 50
```

#### Linux/macOS:

```powershell
~/.local/share/powershell/PSReadLine/ConsoleHost_history.txt
```

## Filtering and Searching History

### Search for Password in History

- Replace `"keyword"` with the term you want to search for in the command history.

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath | Where-Object CommandLine -Match "password"
```

```powershell
Get-History | Where-Object CommandLine -Match "password"
```


### Retrieve the Last 10 Commands

- This command fetches the last 10 executed commands.

```powershell
Get-History -Count 10
```

### Interactive Reverse Search

- Press `Ctrl + R` to interactively search through command history, similar to Bash.

## Deleting History

### Remove Specific Command(s) from Session History

- Replace `<ID>` with the history ID of the command to remove it.

```powershell
Remove-History <ID>
```

### Clear Persistent History

- This command permanently deletes the history file.

```powershell
Remove-Item (Get-PSReadLineOption).HistorySavePath
```


## Alternative Methods to View PowerShell History

### Using `type` Command in Command Prompt

```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

```cmd
type C:\Users\test1\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

```cmd
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

### Using `cat` Command in PowerShell

```powershell
cat (Get-PSReadlineOption).HistorySavePath
```

### Search for Specific Entries in History

- This searches the persistent history file for any commands containing `"passw"`.

```powershell
cat (Get-PSReadlineOption).HistorySavePath | Select-String "passw"
```

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Search for file contents](Search-for-file-contents/Search-for-file-contents.md) — searching files for secrets
- [Windows Privilege Escalation](../README.md) — escalation context
- Password Cracking — reuse harvested credentials
