# CAPA: The Basics

## Overview

CAPA (Common Analysis Platform for Artifacts) is an automated static analysis tool developed by the Mandiant FLARE team. It analyses executable files and identifies their capabilities — what the program can do — without executing it. CAPA maps these capabilities to established frameworks including MITRE ATT&CK, the Malware Behavior Catalogue (MBC), and its own namespace taxonomy. This room covers CAPA's output structure, the frameworks it references, and how to interpret results during malware analysis.

---

## Topics Covered

- What CAPA does and why static analysis matters
- CAPA command-line options
- Output structure: file metadata, ATT&CK mapping, MAEC values, MBC, Capabilities
- MITRE ATT&CK framework in CAPA context
- MAEC values: Launcher and Downloader
- MBC: Objectives, Behaviors, Micro-behaviors, Methods
- CAPA namespaces and rule YAML files

---

## Key Concepts

### Static vs Dynamic Analysis

| Type | Description | Risk |
|------|-------------|------|
| **Static analysis** | Inspecting the file without executing it — examining code, strings, imports, structure | No execution risk to the analyst's machine |
| **Dynamic analysis** | Running the file and observing its behaviour in a controlled environment (sandbox) | Requires isolation — the malware executes |

CAPA performs **static analysis**. It reads the binary, applies a rule set, and reports what behaviours the file appears capable of — all without running it.

---

### What CAPA Does

CAPA analyses executable files (PE, ELF, .NET, shellcode, sandbox reports) against a library of rules written in YAML. Each rule describes a specific behaviour. When the binary matches the conditions in a rule, CAPA reports that capability.

CAPA encapsulates reverse engineering knowledge into an automated tool — analysts who are not reverse engineering experts can use it to rapidly understand a binary's likely behaviour.

Primary use cases:
- Malware analysis
- Threat hunting
- Incident response triage
- Understanding binaries before deeper analysis

---

### Command-Line Options

```
capa [file]            Run default analysis
capa -h                Show help and all available parameters
capa file.bin -v       Verbose output — more detail per capability
capa file.bin -vv      Very verbose — maximum detail, longer processing time
```

---

### CAPA Output Structure

CAPA output is divided into blocks:

#### Block 1: File Metadata

Basic information about the analysed file:

| Field | Description |
|-------|-------------|
| `md5` / `sha1` / `sha256` | Cryptographic hashes for file identification |
| `analysis` | How CAPA performed the analysis |
| `os` | Target operating system context |
| `arch` | CPU architecture (x86, x64, etc.) |
| `path` | Location of the analysed file |

---

### MITRE ATT&CK in CAPA

MITRE ATT&CK is a globally recognised knowledge base documenting the tactics and techniques used by threat actors across the full attack lifecycle. CAPA maps detected capabilities to ATT&CK, allowing analysts to contextualise what they find within the attacker's playbook.

**Output format:**

| Format | Example |
|--------|---------|
| `Tactic::Technique::Identifier` | `Defense Evasion::Obfuscated Files or Information::T1027` |
| `Tactic::Technique::Sub-Technique::Identifier.Sub-ID` | `Defense Evasion::Obfuscated Files or Information::Indicator Removal from Tools::T1027.005` |

Reading example:
- `DEFENSE EVASION` = ATT&CK Tactic (the goal)
- `Obfuscated Files or Information` = ATT&CK Technique (the method)
- `T1027` = Technique identifier
- `005` = Sub-technique identifier (if present)

---

### MAEC Values

MAEC (Malware Attribute Enumeration and Characterization) is a standardised language for describing malware attributes. CAPA reports two commonly seen MAEC values:

| MAEC Value | Description |
|------------|-------------|
| **Launcher** | The file exhibits behaviours that trigger specific actions — dropping payloads, activating persistence, connecting to C2 servers, executing functions |
| **Downloader** | The file downloads and executes other files — indicative of a dropper or loader stage in multi-stage malware |

A `Launcher` tag signals the file can initiate malicious activity. A `Downloader` tag indicates multi-stage behaviour where the initial binary fetches the real payload.

---

### Malware Behavior Catalogue (MBC)

MBC provides a structured catalogue of malware objectives and behaviours for standardised analysis and reporting. It complements ATT&CK — MBC references ATT&CK techniques but adds malware-specific behaviours and micro-behaviors not covered there.

#### MBC Output Format

| Format | Example |
|--------|---------|
| `OBJECTIVE::Behavior::Method [Identifier]` | `ANTI-STATIC ANALYSIS::Executable Code Obfuscation::Argument Obfuscation [B0032.020]` |
| `OBJECTIVE::Behavior:: [Identifier]` | `COMMUNICATION::HTTP Communication:: [C0002]` |

The first format includes a **Method** (sub-technique); the second does not.

#### MBC Objectives

| Objective | Description |
|-----------|-------------|
| Anti-Behavioral Analysis | Malware attempts to detect or hinder dynamic analysis tools (sandboxes, debuggers) |
| Anti-Static Analysis | Malware obfuscates its code or data to resist static inspection |
| Collection | Malware identifies and gathers data from the target |
| Command and Control | Malware establishes communication with attacker-controlled infrastructure |
| Credential Access | Malware steals credentials (usernames, passwords) |
| Defense Evasion | Malware bypasses security controls and detection mechanisms |
| Discovery | Malware enumerates the system or network environment |
| Execution | Malware executes unauthorised commands or code |
| Exfiltration | Malware steals and transmits sensitive data |
| Impact | Malware disrupts, damages, or destroys systems and data |
| Lateral Movement | Malware spreads through the network |
| Persistence | Malware maintains a presence on the system over time |
| Privilege Escalation | Malware elevates its access permissions |
| Micro-Objective | Low-level technical behaviours not inherently malicious but commonly abused |

#### MBC Micro-Objectives

Micro-objectives describe low-level technical actions that are not inherently malicious but are frequently exploited in malicious contexts:

| Micro-Objective | Example Behaviours |
|-----------------|-------------------|
| PROCESS | Create process, set thread context, check mutex |
| MEMORY | Allocate memory, change memory protection, free memory |
| COMMUNICATION | DNS, FTP, HTTP, ICMP, SMTP network traffic |
| DATA | Check strings, compress, decode, encode data |

#### Key MBC Behaviors

| Objective | Behavior | Identifier | Description |
|-----------|----------|------------|-------------|
| ANTI-BEHAVIORAL ANALYSIS | Lab Machine Detection | B0009 | Malware checks for virtual environment indicators |
| ANTI-STATIC ANALYSIS | Executable Code Obfuscation | B0032 | Code intentionally obscured to prevent static analysis |
| EXECUTION | Command and Scripting Interpreter | E1059 | Uses `cmd.exe`, PowerShell, Bash, Python, etc. to run commands |
| DISCOVERY | File and Directory Discovery | E1083 | Enumerates files and directories |
| ANTI-STATIC ANALYSIS / DEFENSE EVASION | Obfuscated Files or Information | E1027 | Encodes or encrypts files/payloads to hinder analysis |

#### Key MBC Micro-Behaviors

| Micro-Objective | Micro-Behavior | Identifier | Description |
|-----------------|----------------|------------|-------------|
| MEMORY | Allocate Memory | C0007 | Allocates memory — common in unpacking routines |
| PROCESS | Create Process | C0017 | Creates a process via WMI, shellcode, or suspended process |
| COMMUNICATION | HTTP Communication | C0002 | Initiates HTTP connections |
| DATA | Check String | C0019 | Inspects strings for specific characteristics |
| DATA | Encode Data | C0026 | Encodes data using Base64 or XOR |
| FILE SYSTEM | Create Directory | C0046 | Creates a directory |
| FILE SYSTEM | Delete File | C0047 | Deletes a file |
| FILE SYSTEM | Read File | C0051 | Reads a file |
| FILE SYSTEM | Write File | C0052 | Writes to a file |

#### MBC Methods (Sub-Techniques)

| Behavior | Method | Identifier | Description |
|----------|--------|------------|-------------|
| Executable Code Obfuscation | Argument Obfuscation | B0032.020 | API call arguments computed at runtime to resist analysis |
| Executable Code Obfuscation | Stack Strings | B0032.017 | Strings built and decrypted on the stack at each use, then discarded |
| HTTP Communication | Read Header | C0002.014 | Reads HTTP response headers |
| Encode Data | Base64 | C0026.001 | Encodes data using Base64 |
| Encode Data | XOR | C0026.002 | Encodes data using XOR |
| Obfuscated Files or Information | Encoding-Standard Algorithm | E1027.m02 | Uses a standard algorithm (e.g. Base64) to encode malware |

---

### CAPA Capabilities and Namespaces

CAPA groups its detection rules into a namespace hierarchy:

```
Top-Level Namespace (TLN) → Namespace → Rule YAML File → Capability (Rule Name)
```

The **Capability** in CAPA output is the name of the rule that matched. Rule names correspond directly to YAML filenames (with spaces replaced by dashes).

**Example:**
- Capability: `reference anti-VM strings`
- Rule file: `reference-anti-vm-strings.yml`
- Namespace: `anti-vm/vm-detection`
- TLN: `Anti-Analysis`

**Interpretation:** CAPA detected that the binary references strings associated with VMware, VirtualBox, or other VM environments — a common anti-analysis technique to detect and evade sandbox execution.

#### Key Top-Level Namespaces

| TLN | Purpose |
|-----|---------|
| `anti-analysis` | Rules detecting obfuscation, packing, anti-debugging, and VM detection |
| `collection` | Rules for data enumeration and collection behaviours |
| `communication` | Rules for network communication patterns |
| `data-manipulation` | Rules for data encoding, encryption, and transformation |
| `executable` | Rules examining PE structure and attributes |
| `host-interaction` | Rules for file system, process, and registry interactions |
| `impact` | Rules for destructive or high-impact behaviours |
| `linking` | Rules for dynamic loading of external libraries |
| `load-code` | Rules for runtime code injection and loading |
| `malware-family` | Rules tied to known malware family signatures |
| `persistence` | Rules for scheduled tasks, registry run keys, and other persistence mechanisms |
| `runtime` | Rules identifying programming language or runtime |
| `nursery` | Staging area for rules not yet fully polished |

> The `nursery` TLN contains rules that are functional but not production-ready. A capability appearing under `nursery` (e.g. `reference cryptocurrency strings`) is still a valid detection but may have higher false positive rates.

#### Capability Output Example

```
┌───────────────────────────────────────────┬───────────────────────────────────────────┐
│ Capability                                │ Namespace                                 │
├───────────────────────────────────────────┼───────────────────────────────────────────┤
│ reference Base64 string                   │ data-manipulation/encoding/base64         │
└───────────────────────────────────────────┴───────────────────────────────────────────┘
```

Reading this:
- **Capability:** `reference Base64 string` — the binary references Base64-encoded data
- **TLN:** `data-manipulation` — data transformation behaviour
- **Namespace:** `encoding/base64` — specifically Base64 encoding/decoding
- **Rule file matched:** `reference-base64-string.yml`

---

## Important Terminology

| Term | Meaning |
|------|---------|
| CAPA | Common Analysis Platform for Artifacts — static binary capability analysis tool |
| Static analysis | Analysing a file without executing it |
| PE | Portable Executable — Windows executable file format (.exe, .dll) |
| MITRE ATT&CK | Framework documenting adversary tactics and techniques |
| MAEC | Malware Attribute Enumeration and Characterization — standardised malware description language |
| MBC | Malware Behavior Catalogue — catalogue of malware objectives and behaviours |
| Capability | A named rule in CAPA that matched during analysis |
| Namespace | Hierarchical grouping of related CAPA rules |
| TLN | Top-Level Namespace — the highest level of CAPA's rule grouping |
| Nursery | CAPA's staging TLN for rules not yet fully polished |
| Launcher | MAEC value — file triggers specific actions (drops payloads, connects to C2) |
| Downloader | MAEC value — file downloads and executes additional files |

---

## Workflow / Process

```
Obtain potentially malicious binary for analysis
        |
        v
Run CAPA against the file:
  capa [file]           basic analysis
  capa [file] -v        verbose
  capa [file] -vv       very verbose
        |
        v
Review Block 1: file metadata (hashes, OS, arch)
        |
        v
Review ATT&CK section: map capabilities to adversary tactics/techniques
        |
        v
Review MAEC values: Launcher? Downloader? Multi-stage?
        |
        v
Review MBC section: specific objectives, behaviors, methods
        |
        v
Review Capabilities/Namespace section: what rules matched and why
        |
        v
Combine findings to build a picture of what the binary can do
        |
        v
Feed results into incident response, threat hunting, or deeper analysis
```

---

## Real-World Relevance

- CAPA is used in malware triage during incident response to quickly assess an unknown binary without running it
- ATT&CK mapping from CAPA output allows SOC teams to correlate binary capabilities with other ATT&CK-mapped detections in their SIEM
- Detecting anti-VM strings (B0009) in a sample is a strong indicator that the binary is aware of analysis environments — relevant for sandbox evasion assessment
- MAEC Downloader tagging identifies multi-stage malware droppers — the binary analysed may just be stage 1 of a larger payload chain
- FLARE's public CAPA rule repository on GitHub can be extended with custom rules — detection engineers write new rules for novel malware families
- Understanding MBC behaviors like `Encode Data::Base64 [C0026.001]` and `HTTP Communication [C0002]` maps directly to real attacker TTPs observed in the wild

---

## Key Learnings

- CAPA performs static analysis — no execution required, no risk to the analyst machine
- Output is organised into: file metadata, ATT&CK mapping, MAEC values, MBC section, and Capabilities/Namespaces
- ATT&CK output maps detected capabilities to adversary tactics and techniques with identifiers (T####)
- MAEC values indicate high-level malware roles: Launcher (triggers actions) and Downloader (fetches more payloads)
- MBC Objectives describe the malware's goal; Behaviors describe the method; Methods describe the sub-technique
- Capability names in CAPA output are the rule names — corresponding YAML files are named the same with dashes replacing spaces
- The `nursery` TLN contains valid but less polished rules

---

## Additional Notes

- CAPA's rule set is open source and regularly updated — keep the tool updated for the latest detection coverage
- A binary triggering anti-VM rules (B0009) does not automatically mean it is malicious — some legitimate software performs environment checks too. Context matters
- CAPA does not execute code, so it can miss behaviours that are only revealed at runtime (e.g. dynamically resolved APIs, encrypted payloads that decrypt in memory)
- Combining CAPA (static) with sandbox analysis (dynamic) gives the most complete picture of a binary's capabilities
- CAPA results can be exported as JSON for integration with SIEM or threat intelligence platforms

---

## Conclusion

CAPA dramatically reduces the time required to understand an unknown binary's capabilities by automating what would otherwise require manual reverse engineering. Its output — structured around ATT&CK, MBC, and its own namespace taxonomy — maps directly to the frameworks SOC teams and IR analysts already use. Understanding how to read and interpret CAPA results is a practical and high-value skill for anyone working in malware analysis, threat hunting, or incident response.
