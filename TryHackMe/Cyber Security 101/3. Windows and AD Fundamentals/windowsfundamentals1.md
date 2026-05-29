# Windows Fundamentals 1

## Overview

Windows is the dominant operating system in enterprise environments and the most common target in cybersecurity. This room covers the foundational components of Windows — the file system, permissions, environment variables, and user account types. Understanding how Windows is structured at this level is essential before moving into offensive or defensive Windows security work.

---

## Topics Covered

- The NTFS file system and its features
- NTFS permissions and Alternate Data Streams (ADS)
- The Windows directory and environment variables
- User account types: Administrator and Standard User

---

## Key Concepts

### The NTFS File System

Modern versions of Windows use NTFS (New Technology File System). Before NTFS, Windows used FAT16/FAT32 and HPFS. FAT partitions are still common on USB drives and SD cards, but not on Windows computers or servers.

NTFS is a journaling file system — in the event of a failure, it can automatically repair files and folders using a log file. FAT does not have this capability.

**NTFS advantages over FAT:**

- Supports files larger than 4GB
- Granular permissions on files and folders
- File and folder compression
- Encryption via EFS (Encrypting File System)

---

### NTFS Permissions

On NTFS volumes, permissions can be set to grant or deny access to files and folders. The available permission levels are:

| Permission | Description |
|---|---|
| Full Control | Read, write, modify, delete, and change permissions |
| Modify | Read, write, and delete files and folders |
| Read & Execute | View and run files |
| List Folder Contents | View the contents of a folder |
| Read | View file contents and attributes |
| Write | Create new files and write data |

Permissions can be set at both the folder and individual file level, and they can be inherited from parent folders.

---

### Alternate Data Streams (ADS)

ADS is an NTFS-specific file attribute that allows a file to contain more than one data stream. Every file has at least one data stream (the default content), but additional named streams can be attached to the same file without being visible in standard file listings.

ADS is legitimate and used by Windows itself (e.g., for zone identifiers on downloaded files), but it is also abused by malware to hide payloads within files.

---

### The Windows Directory and Environment Variables

The Windows operating system is traditionally stored in `C:\Windows`, but this can technically be on any drive or folder. The system environment variable `%windir%` always points to the correct location regardless of where Windows is installed.

Environment variables store information about the OS environment — paths, processor count, temp folder locations, and more. They are referenced throughout the system and by applications.

**Common environment variables:**

| Variable | Points To |
|---|---|
| `%windir%` | Windows installation directory |
| `%SystemRoot%` | Same as `%windir%` |
| `%TEMP%` | Temporary files directory |
| `%USERPROFILE%` | Current user's home directory |
| `%ComSpec%` | Path to the command prompt (`cmd.exe`) |

---

### User Account Types

Windows local accounts fall into two categories:

| Account Type | Capabilities |
|---|---|
| Administrator | Full system control — add/remove users, modify groups, install software, change system settings |
| Standard User | Limited to personal files and settings — cannot install software or make system-level changes |

The principle of least privilege applies here: standard users should be used for day-to-day tasks, with administrator accounts reserved for when elevated access is genuinely needed.

---

## Important Terminology

| Term | Definition |
|---|---|
| NTFS | New Technology File System — the default file system for modern Windows |
| FAT32 | Legacy file system still used on removable media |
| Journaling | A file system feature that logs changes to enable recovery after failure |
| ADS | Alternate Data Streams — hidden data streams attached to NTFS files |
| EFS | Encrypting File System — NTFS feature for file-level encryption |
| Environment Variable | A system-wide variable storing OS configuration information |
| `%windir%` | Environment variable pointing to the Windows installation directory |
| Administrator | Windows account type with full system privileges |
| Standard User | Windows account type with limited, user-scoped privileges |

---

## Practical Examples / Demonstrations

### View ADS on a file (PowerShell)

```powershell
# List all streams on a file
Get-Item file.txt -Stream *

# Read a hidden ADS
Get-Content file.txt -Stream hiddenstream
```

### View environment variables (CMD)

```cmd
set
echo %windir%
echo %ComSpec%
```

### Check NTFS permissions on a file (CMD)

```cmd
icacls C:\Users\user\Documents\file.txt
```

---

## Real-World Relevance

- ADS is used by malware to hide payloads inside legitimate files — a common evasion technique that bypasses basic file inspection
- NTFS permissions misconfigurations are a frequent privilege escalation vector — writable directories in privileged paths allow DLL hijacking and binary replacement
- Environment variables are used in malware and scripts to locate system paths without hardcoding them — understanding them helps when analysing malicious scripts
- The distinction between Administrator and Standard User is the foundation of Windows privilege escalation — most attacks aim to move from Standard to Administrator or SYSTEM

---

## Key Learnings

- NTFS is the standard Windows file system — it supports permissions, encryption, compression, and journaling
- NTFS permissions are granular and can be set per file and folder
- ADS allows hidden data to be stored within files — legitimate but frequently abused
- Environment variables like `%windir%` and `%ComSpec%` are used throughout the OS and by applications
- Administrator accounts have full system control; Standard Users are restricted to their own files and settings

---

## Additional Notes

- Zone identifiers (stored as ADS) are added to files downloaded from the internet — this is how Windows knows to show the "Open File - Security Warning" dialog
- `icacls` is the command-line tool for viewing and modifying NTFS permissions
- The `%TEMP%` directory is a common staging location for malware — it's writable by all users

---

## Conclusion

Windows Fundamentals 1 establishes the baseline knowledge needed to work with Windows from a security perspective. NTFS permissions, ADS, environment variables, and user account types are all concepts that appear repeatedly in both offensive (privilege escalation, evasion) and defensive (hardening, forensics) Windows security work.
