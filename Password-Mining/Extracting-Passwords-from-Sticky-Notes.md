# Extracting Passwords from Sticky Notes  

- The **Sticky Notes** app in **Windows** stores its contents, including potential passwords, in an **SQLite database** file located at:  

```cmd
C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite
````

- Since users often store **sensitive information** like **passwords, notes, and credentials** in **Sticky Notes**, this database can be a valuable target for **forensic analysis** and **security auditing**.

## Locating the Sticky Notes Database  

- To access the database, replace `<user>` with the actual Windows **username** and navigate to the location using **File Explorer** or **Command Prompt**:

```cmd
cd C:\Users\%USERNAME%\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\
````

- Verify if `plum.sqlite` exists:

```cmd
dir plum.sqlite
```

## Extracting Data from Sticky Notes (SQLite)

- Since the **Sticky Notes database** is an **SQLite file**, you can use the **SQLite command-line tool** or **Python** to extract stored content.

### 1. Using SQLite Command-Line Tool

- First, install **SQLite** if you haven’t already:

  - **Windows:** [Download SQLite](https://www.sqlite.org/download.html)
  - Extract and move `sqlite3.exe` to `C:\Windows\System32\` (for easy access)


- Run the following command to open the database:

```cmd
sqlite3 plum.sqlite
```

- Now, list all tables:

```sql
.tables
```

- To view stored **Sticky Notes** content, run:

```sql
SELECT * FROM Note;
```

> This will display all saved notes, including any **potentially stored passwords**.


### 2. Extracting Notes with Python

- Alternatively, use **Python** to automate the extraction:

```python3
import sqlite3

db_path = r"C:\Users\%USERNAME%\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite"

conn = sqlite3.connect(db_path)
cursor = conn.cursor()

cursor.execute("SELECT * FROM Note")
notes = cursor.fetchall()

for note in notes:
    print(note)

conn.close()
```

> This script will extract and print **all Sticky Notes** stored in the **plum.sqlite** database.

## Related
- [Password Mining](Password-Mining.md) — parent hub (Windows credential harvesting)
- [Windows Privilege Escalation](../README.md) — escalation context
- [Search for file contents](Search-for-file-contents/Search-for-file-contents.md) — grepping files for secrets
- Password Cracking — reuse harvested credentials
