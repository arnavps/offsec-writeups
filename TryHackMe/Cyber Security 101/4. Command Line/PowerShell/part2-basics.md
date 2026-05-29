# PowerShell — Part 2: Basics

## Overview

This part covers the core PowerShell cmdlets for filesystem navigation and file management. PowerShell provides a unified set of commands for working with both files and directories, replacing the multiple separate commands used in CMD.

---

## Topics Covered

- Listing directory contents with `Get-ChildItem`
- Navigating directories with `Set-Location`
- Creating items with `New-Item`
- Removing items with `Remove-Item`
- Copying and moving with `Copy-Item` and `Move-Item`
- Reading file contents with `Get-Content`

---

## Key Concepts

### Filesystem Navigation

**`Get-ChildItem`** — lists files and directories. Equivalent to `dir` (CMD) or `ls` (Unix).

```powershell
Get-ChildItem                          # List current directory
Get-ChildItem -Path C:\Users           # List a specific path
Get-ChildItem -Path C:\Users -Recurse  # Recurse into subdirectories
```

**`Set-Location`** — changes the current working directory. Equivalent to `cd`.

```powershell
Set-Location C:\Users
Set-Location ..                        # Go up one level
```

---

### File and Directory Management

PowerShell uses a single set of cmdlets for both files and directories — unlike CMD which has separate commands for each.

**`New-Item`** — creates a file or directory.

```powershell
New-Item -Path C:\temp\notes.txt -ItemType File
New-Item -Path C:\temp\logs -ItemType Directory
```

**`Remove-Item`** — deletes files and directories (replaces both `del` and `rmdir`).

```powershell
Remove-Item -Path C:\temp\notes.txt
Remove-Item -Path C:\temp\logs -Recurse    # Required for non-empty directories
```

**`Copy-Item`** — copies files and directories. Equivalent to `copy`.

```powershell
Copy-Item -Path notes.txt -Destination notes_backup.txt
Copy-Item -Path C:\source -Destination C:\dest -Recurse
```

**`Move-Item`** — moves or renames files and directories. Equivalent to `move`.

```powershell
Move-Item -Path notes.txt -Destination C:\archive\notes.txt   # Move
Move-Item -Path notes.txt -Destination renamed.txt             # Rename
```

**`Get-Content`** — reads and displays file contents. Equivalent to `type` (CMD) or `cat` (Unix).

```powershell
Get-Content -Path C:\Users\user\file.txt
Get-Content notes.txt
```

---

## Important Terminology

| Term | Definition |
|---|---|
| `Get-ChildItem` | Lists files and directories at a given path |
| `Set-Location` | Changes the current working directory |
| `New-Item` | Creates a new file or directory |
| `Remove-Item` | Deletes a file or directory |
| `Copy-Item` | Copies a file or directory |
| `Move-Item` | Moves or renames a file or directory |
| `Get-Content` | Reads and outputs the contents of a file |
| `-Path` | Parameter specifying the target file or directory path |
| `-Recurse` | Flag to process subdirectories recursively |
| `-ItemType` | Specifies whether to create a `File` or `Directory` |

---

## Practical Examples / Demonstrations

```powershell
# List contents of C:\Users
Get-ChildItem -Path C:\Users

# Navigate to a directory
Set-Location C:\Windows\System32

# Create a file and a directory
New-Item -Path C:\temp\test.txt -ItemType File
New-Item -Path C:\temp\testdir -ItemType Directory

# Copy a file
Copy-Item -Path C:\temp\test.txt -Destination C:\temp\test_backup.txt

# Move and rename a file
Move-Item -Path C:\temp\test.txt -Destination C:\archive\test_renamed.txt

# Read a file
Get-Content -Path C:\temp\test_backup.txt

# Delete a file
Remove-Item -Path C:\temp\test_backup.txt
```

---

## CMD vs PowerShell Equivalents

| Task | CMD | PowerShell |
|---|---|---|
| List directory | `dir` | `Get-ChildItem` |
| Change directory | `cd` | `Set-Location` |
| Read file | `type` | `Get-Content` |
| Copy file | `copy` | `Copy-Item` |
| Move file | `move` | `Move-Item` |
| Delete file | `del` | `Remove-Item` |
| Remove directory | `rmdir` | `Remove-Item -Recurse` |
| Create file/dir | `echo >` / `mkdir` | `New-Item` |

---

## Real-World Relevance

- `Get-ChildItem -Recurse` is used during post-exploitation to enumerate filesystem contents and locate sensitive files
- `Get-Content` is used to read configuration files, credentials, and logs
- `Copy-Item` and `Move-Item` are used to stage and exfiltrate files during offensive engagements
- `Remove-Item` is used by attackers to clean up artefacts and by defenders to remove malicious files
- Full path support (`-Path`) makes these cmdlets suitable for scripting and automation across complex directory structures

---

## Key Learnings

- PowerShell uses unified cmdlets for both files and directories — `New-Item`, `Remove-Item`, `Copy-Item`, `Move-Item`
- `Get-ChildItem` replaces `dir`; `Set-Location` replaces `cd`; `Get-Content` replaces `type`
- `-Recurse` is required when operating on non-empty directories
- `-ItemType File` or `-ItemType Directory` specifies what `New-Item` creates
- All cmdlets support full path specification via `-Path`

---

## Conclusion

PowerShell's filesystem cmdlets provide a consistent, unified interface for navigating and managing files and directories. They are more flexible than their CMD counterparts and integrate naturally with PowerShell's object pipeline, making them the foundation for more advanced data manipulation covered in subsequent parts.
