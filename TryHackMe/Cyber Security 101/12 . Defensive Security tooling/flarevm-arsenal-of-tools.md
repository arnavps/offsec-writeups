# FlareVM: Arsenal of Tools

## Overview

FlareVM (Forensics, Logic Analysis, and Reverse Engineering VM) is a Windows-based toolkit curated by the FLARE Team at FireEye/Mandiant. It is a comprehensive collection of pre-installed, specialised tools designed for reverse engineers, malware analysts, incident responders, forensic investigators, and penetration testers. This room introduces the major tool categories in FlareVM and provides a practical overview of the key tools used in initial investigations: Procmon, Process Explorer, HxD, Wireshark, CFF Explorer, PEStudio, and FLOSS.

---

## Topics Covered

- What FlareVM is and who it is for
- Tool categories: reverse engineering, disassembly, static/dynamic analysis, forensics, network analysis, file analysis, scripting
- Sysinternals Suite overview
- Deep dive: Procmon, Process Explorer, HxD, Wireshark, CFF Explorer, PEStudio, FLOSS

---

## Key Concepts

### What is FlareVM?

FlareVM is a curated Windows VM environment containing hundreds of tools across every major security analysis discipline. Unlike REMnux (Linux-based), FlareVM runs on Windows — making it the environment of choice when analysing Windows malware in its native operating system context.

It is used by:
- **Malware analysts** — understanding what a binary does
- **Reverse engineers** — disassembling and decompiling binaries
- **Incident responders** — triaging and investigating compromised systems
- **Forensic investigators** — collecting and analysing digital evidence
- **Penetration testers** — post-exploitation and red team tooling

---

### Tool Categories

#### Reverse Engineering and Debugging

| Tool | Description |
|------|-------------|
| Ghidra | NSA-developed open-source reverse engineering suite — disassembly, decompilation, scripting |
| x64dbg | Open-source debugger for x86 and x64 binaries — dynamic analysis with full debugging capability |
| OllyDbg | Assembly-level debugger — older but widely documented for x86 analysis |
| Radare2 | Comprehensive open-source RE platform — CLI-based, highly scriptable |
| Binary Ninja | Disassembler and decompiler with an accessible UI and scripting API |
| PEiD | Detects packers, cryptors, and compilers used to produce an executable |

#### Disassemblers and Decompilers

| Tool | Description |
|------|-------------|
| CFF Explorer | PE file editor and analyser — structure examination, hash generation, section inspection |
| Hopper Disassembler | Combined debugger, disassembler, and decompiler |
| RetDec | Open-source machine code decompiler |

#### Static and Dynamic Analysis

| Tool | Description |
|------|-------------|
| Process Hacker | Advanced memory editor and process monitor |
| PEview | Lightweight PE file structure viewer |
| Dependency Walker | Displays DLL dependencies of an executable |
| DIE (Detect It Easy) | Packer, compiler, and cryptor identification tool |

#### Forensics and Incident Response

| Tool | Description |
|------|-------------|
| Volatility | RAM dump analysis framework — memory forensics |
| Rekall | Memory forensics framework focused on incident response |
| FTK Imager | Disk image acquisition and analysis |

#### Network Analysis

| Tool | Description |
|------|-------------|
| Wireshark | Network protocol analyser — captures and inspects traffic |
| Nmap | Network scanning and vulnerability detection |
| Netcat | Read and write data across network connections |

#### File Analysis

| Tool | Description |
|------|-------------|
| FileInsight | Binary file viewer and editor |
| Hex Fiend | Lightweight hex editor |
| HxD | Binary file viewing and editing via hex editor |

#### Scripting and Automation

| Tool | Description |
|------|-------------|
| Python | Automation tooling and analysis scripts |
| PowerShell Empire | PowerShell-based post-exploitation framework |

#### Sysinternals Suite

Microsoft's Sysinternals collection provides deep Windows system visibility:

| Tool | Description |
|------|-------------|
| Autoruns | Shows all executables configured to run at system startup |
| Process Explorer | Detailed view of running processes, parent-child relationships, DLL loads |
| Process Monitor | Real-time logging of file system, registry, and process/thread activity |

---

## Key Tools for Initial Investigation

These are the tools used at the beginning of most malware analysis or forensic investigations on FlareVM.

### Process Monitor (Procmon)

Process Monitor records all file system, registry, network, and process/thread activity in real time on a Windows system. It is invaluable for:
- **Malware research** — observe what a binary does the moment it runs (what files it creates, what registry keys it modifies, what network connections it makes)
- **Troubleshooting** — diagnose application failures by watching system interactions
- **Forensic investigation** — build a timeline of system activity during an incident

Key capability: filter by process name, operation type, or result to cut through noise and focus on relevant activity.

---

### Process Explorer (Procexp)

Process Explorer provides a detailed, hierarchical view of all running processes — more information than Windows Task Manager. Key features:

- **Parent-child process tree** — see which process spawned which (e.g. `Word.exe` spawning `cmd.exe` is immediately suspicious)
- **DLL inspection** — see exactly which DLLs each process has loaded
- **File path visibility** — shows the full path of each process executable
- **User account** — shows which account each process is running under

In malware analysis, unexpected parent-child relationships (e.g. a browser spawning PowerShell) are a key detection indicator.

---

### HxD (Hex Editor)

HxD is a fast, flexible hex editor for viewing and editing binary files at the byte level. Uses in malware analysis:

- **File type identification** — check the magic bytes at the start of a file (e.g. `4D 5A` in little-endian = `MZ` = Windows PE executable)
- **Manual data extraction** — read embedded strings, configurations, or encoded payloads directly from binary
- **Data Recovery** — examine and repair corrupted binary files
- **Comparison** — diff two versions of a file at the byte level

**Reading hex output:**
- Left panel: hexadecimal representation of file bytes
- Right panel: ASCII interpretation of the same bytes
- The `MZ` header (`4D 5A`) at the start of a file identifies it as a Windows PE executable regardless of file extension

The **Data Inspector** panel shows the value of selected bytes in multiple data types (integer, float, etc.) — useful for interpreting embedded numeric values.

---

### CFF Explorer

CFF Explorer is a PE (Portable Executable) file analyser and editor. Primary uses in investigation:

- **Hash generation** — produces MD5 and SHA-1/SHA-256 hashes for file integrity verification and threat intelligence lookups
- **File metadata** — displays compilation date, architecture (32/64-bit), file type
- **PE structure inspection** — sections, imports, exports, resources
- **Signature validation** — verify whether system files have been altered

Example: A 64-bit PE file created on September 23, 2024 with specific hashes can be cross-referenced against threat intelligence databases (VirusTotal, MalwareBazaar) for known malware matches.

---

### Wireshark

Wireshark is the standard network protocol analyser. In malware analysis and incident response:

- **Traffic capture and inspection** — record all network traffic during dynamic analysis
- **Protocol dissection** — examine DNS queries, HTTP requests, TLS handshakes in detail
- **Suspicious connection identification** — spot unexpected outbound connections, unusual protocols, or data exfiltration
- **C2 traffic analysis** — identify command-and-control communication patterns

**TLS traffic note:** TLSv1.2/1.3 encrypted traffic appears as ciphertext in Wireshark. The connection metadata (IP, port, timing, certificate) is still visible, but content requires decryption keys to inspect.

---

### PEStudio

PEStudio performs **static analysis** of PE files — inspecting properties and characteristics without execution. Key capabilities:

- **Import analysis** — what Windows API functions does the binary import? (`VirtualAlloc`, `CreateRemoteThread`, `WinExec` are suspicious)
- **String extraction** — embedded URLs, file paths, registry keys, commands
- **Entropy calculation** — high entropy sections suggest packing or encryption
- **Indicator detection** — flagged behaviours or strings known to be associated with malicious activity
- **Threat score** — a low-to-high indicator rating based on observed properties

**Example:** `PsExec.exe` scanned in PEStudio may show low-to-medium indicators and moderately high entropy. In isolation, PsExec is a legitimate Sysinternals tool. Found on a compromised system where remote code execution is unexpected, it warrants investigation as a potential post-exploitation tool.

---

### FLOSS (FLARE Obfuscated String Solver)

FLOSS automatically extracts and de-obfuscates strings from malware binaries using advanced static analysis. It goes beyond the standard `strings` utility by:

- **Extracting stack strings** — strings built on the stack at runtime rather than stored statically (a common obfuscation technique)
- **Decoding encoded strings** — XOR-encoded or otherwise obfuscated strings are decoded automatically
- **Identifying tight loops** — patterns typically associated with decoding routines

Standard `strings` only finds strings stored literally in the binary. FLOSS finds strings that are constructed dynamically and would be invisible to `strings`.

Output from FLOSS can be loaded into IDA Pro or Binary Ninja for deeper analysis.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| FlareVM | Windows-based malware analysis and reverse engineering toolkit |
| PE | Portable Executable — the standard Windows executable format (.exe, .dll, .sys) |
| MZ header | `4D 5A` — the magic bytes identifying a Windows PE file |
| Magic bytes | The first bytes of a file that identify its type regardless of extension |
| Entropy | A measure of randomness in data — high entropy often indicates packing or encryption |
| Packer | Tool that compresses or encrypts an executable to obscure its contents |
| DLL | Dynamic Link Library — a shared code module loaded into processes at runtime |
| Parent-child process | The relationship between a process and the one that spawned it |
| Stack strings | Strings assembled on the stack at runtime to evade static string extraction |
| FLOSS | FLARE Obfuscated String Solver — extracts obfuscated strings from malware |
| Sysinternals | Microsoft's advanced Windows diagnostic and monitoring tool suite |

---

## Workflow / Process

```
Receive suspicious binary for initial investigation
        |
        v
CFF Explorer: generate hashes, check metadata, look up on VirusTotal
        |
        v
HxD: verify magic bytes, inspect raw binary content
        |
        v
PEStudio: static analysis — imports, strings, entropy, indicators
        |
        v
FLOSS: extract obfuscated and stack strings
        |
        v
If proceeding to dynamic analysis (in isolated VM):
        |
        v
Process Monitor: capture all file, registry, network activity during execution
        |
        v
Process Explorer: observe spawned processes, DLL loads, parent-child tree
        |
        v
Wireshark: capture all network traffic generated during execution
        |
        v
Correlate findings from static and dynamic analysis
        |
        v
Document capabilities, IOCs, and ATT&CK TTPs
```

---

## Real-World Relevance

- FlareVM is used by professional malware analysts and DFIR teams for day-to-day analysis of malicious binaries
- The parent-child process relationship in Process Explorer is a primary detection signal — `winword.exe` spawning `powershell.exe` or `cmd.exe` is a well-known Office macro exploitation indicator
- PEStudio's import analysis directly surfaces dangerous API calls before any code is run — functions like `VirtualAllocEx`, `WriteProcessMemory`, and `CreateRemoteThread` are the building blocks of process injection
- FLOSS addresses a specific attacker technique (stack string construction to evade `strings`) — understanding it helps explain why standard string analysis misses certain indicators
- CFF Explorer's hash generation feeds directly into threat intelligence workflows — a SHA-256 hash submitted to VirusTotal can immediately confirm or deny known malware identity
- Entropy analysis in PEStudio catches packed/encrypted malware — if the code section has entropy above ~7.0, the binary is almost certainly packed

---

## Key Learnings

- FlareVM is a Windows-based toolkit covering RE, debugging, static/dynamic analysis, forensics, network analysis, and scripting
- Initial investigation workflow: CFF Explorer (hashes/metadata) → HxD (raw binary) → PEStudio (static analysis) → FLOSS (strings) → Procmon/Procexp/Wireshark (dynamic)
- HxD identifies file types via magic bytes — `4D 5A` (`MZ`) = Windows PE executable
- Process Explorer reveals parent-child process relationships — suspicious spawning patterns indicate exploitation
- PEStudio provides static analysis indicators without execution — imports, entropy, strings, threat scoring
- FLOSS de-obfuscates stack-built and encoded strings invisible to standard `strings` extraction

---

## Additional Notes

- FlareVM is installed on top of a Windows VM — it is not a standalone OS like REMnux
- When doing dynamic analysis, always work in a **snapshot-restored VM** — restore to a clean snapshot before and after each analysis run
- Entropy alone is not definitive — some legitimate software (installers, compressed archives) also has high entropy
- The combination of PEStudio (static) + Procmon (dynamic) gives a comprehensive view of a binary's behaviour and is often sufficient for initial triage
- FLOSS is available as a standalone tool separate from FlareVM — it can be used in any Windows or Linux analysis environment

---

## Conclusion

FlareVM consolidates the tools needed for malware analysis and incident response into a single, curated Windows environment. The key tools covered here — Procmon, Process Explorer, HxD, CFF Explorer, Wireshark, PEStudio, and FLOSS — represent the standard initial investigation toolkit used in professional DFIR and malware analysis work. Each tool addresses a specific aspect of analysis: from raw bytes and static indicators to runtime behaviour and network activity. Understanding when and why to use each one is as important as knowing how to operate them.
