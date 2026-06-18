# Windows Forensics 1

## Overview

Windows is the dominant desktop and enterprise OS, making Windows forensics a core skill for DFIR professionals. Windows creates detailed records of user activity through the registry — tracking installed software, connected devices, recently accessed files, executed programs, and much more. This room covers the Windows Registry structure, forensically relevant registry hives and keys, and the tools used to acquire and analyse registry artifacts.

---

## Topics Covered

- Windows forensic artifacts and why they exist
- Windows Registry: structure and hives
- Registry acquisition tools: KAPE, Autopsy, FTK Imager
- Registry analysis tools: Registry Viewer, Registry Explorer, RegRipper
- System information from the registry: OS version, computer name, timezone, network
- User activity: autorun programs, recent files, ShellBags, UserAssist
- Program execution: ShimCache, AmCache, BAM/DAM
- External device tracking: USB history

---

## Key Concepts

### Why Windows Creates Forensic Artifacts

Windows stores user preferences and activity to personalise the computing experience — remembering recently opened files, preferred folder layouts, installed applications, and more. While created for usability, these same records become forensic evidence during investigations. Artifacts are distributed across the file system, registry, and user profile directories.

---

### The Windows Registry

The Windows Registry is a hierarchical database storing configuration data for hardware, software, and user settings. It is the richest source of forensic information on a Windows system.

**Five root keys:**

| Root Key | Abbreviation | Contents |
|----------|-------------|---------|
| `HKEY_CURRENT_USER` | HKCU | Configuration for the currently logged-in user |
| `HKEY_USERS` | HKU | All loaded user profiles on the computer |
| `HKEY_LOCAL_MACHINE` | HKLM | Machine-wide configuration for all users |
| `HKEY_CLASSES_ROOT` | HKCR | File association and COM object registration (merged view of HKLM and HKCU) |
| `HKEY_CURRENT_CONFIG` | HKCC | Hardware profile used at system startup |

**Registry terminology:**
- **Key** — a container (like a folder) in the registry tree
- **Value** — data stored within a key (like a file)
- **Hive** — a group of keys, subkeys, and values stored in a single file on disk

#### Important Registry Hive Files

| Hive | Location | Contents |
|------|----------|---------|
| SAM | `C:\Windows\System32\Config\SAM` | User accounts, password hashes, login information |
| SYSTEM | `C:\Windows\System32\Config\SYSTEM` | Hardware config, system services, control sets |
| SOFTWARE | `C:\Windows\System32\Config\SOFTWARE` | Installed software, OS version, network history |
| SECURITY | `C:\Windows\System32\Config\SECURITY` | Security policy, audit settings |
| NTUSER.DAT | `C:\Users\<username>\NTUSER.DAT` | Per-user settings and activity |
| USRCLASS.DAT | `C:\Users\<username>\AppData\Local\Microsoft\Windows\USRCLASS.DAT` | Shell folder settings (ShellBags) |
| AmCache | `C:\Windows\AppCompat\Programs\Amcache.hve` | Recently run programs with execution metadata |

**Transaction logs and backups:**
- Transaction logs (`.LOG`, `.LOG1`, `.LOG2`) contain the latest registry changes not yet written to the hive — always check these
- Registry backups are stored in `C:\Windows\System32\Config\RegBack` — copied every 10 days, useful if keys were recently deleted

---

### Registry Acquisition Tools

The registry hive files in `%WINDIR%\System32\Config` are locked by the OS and cannot be copied directly. These tools bypass the lock:

| Tool | Method |
|------|--------|
| **KAPE** | Live data acquisition — automates registry and artifact collection |
| **Autopsy** | Add data source → navigate to hive file → right-click → Extract File(s) |
| **FTK Imager** | Mount disk image or live drive → export registry hive files |

### Registry Analysis Tools

| Tool | Description |
|------|-------------|
| **Registry Viewer** (AccessData) | Similar UI to regedit; loads one hive at a time; does not process transaction logs |
| **Registry Explorer** (Zimmerman) | Loads multiple hives simultaneously; merges transaction logs; includes forensic bookmarks for key artifacts |
| **RegRipper** | Outputs a text report extracting data from forensically important keys; does not process transaction logs |

---

## Forensically Important Registry Keys

### System Information

**OS Version:**
```
SOFTWARE\Microsoft\Windows NT\CurrentVersion
```

**Computer Name:**
```
SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName
```

**Time Zone:**
```
SYSTEM\CurrentControlSet\Control\TimeZoneInformation
```

**Current Control Set (determines which ControlSet is active):**
```
SYSTEM\Select\Current
SYSTEM\Select\LastKnownGood
```

---

### Network Information

**Network interfaces (IP, DHCP, DNS):**
```
SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces
```

**Past networks connected to:**
```
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Unmanaged
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Signatures\Managed
```

---

### User Information

**SAM — user accounts, login counts, last login/failure, password info:**
```
SAM\Domains\Account\Users
```

Contains: relative identifier (RID), login count, last login time, last failed login, password change time, group membership.

---

### Autorun / Persistence

**Programs run at user logon:**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\RunOnce
SOFTWARE\Microsoft\Windows\CurrentVersion\Run
SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer\Run
```

**Services (start value `0x02` = starts at boot):**
```
SYSTEM\CurrentControlSet\Services
```

---

### Recent File Activity

**Recently opened files (Windows Explorer):**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.pdf
```

**Microsoft Office recent files:**
```
NTUSER.DAT\Software\Microsoft\Office\<VERSION>\<Application>
NTUSER.DAT\Software\Microsoft\Office\<VERSION>\UserMRU\LiveID_####\FileMRU
```

**Open/Save dialog history:**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\LastVisitedPidlMRU
```

**Explorer address bar / search history:**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery
```

---

### ShellBags

ShellBags record the window layout preferences when a user opens a folder — providing evidence that a user accessed a specific folder, even if the folder no longer exists.

```
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\Bags
USRCLASS.DAT\Local Settings\Software\Microsoft\Windows\Shell\BagMRU
NTUSER.DAT\Software\Microsoft\Windows\Shell\BagMRU
NTUSER.DAT\Software\Microsoft\Windows\Shell\Bags
```

**Tool:** ShellBag Explorer (Eric Zimmerman)

---

### Program Execution

**UserAssist — applications launched via Windows Explorer (GUI), stored as ROT13-encoded paths:**
```
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count
```

Stores: program name, launch count, last execution time.

**ShimCache (AppCompatCache) — all applications launched on the machine:**
```
SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache
```

Stores: file name, file size, last modified time.
**Tool:** AppCompatCache Parser (`AppCompatCacheParser.exe`)

```
AppCompatCacheParser.exe --csv <output_path> -f <SYSTEM_hive> -c <control_set>
```

**AmCache — similar to ShimCache with additional execution metadata:**
```
C:\Windows\appcompat\Programs\Amcache.hve
Amcache.hve\Root\File\{Volume GUID}\
```

Stores: execution path, installation time, execution time, deletion time, SHA1 hash.

**BAM (Background Activity Monitor) and DAM (Desktop Activity Moderator) — last run times of programs:**
```
SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}
SYSTEM\CurrentControlSet\Services\dam\UserSettings\{SID}
```

---

### External Device (USB) Tracking

**USB devices plugged in (vendor ID, product ID, version):**
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

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Registry hive | A group of registry keys, subkeys, and values stored as a single file |
| NTUSER.DAT | Per-user registry hive containing user-specific settings and activity |
| SAM | Security Account Manager — stores local user account information |
| AmCache | Hive recording recently executed programs with execution metadata and SHA1 hashes |
| ShimCache | AppCompatCache — tracks all launched applications for OS compatibility purposes |
| ShellBags | Registry artifact recording folder access and window layout preferences |
| UserAssist | Registry key tracking GUI-launched applications with run counts and timestamps |
| BAM/DAM | Background/Desktop Activity Monitor — records last run times of background programs |
| MRU | Most Recently Used — Windows maintains MRU lists for files, folders, and typed paths |
| Transaction log | `.LOG` file recording pending registry changes not yet flushed to the hive |

---

## Real-World Relevance

- Registry forensics is a core component of every Windows-based DFIR investigation — it provides a timeline of user activity, software execution, and device connections without requiring memory capture
- ShimCache and AmCache are primary evidence sources for confirming that a specific executable was run on a system — critical when malware is deleted post-execution
- USB device history (USBSTOR) is used in insider threat investigations to establish when and which removable media was connected
- Autorun keys are the most common persistence mechanisms used by malware — they are the first place analysts look during triage
- SAM hive analysis reveals local user accounts, login counts, and failed login attempts — important for detecting account manipulation

---

## Key Learnings

- The Windows Registry is the primary source of forensic artifacts on Windows systems
- Five root keys; most forensic data resides in SYSTEM, SOFTWARE, NTUSER.DAT, SAM, and AmCache hives
- Acquisition requires bypassing OS locks — use KAPE, Autopsy, or FTK Imager
- Registry Explorer (Zimmerman) is the preferred analysis tool — it merges transaction logs and provides forensic bookmarks
- Critical artifact categories: system info, network history, autorun/persistence, recent files, ShellBags, program execution (ShimCache, AmCache, BAM), USB history
- Transaction logs often contain the most recent registry changes — always include them in analysis

---

## Conclusion

The Windows Registry is a comprehensive log of system and user activity. Understanding which keys store what information, and how to extract and analyse them correctly, is foundational to Windows forensics. The artifacts covered here — from autorun keys and ShimCache to ShellBags and USB history — collectively paint a detailed picture of what happened on a Windows system, when it happened, and who was involved.
