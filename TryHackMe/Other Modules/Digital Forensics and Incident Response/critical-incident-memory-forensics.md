# Critical Incident: Memory Forensics Scenario

## Overview

This room presents a practical memory forensics investigation scenario. A user reports encrypted PDF files including a critical company document, and the DFIR team captures a memory dump for analysis. The investigation uses Volatility 3 to analyse the memory dump, identify suspicious network connections, running processes, and other indicators of compromise, building toward understanding what ransomware or credential-stealing malware was active on the system.

---

## Scenario

User "Hattori" reported strange behaviour and discovered that PDF files including `important_document.pdf` had been encrypted. The DFIR team was engaged and a memory dump (`memdump.mem`) was captured using FTK Imager on the Windows system.

**Investigation goals:**
- Identify the OS and environment from the memory dump
- Find suspicious network activity (possible C2 communication)
- Identify the processes involved
- Reconstruct the attack timeline

---

## Key Concepts

### Memory Forensics Phases

| Phase | Description |
|-------|-------------|
| **Memory Acquisition** | Capture live RAM to a dump file before shutdown/reboot destroys volatile data |
| **Memory Analysis** | Analyse the dump to extract running processes, network connections, files, and other runtime state |

### Memory Acquisition Tools

| Platform | Tools |
|----------|-------|
| Windows | FTK Imager, WinPmem |
| Linux | LiME |
| macOS | osxpmem |

In this scenario, FTK Imager captured the dump and it was transferred to a Linux machine running Volatility 3 for analysis.

---

## Volatility 3 Plugin Reference

| Plugin | Command | Description |
|--------|---------|-------------|
| `windows.info` | `vol -f memdump.mem windows.info` | OS and kernel details |
| `windows.pslist` | `vol -f memdump.mem windows.pslist` | List active processes |
| `windows.pstree` | `vol -f memdump.mem windows.pstree` | Process tree by parent PID |
| `windows.cmdline` | `vol -f memdump.mem windows.cmdline` | Process command line arguments |
| `windows.netscan` | `vol -f memdump.mem windows.netscan` | Scan for network objects |
| `windows.netstat` | `vol -f memdump.mem windows.netstat` | Traverse network tracking structures |
| `windows.filescan` | `vol -f memdump.mem windows.filescan` | Scan for file objects in memory |
| `windows.handles` | `vol -f memdump.mem windows.handles` | List open process handles |
| `windows.getsids` | `vol -f memdump.mem windows.getsids` | Print SIDs owning each process |
| `windows.drivermodule` | `vol -f memdump.mem windows.drivermodule` | Detect drivers hidden by rootkits |
| `windows.mftscan` | `vol -f memdump.mem windows.mftscan` | Scan for Alternate Data Streams |
| `windows.malfind` | `vol -f memdump.mem windows.malfind` | Find memory regions with injected code |

---

## Practical Examples / Demonstrations

### Step 1 — Get System Information

```bash
vol -f memdump.mem windows.info
```

Confirms: OS version, architecture, kernel version, build number. Verifies you are analysing the correct context.

### Step 2 — Identify Network Activity

```bash
vol -f memdump.mem windows.netstat
```

Look for:
- Outbound connections to unusual or non-corporate IP addresses — potential C2 communication
- Remote access tools (RDP, VNC, SSH) connecting to unexpected destinations
- Established connections from unexpected processes (e.g. `explorer.exe` with an outbound TCP connection)

### Step 3 — Examine Running Processes

```bash
vol -f memdump.mem windows.pslist
vol -f memdump.mem windows.pstree
```

Look for:
- Processes with unusual names or unusual parent-child relationships
- Known ransomware process names or random-looking executable names
- System processes spawned from unexpected parents

### Step 4 — Get Command Line Arguments

```bash
vol -f memdump.mem windows.cmdline
```

Shows exactly how each process was launched — reveals PowerShell commands, script arguments, and file paths used by malware.

### Step 5 — Scan for Files

```bash
vol -f memdump.mem windows.filescan
```

Identifies file objects referenced in memory — can reveal temporary files, dropped payloads, or encrypted documents still referenced by malware.

### Step 6 — Check for Injected Code

```bash
vol -f memdump.mem windows.malfind
```

Identifies memory regions with executable permissions (RWX) and no corresponding file on disk — a common indicator of process injection by ransomware or credential-stealing malware.

---

## Investigation Workflow

```
Memory dump acquired → transferred to analysis machine
        |
        v
windows.info → Confirm OS version and architecture
        |
        v
windows.netstat → Identify suspicious network connections
  (look for C2 IPs, unexpected outbound connections)
        |
        v
windows.pslist / windows.pstree → Map running processes
  (look for suspicious names, unusual parent-child relationships)
        |
        v
windows.cmdline → Review how processes were launched
  (PowerShell commands, dropped file paths)
        |
        v
windows.filescan → Identify files referenced in memory
  (look for ransomware payloads, encrypted files)
        |
        v
windows.malfind → Check for code injection
  (RWX memory regions, injected PE files)
        |
        v
Correlate findings → Build timeline of what happened
Document IOCs (suspicious IPs, process names, file paths)
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Memory dump | A file containing a bit-for-bit copy of a system's RAM at a point in time |
| Volatile memory | RAM — contents are lost when the system is powered off or rebooted |
| C2 | Command and Control — attacker infrastructure that malware communicates with |
| Process injection | Inserting malicious code into the memory space of a legitimate process |
| `windows.netstat` | Volatility plugin traversing network tracking structures |
| `windows.netscan` | Volatility plugin scanning for all network objects |
| `windows.malfind` | Volatility plugin identifying suspicious RWX memory regions |
| SID | Security Identifier — Windows unique identifier for a user or group |

---

## Real-World Relevance

- Memory forensics is the primary method for investigating fileless malware and ransomware that operates largely in memory
- Network connections captured in memory dumps reveal C2 infrastructure even when malware deletes its network configuration from disk
- `windows.malfind` detecting injected code is a reliable indicator of advanced malware — most commodity ransomware and credential stealers inject into legitimate processes
- The `windows.cmdline` output is critical for reconstructing how malware was launched — especially for PowerShell-based attacks where the command line contains the entire attack chain

---

## Key Learnings

- Memory analysis captures runtime state unavailable from disk — critical for ransomware and fileless malware investigations
- Start with `windows.info` to confirm the analysis context
- `windows.netstat` and `windows.netscan` reveal C2 connections and active network activity
- `windows.pstree` visualises parent-child process relationships — suspicious spawning patterns indicate exploitation
- `windows.cmdline` shows how each process was launched — reveals malicious command arguments
- `windows.malfind` identifies code injection — a near-universal technique in advanced malware

---

## Conclusion

Memory forensics provides visibility into what was happening on a compromised system at the moment of capture — visibility that disk forensics alone cannot provide. In ransomware and credential-theft scenarios, the memory dump captures the malware in its active state, revealing network connections, injected code, and process activity that would otherwise be invisible. Systematic analysis with Volatility's plugin set builds the evidence needed to understand the attack and inform recovery.
