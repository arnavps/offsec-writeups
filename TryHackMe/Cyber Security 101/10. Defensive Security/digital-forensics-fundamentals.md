# Digital Forensics Fundamentals

## Overview

Digital forensics is the branch of forensic science that investigates cyber crimes and digital incidents. It involves applying structured methods and procedures to collect, examine, analyse, and report on evidence from digital devices. This room covers the four-phase forensics process defined by NIST, the different types of digital forensics, evidence acquisition best practices, and the key tools used for Windows forensics.

---

## Topics Covered

- What digital forensics is and why it matters
- The NIST four-phase digital forensics process
- Types of digital forensics
- Evidence acquisition best practices: authorisation, chain of custody, write blockers
- Windows forensics: disk and memory images
- Key forensics tools: FTK Imager, Autopsy, DumpIt, Volatility

---

## Key Concepts

### What is Digital Forensics?

Digital forensics is the application of investigative methods to digital devices to find, preserve, and analyse evidence of criminal or malicious activity. It is used in law enforcement, corporate investigations, and incident response to establish what happened, when, how, and who was responsible.

The range of evidence that can be recovered is broad — files, messages, browsing history, GPS data, call records, deleted content, and more.

---

### The NIST Four-Phase Process

NIST (National Institute of Standards and Technology) defines a structured process for digital forensics investigations, applicable across all case types.

| Phase | Description |
|-------|-------------|
| **Collection** | Identify and collect all relevant digital devices (computers, phones, USBs, hard drives, etc.). Preserve original data integrity — do not alter the evidence. Maintain detailed documentation of everything collected |
| **Examination** | Filter the collected data to extract what is relevant to the case. The raw dataset is often too large to analyse in full — this phase narrows scope. For example: extracting only media files from a specific date range, or isolating data for a specific user account |
| **Analysis** | Correlate the extracted evidence to draw conclusions. Reconstruct the sequence of events chronologically. The approach depends on the case type and available data |
| **Reporting** | Produce a detailed report covering methodology, findings, and where appropriate, recommendations. Includes an executive summary for non-technical stakeholders. The report may be submitted to law enforcement or presented in court |

---

### Types of Digital Forensics

| Type | Focus |
|------|-------|
| **Computer forensics** | Most common type. Investigates desktops, laptops, and personal computers |
| **Mobile forensics** | Investigates smartphones and tablets — call records, messages, GPS locations, app data |
| **Network forensics** | Investigates network-level evidence — primarily traffic logs and packet captures across the whole network |
| **Database forensics** | Investigates intrusions or data modifications within database systems |
| **Cloud forensics** | Investigates data stored on cloud infrastructure — challenging due to limited artefact availability and shared infrastructure |
| **Email forensics** | Analyses emails to determine involvement in phishing campaigns, fraud, or other criminal activity |

---

### Evidence Acquisition Best Practices

#### Proper Authorisation

All evidence collection must be authorised by the relevant legal authorities before it begins. Evidence collected without authorisation may be inadmissible in court and could expose the investigation to legal challenge. This is non-negotiable in any legally significant investigation.

#### Chain of Custody

A chain of custody is a formal document that tracks every piece of evidence from collection to court presentation. It establishes integrity and accountability.

Key fields in a chain of custody document:

- Description of the evidence (name, type, format)
- Name(s) of the individual(s) who collected it
- Date and time of collection
- Storage location of each item
- Record of every access — who accessed it and when

Without a proper chain of custody, the authenticity and reliability of evidence can be challenged. A break in the chain can render evidence inadmissible.

#### Write Blockers

A write blocker is a hardware or software device that prevents any data from being written to a storage medium during acquisition. When a suspect's hard drive is connected to a forensic workstation, background system processes can alter file metadata (timestamps, etc.) — even without the analyst doing anything. A write blocker prevents this entirely.

**Why it matters:** Any modification to the original evidence — even unintentional — can compromise the integrity of the investigation and the admissibility of the evidence.

---

### Windows Forensics

Windows is the most commonly investigated OS in digital forensics due to its prevalence. Evidence is collected in two distinct image types:

#### Disk Image

- A **bit-for-bit copy** of the entire storage device (HDD, SSD)
- Contains **non-volatile** data — persists after shutdown and restart
- Includes: files, documents, media, application data, browser history, registry, deleted file remnants, etc.

#### Memory Image

- A snapshot of the contents of **RAM** at the moment of capture
- Contains **volatile** data — lost permanently when the system is powered off or restarted
- Includes: running processes, open network connections, open files, encryption keys loaded in memory, clipboard contents, logged-in user sessions

> Memory images must be captured **before** the disk image, and ideally before any other action is taken — a shutdown destroys all volatile data.

---

## Key Tools

### FTK Imager

Used for **disk image acquisition** on Windows systems. Provides a graphical interface for creating forensic images in multiple formats (E01, DD, AFF, etc.). Can also browse and analyse disk image contents directly.

**Primary use:** Disk acquisition and basic image browsing.

### Autopsy

An open-source digital forensics platform for **analysing disk images**. Once an image is imported, Autopsy performs extensive automated analysis and provides an investigative interface.

Key features:
- Keyword search across the entire image
- Deleted file recovery
- File metadata extraction
- Extension mismatch detection (files with wrong extensions)
- Timeline analysis
- Browser and email artefact extraction

**Primary use:** Comprehensive disk image analysis.

### DumpIt

A lightweight command-line tool for **memory image acquisition** on Windows. Creates a full RAM dump in formats compatible with analysis tools like Volatility.

**Primary use:** Memory acquisition on Windows.

### Volatility

A powerful open-source framework for **memory image analysis**. It uses plugins to extract specific artefacts from a memory dump. Supports Windows, Linux, macOS, and Android memory images.

Example plugin uses:
- List running processes at the time of capture
- Extract network connections
- Dump loaded DLLs
- Recover registry hives from memory
- Detect injected code or malicious processes

**Primary use:** Comprehensive memory image analysis.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Digital forensics | Investigative methods applied to digital devices to find and analyse evidence |
| NIST | National Institute of Standards and Technology — defines frameworks for digital forensics and cybersecurity |
| Chain of custody | Formal document tracking the handling of evidence from collection to court |
| Write blocker | Device that prevents writes to evidence media during acquisition |
| Disk image | Bit-for-bit copy of a storage device — contains non-volatile data |
| Memory image | Snapshot of RAM contents — contains volatile data |
| Volatile data | Data that is lost when the system is powered off (RAM contents) |
| Non-volatile data | Data that persists after shutdown (stored on disk) |
| Artefact | A piece of digital evidence — a file, log entry, registry key, network connection, etc. |

---

## Workflow / Process

```
Crime scene / incident identified
        |
        v
Obtain proper authorisation for evidence collection
        |
        v
Collection phase:
  - Identify all relevant devices
  - Capture memory image FIRST (volatile data)
  - Capture disk image using write blocker
  - Document everything in chain of custody
        |
        v
Examination phase:
  - Filter collected data for relevant items
  - Extract targeted artefacts (specific user, timeframe, file type)
        |
        v
Analysis phase:
  - Correlate evidence across sources
  - Reconstruct timeline of events
  - Draw conclusions relevant to the case
        |
        v
Reporting phase:
  - Document methodology and findings
  - Include executive summary
  - Submit to law enforcement / management
```

---

## Real-World Relevance

- Digital forensics is a core discipline in incident response — after a breach is contained, forensics determines what happened and how
- Chain of custody is legally critical — failure to maintain it can result in evidence being thrown out of court
- Volatility is one of the most widely used tools in memory forensics and is a key skill in DFIR (Digital Forensics and Incident Response) roles
- Autopsy is used by both law enforcement and enterprise security teams for disk investigation
- Memory forensics is increasingly important in detecting fileless malware — threats that operate entirely in RAM and leave minimal disk artefacts
- Cloud forensics is a growing challenge as organisations migrate infrastructure — limited logging and shared infrastructure complicate evidence collection

---

## Key Learnings

- Digital forensics follows a four-phase process: Collection, Examination, Analysis, Reporting
- Memory images must be captured before disk images — volatile data is lost on shutdown
- Write blockers prevent accidental evidence modification during acquisition
- Chain of custody establishes integrity and admissibility of evidence
- Proper authorisation must precede all evidence collection
- FTK Imager and Autopsy handle disk forensics; DumpIt and Volatility handle memory forensics

---

## Additional Notes

- The order of volatility matters in evidence collection: capture the most volatile data first (memory, then disk)
- Forensic images are typically hashed (MD5, SHA-1, SHA-256) immediately after acquisition — the hash is recorded and used later to prove the image has not been altered
- Deleted files are often recoverable from disk images — forensics tools like Autopsy can reconstruct deleted data using file system metadata
- Time zone handling is important in forensics — all timestamps should be normalised to a consistent timezone (typically UTC) before analysis

---

## Conclusion

Digital forensics provides the structured, evidence-based approach needed to investigate cyber crimes and security incidents. The NIST four-phase framework — Collection, Examination, Analysis, Reporting — ensures investigations are thorough, documented, and legally defensible. Understanding the difference between volatile and non-volatile evidence, the importance of the chain of custody, and the tools used to acquire and analyse Windows evidence is foundational for anyone working in DFIR, SOC analysis, or incident response.
