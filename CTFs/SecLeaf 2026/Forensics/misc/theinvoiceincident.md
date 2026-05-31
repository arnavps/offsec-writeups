# Forensic Analysis Report: The Invoice Incident

## 1. Executive Summary
On May 2, 2026, a security incident occurred involving a finance department workstation (`FIN-WS23`) assigned to employee `finance.aarti`. The compromise was initiated via a targeted phishing email containing a malicious macro-enabled Word document. 

Upon opening the document, a malicious VBA macro executed, spawning an obfuscated PowerShell session. This session established a network connection to a Command and Control (C2) server to retrieve and execute a secondary payload (`a.ps1`).

This writeup details the forensic investigation, logs correlation, key indicators of compromise (IOCs), and strategic remediation steps to prevent similar incidents in the future.

---

## 2. Incident Summary & Key Details

| Field | Value |
|---|---|
| **Incident Date** | 2026-05-02 |
| **Initial Detection Time** | 09:14:03 AM |
| **Target Host** | `FIN-WS23` |
| **Affected User Account** | `finance.aarti` (Finance Department) |
| **Threat Vector** | Phishing Email (Initial Access) |
| **Sender Address** | `billing@micr0soft-support.com` |
| **Malicious Attachment** | `Invoice_April_2026.docm` (Macro-Enabled Word Document) |
| **First-Stage Command** | `powershell.exe -enc SQBFAFgA` (Decodes to `IEX` / `Invoke-Expression`) |
| **C2 Server IP Address** | `198.51.100.24` |
| **Second-Stage Payload URL** | `hxxp://198.51.100.24/a.ps1` |

---

## 3. Chronological Timeline of Events

The attack unfolded within a **2-minute and 21-second window** between email arrival and payload staging:

```mermaid
timeline
    title Attack Lifecycle Timeline (09:14 AM - 09:16 AM)
    09:14:03 : Phishing email received with Invoice_April_2026.docm
    09:16:11 : finance.aarti opens attachment via WINWORD.EXE on FIN-WS23
    09:16:18 : WINWORD executes VBA macro; spawns Base64-encoded powershell.exe
    09:16:24 : powershell.exe connects to C2 IP 198.51.100.24 to retrieve a.ps1
```

1. **09:14:03 AM** — The email gateway allows a suspicious message from `billing@micr0soft-support.com` addressed to `finance.aarti` with the attachment `Invoice_April_2026.docm`.
2. **09:16:11 AM** (T+128s) — The user opens the attachment on workstation `FIN-WS23` using Microsoft Word (`WINWORD.EXE`).
3. **09:16:18 AM** (T+135s) — Word executes an embedded VBA macro that invokes `powershell.exe` using an encoded payload (`-enc SQBFAFgA`).
4. **09:16:24 AM** (T+141s) — PowerShell executes `IEX` (Invoke-Expression) to download and run the stage-two script `a.ps1` from the remote host `198.51.100.24` over port 80.

---

## 4. Technical Analysis of Evidence

### Phase 1: Phishing Email Ingestion
The attacker successfully bypassed initial filters using a typosquatted domain (`micr0soft-support.com` using a zero `0` instead of `o`). 

**Evidence (`mail_gateway.log` & `easy_logs.csv`):**
```log
2026-05-02 09:14:03 ALLOW from=billing@micr0soft-support.com to=finance.aarti subject=Pending Invoice Notice attachment=Invoice_April_2026.docm
```

### Phase 2: User Execution
The log confirms that the finance employee opened the macro-enabled Word document.

**Evidence (`endpoint_events.log` & `easy_logs.csv`):**
```log
2026-05-02 09:16:11 host=FIN-WS23 process=WINWORD.EXE file=Invoice_April_2026.docm
```

### Phase 3: Malicious Macro & Process Spawning
Almost immediately after Word was opened, it spawned `powershell.exe` with a base64 encoded command argument. This behavior is highly indicative of malicious VBA macro execution (e.g., using `Shell` or `WScript.Shell`).

**Evidence (`endpoint_events.log` & `easy_logs.csv`):**
```log
2026-05-02 09:16:18 host=FIN-WS23 parent=WINWORD.EXE process=powershell.exe cmd=-enc SQBFAFgA
```

#### **Payload Decoding Analysis:**
The base64 encoded argument passed to PowerShell was:
```
SQBFAFgA
```
Powershell expects `-enc` (EncodedCommand) strings to be Base64-encoded UTF-16LE (Unicode) byte arrays. Decoding `SQBFAFgA` yields:
* **Base64 decoded bytes (hex)**: `49 00 45 00 58 00`
* **Unicode representation**: `I` `E` `X`
* **Translated Command**: `IEX` (alias for **`Invoke-Expression`**)

`Invoke-Expression` evaluates and runs any string passed to it as a command, commonly used by attackers to execute fileless code directly in memory.

### Phase 4: Staging & Command & Control (C2)
Once PowerShell initialized, it initiated an outbound HTTP request to pull down the script `a.ps1`.

**Evidence (`endpoint_events.log` & `easy_logs.csv`):**
```log
2026-05-02 09:16:24 host=FIN-WS23 process=powershell.exe netconn=198.51.100.24:80
```
And from the detailed activity log:
```log
2026-05-02 09:16:24,FIN-WS23,finance.aarti,powershell.exe,Downloaded stage from hxxp://198.51.100.24/a.ps1
```

---

## 5. Security Recommendations & Hardening

To mitigate future risk and protect the enterprise from macro-based initial access vectors, the following controls should be implemented:

1. **Disable Office Macros Globally**: 
   * Configure Group Policy (GPO) to **block macros from running in Office files downloaded from the Internet** (ADMX settings for Microsoft Word/Excel).
2. **Restrict Scripting Hosts via AppLocker/WDAC**:
   * Block non-admin execution of `powershell.exe` and `wscript.exe`/`cscript.exe`.
   * Enforce **Constrained Language Mode (CLM)** for PowerShell to prevent access to advanced Windows API capabilities.
3. **Attack Surface Reduction (ASR) Rules**:
   * Enable the Microsoft Defender ASR rule: **"Block Office applications from creating child processes"** (this would have blocked `WINWORD.EXE` from spawning `powershell.exe`).
4. **Email Security Gateway Rules**:
   * Block incoming emails containing macro-enabled attachment extensions (`.docm`, `.xlsm`, `.pptm`).
   * Flag incoming external emails that mimic trusted internal or big-tech domains (e.g., look-alike domains containing numbers or minor misspellings like `micr0soft`).
5. **Continuous Security Awareness Training**:
   * Run targeted phishing simulations focusing on urgent invoice lures for employees in financial and administrative departments.
