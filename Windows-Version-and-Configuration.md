# Windows Version and Configuration

This document covers essential commands for extracting detailed information about Windows versions, configurations, patches, environment variables, and drives. Each section includes the command, an explanation, and an example where applicable.

- Displays the current computer's hostname.

```bash
hostname
```


- Retrieves detailed information about the operating system version and configuration.

```bash
systeminfo
```

> Example Output:

```text
OS Name:                   Microsoft Windows 10 Pro  
OS Version:                10.0.19044 N/A Build 19044  
System Manufacturer:       Dell Inc.  
System Model:              XPS 15 9570  
```

> You can filter specific details using `findstr`:

```bash
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

> Explanation:
> `/B` → Match at the beginning of a line
> `/C` → Match a specific string


- Lists all installed patches and updates.

```bash
wmic qfe
```

- Get specific update details:

```bash
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

- Search for specific updates (e.g., `KB5000802`):

```bash
wmic qfe get Caption,Description,HotFixID,InstalledOn | findstr /C:"KB5000802"
```

- Identifies the system's architecture (e.g., `x64`, `ARM`).

```bash
wmic os get osarchitecture
```

- Alternative method using environment variables:

```bash
echo %PROCESSOR_ARCHITECTURE%
```

- Lists all environment variables available to the system.

> Using `set`:

```bash
set
```

> Using PowerShell:

```powershell
Get-ChildItem Env: | ft Key,Value
```


- Retrieves details about logical disks and physical drives.

> List all drives:

```bash
wmic logicaldisk
```

> Get detailed drive information:

```bash
wmic logicaldisk get caption,description,providername
```

> Alternative commands:

```bash
fsutil fsinfo drives
```

> Using PowerShell:

```powershell
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"} | ft Name,Root
```


- Displays detailed information about the OS configuration.

```bash
wmic os get /format:list
```

- Displays the exact OS version from the EULA file.

```bash
type C:\Windows\System32\eula.txt
```


## Usage Tips:

- `wmic` is a powerful tool for querying system information — it's available in most Windows versions.

- PowerShell commands (`Get-PSDrive`, `Get-ChildItem`) provide more flexibility and filtering options.

- `findstr` is helpful for filtering specific outputs when working with large amounts of data.

- Keep an eye on patch versions — they often reveal potential exploitation vectors!

## Related
- [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — patch level drives exploit selection
- [Windows Certificate Dialog Elevation of Privilege](Windows-Certificate-Dialog-Elevation-of-Privilege.md) — OS-version-specific exploit example
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — feed systeminfo to suggesters
- [User Enumeration](User-Enumeration.md) — companion host enumeration
