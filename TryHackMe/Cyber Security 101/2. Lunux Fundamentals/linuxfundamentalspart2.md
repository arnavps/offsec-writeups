# Linux Fundamentals Part 2

## Overview

Building on Part 1, this room covers flags and switches, file management commands, user switching, Linux file permissions, and the key filesystem directories. These concepts are essential for understanding how Linux systems are structured, how access is controlled, and how attackers and defenders interact with the filesystem.

---

## Topics Covered

- Flags and switches (`-a`, `--help`)
- Man pages
- File management: `touch`, `mkdir`, `cp`, `mv`, `rm`, `file`
- Switching users with `su`
- Linux file permissions (symbolic and numeric)
- Key filesystem directories: `/etc`, `/var`, `/root`, `/tmp`

---

## Key Concepts

### Flags and Switches

Commands have default behaviour, but flags modify that behaviour.

```bash
ls          # default — shows visible files only
ls -a       # shows hidden files (files starting with .)
ls -la      # long format + hidden files
```

Most commands also support `--help`:

```bash
ls --help
```

This lists all available options with brief descriptions — useful when you don't want to open the full man page.

### Man Pages

```bash
man <command>
```

The manual page for any command. Covers all flags, syntax, and examples. Available on the machine itself and online.

```bash
man ls
man find
man chmod
```

---

### File Management Commands

| Command | Full Name | Purpose |
|---|---|---|
| `touch` | touch | Create an empty file |
| `mkdir` | make directory | Create a folder |
| `cp` | copy | Copy a file or folder |
| `mv` | move | Move or rename a file or folder |
| `rm` | remove | Delete a file or folder |
| `file` | file | Determine the type of a file |

**Creating files and directories:**
```bash
touch note.txt
mkdir logs
```

**Copying files:**
```bash
cp note.txt note_backup.txt
```

**Moving and renaming:**
```bash
mv note.txt /home/user/Documents/note.txt   # move
mv note2.txt note3.txt                       # rename
```

**Removing files and directories:**
```bash
rm file.txt
rm -R directory/        # -R required to remove directories
```

**Determining file type:**
```bash
file note
```

Files don't always have extensions. `file` inspects the file's content to determine its actual type — useful when a file's purpose isn't obvious from its name.

---

### Switching Users — `su`

```bash
su <username>
su -l <username>
```

- `su` switches to another user account (requires their password unless you're root)
- `-l` or `--login` starts a login shell — inherits the target user's environment variables and drops into their home directory

```bash
su user2          # switch but stay in current directory
su -l user2       # switch and move to user2's home directory
```

---

### Linux File Permissions

Every file and directory has permissions controlling who can read, write, or execute it.

**Symbolic format:**
```
rwxrwxrwx
```

Split into three groups:

| Section | Applies To |
|---|---|
| First 3 chars | Owner |
| Next 3 chars | Group |
| Last 3 chars | Others |

Each character:
- `r` = read (4)
- `w` = write (2)
- `x` = execute (1)
- `-` = no permission (0)

**Numeric (octal) format:**

Add the values for each group:

| Symbolic | Numeric | Meaning |
|---|---|---|
| `rwxrwxrwx` | 777 | Everyone has full access |
| `rwxr-xr-x` | 755 | Owner full, others read + execute |
| `rw-r--r--` | 644 | Owner read/write, others read only |
| `rwx------` | 700 | Owner only, no access for others |

**Changing permissions:**
```bash
chmod 755 script.sh
chmod 644 config.txt
chmod 700 private/
```

Understanding permissions is critical for identifying security misconfigurations — world-writable files and SUID binaries are common privilege escalation vectors.

---

### Key Filesystem Directories

**`/etc`**
Stores system configuration files. Key files:

- `/etc/passwd` — user account information
- `/etc/shadow` — hashed passwords (root-readable only)
- `/etc/sudoers` — defines which users can run sudo and what commands

**`/var`**
Variable data — content that changes frequently:

- `/var/log` — log files from services and applications
- `/var/www` — web server files (Apache/Nginx default)

**`/root`**
Home directory for the root user. Not `/home/root` — just `/root`.

**`/tmp`**
Temporary files. Volatile — contents are cleared on reboot. World-writable by default, making it a common staging area for attackers.

---

## Important Terminology

| Term | Definition |
|---|---|
| Flag / Switch | An argument passed to a command to modify its behaviour |
| Hidden file | A file whose name starts with `.` — not shown by default |
| `su` | Switch User — change to another user account |
| Permissions | Rules controlling read, write, and execute access to files |
| Owner | The user who owns a file |
| Group | A set of users who share access permissions |
| `chmod` | Change file permissions |
| `/etc` | System configuration directory |
| `/var/log` | Directory containing service and application log files |
| `/tmp` | Temporary directory — cleared on reboot, world-writable |

---

## Practical Examples / Demonstrations

### List all files including hidden

```bash
ls -la
```

### Create a directory and file, then copy

```bash
mkdir project
touch project/notes.txt
cp project/notes.txt project/notes_backup.txt
```

### Check file type

```bash
file suspicious_binary
file document
```

### Switch to another user with full login environment

```bash
su -l www-data
```

### Set secure permissions on a script

```bash
chmod 700 privkey.sh      # only owner can read/write/execute
chmod 644 readme.txt      # owner read/write, everyone else read only
```

### Read sensitive system files (as root)

```bash
cat /etc/shadow
cat /etc/sudoers
```

---

## Real-World Relevance

- `/etc/shadow` is a primary target during privilege escalation — it contains hashed passwords that can be cracked offline
- `/etc/sudoers` misconfigurations are one of the most common privilege escalation vectors
- `/tmp` is world-writable and frequently used by attackers to stage payloads and scripts
- SUID binaries (files with the `s` bit set) run with the owner's privileges — finding them with `find / -perm -4000` is a standard escalation enumeration step
- `/var/log` is the first place defenders look during incident response — access logs, auth logs, and application logs all live here
- `file` is used during forensics to identify files that have been renamed to disguise their type

---

## Key Learnings

- Flags modify command behaviour; `--help` lists available options; `man` provides full documentation
- `touch`, `mkdir`, `cp`, `mv`, `rm`, and `file` cover all basic file management operations
- `su -l` switches users with a full login environment
- Linux permissions use symbolic (`rwx`) and numeric (octal) notation — `chmod` applies them
- `/etc`, `/var`, `/root`, and `/tmp` are the most security-relevant directories on a Linux system

---

## Additional Notes

- `rm -R` is irreversible — there's no recycle bin on Linux
- `ls -la` should be your default `ls` — it shows everything including hidden files and permissions
- `stat <file>` shows detailed file metadata including permissions, owner, and timestamps
- World-writable directories (`chmod 777`) are a security risk — anything can be written there

---

## Conclusion

File management, user switching, and permissions are the operational layer of Linux security. Understanding who owns what, who can access it, and how to change that is fundamental to both privilege escalation (finding weak permissions to exploit) and hardening (ensuring permissions are correctly set). The key directories — `/etc`, `/var`, `/root`, `/tmp` — are the first places both attackers and defenders look.
