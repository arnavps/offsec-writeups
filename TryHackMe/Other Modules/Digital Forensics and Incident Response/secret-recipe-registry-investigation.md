# Secret Recipe: Registry Investigation

## Overview

This room is a practical Windows registry forensics challenge. The scenario involves investigating whether an IT employee (James) copied confidential files from another user's machine (Jasmine's laptop at Coffely). Registry hives extracted from the suspect machine are analysed using Eric Zimmerman's tools to determine what activity occurred and whether data was exfiltrated.

---

## Scenario

Jasmine, owner of a coffee shop, kept her secret recipe only on her work laptop. James from IT was consulted to fix the laptop. It is suspected he copied the recipe files to his machine. Registry hives have been extracted from the suspect host and placed in `C:\Users\Administrator\Desktop\Artifacts`. EZ Tools are at `C:\Users\Administrator\Desktop\EZ Tools`.

---

## Artifacts Available

| Hive | Location |
|------|----------|
| SYSTEM | `C:\Users\Administrator\Desktop\Artifacts\SYSTEM` |
| SECURITY | `C:\Users\Administrator\Desktop\Artifacts\SECURITY` |
| SOFTWARE | `C:\Users\Administrator\Desktop\Artifacts\SOFTWARE` |
| SAM | `C:\Users\Administrator\Desktop\Artifacts\SAM` |
| NTUSER.DAT | `C:\Users\Administrator\Desktop\Artifacts\NTUSER.DAT` |
| UsrClass.dat | `C:\Users\Administrator\Desktop\Artifacts\UsrClass.dat` |

---

## Investigation Approach

### Step 1 — System Baseline

Load the SYSTEM and SOFTWARE hives in Registry Explorer.

**OS version:**
```
SOFTWARE\Microsoft\Windows NT\CurrentVersion
```

**Computer name:**
```
SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName
```

**Timezone:**
```
SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```

**Last known good control set:**
```
SYSTEM\Select\LastKnownGood
```

### Step 2 — User Account Review

Load the SAM hive in Registry Explorer.

```
SAM\Domains\Account\Users
```

Records: login count, last login time, last failed login time, account creation date — establishes who was using the machine and when.

### Step 3 — Recent File Access (Focus on Document Files)

Load NTUSER.DAT in Registry Explorer.

**All recently opened files:**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
```

**Recently opened by extension (look for recipe-related formats):**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.pdf
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.docx
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.txt
```

**Explorer typed paths (URLs and paths manually entered):**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
```

**Open/Save dialog history:**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU
```

### Step 4 — ShellBags (Folder Navigation)

Load UsrClass.dat in ShellBag Explorer (EZ Tool).

```
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU
```

ShellBags reveal every folder James navigated to on Jasmine's machine — including folders on USB drives or network shares even if since deleted or disconnected.

### Step 5 — USB Device History

If James copied files to a USB drive, evidence will be in the SYSTEM hive:

**Devices connected:**
```
SYSTEM\CurrentControlSet\Enum\USBSTOR
SYSTEM\CurrentControlSet\Enum\USB
```

**Connection timestamps for a specific device:**
```
SYSTEM\CurrentControlSet\Enum\USBSTOR\Ven_Prod_Version\USBSerial#\Properties\{83da6326-97a6-4088-9453-a19231573b29}\####
```

| Value | Information |
|-------|-------------|
| `0064` | First connection time |
| `0066` | Last connection time |
| `0067` | Last removal time |

**USB device volume name:**
```
SOFTWARE\Microsoft\Windows Portable Devices\Devices
```

### Step 6 — Network Activity

Past networks the machine connected to:
```
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed
```

If James connected to an external network share or cloud storage, traces may appear here.

### Step 7 — Program Execution (Did James Run Any Tools?)

**UserAssist** — GUI applications launched by the user:
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count
```

**BAM** — last run times of programs:
```
SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}
```

**Prefetch** — parse from file system:
```
PECmd.exe -d C:\Windows\Prefetch --csv <output-path>
```

### Step 8 — Autorun / Persistence Check

```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run
SOFTWARE\Microsoft\Windows\CurrentVersion\Run
SYSTEM\CurrentControlSet\Services
```

---

## Tools Used

| Tool | Hive | Purpose |
|------|------|---------|
| Registry Explorer | SYSTEM, SOFTWARE, SAM, NTUSER.DAT | Browse all registry hives |
| ShellBag Explorer | UsrClass.dat, NTUSER.DAT | Analyse folder navigation history |
| AppCompatCacheParser | SYSTEM | Parse ShimCache (program execution) |
| PECmd.exe | Prefetch files | Parse Prefetch execution evidence |
| JLECmd.exe | Jump Lists | Parse recently opened files per application |
| LECmd.exe | LNK files | Parse shortcut files for file access evidence |
| EZviewer | CSV output | View tool output |

---

## What to Look For

- **Recent document access** that includes recipe-related file names in RecentDocs
- **ShellBag entries** showing navigation to folders containing recipe files
- **USB device connection** timestamps correlating with when James had access to the laptop
- **File copy programs** (robocopy, xcopy, cloud sync tools) in UserAssist or BAM
- **Network connections** to cloud storage or external shares around the time James had access

---

## Notes

This is a challenge room — specific answers and flags are not documented here. Apply the registry forensics techniques from Windows Forensics 1 and Windows Forensics 2 to work through the investigation questions using the provided hives and EZ Tools.

---

## Key Learnings

- ShellBags are the most powerful artifact for proving folder access — they persist even after files are deleted or devices disconnected
- USB USBSTOR timestamps provide precise connection and removal times — correlate against user account login times
- RecentDocs and LNK files show which specific files were opened — look for the recipe file names
- UserAssist and BAM show which programs were executed — detect data exfiltration tools
- Building a timeline from multiple artifacts (account login → folder navigation → file access → USB connection → USB removal) is more compelling than any single artifact alone
