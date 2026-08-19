# Password in Alternate Data Stream (ADS)

## What is an Alternate Data Stream (ADS)?

**Alternate Data Streams (ADS)** are a feature of the **NTFS (New Technology File System)** that allow hidden data to be stored within files. While ADS can be used for legitimate purposes, such as metadata storage, they can also be misused for hiding malicious content, including malware, confidential information, or secret messages.


## Creating and Editing Files with ADS  

- To create or edit a normal text file:  

```bash
notepad test1.txt
````

> This opens **Notepad**, allowing you to create or edit `test1.txt`.

- To write simple text data into a file using the **echo** command:

```bash
echo Test > test2.txt
```

> This creates `test2.txt` with the content `"Test"`.


## Creating an Alternate Data Stream

- You can create an ADS using Notepad by specifying a stream name:

```bash
notepad test3.txt:test
```

> This creates or edits an **ADS** named `test` inside `test3.txt`.

- Similarly, another hidden stream can be created using:

```bash
notepad 5.txt:test1
```

> This creates an ADS named `test1` inside `5.txt`.

> **Note:** When opening these files in Notepad, no visible content will appear unless accessed via ADS commands.

## Listing Alternate Data Streams

- To check for **Alternate Data Streams** (ADS) in a directory, use the following command in **Command Prompt (cmd):**

```bash
dir /R
```

> This will list all files and any associated **ADS** in the directory. If a file has an ADS, it will be displayed after the file name in the format:

```text
filename.txt:streamname
```

- To list all files and **ADS** recursively across subdirectories:

```bash
dir /R /b /s
```

> This command shows all files, including hidden streams, in a simple format (`/b`) and across all subdirectories (`/s`).


## Retrieving ADS in PowerShell

- To retrieve all streams associated with files in the current directory using **PowerShell**, use:

```powershell
Get-Item -Path * -Stream *
```

> This command lists all alternate streams in the directory, helping to identify hidden content.

### Viewing a Specific Alternate Data Stream

- If you know the file and stream name, you can access its content using:

```powershell
Get-Item -Path .\5.txt -Stream test1
```

> This retrieves the **"test1"** stream from **"5.txt"**. However, it only shows information about the stream, not its content.

### Reading Hidden Data from an ADS

- To read the content of a specific ADS, use:

```powershell
Get-Content -Path flag.txt -Stream Flag
```

> This command extracts and displays the hidden content stored in the **"Flag"** stream of the file **"flag.txt"**.


## Recreating the File and ADS

- If the file is missing or corrupted, recreate it and add an **ADS**:

```powershell
echo "This is a secret message." > C:\Users\Administrator\secret.txt
Set-Content -Path C:\Users\Administrator\secret.txt -Stream hidden -Value "HiddenPassword123"
```

> Now, check if the **ADS** is properly stored:

```powershell
Get-Content -Path C:\Users\Administrator\secret.txt -Stream hidden
```


## Extracting Data from an ADS

- To retrieve and display the hidden password, use:

```powershell
Get-Content -Path secret.txt -Stream hidden
```

> This prints the content of the **"hidden"** ADS to the console.

### Saving ADS Content to a File

- To save the extracted ADS content into a new file for further analysis, use:

```powershell
Get-Content -Path secret.txt -Stream hidden | Out-File extracted.txt
```

> This will create **extracted.txt** containing the hidden data from the ADS.

## Deleting an ADS

- To remove an **ADS** while keeping the original file intact, use:

```powershell
Remove-Item -Path secret.txt -Stream hidden
```

> This deletes the **"hidden"** stream from **"secret.txt"**, but the main file remains.


## Additional Tools for Managing ADS

- For better visibility and management of **ADS**, third-party tools can be used. One such tool is **AlternateStreamView** from NirSoft:

[NirSoft Alternate Data Streams Utility](https://www.nirsoft.net/utils/alternate_data_streams.html)

> This tool provides a graphical interface to scan, view, and delete ADS from files in NTFS.



## Conclusion

**Alternate Data Streams (ADS)** can be useful for storing metadata but can also be misused to hide data. Understanding how to detect and manage ADS is essential for **cybersecurity** and **system administration**.

## Related
- [Password Mining](Password-Mining.md) — parent hub
- [Search for file contents](Search-for-file-contents/Search-for-file-contents.md) — locating hidden secrets in files
- [Windows Privilege Escalation](../README.md) — escalation context
- Password Cracking — reuse harvested credentials
