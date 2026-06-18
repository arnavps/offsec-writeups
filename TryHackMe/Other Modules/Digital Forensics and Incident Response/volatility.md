# Volatility

## Overview

Volatility is the industry-standard open-source memory forensics framework. It analyses RAM dumps from Windows, Linux, and macOS systems to extract runtime state information that is not available from disk alone — running processes, network connections, loaded DLLs, injected code, and kernel-level artifacts. This room covers Volatility 3 (Python 3), its plugin structure, process and network analysis, and advanced malware hunting capabilities.

---

## Topics Covered

- What Volatility is and why memory forensics matters
- Memory extraction methods
- Volatility 3 vs Volatility 2: key differences
- OS-specific plugin syntax
- Image identification: `windows.info`
- Process analysis: `pslist`, `psscan`, `pstree`
- Network analysis: `netstat`
- DLL analysis: `dlllist`
- Malware hunting: `malfind`, `yarascan`
- Advanced plugins: SSDT hooks, drivers, kernel modules

---

## Key Concepts

### Why Memory Forensics?

Memory forensics captures the **runtime state** of a system — what was happening at the moment of capture. Disk forensics shows what files exist; memory forensics shows what was actively running. This includes:
- Processes that never wrote to disk (fileless malware)
- Network connections active at the time of capture
- Decrypted versions of encrypted files in memory
- Injected code that hides in legitimate processes
- Rootkit hooks in the kernel

### Memory Extraction Methods

| Tool | Notes |
|------|-------|
| FTK Imager | GUI-based, reliable for Windows |
| Redline | FireEye agent-based collection |
| DumpIt.exe | Simple single-executable memory dump |
| win32dd / win64dd | Kernel-mode drivers for memory acquisition |
| Memoryze | FireEye tool with analysis capabilities |
| FastDump | Fast memory acquisition utility |

Extraction from bare-metal systems takes time — plan accordingly.

**Virtual machine memory files** (no additional tools required):
| Hypervisor | Memory file extension |
|------------|----------------------|
| VMware | `.vmem` |
| Hyper-V | `.bin` |
| Parallels | `.mem` |
| VirtualBox | `.sav` (partial memory only) |

---

### Volatility 3 vs Volatility 2

| Aspect | Volatility 2 | Volatility 3 |
|--------|-------------|-------------|
| Language | Python 2 | Python 3 |
| Profiles | Required — specific to OS version and build | Deprecated — auto-detected |
| Plugin naming | Simple name (e.g. `pslist`) | OS-prefixed (e.g. `windows.pslist.PsList`) |
| Plugin syntax | `vol.py --profile=<profile> pslist` | `python3 vol.py -f <file> windows.pslist.PsList` |

Profile selection was a significant overhead in Volatility 2 — Volatility 3 eliminates this.

---

### OS Plugin Prefixes

```
windows.    — Windows plugins
linux.      — Linux plugins
mac.        — macOS plugins
```

---

## Practical Examples / Demonstrations

### Basic Syntax

```bash
python3 vol.py -f <memory_file> <plugin>
```

### Identify Host Information

```bash
python3 vol.py -f memory.vmem windows.info
python3 vol.py -f memory.vmem linux.info
python3 vol.py -f memory.vmem mac.info
```

Outputs OS version, architecture, build number, and other system details.

---

### Process Analysis

#### `windows.pslist` — Active Process List

Lists processes from the doubly-linked list in memory (equivalent to Task Manager).

```bash
python3 vol.py -f memory.vmem windows.pslist.PsList
```

- Shows current and terminated processes (with exit times)
- **Limitation:** Rootkits can unlink themselves from this list to hide

#### `windows.psscan` — Process Scan

Scans memory for `_EPROCESS` data structures directly — finds processes not in the linked list.

```bash
python3 vol.py -f memory.vmem windows.psscan.PsScan
```

- Catches hidden/unlinked processes that `pslist` misses
- **Limitation:** May generate false positives

#### `windows.pstree` — Process Tree

Lists processes in parent-child hierarchy using the same method as `pslist`.

```bash
python3 vol.py -f memory.vmem windows.pstree.PsTree
```

- Visualises the process tree — useful for spotting suspicious parent-child relationships (e.g. `winword.exe` spawning `cmd.exe`)

---

### Network Analysis

#### `windows.netstat` — Network Connections

Lists all memory structures associated with network connections at capture time.

```bash
python3 vol.py -f memory.vmem windows.netstat
```

> Note: This plugin can be unstable on older Windows builds. An alternative is to use `bulk_extractor` to extract a PCAP file from the memory dump for network analysis.

---

### DLL Analysis

#### `windows.dlllist` — DLL List

Lists all DLLs loaded by each process.

```bash
python3 vol.py -f memory.vmem windows.dlllist.DllList
```

- Filter to a specific process or DLL when investigating known malicious indicators
- Unsigned or unexpected DLLs are suspicious

---

### Malware Hunting Plugins

#### `windows.malfind` — Injected Code Detection

Scans the heap and identifies memory regions with executable permissions (RWX or RX) and no corresponding file on disk — a common indicator of code injection.

```bash
python3 vol.py -f memory.vmem windows.malfind.Malfind
```

Output includes:
- Process name and PID
- Memory region offset address
- Hex, ASCII, and disassembly view of the suspected region
- MZ header (`4D 5A`) in the output indicates a Windows PE file was injected

#### `windows.yarascan` — YARA Rule Scanning

Scans memory against YARA rules for strings, patterns, or compound conditions.

```bash
python3 vol.py -f memory.vmem windows.yarascan.YaraScan --yara-rules "rule_string"
python3 vol.py -f memory.vmem windows.yarascan.YaraScan --yara-file /path/to/rules.yar
```

---

### Advanced Malware Analysis Plugins

#### SSDT Hooking — `windows.ssdt`

SSDT (System Service Descriptor Table) contains pointers to Windows kernel functions. Rootkits modify these pointers to intercept system calls.

```bash
python3 vol.py -f memory.vmem windows.ssdt
```

Output will include hundreds of entries — compare against a known-clean baseline to identify modified pointers.

#### Kernel Modules

**`windows.modules`** — lists loaded kernel modules:
```bash
python3 vol.py -f memory.vmem windows.modules
```

**`windows.driverscan`** — scans for driver structures (finds hidden drivers that `modules` misses):
```bash
python3 vol.py -f memory.vmem windows.driverscan
```

**Additional advanced plugins:**
- `windows.modscan` — scans for kernel module structures
- `windows.driverirp` — lists IRP handler functions for drivers
- `windows.callbacks` — kernel callback routines
- `windows.idt` — Interrupt Descriptor Table
- `windows.apihooks` — API hook detection
- `windows.moddump` — dumps kernel module binaries
- `windows.handles` — lists process handles

> Some advanced plugins are available only in Volatility 2 or as third-party plugins.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Memory forensics | Analysing RAM dumps to extract runtime state information |
| Volatility | Open-source memory forensics framework |
| Profile | OS-specific configuration for parsing memory (Volatility 2 only — deprecated in V3) |
| `_EPROCESS` | Windows kernel structure representing a process — scanned by `psscan` |
| Code injection | Inserting malicious code into the memory space of a legitimate process |
| Fileless malware | Malware that operates entirely in memory without writing to disk |
| SSDT | System Service Descriptor Table — kernel lookup table for system functions |
| RWX | Read/Write/Execute memory permissions — suspicious when found in heap regions |
| YARA | Pattern matching language for identifying malware characteristics in files or memory |
| Hooking | Modifying system function pointers to intercept and redirect execution |

---

## Plugin Quick Reference

| Plugin | Command | Purpose |
|--------|---------|---------|
| System info | `windows.info` | OS details from memory |
| Process list | `windows.pslist.PsList` | Active processes from linked list |
| Process scan | `windows.psscan.PsScan` | Find hidden/unlinked processes |
| Process tree | `windows.pstree.PsTree` | Parent-child process hierarchy |
| Network | `windows.netstat` | Active network connections |
| DLLs | `windows.dlllist.DllList` | Loaded DLLs per process |
| Code injection | `windows.malfind.Malfind` | RWX memory regions suggesting injection |
| YARA scan | `windows.yarascan.YaraScan` | Pattern matching in memory |
| SSDT hooks | `windows.ssdt` | Kernel system call table hook detection |
| Kernel modules | `windows.modules` | Loaded kernel modules |
| Driver scan | `windows.driverscan` | Scan for hidden drivers |

---

## Real-World Relevance

- Memory forensics is essential for detecting fileless malware — threats that never touch disk are invisible to traditional endpoint security and disk forensics
- `malfind` is a first-pass tool in malware analysis — every analyst runs it early in a memory investigation
- `psscan` vs `pslist` discrepancy (processes in one but not the other) is a reliable indicator of rootkit activity
- SSDT hook detection is used when investigating kernel-level rootkits — banking trojans and advanced persistent threats commonly use SSDT hooking
- Volatility is used by both incident responders and malware analysts — understanding its plugins is a core competency for DFIR professionals

---

## Key Learnings

- Volatility 3 auto-detects the OS profile — no manual profile selection needed
- Plugin names are OS-prefixed: `windows.pslist.PsList`, not just `pslist`
- `pslist` uses the process list; `psscan` scans memory directly — use both to catch hidden processes
- `malfind` identifies suspicious RWX memory regions and MZ headers indicating code injection
- `netstat` shows network connections; use `bulk_extractor` as an alternative for unreliable builds
- Advanced hunting: SSDT for kernel hooks, `modules`/`driverscan` for malicious kernel drivers

---

## Conclusion

Volatility is the definitive tool for Windows memory forensics. Its plugin architecture provides targeted capabilities for every stage of memory analysis — from basic process listing to advanced rootkit detection. The transition from Volatility 2 to Volatility 3 simplified the workflow by eliminating profile management while maintaining the breadth of analysis capabilities. Proficiency with Volatility's core plugins and an understanding of what each reveals about memory-resident threats is a foundational skill for any DFIR practitioner.
