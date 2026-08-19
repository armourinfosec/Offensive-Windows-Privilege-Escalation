# Step-by-step to Enable WinRM on Windows 11

Windows Remote Management (WinRM, TCP 5985/5986) is the transport behind PowerShell Remoting and **Evil-WinRM** — the tool of choice for landing an interactive shell once you have valid Windows credentials. In a lab or post-exploitation context you often need to *enable* WinRM on a target you control to practise lateral movement and remote command execution. This note is the quick enablement and verification workflow; enabling it requires local administrator rights.

## 1. Open PowerShell as Administrator

- Right-click the Start menu.

- Select **Windows PowerShell (Admin)** or **Windows Terminal (Admin)**.

## 2. Run the WinRM Quick Configuration Command

```bash
winrm quickconfig
```
> This command will:
- Start the WinRM service.

- Set the service to start automatically.

- Create a listener to accept incoming WinRM requests.

- Add firewall exceptions to allow remote management.

- When prompted, type **Y** to confirm and proceed.

## 3. Verify the Configuration

- Check WinRM listeners:

```bash
winrm enumerate winrm/config/listener
```

```bash
nmap -v -sT -sV -sC -A -O -p 5985 192.168.1.71
```

- Check the WinRM service status:

```bash
get-service -Name winrm
```
- The service status should be **Running** and startup type should be **Automatic**.

```bash
evil-winrm -i 192.168.1.71 -u rahul -p 123
```

## 4. (Optional) Configure WinRM for HTTPS

- To secure communication over HTTPS, configure WinRM with SSL certificates.

- Refer to Microsoft's documentation for detailed guidance:

  [Configure WinRM for HTTPS](https://learn.microsoft.com/en-us/troubleshoot/windows-client/system-management-components/configure-winrm-for-https)

## Related
- Evil WinRM(Windows Remote Management) — connect to enabled WinRM as user
- [Network Enumeration](Network-Enumeration.md) — scan port 5985 listener
- [User Enumeration](User-Enumeration.md) — identify accounts for WinRM auth
- Lateral Movement — WinRM enables remote command execution
