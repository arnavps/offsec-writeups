# Windows Forensics 2

## Overview

Beyond the registry, Windows stores forensic artifacts across the file system itself. This room covers file systems used by Windows (FAT, exFAT, NTFS), the forensically critical NTFS Master File Table, deleted file recovery, and artifacts that provide evidence of execution, file/folder access, and external device usage — including Prefetch files, Jump Lists, Shortcut files, and browser history.

---

## Topics Covered

- FAT12/16/32 and exFAT file systems
- NTFS: journaling, access controls, Volume Shadow Copy, ADS, MFT
- MFT Explorer and MFTECmd usage
- Deleted files and disk image forensics
- Evidence of execution: Prefetch, Windows 10 Timeline, Jump Lists
- Evidence of file/folder access: Shortcut files, IE/Edge history, Jump Lists
- Tools: PECmd, WxTCmd, JLECmd, LECmd, Autopsy

---

## Key Concepts

### File Systems Overview

A file system organises the raw bits on a storage device into files and directories. Understanding file systems is essential for forensics because file system metadata — creation times, modification times, cluster allocation — provides evidence about activity.

#### FAT (File Allocation Table)

FAT uses a table to index where clusters (storage units) are located on disk.

| Version | Addressable bits | Max clusters | Max volume size |
|---------|-----------------|-------------|----------------|
| FAT12 | 12 | 4,096 | 32 MB |
| FAT16 | 16 | 65,536 | 2 GB |
| FAT32 | 28 (effective) | 268,435,456 | 2 TB (Windows limits to 32 GB format) |

FAT32 maximum file size is 4 GB minus 1 byte — a significant limitation for large media files.

#### exFAT

Developed by Microsoft for high-capacity flash media (SD cards, USB drives). Now the default for SD cards larger than 32 GB.

- Max file size and volume size: 128 PB
- Cluster size: 4 KB to 32 MB
- Maximum files per directory: ~2.8 million

#### NTFS (New Technology File System)

Introduced with Windows NT, mainstream since Windows XP. Resolves FAT limitations and adds security and reliability features.

| Feature | Description |
|---------|-------------|
| **Journaling** | Logs metadata changes in `$LOGFILE` — enables recovery from crashes |
| **Access controls** | Per-file and per-directory permissions with user ownership |
| **Volume Shadow Copy** | Tracks changes to files enabling previous version recovery |
| **Alternate Data Streams (ADS)** | Multiple data streams per file — used by browsers to mark downloaded files, abused by malware |
| **Master File Table (MFT)** | Structured database tracking all objects on the volume |

---

### NTFS Master File Table (MFT)

The MFT is a structured database where every file and directory on an NTFS volume has at least one entry.

| MFT File | Description |
|----------|-------------|
| `$MFT` | First record — directory of all files and their cluster locations |
| `$LOGFILE` | Transactional log of file system operations — helps maintain integrity |
| `$UsnJrnl` (in `$Extend`) | Update Sequence Number Journal — records all file changes and the reason for each change |

#### MFT Analysis with MFTECmd

```
MFTECmd.exe -f <path-to-$MFT> --csv <output-path>
```

Output can be viewed with EZviewer. Provides: file name, path, created/modified/accessed/MFT record changed timestamps, size, flags.

---

### Deleted Files and Recovery

When a file is deleted from NTFS, the file system marks the clusters as unallocated but does not immediately overwrite the data. The file contents remain on disk until overwritten.

Recovery is possible because:
- The file system metadata entry marks the space as available — the actual data persists
- Forensic tools can scan unallocated space to recover file content

**Disk Image:** A bit-for-bit copy of a storage device preserving all data including unallocated space and metadata. Essential for forensics:
- Original evidence is not contaminated
- Multiple copies can be made for parallel investigation

**Tool for deleted file recovery:** Autopsy — can identify and recover deleted files from disk images.

---

## Evidence of Execution

### Windows Prefetch Files

Windows stores execution information for frequently used programs in Prefetch files to speed up future launches.

- **Location:** `C:\Windows\Prefetch`
- **Extension:** `.pf`
- **Contains:** Last run times, total run count, files and device handles used by the program

**Tool:** PECmd.exe (Eric Zimmerman)

```
PECmd.exe -f <path-to-prefetch-file> --csv <output-path>
PECmd.exe -d <prefetch-directory> --csv <output-path>
```

### Windows 10 Timeline

Windows 10 stores recently used applications and files in an SQLite database.

- **Location:** `C:\Users\<username>\AppData\Local\ConnectedDevicesPlatform\{folder}\ActivitiesCache.db`
- **Contains:** Application name, focus time (how long the app was in focus)

**Tool:** WxTCmd.exe (Eric Zimmerman)

```
WxTCmd.exe -f <path-to-ActivitiesCache.db> --csv <output-path>
```

### Windows Jump Lists

Jump Lists appear when right-clicking taskbar icons and show recently used files in that application. Also stored as files.

- **Location:** `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations`
- **Contains:** Application ID (AppID), first execution time, last execution time, recently opened files per application

**Tool:** JLECmd.exe (Eric Zimmerman)

```
JLECmd.exe -f <path-to-jumplist-file> --csv <output-path>
```

---

## Evidence of File/Folder Access

### Shortcut Files (LNK Files)

Windows creates a shortcut file each time a file is opened locally or remotely.

- **Locations:**
  - `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Recent\`
  - `C:\Users\<username>\AppData\Roaming\Microsoft\Office\Recent\`
- **Contains:** First opened time (creation date of the LNK file), last opened time (modification date), full path of the opened file

**Tool:** LECmd.exe — Lnk Explorer (Eric Zimmerman)

```
LECmd.exe -f <path-to-lnk-file> --csv <output-path>
```

### IE/Edge Browser History

Internet Explorer and Edge history records not only websites visited but also files opened locally — any file opened on the system that the browser was aware of appears with a `file:///` prefix.

- **Location:** `C:\Users\<username>\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat`
- **Tool:** Autopsy — use Logical Files as data source; view under Data Artifacts

### Jump Lists (File Access)

As noted in the execution section, Jump Lists also serve as evidence of file access — they record the files most recently opened in each application, providing both the application used and the file accessed.

---

## Practical Commands Summary

| Task | Command |
|------|---------|
| Parse MFT file | `MFTECmd.exe -f <$MFT path> --csv <output>` |
| Parse Prefetch files | `PECmd.exe -d <prefetch dir> --csv <output>` |
| Parse Windows 10 Timeline | `WxTCmd.exe -f <ActivitiesCache.db> --csv <output>` |
| Parse Jump Lists | `JLECmd.exe -f <jumplist file> --csv <output>` |
| Parse Shortcut files | `LECmd.exe -f <lnk file> --csv <output>` |

All CSV output files can be viewed with EZviewer (Eric Zimmerman's tools).

---

## Important Terminology

| Term | Meaning |
|------|---------|
| FAT | File Allocation Table — older file system using a linked list of clusters |
| NTFS | New Technology File System — Windows default with journaling, access control, and MFT |
| MFT | Master File Table — NTFS database tracking all files on the volume |
| ADS | Alternate Data Streams — NTFS feature allowing multiple data streams per file |
| Volume Shadow Copy | NTFS snapshot feature enabling file version recovery |
| Prefetch | Windows cache files storing execution metadata for frequently run programs |
| Jump List | Windows Taskbar feature recording recently opened files per application |
| LNK file | Windows shortcut file created when a file is opened — stores path and access timestamps |
| Disk image | Bit-for-bit copy of a storage device including all metadata and unallocated space |
| `$UsnJrnl` | NTFS change journal — records all file system changes with reason codes |

---

## Real-World Relevance

- Prefetch files are a primary evidence source for confirming program execution — they exist even after the program is deleted
- Jump Lists reveal which files were recently opened in specific applications — useful in insider threat and data exfiltration investigations
- LNK shortcut files persist even after the referenced file is deleted — the shortcut records the full path, timestamps, and even the original volume serial number
- NTFS Volume Shadow Copies are frequently targeted by ransomware operators for deletion — their presence (or absence) is evidence of ransomware activity
- Alternate Data Streams are used by malware to hide code and payloads in plain sight within legitimate-looking files
- The `$UsnJrnl` change journal provides a chronological log of every file system change — invaluable for timeline reconstruction

---

## Key Learnings

- FAT32 is limited to 4 GB files; exFAT removes this limit for flash media; NTFS is the standard for Windows installations
- NTFS journals file system changes in `$LOGFILE` — enables crash recovery and forensic timeline reconstruction
- Deleted files remain recoverable until their clusters are overwritten — disk image forensics can recover them
- Prefetch, Windows 10 Timeline, and Jump Lists all provide evidence of program execution with timestamps
- Shortcut (LNK) files and IE/Edge history provide evidence of file/folder access
- Eric Zimmerman's tools (PECmd, WxTCmd, JLECmd, LECmd, MFTECmd) are the standard toolkit for parsing these artifacts

---

## Conclusion

Windows file system forensics extends the investigation beyond the registry to the artifacts embedded in NTFS metadata, execution caches, and file access records. Together, Prefetch files, Jump Lists, shortcut files, and browser history provide a rich picture of user activity and program execution that complements registry analysis. Knowing where these artifacts live, what they contain, and which tools to use is essential for any Windows DFIR investigation.
