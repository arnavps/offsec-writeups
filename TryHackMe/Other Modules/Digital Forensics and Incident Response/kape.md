# KAPE

## Overview

KAPE (Kroll Artifact Parser and Extractor) is a portable, extensible forensic collection and processing tool. It significantly reduces the time needed to gather and parse forensic artifacts from a live Windows system by automating both collection and analysis in a single workflow. KAPE uses two core concepts — Targets (what to collect) and Modules (how to process what was collected) — to provide a flexible, scriptable triage capability.

---

## Topics Covered

- What KAPE is and how it works
- Targets: collection configuration
- Compound Targets
- Modules: processing collected artifacts
- The bin directory
- GUI vs CLI usage
- Building KAPE commands from scratch
- Batch mode (`_kape.cli`)

---

## Key Concepts

### What is KAPE?

KAPE serves two primary purposes:
1. **Collect files** (Targets) — copy forensic artifacts from a live system or disk image
2. **Process collected files** (Modules) — run tools against the collected artifacts to extract information

KAPE is portable — it requires no installation and can run from a USB drive or network share. It can collect from:
- Live systems
- Mounted disk images
- F-response utility (remote acquisition)

### How Collection Works

KAPE adds target files to a queue and copies them in two passes:
1. **First pass** — copies files not locked by the OS
2. **Second pass** — uses raw disk reads to bypass OS locks and copy restricted files

Files are saved with original timestamps and directory structure intact.

---

### Targets

A Target defines which files to collect. Target files use the `.tkape` extension and are stored in `KAPE\Targets`.

**Example Target structure (`Prefetch.tkape`):**
- Specifies file mask: `*.pf`
- Specifies paths: `C:\Windows\prefetch` and `C:\Windows.old\prefetch`

The `C:\Windows.old` path contains historical artifacts retained after a Windows upgrade — useful for finding artifacts from a previous OS installation.

#### Target Directory Structure

| Directory | Purpose |
|-----------|---------|
| `Targets\` | Active target configurations |
| `Targets\Compound\` | Compound Targets collecting multiple artifact types together |
| `Targets\!Disabled\` | Targets kept but excluded from active lists |
| `Targets\!Local\` | Custom targets not synced to KAPE's GitHub repository |

#### Compound Targets

Compound Targets collect multiple artifact types with a single command. Examples:
- `!BasicCollection` — basic triage artifacts
- `!SANS_triage` — SANS-recommended triage set
- `KapeTriage` — comprehensive triage collection
- `EvidenceOfExecution` — Prefetch, RecentFileCache, AmCache, Syscache

---

### Modules

A Module defines a tool to run against collected artifacts. Module files use the `.mkape` extension and are stored in `KAPE\Modules`.

**Module configuration includes:**
- Executable to run
- Command-line parameters
- Output format (CSV, TXT, JSON)
- Output filename

#### The bin Directory

`KAPE\bin` contains executables that are not natively present on most Windows systems but are needed by modules — primarily Eric Zimmerman's tools (PECmd, AppCompatCacheParser, RegRipper, etc.).

KAPE runs executables from either the `bin` directory or a full path.

---

## Practical Examples / Demonstrations

### GUI Usage

KAPE includes a GUI (`gkape.exe`) for selecting Targets and Modules visually. For CLI usage, commands are built manually.

### CLI Command Structure

Required flags when collecting Targets:
- `--tsource` — source path to collect from
- `--target` — Target name to use
- `--tdest` — destination directory for collected files

Required flags when processing with Modules:
- `--module` — Module name to use
- `--mdest` — destination directory for module output

### Building a Full Collection and Processing Command

```
kape.exe --tsource C: --target KapeTriage --tdest C:\Users\thm-4n6\Desktop\Target --mdest C:\Users\thm-4n6\Desktop\module --module !EZParser
```

**Breakdown:**
- `--tsource C:` — collect from the C: drive
- `--target KapeTriage` — use the KapeTriage Compound Target
- `--tdest ...Target` — save collected files here
- `--mdest ...module` — save module output here
- `--module !EZParser` — process with the !EZParser Compound Module (runs all Eric Zimmerman's parsers)

**To flush the target destination before collecting:**
```
kape.exe --tsource C: --target KapeTriage --tdest C:\...\Target --tflush ...
```

> Must be run in an elevated (Administrator) command prompt.

### Batch Mode

KAPE supports batch mode via a `_kape.cli` file in the KAPE binary directory. When `kape.exe` is run as Administrator, it checks for this file and executes the commands within.

**`_kape.cli` content:**
```
--tsource C: --target KapeTriage --tdest C:\Users\thm-4n6\Desktop\Target --mdest C:\Users\thm-4n6\Desktop\module --module !EZParser
```

Useful for deploying KAPE to systems where someone else will run it — all configuration is pre-defined, and the operator only needs to right-click and run as Administrator.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Target | KAPE configuration defining which files to collect |
| Module | KAPE configuration defining a tool to run against collected files |
| `.tkape` | Target configuration file extension |
| `.mkape` | Module configuration file extension |
| Compound Target | A Target that collects multiple other Targets together |
| `!EZParser` | Compound Module running all of Eric Zimmerman's parsing tools |
| `KapeTriage` | Compound Target for comprehensive triage collection |
| `bin` directory | Contains third-party executables used by KAPE modules |
| `_kape.cli` | Batch mode file — commands executed when KAPE runs as Administrator |
| `--tflush` | Flag to clear the target destination before collecting |
| Raw disk read | KAPE's method for copying OS-locked files in the second collection pass |

---

## Key Flags Reference

| Flag | Description |
|------|-------------|
| `--tsource` | Source path for Target collection |
| `--target` | Target name to collect |
| `--tdest` | Destination for collected Target files |
| `--tflush` | Flush/clear the target destination before collecting |
| `--msource` | Source path for Module processing (defaults to `--tdest` if omitted) |
| `--module` | Module name to run |
| `--mdest` | Destination for Module output |
| `--gui` | Launch KAPE GUI |

---

## Real-World Relevance

- KAPE is a standard tool in enterprise DFIR engagements — it is often the first tool run on a compromised endpoint to quickly gather all relevant artifacts
- The combination of `KapeTriage` Target and `!EZParser` Module is a widely used triage workflow that produces parsed, investigation-ready output within minutes
- Raw disk read capability makes KAPE effective even on locked system files that standard file copy tools cannot access
- Compound Targets and Modules mean analysts do not need to know every artifact location — KAPE knows where to look and what to do with what it finds
- Batch mode enables deployment at scale — KAPE can be distributed to hundreds of endpoints with a single `_kape.cli` configuration

---

## Key Learnings

- KAPE collects (Targets) and processes (Modules) forensic artifacts — two distinct but chained functions
- Targets use two-pass collection: normal copy first, raw disk reads for locked files second
- Compound Targets combine multiple artifact types — `KapeTriage` is the standard comprehensive option
- Modules run tools against collected files — `!EZParser` runs all Eric Zimmerman's parsers automatically
- The `bin` directory contains tools (primarily EZ tools) not natively present on Windows
- Batch mode via `_kape.cli` enables deployment without operator configuration knowledge

---

## Conclusion

KAPE dramatically accelerates the forensic artifact collection and analysis workflow. By combining automated target collection with modular processing, it reduces what would otherwise take hours of manual collection and tool-by-tool parsing into a process that completes in minutes. Understanding Targets, Modules, and how to build CLI commands gives an analyst full control over KAPE's capabilities and enables rapid, consistent triage across any number of endpoints.
