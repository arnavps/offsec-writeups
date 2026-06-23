# Health Hazard

**Platform:** TryHackMe  
**Room:** Health Hazard (Threat Hunting Simulator)  
**Category:** Supply Chain Attack / SIEM Investigation (Splunk)  
**Difficulty:** Easy-Medium  

---

## Overview

In this Threat Hunting Simulator scenario, co-founder Tom Whiskers of PawPress unknowingly compromises his system. While setting up the company's first website, he installs a malicious dependency package via npm. This supply chain compromise allows the attacker to run commands on the developer machine, download an executable payload, establish C2 communication, and write registry entries for persistence. Acting as a threat hunter, your objective is to navigate the Splunk SIEM logs to reconstruct this attack lifecycle.

---

## Technical Details & Indicators of Compromise (IOCs)

- **Target Victim:** Tom Whiskers (co-founder of PawPress)
- **Attack Vector:** Supply Chain Compromise (trojanized npm package)
- **Malicious npm Package:** `healthchk-lib` (version `1.0.1`)
- **Malicious Trigger Script:** `postinstall.ps1`
- **Downloaded Payload:** `SystemHealthUpdater.exe`
- **Typosquatted C2 Domain:** `global-update.wlndows.thm` (note the spelling `wlndows.thm`)
- **Persistence Mechanism:** Windows Registry Run key `Windows Update Monitor`

---

## Attack Chain Reconstruction

Analyzing the endpoint and network logs in Splunk outlines the following attack chain:

```
[npm install healthchk-lib@1.0.1]
   └── node.exe spawns: cmd.exe
        └── cmd.exe runs: powershell.exe -executionpolicy bypass postinstall.ps1
             └── PowerShell downloads & executes: SystemHealthUpdater.exe
                  ├── Network C2 callback: global-update.wlndows.thm
                  └── Registry Persistence: HKCU\...\Run\Windows Update Monitor -> SystemHealthUpdater.exe
```

---

## Splunk Threat Hunting Queries

Use these search queries in your Splunk instance to investigate the intrusion:

### Step 1 — Investigate npm Installation Flow
Start by hunting process creation events (Sysmon Event ID 1) where `node.exe` is the parent process, indicating activity triggered by npm scripts:
```splunk
index=win_logs sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 ParentImage="*\\node.exe"
```
*Finding:* This reveals `node.exe` spawning `cmd.exe` to run a PowerShell script (`postinstall.ps1`) bundled inside the npm library `healthchk-lib@1.0.1`.

### Step 2 — Trace the Malicious PowerShell Execution
Analyze the child processes of the shell spawned by node to see what commands were executed:
```splunk
index=win_logs sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1 ParentImage="*\\cmd.exe" CommandLine="*powershell*"
```
*Finding:* The PowerShell script bypassed script execution policies to download the malicious executable `SystemHealthUpdater.exe` from an external server and drop it into a user-writable directory.

### Step 3 — Locate the Dropped Payload
Query file creation events (Sysmon Event ID 11) to confirm where `SystemHealthUpdater.exe` was written:
```splunk
index=win_logs sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=11 TargetFilename="*SystemHealthUpdater.exe"
```
*Finding:* This reveals the payload directory (e.g., `C:\Users\TomWhiskers\AppData\Local\Temp` or similar directory).

### Step 4 — Find Registry Persistence Key
The malware established persistence by editing registry run keys. Hunt for registry write events (Sysmon Event ID 13) associated with the dropped executable:
```splunk
index=win_logs sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=13 TargetObject="*\\CurrentVersion\\Run*"
```
*Finding:* An entry named `Windows Update Monitor` was created pointing to `SystemHealthUpdater.exe` under `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`.

### Step 5 — Verify C2 Network Connections
Find the outbound C2 connections made by the payload. Search for DNS queries (Sysmon Event ID 22) or IP connections (Sysmon Event ID 3) associated with `SystemHealthUpdater.exe`:
```splunk
index=win_logs sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode=3 OR EventCode=22) Image="*SystemHealthUpdater.exe"
```
*Finding:* The DNS query resolves the typosquatted C2 domain `global-update.wlndows.thm`, establishing the callback channel.

---

## Common Findings & Answers

| Finding Question | Evidence / Log Value | Log Source |
|---|---|---|
| Compromised npm library name | `healthchk-lib@1.0.1` | Process CommandLine / npm logs |
| Script run after npm install | `postinstall.ps1` | Process CommandLine (EventCode 1) |
| Dropped malicious binary | `SystemHealthUpdater.exe` | TargetFilename (EventCode 11) |
| Registry key name for persistence | `Windows Update Monitor` | TargetObject (EventCode 13) |
| Typosquatted C2 domain name | `global-update.wlndows.thm` | QueryName (EventCode 22) |

---

## Key Learnings

- **Supply Chain Threats:** Modern software development relies on packages from public repositories (npm, PyPI, NuGet). Attackers compromise or upload typosquatted packages, leveraging post-install scripts to compromise developers immediately upon installation.
- **Process Tree Anomalies:** Native dev tools like `node.exe` or `python.exe` rarely spawn administrative command interpreters or PowerShell scripts. Profiling these parent-child chains is highly effective for exposing supply chain intrusions.
- **Run Key Persistence:** The `Run` registry keys are noisy, but monitoring modifications originating from suspicious binaries or user folders exposes threat actors attempting to survive system reboots.
