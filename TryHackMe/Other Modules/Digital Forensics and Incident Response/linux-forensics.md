# Linux Forensics

## Overview

Linux systems are ubiquitous as web servers, database servers, cloud infrastructure, and IoT devices. Performing forensic analysis on Linux requires knowing where the OS stores configuration, user activity, logs, and execution evidence. This room covers the key locations and commands for gathering system information, user accounts, network configuration, running processes, execution history, and log files from a Linux host.

---

## Topics Covered

- OS release and system information
- User account and group files
- Login and authentication logs
- Network configuration and active connections
- Running processes
- Cron jobs and service startup
- Evidence of execution: sudo history, bash history, vim history
- Log files: syslog, auth logs, third-party logs

---

## Key Concepts

### OS and System Information

**OS release:**
```bash
cat /etc/os-release
```

**Hostname:**
```bash
cat /etc/hostname
```

**Timezone:**
```bash
cat /etc/timezone
```

---

### User Accounts and Groups

**User accounts** are stored in `/etc/passwd` — a colon-separated file with seven fields:

```
username:password_info:uid:gid:description:home_dir:default_shell
```

- `x` in the password field means the actual hash is in `/etc/shadow`
- User-created accounts typically have UID ≥ 1000

```bash
cat /etc/passwd | column -t -s :
```

**Group information:**
```bash
cat /etc/group
```

---

### Login and Authentication Logs

**`/var/log/wtmp`** — historical login records (successful)
**`/var/log/btmp`** — failed login attempts

Both are binary files — read with the `last` utility:

```bash
last -f /var/log/wtmp
last -f /var/log/btmp
```

**Authentication log** — all authentication events (sudo usage, SSH logins, PAM events):
```bash
cat /var/log/auth.log
```

---

### Network Configuration

**Network interfaces:**
```bash
cat /etc/network/interfaces
```

**IP addresses and MAC addresses (live system):**
```bash
ip address show
```

**Active network connections (live system):**
```bash
netstat -natp
```

Useful flags: `-n` (numeric IPs), `-a` (all connections), `-t` (TCP), `-p` (show process)

**DNS resolution configuration:**
```bash
cat /etc/resolv.conf
```

**Local hostname/IP mappings:**
```bash
cat /etc/hosts
```

---

### Running Processes (Live System)

```bash
ps aux
```

Fields: USER, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME, COMMAND

---

### Cron Jobs

Scheduled tasks run periodically — a common persistence mechanism for attackers.

**System-wide crontab:**
```bash
cat /etc/crontab
```

Contains: time interval, username, command/script to run.

**Per-user cron directories:**
```
/etc/cron.d/
/etc/cron.daily/
/etc/cron.weekly/
```

---

### Service Startup

Services configured to start at boot:
```bash
ls /etc/init.d/
```

---

### Shell Startup Files (.bashrc)

`.bashrc` runs whenever a bash shell is spawned — an attacker can add persistence commands here.

```bash
cat /home/<username>/.bashrc
cat /root/.bashrc
```

---

## Evidence of Execution

### Sudo Execution History

All commands run with `sudo` are logged to `/var/log/auth.log`:

```bash
cat /var/log/auth.log* | grep -i COMMAND | tail
```

### Bash History

Commands run without `sudo` are stored in each user's bash history file:

```bash
cat ~/.bash_history
cat /root/.bash_history
```

Always check the root user's history separately.

### Vim History

The `.viminfo` file in each user's home directory logs files opened in Vim:

```bash
cat ~/.viminfo
```

Contains: command history, search history, file paths of recently opened files.

---

## Log Files

Linux logs are generally stored in `/var/log/`.

### Syslog

System-level activity log — detail level depends on the configured logging level:

```bash
cat /var/log/syslog* | head
```

Use `tail`, `head`, `more`, or `less` for large files.

### Auth Log

Authentication and authorisation events:

```bash
cat /var/log/auth.log* | head
```

### Third-Party Application Logs

Web server, database, and other application logs:

```bash
ls /var/log/
cat /var/log/apache2/access.log   # Apache web server
cat /var/log/nginx/access.log     # Nginx web server
cat /var/log/mysql/error.log      # MySQL database
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| `/etc/passwd` | User account information file — one line per user |
| `/etc/shadow` | Stores hashed user passwords — accessible only by root |
| `/etc/group` | Group membership information |
| `wtmp` | Binary log of successful logins — read with `last` |
| `btmp` | Binary log of failed login attempts — read with `last` |
| `auth.log` | Authentication events including sudo usage and SSH logins |
| `.bashrc` | Shell startup script — executed when bash launches |
| `bash_history` | Per-user record of commands run in bash |
| `.viminfo` | Vim editor history including recently opened files |
| Crontab | Scheduled task configuration file |
| `syslog` | General system activity log |

---

## Forensic Locations Quick Reference

| Information Needed | Location |
|-------------------|----------|
| OS version | `/etc/os-release` |
| Hostname | `/etc/hostname` |
| Timezone | `/etc/timezone` |
| User accounts | `/etc/passwd` |
| Password hashes | `/etc/shadow` |
| Group memberships | `/etc/group` |
| Successful logins | `/var/log/wtmp` (read with `last`) |
| Failed logins | `/var/log/btmp` (read with `last`) |
| Auth events | `/var/log/auth.log` |
| Network interfaces | `/etc/network/interfaces` |
| DNS servers | `/etc/resolv.conf` |
| Host file | `/etc/hosts` |
| Scheduled tasks | `/etc/crontab`, `/etc/cron.d/` |
| Services at startup | `/etc/init.d/` |
| Shell persistence | `~/.bashrc`, `/root/.bashrc` |
| Sudo history | `/var/log/auth.log` (grep COMMAND) |
| Bash command history | `~/.bash_history` |
| Vim file history | `~/.viminfo` |
| System logs | `/var/log/syslog` |
| Application logs | `/var/log/<application>/` |

---

## Real-World Relevance

- Bash history and auth logs are the first places analysts look when investigating a Linux compromise — they provide a chronological record of what commands were run and by whom
- `/etc/crontab` and `.bashrc` are the most common persistence locations on Linux servers — attackers add entries to run malware on schedule or every time a shell opens
- `wtmp` and `btmp` provide login history even if the attacker cleared bash history — two independent evidence sources are harder to fully clean
- Web server access logs (`/var/log/apache2/access.log`) are critical during web application incident response — they record every HTTP request, enabling reconstruction of the attack
- The `/proc` filesystem on a live system provides real-time process, memory, and network information — useful alongside `ps` and `netstat` for live triage

---

## Key Learnings

- Linux stores forensic artifacts across `/etc`, `/var/log`, and user home directories
- User account data is in `/etc/passwd` and `/etc/group`; password hashes are in `/etc/shadow`
- Login history uses binary formats — use the `last` utility for `wtmp` and `btmp`
- `auth.log` captures all authentication and sudo events
- Bash history (`~/.bash_history`) and vim history (`~/.viminfo`) record user and attacker command activity
- Crontab (`/etc/crontab`) and `.bashrc` are primary persistence locations
- Syslog and application-specific logs in `/var/log/` are key sources for timeline reconstruction

---

## Conclusion

Linux forensics follows the same principles as Windows forensics — find, preserve, and analyse the artifacts that record system and user activity. The key difference is the location and format of those artifacts: configuration files in `/etc`, binary logs in `/var/log`, and per-user histories in home directories. Knowing these locations and the tools to read them enables rapid triage and thorough investigation on any Linux system.
