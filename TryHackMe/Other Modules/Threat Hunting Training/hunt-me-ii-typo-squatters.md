# Hunt Me II: Typo Squatters

**Platform:** TryHackMe  
**Room:** Hunt Me II: Typo Squatters  
**Category:** SIEM Log Analysis (Elastic/Kibana)  
**Difficulty:** Medium  

---

## Overview

In this threat hunting scenario, you investigate an incident where Perry, a software developer, unknowingly downloads and installs malware on his workstation. Perry was attempting to extract a password-protected archive and searched for the 7-Zip utility, falling victim to a typosquatted website. You will query the Elastic Stack/Kibana logs to uncover the initial access vector, the persistence mechanism established, and the post-compromise commands executed by the attacker.

---

## Technical Details & Indicators of Compromise (IOCs)

- **Target Victim:** Perry (`perry`)
- **Initial Download URL:** `http://www.7zipp.org/a/7z2301-x64.msi`
- **Typosquatted Domain:** `www.7zipp.org` (legitimate: `7-zip.org`)
- **Attacker IP Address:** `206.189.34.218`
- **Persistence Service:** `7zService`
- **Malicious Binaries:** `7zipp.exe`, `7zipp.dll`
- **Post-Exploitation Tools:** `Invoke-PowerExtract`, `Invoke-SharpKatz`
- **Incident Reference Date:** September 26, 2023

---

## Attack Chain Reconstruction

Analyzing the events in Kibana outlines the following attack chain:

```
[Developer searches for 7-Zip] 
   └── Typosquat Download: www.7zipp.org/a/7z2301-x64.msi
        └── Execution: MSI runs -> Spawns 7z.ps1 & installs fake legitimate 7-Zip
             └── Persistence: Creates "7zService" running as SYSTEM using sc.exe
                  └── C2 Connection: Establishes callback to 206.189.34.218
                       └── Credential Theft: Attacker executes Invoke-SharpKatz to dump LSASS memory
```

---

## Kibana Threat Hunting Queries

Use these queries within the Elastic SIEM console (filtered for **September 26, 2023**) to investigate the compromise:

### Step 1 — Trace Initial Download via DNS
To identify the malicious domain and IP resolved by the workstation, query DNS events (Sysmon Event ID 22):
```kibana
winlog.event_id: 22 AND winlog.event_data.QueryName: "*7z*"
```
*Finding:* This reveals the resolution of `www.7zipp.org` to the external IP `206.189.34.218`.

### Step 2 — Identify Downloaded Installer
Search for file creation events matching MSI installations:
```kibana
winlog.event_id: 11 AND file.name: "*.msi"
```
*Finding:* This query identifies the creation of `7z2301-x64.msi` downloaded into Perry's downloads directory.

### Step 3 — Locate Persistence Service Creation
When a Windows installer executes, check if it registers a new service. In Windows Security logs, filter for Event ID **4697** (A service was installed in the system) or System Log Event ID **7045**:
```kibana
winlog.event_id: (4697 or 7045) AND winlog.event_data.ServiceName: "*7z*"
```
*Finding:* This identifies the creation of `7zService`. The command line reveals it runs with local `SYSTEM` privileges, executing `7zipp.exe` or executing code via `rundll32.exe 7zipp.dll`.

### Step 4 — Analyze Attacker C2 Traffic
Find the outbound network connections initiated by the installer or the service (Sysmon Event ID 3):
```kibana
winlog.event_id: 3 AND (process.name: "powershell.exe" OR process.name: "7zipp.exe") 
AND destination.ip: "206.189.34.218"
```
*Finding:* This confirms the callback connection from the backdoored service to the attacker IP.

### Step 5 — Detect Credential Dumping
Once the attacker escalated privileges to `SYSTEM` via the malicious service, they downloaded tools to dump hashes. Search for PowerShell process commands containing known dumping keywords:
```kibana
process.name: "powershell.exe" AND process.command_line: ("*SharpKatz*" OR "*PowerExtract*" OR "*lsass*")
```
*Finding:* The attacker executed `Invoke-SharpKatz` via memory injection to bypass local host antivirus and extract cached credentials from LSASS.

---

## Common Findings & Answers

| Finding Question | Evidence / Log Value | Log Source |
|---|---|---|
| Typosquatted C2 Domain | `www.7zipp.org` | Sysmon Event ID 22 |
| Attacker IP Address | `206.189.34.218` | Sysmon Event ID 3 / 22 |
| Malicious service created | `7zService` | Security Log Event ID 4697 |
| Malicious installer downloaded | `7z2301-x64.msi` | Sysmon Event ID 11 |
| Credential harvesting tool | `Invoke-SharpKatz` / `Invoke-PowerExtract` | Process command line (Event ID 1) |

---

## Key Learnings

- **Typosquatted Web Delivery:** Attackers buy domains containing common typographical errors of popular software (e.g., `7zipp.org`, `wlnword.com`) to lure users looking for quick software downloads.
- **Service-Based Privilege Escalation:** Executing an installer as an Administrator allows it to register a boot service running as `SYSTEM`. Even if the user closes the software, the service keeps the attacker connected.
- **In-Memory Tool Execution:** Running credential dumpers via PowerShell in-memory payloads (`iex`) leaves fewer file artifacts on disk, requiring robust endpoint monitoring of LSASS process access (Sysmon Event ID 10) to detect.
