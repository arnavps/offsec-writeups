# Hunt Me I: Payment Collectors

**Platform:** TryHackMe  
**Room:** Hunt Me I: Payment Collectors  
**Category:** SIEM Log Analysis (Elastic/Kibana)  
**Difficulty:** Medium  

---

## Overview

In this threat hunting scenario, you investigate a phishing attack against Michael Ascot, a Senior Finance Director at SwiftSpend. The entry vector was a malicious invoice attachment that bypassed initial email filters. Acting as an analyst, your job is to query the Kibana SIEM instance to trace the attack path, identify indicators of compromise (IOCs), map the post-compromise commands, and determine what data was staged and exfiltrated.

---

## Technical Details & Indicators of Compromise (IOCs)

- **Target Victim:** Michael Ascot (`michael.ascot`)
- **Initial File Attachment:** `Invoice_AT_2023-227.zip`
- **Initial LNK Payload:** `Payment_Invoice.pdf.lnk`
- **C2 Domain Infrastructure:** `haz4rdw4re.io`
- **Staged Archive File:** `exfilt8me.zip`
- **Stolen Sensitive File:** `ClientPortfolioSummary.xlsx`
- **Investigation Reference Date:** September 15, 2023

---

## Attack Chain Reconstruction

Analyzing the endpoint and network logs in Kibana reveals a five-stage attack chain:

```
[Outlook Phishing Email] 
   └── Downloads: Invoice_AT_2023-227.zip 
        └── Extracts: Payment_Invoice.pdf.lnk
             └── Spawns: cmd.exe -> powershell.exe (Base64 Encoded)
                  └── Downloads: PowerCat (Reverse Shell) & PowerView (AD Discovery)
                       └── Action: Stages "exfilt8me.zip" & Exfiltrates via DNS (nslookup) to haz4rdw4re.io
```

---

## Kibana Threat Hunting Queries

To investigate the incident, structure your Kibana searches as follows:

### Step 1 — Isolate the User and Timeframe
Set the global time filter in Kibana to **September 15, 2023**. Filter all events related to the victim's username:
```kibana
user.name: "michael.ascot"
```

### Step 2 — Trace Initial Access & File Extraction
Look for file creation events (Sysmon Event ID 11) originating from the email client (`outlook.exe`) to find the downloaded archive.
```kibana
winlog.event_id: 11 AND process.name: "outlook.exe" AND file.name: "*.zip"
```
*Finding:* This query identifies the creation of `Invoice_AT_2023-227.zip`.

Once the user extracted the file, they ran the malicious LNK file. Look for process creation events (Event ID 1) triggered by `explorer.exe` or `cmd.exe`:
```kibana
winlog.event_id: 1 AND process.parent.name: "explorer.exe" AND process.command_line: "*LNK*"
```
*Finding:* The execution of `Payment_Invoice.pdf.lnk` spawned a shell.

### Step 3 — Analyze Obfuscated Execution
The malicious shortcut spawned PowerShell with an encoded payload. Filter for PowerShell process creation with base64 arguments:
```kibana
process.name: "powershell.exe" AND process.args: "-enc"
```
Decoding the Base64 command-line arguments (using CyberChef or a local shell) reveals:
- Web request queries downloading `powercat.ps1` from the attacker infrastructure `haz4rdw4re.io`.
- Establishment of a reverse shell callback on a specific port.

### Step 4 — Map Post-Exploitation Discovery
Query commands executed under the spawned shell to identify what host and Active Directory enumeration was performed:
```kibana
process.parent.name: "powershell.exe" AND process.name: ("systeminfo.exe" or "whoami.exe" or "net.exe")
```
The attacker imported `PowerView.ps1` to query the domain controllers and enumerate high-value target assets.

### Step 5 — Investigate Data Staging and Exfiltration
Find where the attacker staged files by looking for command-line compression utilities or file creations:
```kibana
winlog.event_id: 11 AND file.name: "*exfil*"
```
*Finding:* The attacker compressed confidential directories into `exfilt8me.zip`.

To trace the exfiltration, look for high-volume network logs or script-based exfiltration. The attacker utilized DNS exfiltration (DNS Tunneling) using `nslookup` queries to leak data slice-by-slice:
```kibana
process.name: "nslookup.exe" AND process.command_line: "*.haz4rdw4re.io"
```
Each DNS query queried a subdomain containing encoded slices of `exfilt8me.zip`, allowing the attacker to reconstruct the archive on their server.

---

## Common Findings & Answers

| Finding Question | Evidence / Log Value | Log Source |
|---|---|---|
| Name of phishing attachment | `Invoice_AT_2023-227.zip` | Sysmon Event ID 11 |
| Decoded C2 host | `haz4rdw4re.io` | decoded `powershell.exe -enc` |
| AD Discovery Tool used | `PowerView.ps1` | process command line |
| Staged data archive name | `exfilt8me.zip` | Sysmon Event ID 11 |
| Exfiltration vector | DNS queries via `nslookup` | Sysmon Event ID 1 & 22 |
| Sensitive file compromised | `ClientPortfolioSummary.xlsx` | file access logs |

---

## Key Learnings

- **LNK File Abuse:** Attackers continue to mask `.lnk` files with PDF or Document icons to trick users into running command-line payloads.
- **Outbound Network Profiling:** In a secure network, utilities like `powershell.exe` and `nslookup.exe` should not be executing arbitrary requests to untrusted external domains.
- **DNS Exfiltration Detection:** Traditional firewalls do not block DNS traffic (UDP 53). Organizations must monitor DNS queries for volume spikes and long, randomized subdomains to detect data leakage.
