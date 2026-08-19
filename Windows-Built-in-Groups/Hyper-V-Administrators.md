# Hyper-V Administrators

**Hyper-V Administrators** have complete control over the hypervisor, virtual machines, and their virtual-disk files. That control translates into host privilege escalation in two practical ways: reading arbitrary host files by mounting them into a VM (or attaching the host disk), and abusing the VM management service's file operations. On a virtualization host — including some Domain Controllers running as Hyper-V guests' parent — this group is a route to SYSTEM.

## Confirm membership

```cmd
whoami /groups | findstr /i "Hyper-V Administrators"
```

## Exploitation paths

**Read protected host files via a VHD.** A Hyper-V Admin can attach/mount virtual disks and, in some configurations, read files outside their normal ACL — e.g. dumping the host SAM/SYSTEM or a VM's disk that contains credentials:

```powershell
Mount-VHD -Path C:\VMs\target.vhdx
# assign a drive letter, then read the guest filesystem offline
```

**vmms/vmcompute DLL hijack (CVE-2018-0952 class).** Historically, the VM management service could be coerced into loading an attacker DLL or following a symlink a Hyper-V Admin created, yielding SYSTEM. Where the host is unpatched, planting a hijackable DLL in a path the service loads and triggering a VM operation runs code as SYSTEM.

**Reset a guest's local admin.** For a VM you can control, mounting its VHD offline lets you plant a payload or clear the local Administrator password, then boot it.

## Detection and defenses

- **Detection:** unexpected `Mount-VHD`/disk-attach activity, VM management service loading non-standard DLLs, Hyper-V Administrators membership changes.
- **Defenses:** minimise Hyper-V Administrators membership; keep the virtualization host fully patched; isolate management of tier-0 hosts.

## Related
- [Windows Built in Groups](Windows-Built-in-Groups.md) — group overview and escalation map
- [Mounting VHD and VHDX](../Password-Mining/Mounting-VHD-and-VHDX.md) — mounting virtual disks to read protected files
- [SAM and SYSTEM files](../Password-Mining/SAM-and-SYSTEM-files.md) — the host hives you can recover this way
