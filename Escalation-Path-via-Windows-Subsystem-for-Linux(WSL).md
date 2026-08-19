# Escalation Path via Windows Subsystem for Linux (WSL)

The **Windows Subsystem for Linux (WSL)** allows users to run Linux distributions natively on Windows. If misconfigured, it can become a vector for **privilege escalation**, **policy bypass**, or **unauthorized code execution**.

## Finding WSL Binaries

- To locate binaries associated with WSL on a Windows system, use the following commands.

### 1. Finding `bash.exe` (WSL 1)

- `bash.exe` was used as the entry point in early WSL 1 implementations.

```cmd
where /R c:\windows bash.exe
````

### 2. Finding `wsl.exe` (WSL 2)

- The modern `wsl.exe` binary is used for WSL 2.

```cmd
C:\> where /R c:\windows wsl.exe
```

## How WSL Can Be Abused for Privilege Escalation

### 1. WSL with Sudo

- If WSL is accessible to a user who already has elevated rights or if WSL is misconfigured, attackers can use `sudo` inside the Linux environment to execute commands as **root**. While root access is scoped to WSL, it can be abused to impact the Windows host in certain scenarios.

### 2. Bypassing Execution Controls

- Linux binaries executed via WSL may bypass Windows execution controls such as **AppLocker** or **Windows Defender Application Control (WDAC)** if those policies do not explicitly cover WSL binaries. This can allow execution of otherwise blocked payloads.

### 3. Access to Windows File System

- WSL mounts Windows drives under `/mnt/c`, `/mnt/d`, etc. This allows Linux tools to read or modify Windows files, potentially enabling:

  - Tampering with user files

  - Dropping payloads into startup locations

  - Modifying scripts or binaries executed by Windows users

### 4. Legacy WSL 1 Risks

- WSL 1 lacks the isolation provided by WSL 2. Older versions have had weaknesses that could be leveraged for escalation or sandbox escape. Systems still using WSL 1 are at higher risk.


## Defense and Hardening Against WSL Abuse

### 1. Disable WSL if Not Required

- If WSL is not needed, disable it completely.

**Via Windows Features**

- Control Panel → Programs → Turn Windows features on or off

- Uncheck **Windows Subsystem for Linux**

**Via PowerShell**

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```


### 2. Limit WSL Access

- Restrict which users can install or execute WSL distributions.

- Use **Group Policy** or privilege management to limit WSL usage to trusted users.


### 3. Monitor WSL Activity

- Monitor execution of WSL-related binaries such as:

  - `wsl.exe`

  - `bash.exe`

Use **Sysmon** or EDR solutions to alert on:

-  Unexpected WSL execution

-  Access to sensitive Windows directories from WSL

- Suspicious child processes spawned by `wsl.exe`


### 4. Use WSL 2 and Keep It Updated

WSL 2 runs inside a lightweight virtual machine, providing better isolation than WSL 1. Ensure:

- WSL 2 is used instead of WSL 1

- Windows and WSL components are regularly patched


## Additional Mitigation Techniques

1. **AppLocker / WDAC**

   - Explicitly control or block execution of `wsl.exe` and `bash.exe` if WSL is not required.

2. **EDR Configuration**

   - Detect unusual Linux-to-Windows interactions.

   - Flag attempts to modify Windows startup folders or system binaries from WSL.


## Useful Commands

### Checking Permissions on WSL Binaries

```cmd
icacls "C:\Windows\System32\wsl.exe"
```

- Ensure standard users do not have write permissions.


### Monitoring WSL Execution with Sysmon

- Configure Sysmon rules to log:

  - Process creation events for `wsl.exe`
  
  - File modifications in sensitive Windows paths initiated by WSL

## Related
- Linux Privilege Escalation — sudo/root abuse inside WSL
- [Services Exploitation](Services-Exploitation/Services-Exploitation.md) — alternative misconfiguration escalation vector
- [Windows Kernel Exploits](Windows-Kernel-Exploits/Windows-Kernel-Exploits.md) — another local escalation route
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — enumerate WSL misconfigurations
