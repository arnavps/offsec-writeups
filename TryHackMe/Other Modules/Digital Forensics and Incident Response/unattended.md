# Unattended

## Overview

Unattended is a practical Windows forensics challenge room on TryHackMe. The scenario involves investigating a Windows system to determine what activity occurred while the machine was left unattended. The investigation relies on Windows registry artifacts, file system artifacts, and user activity traces to reconstruct what happened.

---

## Key Concepts Applied

This room is a practical application of the forensic techniques covered in Windows Forensics 1 and Windows Forensics 2. The investigation uses:

- Registry hive analysis with Registry Explorer and Eric Zimmerman's tools
- User activity artifacts: RecentDocs, ShellBags, UserAssist
- Program execution evidence: Prefetch files, ShimCache/AmCache
- File access traces: LNK shortcut files, Jump Lists
- Browser history analysis

---

## Investigation Approach

### Step 1 — Establish Context

Before diving into specific artifacts, establish basic system context:

```
SOFTWARE\Microsoft\Windows NT\CurrentVersion          → OS version
SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName  → Computer name
SYSTEM\CurrentControlSet\Control\TimeZoneInformation  → Timezone
```

### Step 2 — Identify User Accounts

```
SAM\Domains\Account\Users    → Local user accounts, login counts, timestamps
```

### Step 3 — Check Recent File Access

```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
```

Parse LNK files for full file paths and access timestamps:
```
C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Recent\
```

Command: `LECmd.exe -d <recent-files-dir> --csv <output-path>`

### Step 4 — Check Program Execution

**Prefetch files:**
```
C:\Windows\Prefetch\*.pf
```
Command: `PECmd.exe -d <prefetch-dir> --csv <output-path>`

**ShimCache** — all launched applications:
```
SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache
```
Command: `AppCompatCacheParser.exe --csv <output-path> -f <SYSTEM-hive>`

**AmCache** — recently executed programs with SHA1 hashes:
```
C:\Windows\appcompat\Programs\Amcache.hve
```

### Step 5 — Check ShellBags for Folder Access

ShellBags record folders a user navigated to — even if the folder no longer exists:
```
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags
NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU
```

Tool: ShellBag Explorer (Eric Zimmerman)

### Step 6 — Check Autorun and Persistence Keys

```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run
SOFTWARE\Microsoft\Windows\CurrentVersion\Run
SYSTEM\CurrentControlSet\Services
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Registry Explorer | Browse and analyse registry hives |
| LECmd.exe | Parse LNK shortcut files |
| PECmd.exe | Parse Prefetch files |
| AppCompatCacheParser.exe | Parse ShimCache from SYSTEM hive |
| ShellBag Explorer | Parse ShellBag artifacts |
| EZviewer | View CSV output from all EZ tools |

---

## Notes

This is a challenge room — specific answers and flags are not documented here, as the investigation is designed to develop hands-on registry forensics skills. Work through the questions by applying the artifact locations and tool commands documented in the Windows Forensics 1 and Windows Forensics 2 writeups.

---

## Key Learnings

- Practical application of Windows registry forensics across all major artifact categories
- The combination of RecentDocs, LNK files, ShellBags, and Prefetch provides comprehensive evidence of file access and program execution
- ShellBags prove folder navigation even when the folder has been deleted
- Prefetch files prove program execution even when the executable has been deleted
