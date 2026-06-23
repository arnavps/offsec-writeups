# Typo Snare

**Platform:** TryHackMe  
**Room:** Typo Snare (Threat Hunting Simulator)  
**Category:** Incident Response / Ransomware Investigation (Elasticsearch)  
**Difficulty:** Medium  

---

## Overview

At approximately 4:00 PM, a critical alert hits the SOC: two developer workstations have been fully encrypted, and ransom notes have been placed across the filesystems. To make matters worse, the central SIEM log collector suffered a brief outage during the incident, leaving gaps in real-time correlation. Acting as an incident responder, your job is to query the Elasticsearch logs recovered from the affected endpoints to piece together the intrusion timeline, identify the persistence mechanisms, and map out the attacker's post-exploitation activities.

---

## Technical Details & Indicators of Compromise (IOCs)

- **Scenario Trigger Time:** ~4:00 PM
- **Initial Access Vector:** Download of a malicious MSI installer from a typosquatted domain (`7zipp.org`) masquerading as 7-Zip.
- **Persistence Mechanism:** Windows Service named `7zService` running with `SYSTEM` privileges.
- **Defense Evasion Technique:** DLL hijacking/side-loading via `rundll32.exe 7zipp.dll`.
- **Target Domain User:** `anna.jones` (attacker attempted to compromise and reset password).
- **Post-Exploitation Channels:** WinRM (`wsmprovhost.exe`) remote execution and PowerSploit scripts.
- **Impact:** Deployment of ransomware resulting in mass file encryption.

---

## Attack Chain Reconstruction

Analyzing the Elasticsearch logs reveals the following attacker movement:

```
[Initial Access] 
   └── Typosquat Download: 7zipp.org MSI installer
        └── Persistence: sc.exe registers "7zService" running as SYSTEM
             └── Defense Evasion: Service executes "rundll32.exe 7zipp.dll"
                  └── Pivoting/Post-Exploitation: Remote WinRM session (wsmprovhost.exe) established
                       └── Privilege Access: Attacker runs net.exe to reset anna.jones domain password
                            └── Impact: Ransomware executed, mass encrypting files
```

---

## Elasticsearch Threat Hunting Queries

Query the recovered indices in Elasticsearch using Kibana to trace the intrusion:

### Step 1 — Spotting the Persistence Service
Windows Services running as SYSTEM are prime persistence targets. Search Security logs for Event ID **4697** (Service Creation) or System logs for Event ID **7045**:
```kibana
winlog.event_id: (4697 or 7045) AND winlog.event_data.ServiceName: "*7z*"
```
*Finding:* This query uncovers the malicious `7zService` created by the typosquatted installer. The configuration launches the payload under `SYSTEM` privileges on boot.

### Step 2 — Tracing Evasive Execution (LOLBins)
The malicious service executed a DLL payload by hijacking `rundll32.exe`. Look for `rundll32.exe` process logs loading custom DLLs:
```kibana
process.name: "rundll32.exe" AND process.command_line: "*7z*"
```
*Finding:* This reveals `rundll32.exe` being invoked to execute functions within `7zipp.dll`, attempting to hide the C2 activity behind a legitimate system process.

### Step 3 — Identifying Remote Command Session
The attacker pivoted to host interaction using WinRM (Windows Remote Management). Identify PowerShell or command prompts spawned under the WinRM worker process (`wsmprovhost.exe`):
```kibana
process.parent.name: "wsmprovhost.exe" AND process.name: ("cmd.exe" or "powershell.exe")
```
*Finding:* The query shows the attacker establishing an interactive terminal session remotely, allowing command line interaction without RDP logs.

### Step 4 — Detecting Password Reset Attempt
Under the WinRM shell, the attacker executed reconnaissance commands (`whoami`) and attempted to hijack a domain user account (`anna.jones`) by resetting her password. Query process executions containing command line operations on local accounts:
```kibana
process.name: "net.exe" AND process.command_line: "*user*anna.jones*"
```
*Finding:* This exposes the exact commands used by the attacker to attempt a domain password modification.

### Step 5 — Ransomware Impact Analysis
Search for the sudden creation of ransom files or mass renaming of document extensions (Sysmon Event ID 11):
```kibana
winlog.event_id: 11 AND file.name: ("*README*" OR "*HELP_DECRYPT*" OR "*DECRYPT*")
```
Tracing the processes that created these files will map directly back to the malicious `7zipp.exe` or `rundll32.exe` loader, confirming the root cause process of the encryption.

---

## Common Findings & Answers

| Finding Question | Evidence / Log Value | Log Source |
|---|---|---|
| Malicious service installed | `7zService` | Event ID 4697 (Service Created) |
| System utility used to execute DLL | `rundll32.exe` | Process Image |
| Compromised DLL file name | `7zipp.dll` | Process CommandLine |
| Remote management host process | `wsmprovhost.exe` | ParentProcessName (Event ID 1) |
| Targeted domain user account | `anna.jones` | Process CommandLine (`net.exe user`) |
| Typosquatted source domain | `7zipp.org` | Sysmon Event ID 22 / Web proxy logs |

---

## Key Learnings

- **WinRM Monitoring:** WinRM is a silent administrative helper. Attackers leverage it to run tools like PowerSploit without generating active GUI session logs. Monitoring `wsmprovhost.exe` process spawns is a critical detection rule.
- **Rundll32 Evasion:** Attackers frequently wrap C2 beacons inside DLLs and run them via `rundll32.exe`. Detections must profile what DLLs are loaded from user-writable directories rather than standard system libraries.
- **Account Modification Alerts:** Modifying account passwords using CLI commands (`net.exe user`) is a high-risk activity that should immediately alert the security team, especially when initiated by non-interactive admin shells.
