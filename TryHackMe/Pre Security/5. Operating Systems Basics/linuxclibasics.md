# Linux CLI Basics

## Overview

The Linux command-line interface (CLI) is the primary way security professionals interact with Linux systems. Most security tools are terminal-based, remote servers have no GUI, and the CLI provides speed, precision, and scriptability that a graphical interface can't match. This room covers the essential commands for navigating the filesystem, finding files, reading content, and gathering system information.

---

## Topics Covered

- Terminal basics
- Filesystem navigation
- Finding files
- Reading file contents
- System information commands

---

## Key Concepts

### The Terminal

The terminal is a text-based interface for controlling a Linux system. You type commands and the system executes them. Security professionals use it because:

- Faster than navigating a GUI
- Provides direct, precise control
- Most security tools are CLI-only
- Remote systems (servers, VMs) are often accessed without a GUI

---

### Filesystem Navigation

**Where am I?**

```bash
pwd
```

Prints the current working directory (your location in the filesystem).

**What's in this directory?**

```bash
ls
```

Lists files and directories in the current location.

```bash
ls -l
```

Long format — shows permissions, owner, size, and modification date.

```bash
ls -a
```

Shows hidden files (files starting with `.`).

```bash
ls -al
```

Combines both — long format including hidden files.

**Move to a different directory:**

```bash
cd Documents
```

```bash
cd ..
```

Go up one level in the directory tree.

```bash
cd ~
```

Go to the home directory.

---

### Finding Files

```bash
find <starting_point> -name <filename>
```

Recursively searches for a file by name from the specified starting point.

**Example — find a file in the home directory:**

```bash
find ~ -name mission_brief.txt
```

`~` is shorthand for the current user's home directory.

---

### Reading File Contents

```bash
cat filename.txt
```

Prints the full contents of a file to the terminal. Short for "concatenate."

---

### System Information Commands

**Who am I logged in as?**

```bash
whoami
```

Returns the current username.

**System and kernel information:**

```bash
uname -a
```

Returns detailed system information:

```
Linux tryhackme 5.x.x-aws x86_64 GNU/Linux
```

| Field | Meaning |
|---|---|
| `Linux` | Kernel type |
| `tryhackme` | Hostname (machine name) |
| `5.x.x-aws` | Kernel version |
| `x86_64` | CPU architecture (64-bit) |
| `GNU/Linux` | OS type |

```bash
uname
```

Returns just the OS/kernel name.

**Disk and storage usage:**

```bash
df -h
```

Shows disk space usage for all mounted filesystems in human-readable format (`-h`).

---

### Reading System Files

Linux stores configuration and system information files in `/etc`. To explore it:

```bash
cd /etc
ls
```

Many important files live here — network configuration, user accounts (`/etc/passwd`), password hashes (`/etc/shadow`), and more.

---

## Important Terminology

| Term | Definition |
|---|---|
| Terminal | Text-based interface for interacting with the OS |
| Shell | The program that interprets and executes commands (e.g., bash, zsh) |
| Directory | A folder in the Linux filesystem |
| Path | The location of a file or directory (e.g., `/home/user/Documents`) |
| `~` | Shorthand for the current user's home directory |
| Hidden File | A file whose name starts with `.` — not shown by default with `ls` |
| `/etc` | System configuration directory |
| `pwd` | Print Working Directory — shows current location |
| `ls` | List directory contents |
| `cd` | Change Directory |
| `find` | Search for files in the filesystem |
| `cat` | Print file contents to the terminal |
| `whoami` | Display the current logged-in username |
| `uname -a` | Display detailed system and kernel information |
| `df -h` | Display disk usage in human-readable format |

---

## Practical Examples / Demonstrations

### Mini challenge workflow

```bash
# Find a file named day1_report.txt in the home directory
find ~ -name day1_report.txt

# Navigate to the directory where it was found
cd /home/user/some/path/

# Read the file
cat day1_report.txt
```

### Check who you are and what system you're on

```bash
whoami
uname -a
```

### List all files including hidden ones with details

```bash
ls -al
```

---

## Workflow / Process

### General CLI Orientation Workflow

```
Open terminal
        |
pwd  →  confirm current location
        |
ls -al  →  see all files including hidden
        |
cd <directory>  →  navigate to target
        |
find ~ -name <file>  →  locate a specific file
        |
cat <file>  →  read its contents
        |
whoami / uname -a  →  confirm identity and system info
```

---

## Real-World Relevance

- Post-exploitation on Linux systems is done entirely via CLI — navigating filesystems, reading config files, and escalating privileges all use these commands
- `find` is used to locate SUID binaries, world-writable files, and sensitive files during privilege escalation enumeration
- `cat /etc/passwd` and `cat /etc/shadow` are standard commands during Linux privilege escalation to enumerate users and password hashes
- `uname -a` reveals the kernel version — used to identify whether a known kernel exploit applies
- `df -h` and disk analysis commands are used in forensics to understand storage layout
- Hidden files (`.bash_history`, `.ssh/`) often contain sensitive information like command history and SSH keys

---

## Key Learnings

- The terminal is the primary interface for Linux security work
- `pwd`, `ls`, and `cd` are the core navigation commands
- `find` locates files anywhere in the filesystem by name or other attributes
- `cat` reads file contents directly in the terminal
- `whoami` and `uname -a` quickly establish identity and system context
- `/etc` is a critical directory containing system configuration files

---

## Additional Notes

- `less` and `more` are alternatives to `cat` for reading large files page by page
- `grep` searches for patterns within files — essential for filtering output
- Tab completion speeds up command entry significantly
- Command history is stored in `~/.bash_history` — a valuable artefact during forensic investigation and a risk if sensitive commands (passwords) were typed

---

## Conclusion

Linux CLI basics are non-negotiable for anyone in cybersecurity. Whether you're navigating a compromised system, running security tools, or administering a server, these commands are used constantly. Mastering navigation, file discovery, and system enumeration is the foundation everything else builds on.
