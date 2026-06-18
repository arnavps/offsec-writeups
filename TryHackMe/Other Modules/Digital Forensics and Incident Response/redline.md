# Redline

## Overview

Redline is a free incident response tool developed by FireEye (now Trellix) that provides a rapid, high-level view of a Windows, Linux, or macOS endpoint. It collects forensic data from a live system or memory dump and presents it through a graphical interface, enabling analysts to quickly triage a potentially compromised endpoint and determine the nature and extent of a security event. This room covers Redline's collection methods, its interface, IOC-based analysis using the IOC Editor, and a practical ransomware scenario.

---

## Topics Covered

- What Redline does and when to use it
- Three collection methods: Standard, Comprehensive, IOC Search
- Redline interface: Analysis Data categories
- Timeline analysis in Redline
- IOC Editor: creating and using Indicators of Compromise
- IOC Search Collector workflow
- Practical scenarios: lateral movement investigation and ransomware analysis

---

## Key Concepts

### What is Redline?

Redline provides a 30,000-foot view of an endpoint — a rapid triage capability that collects a broad range of forensic data and presents it through a clean GUI without requiring deep manual analysis. It is particularly useful when time is critical and a quick assessment is needed before committing to a full forensic investigation.

**What Redline can collect:**
- Registry data (Windows only)
- Running processes
- Memory images (Windows versions before Windows 10)
- Browser history
- Suspicious strings
- Network connections, services, tasks, event logs, and more

---

### Data Collection Methods

| Method | Description | Use Case |
|--------|-------------|---------|
| **Standard Collector** | Minimum data set — fastest collection (minutes) | Initial triage, time-critical situations |
| **Comprehensive Collector** | Maximum data — most thorough (up to an hour+) | Full investigation when time permits |
| **IOC Search Collector** (Windows only) | Collects data matching known IOCs from IOC Editor | Targeted hunting based on threat intelligence |

#### Standard Collector Configuration Tabs

| Tab | What to Configure |
|-----|------------------|
| Memory | Process listings, driver enumeration, hook detection (disable Hook Detection for most cases; leave Acquire Memory Image unchecked unless needed) |
| Disk | Disk partitions, volumes, file enumeration |
| System | Machine info, OS info, registry hives, user accounts, prefetch cache |
| Network | Network information, browser history, inbound/outbound connections |
| Other | Additional collection options |

---

### Redline Interface: Analysis Data

After importing an analysis session (`.mans` file), the left panel shows:

| Category | Contents |
|----------|---------|
| **System Information** | Machine info, BIOS (Windows), OS details, user information |
| **Processes** | Process name, PID, path, arguments, parent process, username |
| → Handles | Connections from processes to OS objects (files, registry keys, resources) |
| → Memory Sections | Unsigned memory sections — flagged as potentially suspicious |
| → Strings | Captured strings from process memory |
| → Ports | Network connections per process — key for C2 detection |
| **File System** | Files on disk |
| **Registry** | Registry data |
| **Windows Services** | Running and stopped services |
| **Tasks** | Scheduled tasks — common persistence mechanism |
| **Event Logs** | Windows event logs including PowerShell and logon events |
| **ARP and Route Entries** | Network layer information |
| **Browser URL History** | Browsing history |
| **File Download History** | Files downloaded via browser |

**Key investigation areas:**
- **Ports** — identify suspicious outbound connections (C2 communication); legitimate system processes like `explorer.exe` or `notepad.exe` should not have outbound connections
- **Tasks** — attackers create scheduled tasks for persistence
- **Event Logs** — PowerShell events, logon/logoff, user creation

---

### Timeline

Redline's Timeline provides a chronological view of file creation, modification, access, and change events. It helps:
- Establish when a compromise began
- Trace the sequence of attacker actions
- Identify the initial infection vector

Timeline supports keyword search to quickly locate events of interest.

---

### IOC Editor

An IOC (Indicator of Compromise) is an observable artifact that indicates potential compromise — file hashes, IP addresses, domain names, file paths, registry keys, process names, strings, etc.

**IOC Editor** (FireEye, free) allows analysts to create, edit, and save IOC files in a structured format that Redline's IOC Search Collector can use.

**Creating an IOC file:**
1. Open IOC Editor
2. Create a new IOC file
3. Set the IOC name, author, and description
4. Add indicators using the **Add** button:
   - File Strings
   - File MD5 hash
   - File Size
   - Registry keys
   - IP addresses
   - Domain names

**Example IOC for a keylogger:**
- File String: `psylog.exe`
- File String: `RIDEV_INPUTSINK`
- File MD5: (hash value)
- File Size: 35400 bytes

IOC files are saved in XML format and can be shared across the security community.

---

### IOC Search Collector Workflow

1. Create IOC files using IOC Editor
2. In Redline, select **IOC Search Collector**
3. Point Redline to the IOC directory
4. Configure what additional data to collect alongside IOC matching
5. Run the collector — it filters results to only data matching the IOCs
6. Review hits in the analysis session

**Supported vs Unsupported Search Terms:** Not all IOC fields are supported by Redline. Unsupported terms will produce no hits. Check Redline's documentation for the current supported list.

---

## Practical Scenarios

### Scenario 1: Lateral Movement Investigation

**Context:** A corporation suspects a "pass-the-hash" attack was used for lateral movement. A specific file was planted on the victim's machine.

**Known artifacts:**
- File Strings: known embedded strings in the malicious file
- File Size: 834,936 bytes

**Approach:**
1. Create an IOC file in IOC Editor with these file attributes
2. Run IOC Search Collector or load an existing analysis session
3. Search the Timeline and File System for matching entries
4. Confirm the file's presence, location, and timestamps

### Scenario 2: Ransomware Analysis

**Context:** An accountant's files are encrypted and their wallpaper was replaced with a ransom note.

**Approach:**
1. Load the analysis session into Redline
2. **Processes:** Look for unusual processes — ransomware typically runs as a user-space process
3. **Ports:** Check for outbound connections to C2 infrastructure
4. **Tasks:** Identify scheduled tasks created by the ransomware for persistence
5. **File System:** Identify encrypted files, the ransomware executable, and any dropped notes
6. **Timeline:** Establish when encryption began and what preceded it
7. **Event Logs:** Look for shadow copy deletion commands (ransomware typically deletes VSS backups)

---

## Important Terminology

| Term | Meaning |
|------|---------|
| IOC | Indicator of Compromise — artifact indicating potential malicious activity |
| `.mans` | Redline analysis session file — double-click to load into Redline |
| Standard Collector | Redline's fastest collection method — minimum necessary data |
| IOC Search Collector | Collects data matching specified IOCs from IOC Editor |
| Handle | Windows OS connection from a process to a resource (file, registry key, etc.) |
| Memory Section | A mapped region of process memory — unsigned sections are suspicious |
| Pass-the-hash | Lateral movement technique using captured NTLM hashes instead of plaintext passwords |
| C2 | Command and Control — attacker infrastructure that compromised systems beacon to |

---

## Real-World Relevance

- Redline is commonly used in the first hours of an incident response engagement when time is critical and a full forensic image is not yet available
- The Ports section is one of the most valuable for identifying active C2 connections — unexpected outbound connections from system processes are a reliable indicator
- IOC-based hunting with Redline is used after threat intelligence is available — analysts can immediately check whether specific known-bad indicators are present on endpoints
- Scheduled tasks and services are the two most common attacker persistence mechanisms on Windows — Redline surfaces both
- Timeline analysis in Redline is used to establish the initial infection timeline before committing to a full Autopsy investigation

---

## Key Learnings

- Redline provides rapid endpoint triage through three collection methods: Standard, Comprehensive, and IOC Search
- The Standard Collector is fastest and sufficient for initial triage
- Key investigation areas: Ports (C2 connections), Tasks (persistence), Event Logs (PowerShell, logon events)
- IOC Editor creates structured indicator files that the IOC Search Collector uses for targeted hunting
- Timeline helps reconstruct the sequence of events from file creation and modification timestamps
- Redline analysis sessions are saved as `.mans` files — portable and shareable

---

## Conclusion

Redline provides a structured, GUI-based approach to rapid endpoint triage that bridges the gap between a raw incident alert and a full forensic investigation. Its combination of broad artifact collection, IOC-based hunting, and timeline analysis makes it a versatile first-response tool. Understanding how to configure the collector, navigate the interface, and use IOC Editor effectively enables analysts to quickly determine whether an endpoint has been compromised and what the attacker was doing.
