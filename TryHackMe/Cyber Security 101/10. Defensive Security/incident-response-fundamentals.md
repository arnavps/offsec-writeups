# Incident Response Fundamentals

## Overview

Incident response (IR) is the structured process an organisation follows when a confirmed security incident occurs. It covers everything from identifying true threats among the noise of alerts, to containing and eliminating the threat, recovering from the damage, and improving defences afterwards. This room covers the difference between false and true positives, the types of security incidents, the SANS/PICERL incident response framework, the tools used during IR, and playbooks and runbooks.

---

## Topics Covered

- False positives vs true positives
- Types of security incidents
- The SANS IR framework (PICERL)
- Comparison with NIST IR framework
- Security tools used during IR
- Playbooks and runbooks

---

## Key Concepts

### False Positives vs True Positives

Security solutions generate alerts constantly. Not all alerts indicate real threats.

| Term | Definition | Example |
|------|-----------|---------|
| **False Positive** | An alert that fires on benign activity — it looks harmful but isn't | A SIEM alerts on high data transfer from a system. Investigation reveals it was a scheduled cloud backup — not an attack |
| **True Positive** | An alert that correctly identifies genuinely malicious activity | A SIEM alerts on a phishing email. Investigation confirms the email was malicious and targeted a user |

True positive alerts that indicate real harmful activity are classified as **incidents** and trigger the incident response process.

---

### Types of Security Incidents

Incidents are not a monolithic category. Different types of incidents have different impacts, different response procedures, and different levels of severity depending on the organisation.

| Incident Type | Description |
|---------------|-------------|
| **Malware Infection** | A malicious program causes damage to a system, network, or application. Delivered through files — documents, executables, scripts. The most common type of incident |
| **Security Breach** | An unauthorised person gains access to confidential data they are not permitted to access. Critical for any organisation handling sensitive business or customer data |
| **Data Leak** | Confidential information is exposed to unauthorised entities. Can be caused intentionally by attackers or accidentally through human error or misconfiguration. Often used for reputational damage or extortion |
| **Insider Attack** | A malicious action initiated by someone inside the organisation — an employee, contractor, or former staff member with existing access. Particularly dangerous because insiders have elevated access compared to external attackers |
| **Denial of Service (DoS)** | The attacker floods a system, network, or application with requests until it is exhausted of resources and becomes unavailable to legitimate users. Attacks availability — one of the three pillars of the CIA triad |

> A given incident can combine multiple types. Severity is always context-dependent — a DoS attack that takes down a website may be catastrophic for one organisation and minor for another.

---

### The SANS Incident Response Framework (PICERL)

SANS defines a six-phase IR framework. The acronym **PICERL** covers each phase in order.

| Phase | Name | Description |
|-------|------|-------------|
| P | **Preparation** | Build the resources, teams, tools, and plans needed before an incident occurs. Includes training employees, creating IR plans, and deploying security solutions. Example: running phishing awareness training |
| I | **Identification** | Detect and confirm that an incident has occurred. Uses security monitoring tools and analyst investigation to separate true positives from noise. Example: detecting a host sending unusual outbound data, tracing it back to a malicious attachment from a phishing email |
| C | **Containment** | Minimise the spread and impact of the incident. Isolate affected systems, disable compromised accounts, block malicious traffic. Example: isolating the infected host from the network to prevent lateral movement |
| E | **Eradication** | Remove the threat from the environment completely. Run deep malware scans, remove malicious files, patch the exploited vulnerability. Example: executing a full malware scan and removing the malicious software from the host |
| R | **Recovery** | Restore affected systems from clean backups or rebuild them. Verify systems are clean and working correctly before returning them to production. Example: reconfiguring the compromised host and restoring exfiltrated data from backup |
| L | **Lessons Learned** | Conduct a post-incident review. Document what happened, identify gaps in detection or response, and implement improvements to prevent recurrence. Example: holding a post-incident review meeting to analyse root cause and update detection rules |

---

### NIST IR Framework Comparison

NIST and SANS define similar frameworks. NIST uses four phases:

| NIST Phase | Equivalent SANS Phase(s) |
|------------|--------------------------|
| Preparation | Preparation |
| Detection and Analysis | Identification |
| Containment, Eradication, and Recovery | Containment + Eradication + Recovery |
| Post-Incident Activity | Lessons Learned |

Both frameworks cover the same core IR lifecycle — NIST groups the middle phases together while SANS separates them for granularity.

---

### Security Tools in Incident Response

These tools are used primarily during the Identification phase but support the full IR lifecycle.

| Tool | Role in IR |
|------|------------|
| **SIEM** | Collects and correlates logs from across the network to detect incidents. Central detection platform in most SOC/IR environments |
| **Antivirus (AV)** | Detects known malware signatures on systems. Runs automated scans to identify and quarantine known threats |
| **EDR** | Deployed on endpoints — detects advanced threats, provides detailed visibility into endpoint activity, and can perform automated responses such as isolating a host or terminating a malicious process |

---

### Playbooks and Runbooks

When an incident is confirmed, responders need structured guidance to act efficiently and consistently. Two types of documents serve this purpose.

#### Playbooks

A **playbook** is a high-level set of guidelines for responding to a specific type of incident. It defines what actions to take and in what general order.

Example playbook for a **phishing email incident**:
1. Notify all stakeholders of the phishing email incident
2. Determine if the email was malicious — conduct header and body analysis
3. Identify any attachments and analyse them
4. Determine if any users opened the attachments
5. Isolate infected systems from the network
6. Block the sender's email address

#### Runbooks

A **runbook** is the detailed, step-by-step technical execution of a specific task within a playbook. Runbooks are more granular — they contain the exact commands, tools, and decision trees a responder follows. They may vary depending on available tools and infrastructure.

**Relationship:** A playbook defines the strategy; runbooks implement the tactics.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Incident | A confirmed true positive — a harmful security event requiring a structured response |
| False Positive | An alert on benign activity mistakenly flagged as malicious |
| True Positive | An alert that correctly identifies real malicious activity |
| PICERL | Mnemonic for SANS IR phases: Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned |
| Containment | Limiting the spread or impact of an active incident |
| Eradication | Removing the threat completely from the affected environment |
| Lateral Movement | An attacker moving from one compromised system to other systems on the same network |
| Playbook | High-level guidelines for responding to a specific incident type |
| Runbook | Detailed, step-by-step technical instructions for executing specific IR tasks |
| DFIR | Digital Forensics and Incident Response — the combined discipline |

---

## Workflow / Process

```
Security solution generates alert
        |
        v
Analyst investigates: False Positive or True Positive?
  False Positive --> Document and close
  True Positive  --> Classify incident type
        |
        v
Initiate IR process (PICERL):
        |
        v
Preparation (ongoing before incident occurs)
        |
        v
Identification: confirm incident, determine scope
        |
        v
Containment: isolate affected systems, disable accounts
        |
        v
Eradication: remove malware, patch vulnerabilities, clean systems
        |
        v
Recovery: restore from backup, verify systems, return to production
        |
        v
Lessons Learned: post-incident review, improve detection and response
```

---

## Real-World Relevance

- PICERL is the framework taught in SANS courses and widely used in enterprise IR teams. Knowing the phases and what happens in each is expected in any IR or SOC role
- Alert triage (separating false from true positives) is one of the most time-consuming daily tasks in a SOC — reducing false positive rates is a major focus of detection engineering
- Containment speed directly limits the blast radius of an attack — fast isolation of a compromised host prevents lateral movement and ransomware spread
- Lessons Learned is frequently undervalued but is where the most long-term security improvement happens — post-incident reviews identify the gaps that allowed the incident to occur
- Playbooks and runbooks are standard artefacts in mature IR programmes and are often required for security compliance frameworks (ISO 27001, NIST CSF, SOC 2)
- EDR's ability to automate containment (isolate endpoint from the network with one click) is a major efficiency multiplier in active incident response

---

## Key Learnings

- Not every alert is an incident — true positives are genuine harmful events; false positives are benign activity misidentified as threats
- Security incidents fall into several types: malware infections, security breaches, data leaks, insider attacks, DoS attacks
- The SANS IR framework has six phases: Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned (PICERL)
- NIST and SANS cover the same lifecycle with slightly different phase groupings
- SIEM detects via log correlation; AV detects known malware; EDR provides endpoint-level detection and automated response
- Playbooks provide high-level response guidance; runbooks provide granular step-by-step execution

---

## Additional Notes

- **Mean Time to Detect (MTTD)** and **Mean Time to Respond (MTTR)** are key SOC metrics — lower values indicate a more effective detection and response capability
- Ransomware incidents combine multiple IR phases that must happen extremely quickly — slow containment allows encryption to spread across the network
- The Preparation phase is often the most neglected — organisations that skip it spend far more time and effort during actual incidents
- Tabletop exercises simulate incidents in a low-stakes environment, allowing IR teams to practise and refine their playbooks before a real incident occurs

---

## Conclusion

Incident response is the operational response to security failures. Understanding the PICERL framework gives structure to what can otherwise be a chaotic and reactive process. Knowing how to differentiate false from true positives, what types of incidents exist, what tools support each phase, and how playbooks and runbooks operationalise response procedures is essential for anyone working in a SOC, DFIR, or security engineering capacity.
