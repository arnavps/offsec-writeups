# PowerShell — Part 5: Real-Time System Analysis

## Overview

Beyond static system information, PowerShell provides cmdlets for monitoring dynamic system state — running processes, active services, live network connections, and file integrity. These are the tools used in incident response, threat hunting, and malware analysis to understand what a system is doing right now.

---

## Topics Covered

- Process monitoring with `Get-Process`
- Service enumeration with `Get-Service`
- Active network connections with `Get-NetTCPConnection`
- File hashing with `Get-FileHash`
- Alternate Data Streams (ADS) detection

---

## Key Concepts

### Process Monitoring — `Get-Process`

Lists all currently running processes with CPU usage, memory consumption, and process details.

```powershell
Get-Process
Get-Process -Name svchost
Get-Process | Sort-Object -Property CPU -Descending | Select-Object -First 10
```

More detailed than `tasklist` — returns objects with properties like `Id`, `Name`, `CPU`, `WorkingSet`, `Path`.

---

### Service Enumeration — `Get-Service`

Lists all services on the system and their current state (Running, Stopped, Paused).

```powershell
Get-Service
Get-Service | Where-Object -Property Status -eq "Running"
Get-Service | Where-Object -Property DisplayName -like "*update*"
```

Used in troubleshooting and forensics to identify anomalous or tampered services.

---

### Active Network Connections — `Get-NetTCPConnection`

Displays current TCP connections — local and remote endpoints, connection state, and the owning process ID.

```powershell
Get-NetTCPConnection
Get-NetTCPConnection | Where-Object -Property State -eq "Established"
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
```

The `OwningProcess` property contains the PID of the process that opened the connection — combine with `Get-Process` to identify the responsible application.

---

### File Hashing — `Get-FileHash`

Generates a cryptographic hash of a file. Used to verify file integrity, detect tampering, and match files against known malware hashes.

```powershell
Get-FileHash -Path C:\Windows\System32\notepad.exe
Get-FileHash -Path file.txt -Algorithm SHA256    # Default is SHA256
Get-FileHash -Path file.txt -Algorithm MD5
```

---

### Alternate Data Streams (ADS)

NTFS files can have hidden data streams attached to them. `Get-Item` with the `-Stream` parameter reveals them.

```powershell
# List all streams on a file
Get-Item file.txt -Stream *

# Read a specific ADS
Get-Content file.txt -Stream streamname
```

Two streams you may encounter:
- `:$DATA` — the default data stream (normal file content)
- Any other named stream — a potential ADS, possibly hiding data

---

## Important Terminology

| Term | Definition |
|---|---|
| `Get-Process` | Lists running processes with resource usage details |
| `Get-Service` | Lists services and their current state |
| `Get-NetTCPConnection` | Shows active TCP connections and their owning process |
| `Get-FileHash` | Generates a cryptographic hash of a file |
| `OwningProcess` | Property of a TCP connection containing the PID of the responsible process |
| ADS | Alternate Data Stream — hidden data stream attached to an NTFS file |
| IOC | Indicator of Compromise — evidence of malicious activity |
| SHA256 | Default hashing algorithm used by `Get-FileHash` |

---

## Practical Examples / Demonstrations

```powershell
# Top 10 CPU-consuming processes
Get-Process | Sort-Object -Property CPU -Descending | Select-Object -First 10

# Find a specific service by display name pattern
Get-Service | Where-Object { $_.DisplayName -like "*merry*" } | Select-Object Name, DisplayName

# View established TCP connections with process info
Get-NetTCPConnection | Where-Object -Property State -eq "Established" |
    Select-Object LocalPort, RemoteAddress, RemotePort, OwningProcess

# Hash a file
Get-FileHash -Path C:\Users\user\Documents\big-treasure.txt

# Check for ADS on a file
Get-Item C:\logs\house_log.txt -Stream *
```

### Correlating a connection to a process

```powershell
# Get the PID from the connection
$pid = (Get-NetTCPConnection | Where-Object RemotePort -eq 4444).OwningProcess

# Identify the process
Get-Process -Id $pid
```

---

## Real-World Relevance

- `Get-Process` is used during incident response to identify malicious processes — suspicious names, unusual paths, or high resource usage
- `Get-Service` is used to detect persistence via malicious services — attackers install services to survive reboots
- `Get-NetTCPConnection` reveals active C2 connections — combining it with `Get-Process` via `OwningProcess` identifies the malware responsible
- `Get-FileHash` is used to verify file integrity and match against threat intelligence feeds (VirusTotal, MISP)
- ADS detection is important in malware analysis — malware hides payloads in alternate data streams to evade basic file inspection

---

## Key Learnings

- `Get-Process` provides detailed process information including CPU and memory — more useful than `tasklist`
- `Get-Service` reveals all services including stopped ones — anomalous services are a common persistence mechanism
- `Get-NetTCPConnection` shows live connections with `OwningProcess` — essential for identifying malicious network activity
- `Get-FileHash` generates file hashes for integrity verification and threat intelligence matching
- ADS can hide data within NTFS files — `Get-Item -Stream *` reveals hidden streams

---

## Conclusion

Real-time system analysis with PowerShell gives security professionals a live view of what a system is doing — which processes are running, which services are active, what network connections exist, and whether files have been tampered with. These cmdlets are the core of PowerShell-based incident response and threat hunting workflows.
