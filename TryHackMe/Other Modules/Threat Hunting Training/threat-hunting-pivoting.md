# Threat Hunting: Pivoting

**Platform:** TryHackMe  
**Room:** Threat Hunting: Pivoting  
**Category:** Active Directory Forensics / Threat Hunting  
**Difficulty:** Medium  

---

## Overview

After establishing an initial foothold, threat actors seek to expand their control across the target infrastructure. This involves finding high-value targets, harvesting credentials, and laterally moving from one machine to another. These activities map to the MITRE ATT&CK tactics **Credential Access** and **Lateral Movement**. This guide covers methodologies and event log analysis to hunt for suspicious pivoting and propagation behaviors within Windows and Active Directory environments.

---

## Key Areas to Investigate

When hunting for lateral movement and credential access, focus on:

1. **Remote Execution Logs:** Detecting command execution via Windows Management Instrumentation (WMI), WinRM (Windows Remote Management), or Remote Services.
2. **Interactive & Network Logons:** Analyzing Remote Desktop Protocol (RDP) logons, session reconnections, and anomalous network authentication patterns.
3. **LSASS Memory Access:** Monitoring access to the Local Security Authority Subsystem Service (`lsass.exe`) process, which is targeted for credential harvesting (e.g., Mimikatz).
4. **Anomalous SMB Activity:** Tracking connections to hidden administrative shares (such as `ADMIN$` or `C$`).

---

## Sysmon and Event Log Indicators

The following Event IDs are critical when hunting for pivoting activities:

### 1. Logon and Remote Connection Events
- **Event ID 4624 (Security Log):** Successful Logon. Focus on:
  - **Logon Type 3:** Network Logon (common for SMB, WinRM, WMI).
  - **Logon Type 10:** Remote Interactive (Remote Desktop / RDP).
- **Event ID 4778 (Security Log):** A session was reconnected to a Window Station (RDP session resumption).
- **Event ID 4625 (Security Log):** Failed Logon attempts (often indicating credential brute-forcing).

### 2. Execution and Service Events
- **Event ID 7045 (System Log) / 4697 (Security Log):** A new service was installed on the system (indicates tools like PsExec or custom C2 installers).
- **Event ID 4688 (Security Log) / Sysmon Event ID 1:** Process Creation. Look for WMI hosts (`wmic.exe`, `scrcons.exe`) or WinRM processes (`wsmprovhost.exe`) spawning shells.

### 3. Credential Harvesting (Sysmon)
- **Event ID 10 (ProcessAccess):** Process access. Look for access requested to `lsass.exe` with suspicious call traces or source binaries.
- **Event ID 11 (FileCreate):** Check for the creation of `.dmp` files (commonly named `lsass.dmp` or similar) in temp directories.

---

## Hunt Playbooks & Queries

### Playbook 1 — Hunting Credential Access (LSASS Access)
Adversaries dump LSASS memory to extract cleartext passwords or NTLM hashes.

**Kibana Query for LSASS Access:**
```kibana
winlog.event_id: 10 AND winlog.event_data.TargetImage: "*\\lsass.exe"
AND NOT winlog.event_data.SourceImage: ("*\\MsMpEng.exe" OR "*\\svchost.exe" OR "*\\taskhostw.exe")
```

Look for:
- Source processes running from user-writable directories (e.g., `C:\Users\Public`).
- GrantedAccess masks like `0x1F0FFF` (Process All Access) or `0x0010` (Process VM Read).

### Playbook 2 — Detecting WinRM / WMI Execution
WMI and WinRM allow remote administrative commands, often abused by attackers to run malware remotely.

- **WinRM Detection:** Look for `wsmprovhost.exe` spawning child processes.
  ```kibana
  process.parent.name: "wsmprovhost.exe" AND process.name: ("cmd.exe" or "powershell.exe")
  ```
- **WMI Detection:** Monitor process creation where the parent is WMI Prereq (`WmiPrvSE.exe`).
  ```kibana
  process.parent.name: "wmiprvse.exe" AND process.name: ("cmd.exe" or "powershell.exe" or "scrcons.exe")
  ```

### Playbook 3 — New Remote Service Installation
Tools like `PsExec` or lateral movement scripts create temporary Windows services.

**Kibana Query for Service Installation:**
```kibana
winlog.event_id: (4697 or 7045) AND NOT winlog.event_data.ServiceStartType: "Disabled"
```

Inspect the `ImagePath` parameter in the events. If it points to `cmd.exe`, `powershell.exe`, or file shares (e.g., `\\10.10.10.5\share\payload.exe`), it is highly indicative of lateral movement.

### Playbook 4 — RDP Profiling
RDP connections are traced via Event ID 4624 (Logon Type 10) and Event ID 4778.

**Kibana Query for Remote interactive logins:**
```kibana
winlog.event_id: 4624 AND winlog.event_data.LogonType: 10
```

Cross-reference the `IpAddress` (source machine) with the `TargetUserName`. Check if this user typically logs in from that source IP, or if a single source IP is logging into multiple machines sequentially.

---

## Pivoting Indicators of Compromise

| Technique | Attack Tool / Command | Primary Log Event |
|---|---|---|
| Remote Service Pivot | `psexec.exe \\target cmd.exe` | Service Install (7045) / Event ID 1 (`PSEXESVC.exe`) |
| WMI Execution | `wmic /node:target process call create "..."` | `WmiPrvSE.exe` spawning shells |
| Credential Theft | `procdump.exe -ma lsass.exe lsass.dmp` | Event ID 1 (procdump) & ID 11 (lsass.dmp) |
| Pass-the-Hash | Mimikatz `sekurlsa::pth` | Event ID 4624 (Logon Type 9 - NewCredentials) |

---

## Real-World Relevance

- Credential dumping is a mandatory step for attackers to escalate privileges and move laterally across Active Directory. Detecting LSASS access blocks standard dumping tools.
- Modern lateral movement frequently leverages native administration channels (WinRM, WMI, RDP) rather than custom exploits, making process tree behavioral analysis (e.g. `wsmprovhost.exe` spawning shell) crucial.
- Correlating network logs with logon logs helps detect Pass-the-Hash or Pass-the-Ticket attacks where NTLM hashes or Kerberos tickets are abused without password cracking.
