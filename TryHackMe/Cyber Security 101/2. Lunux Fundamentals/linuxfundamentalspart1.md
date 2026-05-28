# Linux Fundamentals Part 1

## Overview

Linux is everywhere — web servers, embedded systems, critical infrastructure, and the majority of cybersecurity tooling runs on it. This room introduces the Linux command line from scratch: basic commands, filesystem navigation, file searching, and shell operators. These are the foundational skills required for every Linux-based security task.

---

## Topics Covered

- Where Linux is used
- Basic commands: `echo`, `whoami`
- Filesystem navigation: `ls`, `cd`, `cat`, `pwd`
- Finding files with `find`
- Searching file contents with `grep`
- Shell operators: `&`, `&&`, `>`, `>>`

---

## Key Concepts

### Where Linux Is Used

Linux is far more prevalent than most people realise. It powers:

- The majority of websites and web servers
- Car entertainment and control systems
- Point of Sale (PoS) systems
- Critical infrastructure (traffic controllers, industrial sensors)
- Most cybersecurity tools and platforms

---

### Basic Commands

| Command | Description |
|---|---|
| `echo` | Output text to the terminal |
| `whoami` | Display the current logged-in username |

```bash
echo "Hello World"
whoami
```

---

### Filesystem Navigation

| Command | Full Name | Description |
|---|---|---|
| `ls` | listing | List files and directories in the current location |
| `cd` | change directory | Move to a different directory |
| `cat` | concatenate | Print the contents of a file |
| `pwd` | print working directory | Show the current directory path |

```bash
ls
cd Documents
cat note.txt
pwd
```

---

### Finding Files — `find`

The `find` command searches the filesystem for files matching specified criteria.

**Basic syntax:**
```bash
find <starting_point> -name <filename>
```

**Examples:**
```bash
# Find a file by name starting from the home directory
find / -name passwords.txt

# Find all .txt files
find / -name "*.txt"

# Find in home directory
find ~ -name mission_brief.txt
```

---

### Searching File Contents — `grep`

`grep` searches inside files for lines matching a pattern.

```bash
grep <pattern> <file>
```

**Examples:**
```bash
# Search for "THM" in access.log
grep THM /home/tryhackme/access.log

# Case-insensitive search
grep -i "error" /var/log/syslog
```

**Recursive search** — search all files in a directory and its subdirectories:

```bash
grep -R "password" /etc/
```

The `-R` flag tells grep to descend into subdirectories rather than searching a single file.

---

### Shell Operators

Operators extend what you can do with commands in the terminal.

| Operator | Description |
|---|---|
| `&` | Run a command in the background |
| `&&` | Run the second command only if the first succeeds |
| `>` | Redirect output to a file (overwrites existing content) |
| `>>` | Redirect output to a file (appends to existing content) |

**`&` — Background execution**
```bash
cp large_file.iso /backup/ &
```
Runs the copy in the background, freeing the terminal for other commands.

**`&&` — Chained commands**
```bash
mkdir logs && cd logs
```
Creates the directory, then changes into it — but only if `mkdir` succeeds.

**`>` — Output redirection (overwrite)**
```bash
echo "hello" > welcome.txt
```
Creates `welcome.txt` with the content "hello". If the file already exists, it is overwritten.

**`>>` — Output redirection (append)**
```bash
echo "world" >> welcome.txt
```
Adds "world" to the end of `welcome.txt` without removing existing content.

---

## Important Terminology

| Term | Definition |
|---|---|
| Shell | The program that interprets and executes commands (e.g., bash) |
| Directory | A folder in the Linux filesystem |
| `~` | Shorthand for the current user's home directory |
| Recursive | Descending into subdirectories automatically |
| Redirect | Sending command output to a file instead of the terminal |
| Background process | A command running without occupying the terminal |

---

## Practical Examples / Demonstrations

### Navigate and read a file

```bash
pwd                          # Where am I?
ls                           # What's here?
cd /home/tryhackme           # Move to directory
cat access.log               # Read the file
```

### Find and read a specific file

```bash
find / -name secret.txt 2>/dev/null
cat /path/to/secret.txt
```

### Search a log file for a flag

```bash
grep THM /home/tryhackme/access.log
```

### Chain commands and redirect output

```bash
echo "scan started" > scan.log && nmap -sV 10.10.10.10 >> scan.log
```

---

## Real-World Relevance

- `find` is used during privilege escalation to locate SUID binaries, world-writable files, and sensitive configuration files
- `grep -R` is used to search for credentials, API keys, and sensitive strings across entire directory trees
- Shell operators are used in one-liners during post-exploitation and in automation scripts
- Output redirection (`>`, `>>`) is used to save command output to files for later analysis or exfiltration
- `cat` on files like `/etc/passwd`, `/etc/shadow`, and config files is a standard enumeration step

---

## Key Learnings

- Linux powers most of the internet and the majority of security tooling
- `ls`, `cd`, `cat`, and `pwd` are the core navigation commands
- `find` locates files anywhere in the filesystem by name or attribute
- `grep` searches file contents for patterns; `-R` makes it recursive
- Shell operators (`&`, `&&`, `>`, `>>`) extend command functionality significantly

---

## Additional Notes

- `2>/dev/null` suppresses error messages (e.g., "permission denied") when running `find` as a non-root user
- `ls -la` shows hidden files and detailed permissions — use this instead of plain `ls`
- `cat` is fine for small files; use `less` for large files to avoid flooding the terminal

---

## Conclusion

Linux CLI fundamentals are the entry point to everything in cybersecurity. Navigation, file discovery, content searching, and shell operators are used constantly — in CTFs, penetration tests, incident response, and daily administration. These commands are not optional knowledge; they're the baseline.
