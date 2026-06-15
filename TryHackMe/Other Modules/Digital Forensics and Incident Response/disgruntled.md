# Disgruntled

## Overview

Disgruntled is a practical Linux forensics challenge room on TryHackMe. The scenario involves investigating a Linux system to identify suspicious activity, likely by a disgruntled employee or insider threat. The investigation uses Linux forensic artifacts — log files, bash history, cron jobs, authentication records, and file system traces — to reconstruct what occurred.

---

## Key Concepts Applied

This room is a practical application of the Linux forensics techniques covered in the Linux Forensics writeup. The investigation focuses on:

- User account and authentication logs
- Bash command history
- Sudo usage from auth logs
- Cron job configuration
- File modification evidence
- Service or persistence mechanism analysis

---

## Investigation Approach

### Step 1 — Establish User Context

```bash
cat /etc/passwd          # User accounts on the system
cat /etc/group           # Group memberships
last -f /var/log/wtmp    # Login history
last -f /var/log/btmp    # Failed login attempts
```

### Step 2 — Review Authentication and Sudo Activity

```bash
cat /var/log/auth.log* | grep -i COMMAND | tail -50
cat /var/log/auth.log* | head -100
```

Authentication logs record all `sudo` executions with the exact command, timestamp, and user — critical for insider threat investigation.

### Step 3 — Examine Bash History

```bash
cat /home/<username>/.bash_history
cat /root/.bash_history
```

Each user's bash history is stored separately. The root history is especially important if the suspect escalated privileges.

### Step 4 — Check Cron Jobs for Persistence

```bash
cat /etc/crontab
ls /etc/cron.d/
crontab -l -u <username>
```

Insider threats commonly add cron jobs to maintain access or schedule destructive actions.

### Step 5 — Check Shell Startup Files

```bash
cat /home/<username>/.bashrc
cat /home/<username>/.bash_profile
cat /root/.bashrc
```

Persistence commands or malicious aliases may be hidden in startup scripts.

### Step 6 — Check for Modified Files

```bash
find / -newer /var/log/auth.log -type f 2>/dev/null
find /home -mtime -1 -type f
```

Finding files modified during the suspicious timeframe narrows the investigation.

### Step 7 — Review System Logs

```bash
cat /var/log/syslog* | grep -i <suspicious_keyword>
cat /var/log/auth.log* | grep -i failed
```

---

## Key Artifact Locations

| Artifact | Location |
|----------|----------|
| User accounts | `/etc/passwd`, `/etc/shadow` |
| Group info | `/etc/group` |
| Login history | `/var/log/wtmp` (use `last`) |
| Failed logins | `/var/log/btmp` (use `last`) |
| Auth events + sudo | `/var/log/auth.log` |
| Bash history | `~/.bash_history`, `/root/.bash_history` |
| Shell startup scripts | `~/.bashrc`, `~/.bash_profile` |
| Cron jobs | `/etc/crontab`, `/etc/cron.d/`, per-user crontabs |
| Syslog | `/var/log/syslog` |
| Vim file history | `~/.viminfo` |

---

## Tools Used

| Tool / Command | Purpose |
|----------------|---------|
| `cat`, `grep`, `tail` | Read and filter log files |
| `last` | Read binary wtmp/btmp login logs |
| `find` | Locate recently modified files |
| `ls -la` | Check file timestamps and permissions |

---

## Notes

This is a challenge room — specific answers and flags are not documented here, as the investigation is designed to develop hands-on Linux forensics skills. Apply the Linux forensics techniques and artifact locations from the Linux Forensics writeup to work through the questions systematically.

---

## Key Learnings

- Linux auth logs provide a complete sudo command history — critical for insider threat investigation
- Bash history is per-user and stored in the home directory — always check the root user separately
- Cron jobs and `.bashrc` are the primary Linux persistence mechanisms to check
- File modification timestamps help narrow the investigation to a specific timeframe
- Correlating evidence across auth logs, bash history, and cron jobs builds a complete timeline of suspicious activity
