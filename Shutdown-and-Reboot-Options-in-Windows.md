# Shutdown and Reboot Options in Windows

In Windows, users can trigger system shutdowns or reboots through the`shutdown` command. Additionally,Group Policy (GPO) settings can be used to control which users are allowed to shut down or reboot the system, including remotely.

##  Shutdown and Reboot Commands

The`shutdown` command allows users to shut down or reboot the system with various options.

### Shutdown Command Syntax:

1. Reboot the System:

   - Command:

```cmd
shutdown /r /t 0 /f
```

> `/r`: Reboot the system.
> `/t 0`: Sets the timeout to 0 seconds (immediate shutdown).
> `/f`: Forces running applications to close without warning.

2. Shutdown the System:
- Command:

```cmd
shutdown /s /t 0 /f
```

> `/s`: Shuts down the system.
> `/t 0`: Sets the timeout to 0 seconds (immediate shutdown).
> `/f`: Forces running applications to close without warning.


## Allow / Prevent Shutdown and Reboot via GPO (Group Policy)

Group Policy allows administrators to control shutdown and reboot permissions for local users or remote systems. This can be configured using`gpedit.msc` onnon-Active Directory (non-AD) systems.

### Steps to Configure Shutdown Permissions Using GPO:

1. Open Group Policy Editor:
   - Press`Win + R` and type `gpedit.msc` to openGroup Policy Editor.

2. Navigate to Shutdown Settings:
   - Go to:

     ```text
     Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> User Rights Assignment
     ```

3. Modify Shutdown Permissions:

   - Shut down the system: This policy controls which users can shut down the system locally.
	    - Default users/groups with permission:
	    - Administrator
	    - Backup Operators
	    - Add any other user (e.g., `rahul`) to allow them to shut down the system:
	    - Example: `rahul` is added to the list of users who can shut down the system.


### Allow Remote Shutdown / Restart Without Admin Permissions

- If you want users to be able to remotely shut down or restart the system without having administrative permissions, you can configure the following setting inGroup Policy:

1. Open Group Policy Editor by typing`gpedit.msc` inRun.

2. Navigate to:

```text
Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> User Rights Assignment
```

3. Modify the following policy:

- Force Shutdown from a Remote System: Allows users to initiate a shutdown or reboot from another machine without requiring admin privileges.
  - Default users/groups with permission:
    - Administrator
  - Add a user (e.g., `rahul`) to grant them remote shutdown/restart permissions.


## Security Considerations

- Limit Shutdown Permissions: Allowing users to shut down or reboot the system can disrupt service or compromise security. Carefully limit shutdown permissions to trusted users.

- Remote Shutdown/Reboot: Granting remote shutdown/reboot permissions should be restricted to avoid potential misuse by unauthorized users.

- Audit and Monitor: Regularly audit users with shutdown/reboot permissions, especially for remote actions, to prevent unauthorized system shutdowns.


## Defensive Measures

1. Restrict Access to GPO Settings: Ensure only administrators have access to modify Group Policy settings related to shutdown and reboot.

2. Use AppLocker or WDAC to prevent unauthorized execution of shutdown or restart commands.

3. Monitor Event Logs for any shutdown/reboot activities, particularly if triggered remotely, to detect potential abuse.

## Related
- [User Enumeration](User-Enumeration.md) — check assigned user rights
- [Token Impersonation](Impersonation-and-Potato-Attacks/Token-Impersonation.md) — parallel privilege/rights abuse
- [Registry Exploitation Techniques](Registry-Exploitation/Registry-Exploitation-Techniques.md) — GPO settings backed by registry
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — audit privileges and rights
