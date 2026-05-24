# Windows CLI Basics

## Overview

The Windows Command Prompt (CMD) is the built-in text-based interface for interacting with Windows. While most users rely on the GUI, the CLI is faster, more precise, and essential for security work — many tasks in incident response, forensics, and post-exploitation are performed entirely through the command line. This room covers the core CMD commands for filesystem navigation, file discovery, and system enumeration.

---

## Topics Covered

- Navigating the filesystem with CMD
- Finding files
- Reading file contents
- System and network information commands

---

## Key Concepts

### The Command Prompt

CMD is a text-based interface where you type commands to interact with Windows directly. It's available on every Windows system and is widely used in:

- IT administration and troubleshooting
- Incident response and forensic investigation
- Post-exploitation and lateral movement
- Automation via batch scripts

---

### Filesystem Navigation

**Where am I?**

```cmd
cd
```

Running `cd` with no arguments prints the current directory path.

**What's in this directory?**

```cmd
dir
```

Lists files and folders in the current directory, including sizes and timestamps.

**Show hidden files:**

```cmd
dir /a
```

The `/a` flag shows all files including hidden and system files.

**Move to a different directory:**

```cmd
cd Documents
```

```cmd
cd ..
```

Go up one level.

```cmd
cd \
```

Go to the root of the current drive.

---

### Finding Files

```cmd
dir /s filename.txt
```

Searches recursively from the current directory for a file matching the name.

**Example:**

```cmd
dir /s task_brief.txt
```

The `/s` flag tells `dir` to search all subdirectories.

---

### Reading File Contents

```cmd
type filename.txt
```

Prints the contents of a text file directly to the terminal.

---

### System Information Commands

**Who am I logged in as?**

```cmd
whoami
```

Returns the current username in `domain\username` or `hostname\username` format.

**What is this computer's name?**

```cmd
hostname
```

Returns the computer's hostname.

**Detailed system information:**

```cmd
systeminfo
```

Returns comprehensive information including OS version, build number, installed hotfixes, RAM, and domain membership. Useful for identifying patch levels and system configuration.

**Network configuration:**

```cmd
ipconfig
```

Displays IP address, subnet mask, and default gateway for all network adapters.

```cmd
ipconfig /all
```

Extended output — includes MAC addresses, DNS servers, DHCP status, and lease information.

---

## Important Terminology

| Term | Definition |
|---|---|
| CMD | Command Prompt — Windows' built-in CLI |
| `cd` | Change Directory / print current directory |
| `dir` | List directory contents |
| `dir /a` | List all files including hidden and system files |
| `dir /s` | Recursive file search |
| `type` | Print file contents to the terminal |
| `whoami` | Display the current logged-in username |
| `hostname` | Display the computer's name |
| `systeminfo` | Display detailed OS and hardware information |
| `ipconfig` | Display network adapter configuration |

---

## Practical Examples / Demonstrations

### Full orientation workflow

```cmd
cd
dir /a
dir /s task_brief.txt
type task_brief.txt
whoami
hostname
systeminfo
ipconfig
```

### Find a hidden file and read it

```cmd
dir /a
dir /s secret.txt
type secret.txt
```

---

## Workflow / Process

### CMD Orientation Workflow

```
Open CMD
        |
cd  →  confirm current location
        |
dir /a  →  list all files including hidden
        |
dir /s <filename>  →  locate a specific file recursively
        |
type <filename>  →  read its contents
        |
whoami / hostname / systeminfo  →  enumerate system info
        |
ipconfig  →  check network configuration
```

---

## Real-World Relevance

- `whoami` and `systeminfo` are among the first commands run during post-exploitation to understand the compromised environment
- `ipconfig` reveals network configuration — used to identify the internal network range for lateral movement
- `dir /a` reveals hidden files that may contain sensitive data or malware artefacts
- `dir /s` is used to locate specific files (config files, credentials, flags) across the filesystem
- `systeminfo` reveals the OS version and installed patches — used to identify missing patches and applicable exploits
- CMD is used in many malware payloads and living-off-the-land attacks because it's always present on Windows

---

## Key Learnings

- CMD provides direct, precise control over Windows without a GUI
- `cd` and `dir` are the core navigation commands
- `dir /a` reveals hidden files; `dir /s` searches recursively
- `type` reads file contents in the terminal
- `whoami`, `hostname`, `systeminfo`, and `ipconfig` are the core system enumeration commands
- These skills apply directly to IT administration, incident response, and offensive security

---

## Additional Notes

- PowerShell is the modern successor to CMD and is significantly more powerful — most advanced Windows administration and offensive tooling uses PowerShell
- `net user` lists local user accounts; `net localgroup administrators` lists members of the local admin group — both are standard post-exploitation enumeration commands
- `tasklist` shows running processes (CMD equivalent of Task Manager)
- `netstat -ano` shows active network connections with associated process IDs

---

## Conclusion

CMD is a foundational tool for Windows security work. The ability to navigate the filesystem, locate files, read content, and enumerate system information without a GUI is a skill used constantly in both administration and security contexts. These commands are the starting point for understanding Windows post-exploitation and incident response.
