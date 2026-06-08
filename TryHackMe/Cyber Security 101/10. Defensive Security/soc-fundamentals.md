# SOC Fundamentals

## Overview

A Security Operations Center (SOC) is the operational heart of an organisation's defensive security posture. It is a 24/7 facility staffed by a specialised team responsible for continuously monitoring, detecting, and responding to security threats across the organisation's network and systems. This room covers the structure of a SOC, the roles within it, the technology that supports it, and the core processes that keep it functioning.

---

## Topics Covered

- What a SOC is and what it does
- Detection and response as the SOC's core mission
- SOC team roles and responsibilities
- Alert triage and the 5 Ws
- Escalation and reporting
- Incident response and forensics within the SOC
- Core security technologies: SIEM, EDR, Firewall

---

## Key Concepts

### What is a SOC?

A SOC is a dedicated security facility that operates continuously (24/7/365). Its primary mission is to keep **Detection and Response** intact — detecting suspicious or malicious activity across the organisation's infrastructure and responding to it before damage occurs.

The SOC achieves this by centralising monitoring across all network devices, endpoints, and applications through integrated security solutions.

### Detection Functions

| Detection Type | Description |
|----------------|-------------|
| Vulnerability detection | Identifying unpatched weaknesses in systems and software that could be exploited |
| Unauthorised activity | Detecting logins or actions that fall outside expected behaviour (e.g. login from an unusual geographic location) |
| Policy violations | Identifying activity that violates security policy, such as sharing confidential files insecurely or downloading pirated software |
| Intrusions | Detecting unauthorised access to systems or networks, whether via exploitation or malware infection |

### Response Functions

- **Incident response support** — Once an incident is confirmed, the SOC team assists in minimising impact, coordinating containment, and performing root cause analysis
- The SOC feeds into broader incident response activities and works closely with dedicated IR teams on high-severity events

---

### The Three Pillars: People, Process, Technology

A mature SOC environment requires all three working together:

- **People** — skilled analysts and engineers using the right tools
- **Process** — defined procedures for triage, escalation, and response
- **Technology** — security solutions that automate detection and reduce manual effort

---

### SOC Team Roles

| Role | Responsibilities |
|------|----------------|
| SOC Analyst Level 1 | First responders. Perform initial alert triage to determine if a detection is harmful. Escalate and report findings through proper channels |
| SOC Analyst Level 2 | Handle detections requiring deeper investigation. Correlate data across multiple sources to perform thorough analysis |
| SOC Analyst Level 3 | Experienced analysts who proactively hunt for threat indicators. Lead incident response activities including containment, eradication, and recovery |
| Security Engineer | Deploy and configure the security solutions the analyst team operates on |
| Detection Engineer | Write and maintain the detection rules and logic within security solutions. Often performed by L2/L3 analysts or as a dedicated role |
| SOC Manager | Manages SOC processes and team operations. Liaises with the CISO to report on security posture and ongoing efforts |

> Role structure scales with the size and security requirements of the organisation — smaller SOCs may combine several of these responsibilities.

---

### Alert Triage

Triage is the foundation of SOC operations. Every alert that fires is first triaged to determine its severity and whether it requires escalation. Triage is structured around answering the **5 Ws**.

#### The 5 Ws

| Question | Purpose |
|----------|---------|
| **What?** | What happened — what was the nature of the detection? |
| **When?** | When did the event occur? |
| **Where?** | Where did it occur — which host, directory, or system? |
| **Who?** | Which user or account was involved? |
| **Why?** | Why did it happen — what is the root cause or intent? |

#### Example: Malware Detected on GEORGE-PC

| W | Answer |
|---|--------|
| What | A malicious file was detected on a host inside the organisation's network |
| When | 13:20 on June 5, 2024 |
| Where | Directory on host: GEORGE-PC |
| Who | User: George |
| Why | The file was downloaded from a pirated software website — the user downloaded it to use paid software for free |

---

### Reporting and Escalation

Harmful alerts are escalated as **tickets** to higher-level analysts. A good report includes:

- All 5 Ws answered thoroughly
- A detailed analysis of the alert and supporting data
- Screenshots or log excerpts as evidence of the activity

Reports ensure that the right people are informed in a timely manner and that there is a documented trail of the investigation.

---

### Incident Response and Forensics

Some alerts escalate beyond standard triage into full **incident response**, particularly when they indicate critical or large-scale compromise. High-level teams are engaged to manage containment, eradication, and recovery.

In some cases, **digital forensics** is also required to:
- Determine the root cause of the incident
- Analyse artefacts from compromised systems or network traffic
- Provide evidence for legal or disciplinary action

---

## Security Technologies

| Technology | Primary Role |
|------------|-------------|
| **SIEM** (Security Information and Event Management) | Collects logs from across the network, correlates them against detection rules, and generates alerts. Modern SIEMs also include user behaviour analytics (UBA) and threat intelligence integration. **Provides detection only** — not response |
| **EDR** (Endpoint Detection and Response) | Deployed on individual endpoints. Provides real-time and historical visibility into endpoint activity. Can perform automated responses (isolate a host, kill a process). Covers both detection and response at the endpoint level |
| **Firewall** | Network-layer security device. Monitors and filters incoming/outgoing traffic based on defined rules. Blocks suspicious traffic before it reaches the internal network |

Other technologies deployed in SOC environments include: Antivirus (AV), Endpoint Protection Platform (EPP), Intrusion Detection/Prevention Systems (IDS/IPS), Extended Detection and Response (XDR), Security Orchestration Automation and Response (SOAR), and more. Technology selection is driven by the organisation's threat surface and available resources.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SOC | Security Operations Center — 24/7 security monitoring facility |
| Alert Triage | The process of analysing an alert to determine its severity and appropriate response |
| False Positive | An alert that fires on benign activity and is not actually malicious |
| True Positive | An alert that correctly identifies genuinely malicious or harmful activity |
| SIEM | Centralised log collection and correlation platform for detection |
| EDR | Endpoint-level detection and response solution |
| CISO | Chief Information Security Officer — executive responsible for security strategy |
| Incident Response | Structured process for managing and recovering from a confirmed security incident |
| Detection Engineering | The practice of writing and maintaining rules that power automated detection |

---

## Workflow / Process

```
Security solution generates an alert
        |
        v
Level 1 Analyst performs initial triage (5 Ws)
        |
        v
Is it a false positive?
  Yes --> Document and close
  No  --> Escalate to Level 2 for deeper investigation
        |
        v
Level 2 correlates data across sources, confirms true positive
        |
        v
Is it a critical incident?
  No  --> Document findings, close ticket
  Yes --> Escalate to Level 3 / Incident Response team
        |
        v
Level 3 leads containment, eradication, and recovery
        |
        v
Post-incident documentation and lessons learned
```

---

## Real-World Relevance

- Every large organisation and enterprise runs a SOC or contracts a Managed SOC (MSOC) service
- The tiered analyst structure mirrors real-world SOC environments — L1 roles are the most common entry point for careers in defensive security
- SIEM platforms (Splunk, Microsoft Sentinel, IBM QRadar, Elastic SIEM) are the dominant tools in the industry — familiarity with at least one is expected in SOC roles
- Alert fatigue is a real and significant problem in SOCs — good detection engineering and triage processes are essential to manage it
- Understanding SOC operations is equally valuable for offensive security — knowing what the blue team monitors directly informs evasion and stealth techniques in red team engagements

---

## Key Learnings

- A SOC is a 24/7 security monitoring facility focused on detection and response
- The three pillars — People, Process, Technology — must all be present for an effective SOC
- SOC analysts are structured in three tiers: L1 (triage), L2 (investigation), L3 (incident response/hunting)
- Alert triage is answered through the 5 Ws: What, When, Where, Who, Why
- SIEM provides detection; EDR provides endpoint detection and automated response; Firewall filters network traffic
- Escalation of true positives as tickets, with complete documentation, is a core SOC process

---

## Additional Notes

- A SOC can be internal (in-house) or managed/outsourced (MSOC/MSSP)
- The Detection Engineer role is increasingly being recognised as a separate discipline from analyst work, particularly in mature SOCs
- SOAR platforms automate repetitive SOC tasks (e.g. automatically blocking an IP, enriching an alert with threat intelligence) to reduce analyst workload
- Threat hunting — proactively looking for threats that haven't triggered an alert — is a key function of Level 3 analysts and a sign of a mature SOC

---

## Conclusion

The SOC is the operational core of an organisation's defensive security capability. Understanding how it's structured — the roles, the triage process, the escalation flow, and the technology stack — is foundational knowledge for anyone pursuing a career in defensive security. It also contextualises what the blue team monitors and responds to, which is directly relevant when approaching problems from the offensive side.
