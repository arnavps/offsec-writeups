# Windows Command Line

## Overview

The Windows Command Prompt (CMD) is the built-in text-based interface for interacting with Windows. While the GUI handles most everyday tasks, the CLI is faster, more scriptable, and essential for remote administration and security work. This room covers system information, network commands, filesystem navigation, file management, process management, and utility commands.

---

## Topics Covered

- System information commands
- Network configuration and troubleshooting
- Filesystem navigation
- File management
- Process management
- Utility commands and help

---

## Key Concepts

### Why Use the CLI?

- Faster than navigating a GUI for repetitive tasks
- Lower resource usage — runs on older or limited hardware
- Easier automation via batch scripts
- Essential for remote management over SSH on slow connections

---

### System Information

```cmd
ver                  # Display Windows OS version
systeminfo           # Detailed system info: OS, hardware, memory, network
set                  # Display all environment variables including PATH
```

---

### Network Configuration

```cmd
ipconfig             # Show IP address, subnet mask, default gateway
ipconfig /all        # Full network details: MAC address, DNS, DHCP status
```

**Troubleshooting:**

```cmd
ping example.com                  # Test connectivity using ICMP echo
tracert example.com               # Trace the network route to a target
nslookup example.com              # Resolve a domain to its IP address
nslookup example.com 1.1.1.1      # Resolve using a specific DNS server
```

**Active connections:**

```cmd
netstat               # Show established connections
netstat -a            # All connections and listening ports
netstat -b            # Show the program associated with each connection
netstat -o            # Show the PID associated with each connection
netstat -n            # Use numerical addresses and port numbers
netstat -abon         # Combine all of the above
```

---

### Filesystem Navigation

```cmd
cd                    # Print current directory (where am I?)
cd target_directory   # Change to a directory
cd ..                 # Go up one level
dir                   # List files and folders in current directory
dir /a                # Include hidden and system files
dir /s                # Include all subdirectories recursively
tree                  # Visual representation of directory structure
mkdir directory_name  # Create a new directory
rmdir directory_name  # Remove a directory
```

---

### File Management

```cmd
type file.txt         # Display contents of a text file
more file.txt         # Display file contents one page at a time
some_command | more   # Pipe long output through more for paging
copy source dest      # Copy a file
move source dest      # Move a file
del file.txt          # Delete a file
erase file.txt        # Delete a file (alias for del)
```

---

### Process Management

```cmd
tasklist                                    # List all running processes
tasklist /FI "imagename eq sshd.exe"        # Filter by process name
taskkill /PID 4567                          # Kill a process by PID
```

---

### Utility Commands

```cmd
chkdsk                # Check file system and disk for errors and bad sectors
driverquery           # List installed device drivers
sfc /scannow          # Scan and repair corrupted system files
shutdown /s           # Shut down the system
shutdown /r           # Restart the system
shutdown /a           # Abort a scheduled shutdown
<command> /?          # Display help for any command
```

---

## Important Terminology

| Term | Definition |
|---|---|
| CLI | Command-Line Interface — text-based system interaction |
| `ipconfig` | Displays network adapter configuration |
| `ping` | Tests connectivity using ICMP echo packets |
| `tracert` | Traces the network route to a destination |
| `nslookup` | Resolves domain names to IP addresses |
| `netstat` | Displays active network connections and listening ports |
| `tasklist` | Lists running processes |
| `taskkill` | Terminates a process by PID or name |
| `systeminfo` | Displays detailed OS and hardware information |
| TTL | Time to Live — limits how long a packet can exist on the network |
| PID | Process ID — unique identifier for a running process |

---

## Practical Examples / Demonstrations

### Find your IP address

```cmd
ipconfig
```

### Check if a host is reachable

```cmd
ping 8.8.8.8
ping example.com
```

### Trace the route to a target

```cmd
tracert example.com
```

### View all active connections with process info

```cmd
netstat -abon
```

### Find processes related to a specific executable

```cmd
tasklist /FI "imagename eq notepad.exe"
```

### Kill a process

```cmd
taskkill /PID 1234
```

### View a file page by page

```cmd
more C:\Windows\System32\drivers\etc\hosts
type largefile.txt | more
```

---

## Real-World Relevance

- `ipconfig`, `netstat`, and `systeminfo` are among the first commands run during post-exploitation to enumerate the environment
- `netstat -abon` reveals active connections and the processes behind them — used to detect backdoors and C2 connections
- `tasklist` and `taskkill` are used to identify and terminate security tools (AV, EDR) during offensive engagements
- `tracert` and `ping` are used in network troubleshooting and basic reachability testing during reconnaissance
- `nslookup` is used for DNS enumeration and verifying DNS resolution
- Batch scripts using these commands automate reconnaissance and enumeration tasks

---

## Key Learnings

- CMD provides direct, fast access to system information, network state, and process management
- `ipconfig /all` gives the full network picture including MAC, DNS, and DHCP
- `netstat -abon` is the most useful netstat variant — shows connections, programs, and PIDs
- `tasklist` with `/FI` filters processes by name; `taskkill /PID` terminates them
- `/?` works with almost every command to display its help page
- `more` is used both to read files and to page through long command output

---

## Additional Notes

- `netstat -b` requires administrator privileges
- `sfc /scannow` must be run as administrator
- `shutdown /a` only works if a shutdown has been scheduled — useful for cancelling accidental or malicious shutdowns
- PowerShell has more powerful equivalents for most of these commands — CMD remains relevant for compatibility and simplicity

---

## Conclusion

The Windows Command Line is a foundational tool for both administration and security work. The commands covered here — network diagnostics, process management, filesystem navigation, and system information — are used constantly in incident response, post-exploitation enumeration, and day-to-day administration. Knowing them well and knowing how to combine them efficiently is a practical skill that applies across every Windows security scenario.
