# Threat Hunting: Foothold

**Platform:** TryHackMe  
**Room:** Threat Hunting: Foothold  
**Category:** Endpoint Forensics / Threat Hunting  
**Difficulty:** Medium  

---

## Overview

The foothold phase of an intrusion corresponds to MITRE ATT&CK tactics **Initial Access** and **Execution**. Threat actors establish a foothold on a system using phishing attachments, malicious links, web server exploitation, or compromised credentials. Once inside, they execute code to prepare the host for secondary operations. This guide explores strategies, tools, and queries to detect initial user or host compromise using Windows Event Logs and Sysmon data.

---

## Key Areas to Investigate

When hunting for a foothold, focus your investigation on:

1. **Suspicious Parent-Child Process Relationships:** Identifying non-standard processes spawned by common initial access applications (e.g., MS Office spawning shells).
2. **Living off the Land Binaries (LoLBins):** Monitoring legitimate Windows system binaries (like `powershell.exe`, `cmd.exe`, `wscript.exe`, `certutil.exe`, or `mshta.exe`) used for malicious downloads or execution.
3. **Execution from Non-Standard Paths:** Detecting binaries running from directories with write access for standard users (e.g., `C:\Users\<user>\AppData\Local\Temp` or `C:\Users\Public`).
4. **Mark of the Web (MoTW) Bypass:** Checking file downloads and execution that bypass standard system trust mechanisms.

---

## Sysmon and Event Log Indicators

Sysmon (System Monitor) is the primary resource for endpoint detection. The following Event IDs are critical when hunting for footholds:

| Event ID | Event Name | Key Hunting Indicators |
|---|---|---|
| **Event ID 1** | Process Creation | Abnormal parent processes, encoded command lines, suspicious execute locations. |
| **Event ID 3** | Network Connection | System utilities (`cmd.exe`, `powershell.exe`, `certutil.exe`) initiating outbound web requests. |
| **Event ID 11** | File Created | Bulk creation of files in Temp folders, or creation of `.exe`, `.dll`, `.bat` files by office applications or web browsers. |
| **Event ID 15** | FileCreateStreamHash | Identifies file downloads with Zone.Identifier indicating web origin (MoTW). |
| **Event ID 12/13** | Registry Event | Registry modifications for persistence, specifically Run/RunOnce keys. |

---

## Investigation Playbooks & Queries

### Playbook 1 — Identifying Suspicious Parent Processes
Web browsers, email clients, and document readers should rarely spawn command shells or script engines.

**Elasticsearch / Kibana Query:**
```kibana
process.parent.name: ("outlook.exe" or "winword.exe" or "excel.exe" or "chrome.exe" or "msedge.exe" or "acrobat.exe") 
AND process.name: ("cmd.exe" or "powershell.exe" or "wscript.exe" or "cscript.exe" or "mshta.exe" or "scrcons.exe")
```

**PowerShell Query:**
```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | 
Where-Object {$_.Id -eq 1} | 
Where-Object {
    $xml = [xml]$_.ToXml()
    $parentName = ($xml.Event.EventData.Data | Where-Object {$_.Name -eq "ParentImage"})."#text"
    $imageName = ($xml.Event.EventData.Data | Where-Object {$_.Name -eq "Image"})."#text"
    $parentName -match "outlook|winword|excel|chrome|msedge" -and $imageName -match "cmd|powershell|wscript|mshta"
} | Select-Object TimeCreated, Message | Format-List
```

### Playbook 2 — Detecting LoLBin Exploitation
Legitimate Windows tools are often repurposed to bypass application whitelisting and network restrictions:

1. **`certutil.exe` (Download & Decode):**
   Used to download payloads disguised as text files and decode them.
   ```kibana
   process.name: "certutil.exe" AND process.args: ("*urlcache*" OR "*decode*")
   ```
2. **`mshta.exe` (HTML Applications):**
   Executes malicious HTML/JavaScript/VBScript files directly from web hosts.
   ```kibana
   process.name: "mshta.exe" AND process.args: "*http*"
   ```
3. **`regsvr32.exe` (Squiblydoo Technique):**
   Downloads and runs scriptlets (`.sct`) from web hosts bypassing app control.
   ```kibana
   process.name: "regsvr32.exe" AND process.args: ("*/i:http*" OR "*scrobj.dll*")
   ```

### Playbook 3 — Execution from User-Writable Folders
Standard users cannot write to `C:\Windows` or `C:\Program Files`, but they have full write access to their own AppData and Temp folders. Attackers leverage this to drop and run payloads.

**Kibana Query for Suspicious Execution Paths:**
```kibana
process.executable: (*\\AppData\\Local\\Temp\\* OR *\\Users\\Public\\* OR *\\AppData\\Roaming\\*)
AND NOT process.executable: (*\\Microsoft\\Teams\\* OR *\\AppData\\Local\\Microsoft\\OneDrive\\*)
```

---

## Common Foothold Findings

| Attacker Action | Detection Signal | Log Context |
|---|---|---|
| Malicious Attachment | `outlook.exe` → `cmd.exe` → `powershell.exe` | Sysmon Event ID 1 |
| Payload Download | `certutil.exe -urlcache -split -f http://...` | Sysmon Event ID 1 & 3 |
| Registry Run Key | `reg.exe add HKCU\Software\Microsoft\Windows\CurrentVersion\Run ...` | Event ID 13 |
| Unsigned execution | Executable running without a valid signature signature verification status | Sysmon Event ID 1 (`SignatureStatus`) |

---

## Real-World Relevance

- Initial access techniques evolve rapidly, shifting from macro-enabled Word documents (VBA) to LNK files, ISO mounts, and supply chain updates.
- Restricting LolBin execution or monitoring outbound connection logs from system utilities like `powershell.exe` and `certutil.exe` blocks or exposes a vast majority of commodity and APT foothold attempts.
- Creating alerts for Sysmon Event ID 15 (Mark of the Web) helps trace download chains back to their external origins when investigating compromised systems.
