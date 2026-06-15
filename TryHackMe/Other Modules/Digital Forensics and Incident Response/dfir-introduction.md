# DFIR: An Introduction

## Overview

DFIR (Digital Forensics and Incident Response) is the discipline that combines the technical investigation of digital devices with the operational response to security incidents. It provides the methodology and tooling to identify attacker activity, determine the extent of compromise, remove the threat, and restore normal operations. This room introduces what DFIR is, who performs it, core concepts like artifacts and chain of custody, and the incident response frameworks (NIST and SANS/PICERL) used to guide the process.

---

## Topics Covered

- What DFIR is and why it matters
- Skills required for DFIR professionals
- Artifacts: what they are and how they are collected
- Evidence preservation and write protection
- Chain of custody
- Order of volatility
- Timeline creation
- Key DFIR tools
- NIST and SANS incident response frameworks

---

## Key Concepts

### What is DFIR?

DFIR stands for Digital Forensics and Incident Response. It covers:
- **Digital Forensics** — collecting and examining evidence from digital devices (computers, media, smartphones) to identify what happened
- **Incident Response** — the operational process of detecting, containing, eradicating, and recovering from a security incident

The two disciplines are deeply interdependent: IR defines the scope and goals of the forensic investigation, and forensics provides the evidence that drives IR decisions.

### Why DFIR Matters

| Use Case | Description |
|----------|-------------|
| Identifying attacker activity | Separating real incidents from false alarms |
| Removing the attacker | Ensuring complete eradication with forensic confidence |
| Determining breach scope | Understanding what was accessed, modified, or exfiltrated |
| Finding root cause | Identifying the vulnerability or misconfiguration that was exploited |
| Understanding attacker behaviour | Informing future preventative controls |
| Threat intelligence sharing | Contributing findings to the broader security community |

### Who Performs DFIR?

DFIR professionals require skills from both parent disciplines:
- **Digital Forensics** — expertise in identifying artifacts and evidence of human activity across digital devices
- **Incident Response** — cybersecurity expertise to interpret forensic findings in the context of an attack

In practice, these roles often overlap — the same analyst performs both the forensic collection and the IR analysis.

---

### Artifacts

An **artifact** is a piece of evidence that points to activity performed on a system. In DFIR, artifacts are collected to support or refute a hypothesis about attacker activity.

**Example:** If claiming an attacker used a specific Windows registry key for persistence, that registry key is the artifact supporting the claim.

Artifacts can be found in:
- **File system** — files, directories, metadata, timestamps
- **Memory (RAM)** — running processes, network connections, loaded modules
- **Network** — traffic logs, connection records, DNS queries

Enterprise environments primarily consist of Windows and Linux systems. Windows is common for endpoints and servers (Active Directory, Exchange). Linux is common for web servers, databases, and infrastructure services.

---

### Evidence Preservation

Forensic analysis inherently alters the evidence — even reading a file updates its access timestamp. To maintain integrity:

1. Collect evidence and immediately **write-protect** it
2. Create a **forensic copy** (bit-for-bit image)
3. Perform all analysis on the copy, never on the original
4. If the copy is corrupted, the original remains intact for a new copy

---

### Chain of Custody

The chain of custody documents every person who has handled the evidence from collection to court presentation. A broken chain of custody raises questions about whether the evidence was tampered with.

A chain of custody document records:
- Description of the evidence (name, type, format)
- Names of individuals who collected it
- Date and time of collection
- Storage location
- Every access — who accessed it and when

Without a maintained chain of custody, the integrity of the evidence can be challenged and it may be ruled inadmissible in legal proceedings.

---

### Order of Volatility

Some evidence is lost permanently when a system is powered off. Evidence must be collected in order of how quickly it disappears:

| Evidence Source | Volatility | Notes |
|----------------|-----------|-------|
| RAM / Memory | Most volatile | Lost on shutdown or reboot — collect first |
| Network connections | Very volatile | Active connections terminate when disrupted |
| Running processes | Volatile | Cleared on shutdown |
| Disk / File system | Least volatile | Persists after power loss |

Always capture more volatile evidence before less volatile evidence.

---

### Timeline Creation

Once artifacts are collected and preserved, a **timeline** organises all events in chronological order. Timeline creation:
- Puts activities from multiple sources in sequence
- Provides perspective on the progression of the attack
- Helps identify gaps or inconsistencies in the evidence

This is a critical step for turning raw artifacts into a coherent narrative of what happened.

---

### Key DFIR Tools

| Tool | Purpose |
|------|---------|
| **Eric Zimmerman's tools** | Suite of Windows forensic utilities covering registry, file system, timeline, and more |
| **KAPE** | Kroll Artifact Parser and Extractor — automates collection and parsing of forensic artifacts |
| **Autopsy** | Open-source forensics platform for analyzing mobile devices, hard drives, and removable media |
| **Volatility** | Memory forensics framework — analyzes RAM dumps from Windows and Linux systems |
| **Redline** | FireEye incident response tool for gathering and analyzing forensic data from a live system |
| **Velociraptor** | Advanced open-source endpoint monitoring, forensics, and response platform |

---

## Incident Response Frameworks

### NIST SP-800-61 (4 Phases)

| Phase | Description |
|-------|-------------|
| Preparation | Build teams, processes, and technology before an incident occurs |
| Detection and Analysis | Identify indicators of incidents and analyze them |
| Containment, Eradication, and Recovery | Limit impact, remove the threat, restore services |
| Post-incident Activity | Review, document, and improve for the future |

### SANS PICERL (6 Phases)

| Phase | Description |
|-------|-------------|
| **P**reparation | Establish people, processes, and tools ready for incidents |
| **I**dentification | Detect and confirm incidents; filter false positives |
| **C**ontainment | Limit the spread and impact of the incident |
| **E**radication | Remove the threat from the environment completely |
| **R**ecovery | Restore affected systems and services to normal operation |
| **L**essons Learned | Review the incident, document findings, improve future response |

NIST and SANS cover the same lifecycle. NIST groups Containment/Eradication/Recovery together; SANS separates them. PICERL is easier to remember as an acronym.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| DFIR | Digital Forensics and Incident Response |
| Artifact | A piece of digital evidence pointing to activity on a system |
| Chain of custody | Documentation tracking the handling of evidence from collection to court |
| Write protection | Preventing any writes to original evidence media during acquisition |
| Order of volatility | Priority order for collecting evidence based on how quickly it disappears |
| Timeline | Chronological ordering of events from multiple evidence sources |
| PICERL | SANS IR framework: Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned |
| IOC | Indicator of Compromise — observable artifact suggesting malicious activity |

---

## Workflow / Process

```
Incident detected / suspected
        |
        v
Preparation: team, tools, and processes in place
        |
        v
Identification: confirm incident, classify type and severity
        |
        v
Evidence Collection (ordered by volatility):
  Memory → Network connections → Running processes → Disk
        |
        v
Evidence preserved with write protection + chain of custody maintained
        |
        v
Forensic copies created for analysis
        |
        v
Containment: isolate affected systems
        |
        v
Eradication: remove threat, patch entry point
        |
        v
Recovery: restore systems from clean state
        |
        v
Timeline creation and full incident documentation
        |
        v
Lessons Learned: improve detection, response, and preventative controls
```

---

## Real-World Relevance

- DFIR is a core function in every mature security operations programme — SOC analysts, IR teams, and forensic investigators all use these frameworks and tools
- The order of volatility principle is directly applied in breach response — attackers who know forensics often attempt to clear memory and logs before defenders can capture them
- Chain of custody matters beyond legal proceedings — it establishes defensible evidence for disciplinary actions, insurance claims, and regulatory reporting
- KAPE and Eric Zimmerman's tools are standard in enterprise DFIR engagements — their output feeds directly into SIEM correlation and threat hunting
- Volatility is used in malware analysis labs worldwide to reconstruct what malware was doing in memory during an infection

---

## Key Learnings

- DFIR combines Digital Forensics and Incident Response — two interdependent disciplines
- Artifacts are collected to support hypotheses about attacker activity
- Evidence must be write-protected and copied before analysis — never analyse the original
- Chain of custody tracks every person who handles evidence
- Collect evidence in order of volatility — RAM first, disk last
- NIST (4 phases) and SANS/PICERL (6 phases) describe the same IR lifecycle with different groupings
- Key tools: KAPE (collection), Autopsy (disk analysis), Volatility (memory), Redline (IR triage), Velociraptor (endpoint)

---

## Conclusion

DFIR provides the structured methodology that turns a chaotic security incident into an understandable, documented chain of events. Understanding the core concepts — artifacts, chain of custody, order of volatility, timeline creation — and the IR frameworks that govern the process is foundational for anyone working in incident response, SOC analysis, threat hunting, or digital investigations.
