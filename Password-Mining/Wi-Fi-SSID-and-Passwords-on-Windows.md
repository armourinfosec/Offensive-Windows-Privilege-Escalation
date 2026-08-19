# Finding Wi-Fi SSID and Passwords on Windows

## Find Available Wi-Fi Profiles (SSIDs)

- To list all saved Wi-Fi profiles (SSIDs) on your system, run the following command in the Command Prompt:

```cmd
netsh wlan show profile
````

> This command displays a list of all the Wi-Fi profiles stored on the computer.

## Retrieve the Cleartext Password of a Wi-Fi Profile

- To extract the saved password (key) for a specific Wi-Fi profile, use the following command:

```cmd
netsh wlan show profile <SSID> key=clear
```

> Replace `<SSID>` with the actual name of the Wi-Fi network.

### Example:

- If the Wi-Fi profile name is `MyWiFi`, run:

```cmd
C:\> netsh wlan show profile MyWiFi key=clear
```

> The output will include security details, including the password under the **Key Content** field.

## Extract Wi-Fi Passwords for All Access Points (One-liner)

- To retrieve passwords for all saved Wi-Fi networks in a single command:

```cmd
cls & echo. & for /f "tokens=4 delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name=%a key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on
```

### Explanation:

- Clears the screen (`cls`).

- Extracts all stored Wi-Fi profile names.

- Runs `netsh wlan show profile` for each SSID.

- Filters the output to display only SSID, encryption type, and password.

- Ignores unnecessary information like "Number of SSIDs."


### Example Output:

```text
SSID name        : MyWiFi
Authentication   : WPA2-Personal
Cipher           : CCMP
Key Content      : mypassword123
```

## Additional Notes

- These commands require administrator privileges. Open Command Prompt as Administrator before executing them.

- If the output does not show **Key Content**, the password might not be stored in plaintext or the system may have encryption policies preventing its display.

- Use this information responsibly and only on networks you own or have permission to access.

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Windows Privilege Escalation](../README.md) — escalation context
- [Search for file contents](Search-for-file-contents/Search-for-file-contents.md) — searching for stored secrets
- Password Cracking — reuse harvested credentials
