# Threat Hunting: Endgame

**Platform:** TryHackMe  
**Room:** Threat Hunting: Endgame  
**Category:** Incident Response / Threat Hunting  
**Difficulty:** Medium-Hard  

---

## Overview

The "Endgame" phase refers to the final phase of the intrusion lifecycle, mapping to the MITRE ATT&CK tactics **Exfiltration** and **Impact**. At this stage, the adversary has achieved their prerequisites and proceeds to execute their ultimate objective: stealing sensitive business data (exfiltration) or disrupting system operations (e.g., encrypting files via ransomware). This guide covers detections and log analysis to hunt for data staging, exfiltration tunnels, and ransomware indicators.

---

## Key Areas to Investigate

When hunting for endgame behaviors, look for:

1. **Data Staging:** The consolidation and compression of files (e.g., `.zip`, `.rar`, `.7z`, `.tar`) in user-writable directories (Temp, AppData) prior to exfiltration.
2. **Exfiltration Tunnels:** Non-standard network protocols or utilities used to push data out of the network (e.g., DNS tunneling, ICMP tunneling, Rclone, or WebDAV uploads).
3. **Volume Shadow Copy Tampering:** Attacker actions to prevent system recovery by deleting backups and shadow copies.
4. **Bulk File Modification:** High-frequency file write and rename operations characteristic of ransomware encryption processes.

---

## Sysmon and Event Log Indicators

The following Event IDs are critical during the final stage of an investigation:

| Event ID | Event Name | Hunting Context |
|---|---|---|
| **Event ID 1** | Process Creation | Identifying tools like `vssadmin.exe`, `bcdedit.exe`, `rclone.exe`, or `7z.exe`. |
| **Event ID 11** | File Created | Spotting bulk file writes, ransom notes (`README.txt`), or staged zip files. |
| **Event ID 22** | DNS Query | Detecting DNS Tunneling (massive amounts of TXT/CNAME queries to a single domain). |
| **Event ID 3** | Network Connection | Spikes in outbound data to cloud storage or unauthorized IPs. |
| **Event ID 4663** | Object Access | Identifying rapid, successive modifications to files (requires File Auditing enabled). |

---

## Hunt Playbooks & Queries

### Playbook 1 — Identifying Data Staging
Attackers consolidate files using tools like `7z.exe`, `rar.exe`, `tar.exe`, or built-in PowerShell compression before exfiltration.

**Kibana Query for Archiving Tools:**
```kibana
process.name: ("7z.exe" or "7za.exe" or "rar.exe" or "tar.exe" or "zip.exe")
AND process.args: ("*\\Temp\\*" OR "*\\Users\\Public\\*" OR "*\\AppData\\*")
```

Look for:
- Large archive files created in Temp folders (e.g., `C:\Windows\Temp\exfil.zip`).
- Command lines containing files targeting source code repositories, finance folders, or database backups.

### Playbook 2 — Detecting DNS Tunneling Exfiltration
DNS tunneling allows attackers to encode stolen data into DNS subdomains and query their external server, leaking data in DNS queries.

**Kibana Query for DNS Queries:**
```kibana
winlog.event_id: 22 AND NOT winlog.event_data.QueryName: (*.microsoft.com OR *.google.com OR *.windows.net)
```

Look for:
- A massive volume of queries (thousands in minutes) to a single domain.
- Query names containing long, random, or Base64-like subdomains (e.g., `aGJ4Z2...234.attacker.com`).
- Queries requesting `TXT`, `CNAME`, or `NULL` records.

### Playbook 3 — Ransomware Execution (Shadow Copy & Recovery Tampering)
To force payment, ransomware operators disable system recovery mechanisms before encrypting.

**Kibana Query for Disabling Backups:**
```kibana
process.name: ("vssadmin.exe" or "wmic.exe" or "bcdedit.exe" or "wbadmin.exe")
AND process.args: ("*delete*shadows*" or "*shadowcopy*delete*" or "*recoveryenabled*No*" or "*ignoreallfailures*")
```

**Common commands used by ransomware:**
- `vssadmin.exe delete shadows /all /quiet`
- `wmic shadowcopy delete`
- `bcdedit.exe /set {default} recoveryenabled No`
- `bcdedit.exe /set {default} bootstatuspolicy ignoreallfailures`

### Playbook 4 — Ransomware Encryption Activity
Ransomware modifies hundreds of files in seconds, renaming them or adding a specific extension.

**Kibana Query for Bulk File Creation:**
```kibana
winlog.event_id: 11 AND file.extension: ("*.locked" OR "*.crypto" OR "*.crypt" OR "*.enc")
```

If Sysmon is configured, trace back Event ID 11. Count the number of files created per minute grouped by `ProcessId`. A count exceeding 100 files in a short time frame by a non-system process (e.g., `svchost.exe` is expected, but `7zipp.exe` or `updater.exe` is not) suggests active ransomware encryption.

---

## Endgame Indicators of Compromise

| Category | Indicator / Activity | Logs |
|---|---|---|
| Backup Deletion | `vssadmin.exe delete shadows /all /quiet` | Sysmon Event ID 1 |
| Exfiltration | Outbound connections to `mega.nz`, `dropbox.com`, `gofile.io` | Sysmon Event ID 3 |
| Ransomware | Spawning `README.txt`, `HELP_DECRYPT.txt` | Sysmon Event ID 11 |
| Recovery Tampering| `bcdedit /set {default} recoveryenabled No` | Sysmon Event ID 1 |

---

## Real-World Relevance

- Exfiltration is the primary goal of modern cyber espionage and double-extortion ransomware campaigns. Early detection during the "staging" phase blocks data leaks before they leave the perimeter.
- Volume shadow copy deletion commands (`vssadmin delete shadows`) are high-fidelity alerts. Any execution of these commands in an enterprise network should immediately trigger host isolation.
- DNS tunneling can bypass standard firewalls because it utilizes port 53. Monitoring DNS query sizes, patterns, and frequencies is critical for networks handling sensitive intellectual property.
