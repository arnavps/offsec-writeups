# Defensive Security Intro

## Overview

Defensive security focuses on protecting systems, networks, and data from cyber threats. While offensive security simulates attacks, the defensive side works to prevent, detect, and respond to them. Blue teams are the primary practitioners of defensive security, operating continuously to keep organizations secure.

---

## Topics Covered

- Core defensive security tasks and responsibilities
- Security Operations Center (SOC)
- Threat Intelligence
- Digital Forensics and Incident Response (DFIR)
- Malware Analysis

---

## Key Concepts

### Core Defensive Security Tasks

Defensive security covers a broad range of ongoing responsibilities:

- **User awareness training** — Educating users reduces the risk of social engineering and phishing attacks, which target human behaviour rather than technical vulnerabilities
- **Asset management** — Maintaining an accurate inventory of systems and devices is a prerequisite for protecting them
- **Patch management** — Keeping systems updated closes known vulnerabilities that attackers actively exploit
- **Preventative security devices** — Firewalls and Intrusion Prevention Systems (IPS) form the first line of defence:
  - Firewalls control inbound and outbound network traffic based on defined rules
  - IPS actively blocks traffic that matches known attack signatures or suspicious patterns
- **Logging and monitoring** — Comprehensive logs and real-time monitoring enable detection of malicious activity, unauthorized devices, and intrusions

---

### Security Operations Center (SOC)

A SOC is a dedicated team of cybersecurity professionals responsible for continuously monitoring an organization's network and systems for malicious activity. Key areas of focus include:

- **Vulnerabilities** — Identifying and tracking unpatched weaknesses; coordinating remediation or applying mitigating controls when patches aren't available
- **Policy violations** — Detecting when users or systems deviate from security policy (e.g., uploading confidential data to unauthorized cloud storage)
- **Unauthorized activity** — Identifying compromised accounts or sessions, such as an attacker using stolen credentials to access internal systems
- **Network intrusions** — Detecting breaches whether caused by user error (clicking a malicious link) or direct exploitation of exposed services

---

### Threat Intelligence

Threat intelligence is the process of collecting and analyzing information about current and potential adversaries to build a threat-informed defence.

Key points:

- Intelligence focuses on understanding who might attack, why, and how
- Different organizations face different threat actors — a telecom company faces different adversaries than an energy company
- Common adversary types include nation-state groups (politically motivated) and ransomware operators (financially motivated)
- The output of threat intelligence feeds directly into SOC operations, incident response planning, and security controls

---

### Digital Forensics and Incident Response (DFIR)

#### Digital Forensics

Digital forensics applies scientific investigation methods to digital systems. In a defensive security context, it focuses on analyzing evidence of attacks, data breaches, intellectual property theft, and unauthorized activity.

Key forensic evidence sources:

| Source | What It Reveals |
|---|---|
| File System | Installed programs, created/modified/deleted files, partial overwrites |
| System Memory | Malware running only in RAM (never written to disk), active processes, credentials |
| System Logs | Timeline of events on a host; traces often remain even after attacker cleanup |
| Network Logs | Traffic patterns, connections to malicious infrastructure, data exfiltration evidence |

#### Incident Response

An incident is any event that negatively impacts the confidentiality, integrity, or availability of systems or data — from a full data breach to a policy violation or misconfiguration.

The four phases of incident response:

1. **Preparation** — Establish a trained response team, define procedures, and implement preventative controls before incidents occur
2. **Detection and Analysis** — Identify that an incident has occurred and assess its scope and severity
3. **Containment, Eradication, and Recovery** — Stop the spread, remove the threat, and restore affected systems to normal operation
4. **Post-Incident Activity** — Document findings, produce a report, and apply lessons learned to prevent recurrence

---

### Malware Analysis

Malware (malicious software) encompasses any program designed to harm, exploit, or gain unauthorized access to systems.

**Common malware types:**

| Type | Description |
|---|---|
| Virus | Attaches to legitimate programs and spreads between systems; alters, overwrites, or deletes files |
| Trojan Horse | Disguises itself as legitimate software while executing a hidden malicious function |
| Ransomware | Encrypts victim files and demands payment in exchange for the decryption key |

**Analysis approaches:**

- **Static analysis** — Examining the malware without executing it; involves inspecting code, strings, file structure, and assembly instructions. Requires knowledge of assembly language and binary analysis tools
- **Dynamic analysis** — Running the malware in an isolated, controlled environment (sandbox) and observing its behaviour — network connections, file modifications, registry changes, process spawning, etc.

---

## Important Terminology

| Term | Definition |
|---|---|
| Blue Team | The defensive security team responsible for protecting systems |
| SOC | Security Operations Center — team monitoring for threats 24/7 |
| IPS | Intrusion Prevention System — actively blocks malicious traffic |
| Threat Intelligence | Collected information about adversaries used to inform defences |
| DFIR | Digital Forensics and Incident Response |
| Forensic Image | A low-level, bit-for-bit copy of storage or memory for analysis |
| Malware | Any software designed to cause harm or gain unauthorized access |
| Sandbox | An isolated environment used to safely run and analyze malware |
| Static Analysis | Analyzing malware without executing it |
| Dynamic Analysis | Analyzing malware by running it in a controlled environment |

---

## Workflow / Process

### Incident Response Flow

```
Preparation
    |
Detection & Analysis
    |
Containment, Eradication & Recovery
    |
Post-Incident Activity (Report + Lessons Learned)
```

### Malware Analysis Flow

```
Receive malware sample
    |
Static Analysis (strings, headers, disassembly)
    |
Dynamic Analysis (sandbox execution, behaviour monitoring)
    |
Report findings (IOCs, TTPs, impact assessment)
```

---

## Real-World Relevance

- SOCs are the operational backbone of enterprise security — most large organizations run one internally or outsource to a managed security service provider (MSSP)
- Threat intelligence feeds are used to update firewall rules, IPS signatures, and detection logic in real time
- DFIR is critical after any breach — forensic evidence determines what happened, what was accessed, and how to prevent recurrence
- Malware analysis underpins the creation of antivirus signatures, detection rules, and threat intelligence reports

---

## Key Learnings

- Defensive security is proactive and reactive — prevention, detection, and response all matter
- The SOC is the central hub for monitoring and responding to threats
- Threat intelligence makes defences smarter by focusing on real, relevant adversaries
- Digital forensics reconstructs what happened; incident response limits the damage
- Understanding malware behaviour (static and dynamic) is essential for building effective detections

---

## Additional Notes

- Blue team roles include SOC Analyst, Threat Intelligence Analyst, Incident Responder, Digital Forensics Examiner, and Malware Analyst
- Tools commonly used in defensive security include SIEM platforms (Splunk, Microsoft Sentinel), EDR solutions, network monitoring tools, and sandbox environments (Any.run, Cuckoo)
- The NIST Incident Response framework and SANS PICERL model are widely referenced standards for incident response processes

---

## Conclusion

Defensive security is a multi-layered discipline covering everything from user training and patch management to advanced forensics and malware analysis. The SOC sits at the center of it all, continuously monitoring for threats and coordinating response. Understanding both the tools and the processes is essential for anyone working on the blue team side of cybersecurity.
