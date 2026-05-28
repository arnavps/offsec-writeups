# Linux Fundamentals Part 3

## Overview

The final Linux Fundamentals room covers the practical skills needed to work effectively on a Linux system: text editors, file transfer methods, process management, scheduled tasks with cron, and package management. These are the tools used daily in system administration, post-exploitation, and maintaining persistence on Linux systems.

---

## Topics Covered

- Terminal text editors: nano and vim
- Downloading files with `wget`
- Secure file transfer with `scp`
- Serving files with Python's HTTP server
- Process management: `ps`, `kill`, signals
- Background and foreground processes
- `systemctl` and service management
- Crontabs and scheduled tasks
- Package management with `apt`

---

## Key Concepts

### Terminal Text Editors

**Nano**

The most beginner-friendly terminal text editor.

```bash
nano filename.txt
```

Key shortcuts (use `Ctrl` + letter):

| Shortcut | Action |
|---|---|
| `Ctrl + X` | Exit |
| `Ctrl + O` | Save (write out) |
| `Ctrl + W` | Search |
| `Ctrl + K` | Cut line |
| `Ctrl + U` | Paste |

**Vim**

A more advanced editor with a steeper learning curve but significantly more powerful. Operates in modes (normal, insert, command).

```bash
vim filename.txt
```

TryHackMe has a dedicated Vim room for deeper coverage.

---

### Downloading Files — `wget`

Downloads files from the web over HTTP/HTTPS directly to the current directory.

```bash
wget https://example.com/file.txt
wget https://assets.tryhackme.com/additional/linux-fundamentals/part3/myfile.txt
```

---

### Secure File Transfer — `scp`

SCP (Secure Copy) transfers files between systems over SSH — providing both authentication and encryption.

**Copy a file to a remote system:**
```bash
scp important.txt ubuntu@192.168.1.30:/home/ubuntu/transferred.txt
```

**Copy a file from a remote system:**
```bash
scp ubuntu@192.168.1.30:/home/ubuntu/documents.txt notes.txt
```

Format: `scp <source> <destination>` — works in both directions.

---

### Serving Files — Python HTTP Server

Python's built-in HTTP server module turns any directory into a temporary web server. Useful for transferring files between machines on the same network.

**Start the server (serves current directory on port 8000):**
```bash
python3 -m http.server
```

**Download a file from it on another machine:**
```bash
wget http://10.49.190.81:8000/myfile
```

Run the server in one terminal and wget in another — the server must stay running during the transfer.

---

### Process Management

**View running processes:**
```bash
ps aux
```

Shows all running processes with PID, CPU/memory usage, and the command that started them.

**Kill a process:**
```bash
kill <PID>
kill 1337
```

**Process signals:**

| Signal | Description |
|---|---|
| `SIGTERM` | Graceful termination — allows the process to clean up before exiting |
| `SIGKILL` | Immediate termination — no cleanup |
| `SIGSTOP` | Pause/suspend the process |

```bash
kill -SIGTERM 1337    # graceful
kill -9 1337          # SIGKILL (force kill)
```

---

### Background and Foreground Processes

**Run a command in the background:**
```bash
command &
```

**Suspend a running process (send to background):**
```
Ctrl + Z
```

**Bring a background process back to the foreground:**
```bash
fg
```

**Check background jobs:**
```bash
ps aux
jobs
```

---

### Service Management — `systemctl`

`systemctl` interacts with systemd to manage services.

```bash
systemctl <option> <service>
```

| Option | Description |
|---|---|
| `start` | Start a service |
| `stop` | Stop a service |
| `enable` | Start the service automatically on boot |
| `disable` | Prevent the service from starting on boot |
| `status` | Check the current status of a service |

```bash
systemctl start apache2
systemctl stop apache2
systemctl enable apache2
systemctl status apache2
```

---

### Crontabs — Scheduled Tasks

Cron runs scheduled tasks automatically. The crontab file defines when and what to run.

**Edit the crontab:**
```bash
crontab -e
```

**Crontab field format:**

| Field | Description |
|---|---|
| MIN | Minute (0–59) |
| HOUR | Hour (0–23) |
| DOM | Day of month (1–31) |
| MON | Month (1–12) |
| DOW | Day of week (0–7, 0 and 7 = Sunday) |
| CMD | Command to execute |

`*` = wildcard (any value)

**Example — back up Documents every 12 hours:**
```
0 */12 * * * cp -R /home/user/Documents /var/backups/
```

**Example — run a script every day at 3am:**
```
0 3 * * * /home/user/scripts/backup.sh
```

Use [crontab.guru](https://crontab.guru) to generate and verify crontab expressions.

---

### Package Management — `apt`

`apt` manages software installation, updates, and removal on Debian/Ubuntu systems.

**Install software:**
```bash
sudo apt install <package>
sudo apt install nmap
```

**Remove software:**
```bash
sudo apt remove <package>
```

**Update package lists:**
```bash
sudo apt update
```

**Adding a third-party repository (example: Sublime Text):**

```bash
# 1. Add the GPG key
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -

# 2. Create a sources list file
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list

# 3. Update apt
sudo apt update

# 4. Install
sudo apt install sublime-text
```

GPG keys verify the integrity of packages — if the key doesn't match, the package won't install.

**Remove a repository:**
```bash
sudo add-apt-repository --remove ppa:PPA_Name/ppa
sudo apt remove sublime-text
```

---

## Important Terminology

| Term | Definition |
|---|---|
| `wget` | Command-line tool for downloading files over HTTP/HTTPS |
| `scp` | Secure Copy — file transfer over SSH |
| PID | Process ID — unique identifier for a running process |
| `SIGTERM` | Signal to gracefully terminate a process |
| `SIGKILL` | Signal to immediately kill a process |
| `systemd` | The init system on modern Linux — manages services and processes |
| `systemctl` | Command to interact with systemd |
| Crontab | A file defining scheduled tasks for the cron daemon |
| `apt` | Package manager for Debian/Ubuntu systems |
| GPG key | Cryptographic key used to verify the authenticity of software packages |

---

## Practical Examples / Demonstrations

### Transfer a file to a remote machine

```bash
scp report.txt user@10.10.10.5:/home/user/report.txt
```

### Serve a file and download it on another machine

```bash
# Machine A (server)
python3 -m http.server

# Machine B (client)
wget http://<Machine_A_IP>:8000/file.txt
```

### Find and kill a process

```bash
ps aux | grep apache
kill -SIGTERM 2341
```

### Schedule a daily backup at midnight

```
0 0 * * * cp -R /home/user/data /backup/data
```

### Install and enable a web server

```bash
sudo apt install apache2
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2
```

---

## Workflow / Process

### How systemd Starts Processes

```
System boots
        |
systemd (PID 1) initialises
        |
Services configured with "enable" start automatically
        |
Each service runs as a child process of systemd
        |
systemctl manages start/stop/status of each service
```

---

## Real-World Relevance

- Crontabs are a common persistence mechanism — attackers add cron jobs to maintain access after a reboot
- `python3 -m http.server` is used constantly during CTFs and pentests to transfer tools and payloads between machines
- `scp` is used for secure file exfiltration and tool deployment during authorised engagements
- `ps aux` is one of the first commands run after gaining a shell — it reveals running processes and can expose credentials passed as command-line arguments
- `systemctl enable` is used by attackers to make malicious services persist across reboots
- `/var/log` (managed by running services) is the primary source of evidence during incident response
- `apt` package management is relevant for understanding supply chain attacks — malicious packages in repositories

---

## Key Learnings

- Nano is the go-to beginner text editor; Vim is more powerful but requires learning
- `wget` downloads files; `scp` transfers files securely over SSH; Python HTTP server serves files on the local network
- `ps aux` lists all processes; `kill` terminates them using signals
- `SIGTERM` is graceful; `SIGKILL` is immediate
- `systemctl` manages services — `enable` makes them start on boot
- Crontabs schedule recurring tasks using a 6-field time format
- `apt` manages packages; GPG keys verify package integrity

---

## Additional Notes

- `crontab -l` lists the current user's cron jobs — check this during privilege escalation enumeration
- `/etc/crontab` and `/etc/cron.d/` contain system-wide cron jobs — also worth checking
- `jobs` lists background jobs in the current shell session
- `nohup command &` runs a command that persists even after the terminal is closed
- `apt list --installed` shows all installed packages — useful for identifying outdated software

---

## Conclusion

Linux Fundamentals Part 3 rounds out the core Linux skill set. Text editors, file transfer, process management, scheduled tasks, and package management are the operational tools used in every Linux environment. For security professionals, these aren't just administration skills — they're the techniques used to maintain access, transfer tools, manage persistence, and understand what's running on a compromised system.
