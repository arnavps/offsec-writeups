# Auditing and Monitoring

## Executive Summary

Auditing and monitoring are the verification and oversight mechanisms that determine whether security controls are actually working. Without auditing, there is no way to know whether compliance is being achieved. Without monitoring, there is no way to detect attacks in progress or after the fact. This room covers the purpose and structure of information systems auditing, audit frameworks, the six-phase audit process, Linux and Windows logging, and the relationship between logging, monitoring, and auditing.

**Major concepts covered:** Audit definitions and objectives, internal vs external vs third-party audits, audit frameworks (COSO, COBIT, ISO 27001, ISAE 3402, ITIL, PCI DSS, SOX), the audit process, Linux logging with aureport and ausearch, Windows Event Viewer with security event IDs, and the distinctions between logging, monitoring, and auditing.

**Why it matters:** Every compliance framework requires auditing. Every incident investigation requires logs. Every security operations programme depends on monitoring. These three disciplines — logging, monitoring, and auditing — are the feedback mechanisms that close the security improvement loop. Without them, controls exist only on paper.

---

## Big Picture Overview

Consider the hospital scenario from the room: staff are trained on data protection requirements. But do they follow them? Are patients' records being copied to USB drives? Are paper records being properly shredded? Auditing answers the question: "Are the controls we put in place actually working?"

```
Security Policy (sets requirements)
         ↓
Security Controls (implements requirements)
         ↓
Logging (records what happened)
         ↓
Monitoring (real-time observation of what is happening)
         ↓
Auditing (systematic evaluation of whether controls work)
         ↓
Findings and Recommendations (close the gap)
         ↓
(back to Security Policy — continuous improvement cycle)
```

---

## Core Concepts

---

### Concept 1 — What Is Auditing?

**Informal Definition:**
Auditing is a check-up for a company or organisation. It involves carefully examining the company's processes, internal controls, and financial statements to ensure everything runs correctly according to policies and laws. Auditors identify problems — errors, inefficiencies, policy violations — and suggest fixes.

**Formal Definition:**
Auditing is a **systematic, independent, and objective process** of gathering and evaluating evidence to determine whether an organisation, its policies, processes, controls, or financial statements comply with applicable laws, regulations, and industry standards.

**Financial vs Information Systems Auditing:**

| Aspect | Financial Audit | Information Systems Audit |
|--------|---------------|--------------------------|
| Focus | Financial records and procedures | IT infrastructure, processes, controls |
| Purpose | Verify financial accuracy and compliance | Evaluate effectiveness, security, and compliance of IT systems |
| Scope | Narrower | Broader |
| Regulatory drivers | SOX, GAAP, IFRS | ISO 27001, PCI DSS, HIPAA, NIST |

---

### Concept 2 — Information Systems Audit Objectives

Information systems audits serve multiple objectives simultaneously:

| Objective | What It Evaluates |
|-----------|-----------------|
| **Risk assessment** | Risks to CIA of information assets; adequacy of risk mitigation strategies |
| **Regulatory compliance** | Adherence to GDPR, HIPAA, PCI DSS, SOX, and industry standards |
| **IT governance** | Decision-making processes, resource allocation, performance management |
| **Security management** | Effectiveness of information security policy, processes, and controls |
| **Operational evaluation** | System design, development, implementation, and maintenance processes |
| **Data management** | Data storage, retention, backup, and recovery controls |
| **Business continuity** | Disaster recovery and business continuity plan adequacy |
| **Fraud detection** | Identification of fraudulent activities, resource misuse, and control gaps |

---

### Concept 3 — Audit Types

**Internal Audits:**
Conducted by organisation's own personnel assigned to the internal audit function.
- Cost-effective; auditors understand the organisation
- Less objective than external; potential conflict of interest
- Good starting point: verify controls work before paying for external audit

**External Audits:**
Conducted by independent auditors from external firms — not employed by the audited organisation.
- Impartial and objective — no conflict of interest
- Required for certification (ISO 27001), compliance (SOC 2 Type II), and regulatory (PCI DSS Level 1)
- More expensive than internal

**Third-Party Audits:**
Assessment of vendors, service providers, or subcontractors that the organisation depends on.
- Purpose: ensure third parties meet required security, data protection, and compliance standards
- Critical for supply chain risk management
- Results inform vendor selection, contract terms, and risk acceptance decisions

**Recommended Sequence:**
```
1. Internal audit → verify controls are in place and working
         ↓
2. External audit → independent verification; finds what internal missed
         ↓
3. Remediate findings
         ↓
4. Third-party audits → ongoing supply chain validation
```

Starting directly with external audits (without an internal audit first) typically requires multiple external audit cycles — significantly more expensive.

---

### Concept 4 — Audit Frameworks

Audit frameworks provide structured approaches for conducting audits consistently and objectively.

| Framework | Full Name | Focus | Primary Industries |
|-----------|-----------|-------|-------------------|
| **COSO** | Committee of Sponsoring Organizations | Internal control framework — comprehensive internal control design | All industries |
| **COBIT** | Control Objectives for IT | IT governance and management; aligns IT with business goals | All industries |
| **ISAE 3402** | International Standard on Assurance Engagements 3402 | Controls over financial reporting by service organisations | Financial services |
| **ISO 27001** | International Organisation for Standardisation 27001 | Information security management systems (ISMS) | All industries |
| **ITIL** | Information Technology Infrastructure Library | IT service management (ITSM) best practices | All industries |
| **PCI DSS** | Payment Card Industry Data Security Standard | Protecting cardholder data | Financial, retail, hospitality |
| **SOX** | Sarbanes-Oxley Act | Internal controls over financial reporting | Public companies (US) |

**Selecting the Right Framework:**
- Financial services: PCI DSS (card data), GLBA (financial privacy), SOX (public companies)
- Healthcare: HIPAA (US), Data Protection Act (UK)
- General IT governance: COBIT, ISO 27001
- Service organisations: ISAE 3402, SOC 2

**COBIT 2019 Domains (for reference):**
- Plan and Organise (PO) — 13 control objectives
- Acquire and Implement (AI) — 9 control objectives
- Deliver and Support (DS) — 11 control objectives
- Monitor and Evaluate (ME) — 7 control objectives
- Resilience (RES) — 1 control objective

---

### Concept 5 — The Audit Process

The audit process follows consistent stages regardless of the framework used:

#### Stage 1: Planning

- Define audit scope (systems, processes, locations in scope)
- Identify relevant framework controls
- Understand the organisation's IT environment
- Develop audit plan with timeline and testing approach

#### Stage 2: Information Gathering

- Interview key personnel (IT managers, system administrators, security team)
- Review documentation (policies, procedures, architecture diagrams)
- Analyse existing logs and reports
- Understand the control environment

#### Stage 3: Risk Assessment and Control Evaluation

- Identify risks and vulnerabilities in scope systems
- Evaluate effectiveness of existing controls
- Assess compliance with applicable policies and regulations
- Map control gaps

#### Stage 4: Testing

- Validate control effectiveness through testing:
  - Vulnerability scanning
  - Penetration testing
  - Controls testing (test that the control actually works, not just that it exists)
  - Data analysis (sampling, statistical analysis)
  - Technical configuration review

#### Stage 5: Analysis and Findings

- Analyse test results and identify deviations, vulnerabilities, and gaps
- Evaluate implications of each finding
- Assess whether risks are adequately mitigated

#### Stage 6: Reporting

- Document findings with supporting evidence
- Make recommendations for improvement
- Assign risk ratings to findings
- Distribute to management, audit committee, stakeholders

#### Stage 7: Follow-Up

- Verify that recommended improvements have been implemented
- Confirm corrective actions resolved the identified issues

**Example COBIT Audit Scenario:**
```
Step 1: Planning
  → Define scope (IT governance practices)
  → Identify relevant COBIT domains and controls
  → Develop audit plan

Step 2: Execution
  → Gather evidence of compliance with COBIT controls
  → Assess evidence

Step 3: Assessment
  → Identify compliance gaps
  → Make improvement recommendations

Step 4: Reporting
  → Prepare and communicate audit report

Step 5: Follow-up
  → Monitor implementation of recommendations
```

---

### Concept 6 — Audit Areas

Information systems audits may cover any or all of the following areas:

| Area | What Is Audited |
|------|----------------|
| Information Systems Hardware | Configuration, performance, asset inventory |
| Operating Systems | Configuration, patching, security hardening |
| File Systems | Permissions, access controls, sensitive data locations |
| Database Management Systems | Configuration, security controls, access logs |
| Network Infrastructure | Topology, security controls, access logs |
| Network Operating Controls | Access controls, network monitoring |
| IT Operations | Service delivery, change management, incident management |
| Lights-Out Operations | Automated IT processes running without human intervention |
| Problem Management | Incident tracking, resolution timeliness |
| Monitoring Operations | Detection and response capabilities |
| Procurement | Security requirements in procurement processes |
| Business Continuity | BCP adequacy and testing |
| Disaster Recovery | DR plan adequacy, testing frequency, RTO/RPO verification |

---

### Concept 7 — Logging

**What Is Logging?**
Logging is the process of recording events as they occur on a system. Log entries capture event type, timestamp, source, severity, and relevant context.

**Purposes of Logging:**

| Purpose | Description |
|---------|-------------|
| **Troubleshooting** | Diagnose application and system failures — understand what went wrong and why |
| **Monitoring** | Track system resource utilisation and performance |
| **Auditing** | Record user activities — who accessed what, when, and what changes were made |
| **Compliance** | Satisfy regulatory requirements for event recording and retention |

---

### Concept 8 — Linux Logging

**Key Linux Log Files:**

| File | Contents | Distro |
|------|---------|--------|
| `/var/log/messages` | General system messages | RHEL, CentOS |
| `/var/log/syslog` | General system messages | Debian, Ubuntu |
| `/var/log/auth.log` | Authentication events | Debian, Ubuntu |
| `/var/log/secure` | Authentication events | RHEL, CentOS, Fedora |
| `/var/log/utmp` | Currently logged-in users | All |
| `/var/log/wtmp` | Historical login/logout records | All |
| `/var/log/kern.log` | Kernel messages | All |
| `/var/log/boot.log` | Boot process messages | All |
| `/var/log/audit/audit.log` | Linux Audit subsystem events | All (with auditd) |

**Linux Logging Daemons:**
- **rsyslog:** Most common; flexible; supports forwarding to remote syslog servers
- **syslog-ng:** Enhanced features; complex filtering
- **journald:** systemd's journal; binary format; `journalctl` for viewing

**Linux Log Analysis Commands:**

```bash
# View last N lines of a log file
tail -n 12 /var/log/auth.log
tail -f /var/log/auth.log  # Follow in real-time

# Search for keyword in log
grep FAILED /var/log/auth.log
grep -i "authentication failure" /var/log/auth.log

# Audit report summary
aureport --summary

# Failed events summary
aureport --failed

# Search successful logins
ausearch --message USER_LOGIN --success yes --interpret
# --message: event type to search
# --success: yes (successful) or no (failed)
# --interpret: convert numeric IDs to readable names

# Failed login attempts
ausearch --message USER_LOGIN --success no --interpret

# Count failed root login attempts
ausearch -m USER_LOGIN -sv no -i | grep ct=root | wc -l

# Short form flags:
# -m = --message
# -sv = --success
# -i = --interpret
```

**aureport Output Example:**
```
Failed Summary Report
=====================
Range: 06/08/2023 - 07/06/2023
Number of failed logins: 87
Number of failed authentications: 421
```

Note: Failed authentications > failed logins because one failed login attempt triggers multiple authentication attempts depending on configuration (PAM modules, SSH retry settings).

**Managing Linux Logs — Best Practices:**

1. **Centralise:** Forward to a central syslog server or SIEM — prevents local deletion by attackers
2. **Filter and parse:** Use tools to extract meaningful events from raw log data
3. **Set up alerts:** Define thresholds and patterns that trigger notifications (e.g., 10+ failed logins in 5 minutes)

---

### Concept 9 — Windows Logging

**Windows Event Log Types:**

| Log | Contents | Security Relevance |
|-----|---------|-------------------|
| **Application** | Application-level events | Software crashes, application errors |
| **System** | OS component events | Driver failures, service state changes |
| **Security** | Authentication, policy, privilege events | Primary source for security investigations |
| **Forwarded Events** | Collected from other systems via Windows Event Forwarding | Centralised monitoring in Windows environments |

**Windows Log Location:** `C:\WINDOWS\system32\config\`

**Key Security Event IDs:**

| Event ID | Description | Security Significance |
|----------|------------|----------------------|
| **4624** | Successful logon | Baseline; unusual times or sources = investigation |
| **4625** | Failed logon | Multiple failures = brute force or password spray |
| **4634** | Logoff completed | Session tracking |
| **4647** | User initiated logoff | User-initiated session end |
| **4720** | User account created | New accounts outside change windows = suspicious |
| **4722** | User account enabled | Re-enabling dormant accounts |
| **4723** | Password change attempt | Credential management |
| **4728** | Member added to security-enabled global group | Privilege escalation |
| **4740** | User account locked out | Result of brute force attempts |
| **4779** | Remote session disconnect without logoff | RDP session abandoned |
| **7045** | New service installed | Malware service installation |

**Access:** `eventvwr` in Run dialog or `Win+R → eventvwr`

**Windows Audit Policy:**
```
Group Policy → Computer Configuration → Windows Settings →
Security Settings → Advanced Audit Policy Configuration

Enable:
- Account Logon (Kerberos authentication logging on DCs)
- Account Management (account creation, modification, deletion)
- Logon/Logoff (interactive and network logons)
- Object Access (file, registry, AD object access)
- Policy Change (security policy modifications)
- Privilege Use (use of sensitive privileges)
- System (security state changes, kernel driver events)
```

**Windows vs Linux Logging Comparison:**

| Feature | Linux Logs | Windows Logs |
|---------|-----------|-------------|
| Location | `/var/log` | `%SystemRoot%\System32\Logfiles` |
| Format | Syslog (text-based) | EventLog (binary .evtx) |
| Viewing tools | `tail`, `grep`, `less`, `ausearch` | Event Viewer, PowerShell (`Get-EventLog`) |
| Logging levels | Debug, Info, Notice, Warning, Error, Critical | Debug, Information, Warning, Error, Critical |
| Advantages | Flexible; easy to parse; grep-friendly | Integrated with Windows; structured data |

---

### Concept 10 — Monitoring

**What Is Monitoring?**
Monitoring is the continuous, real-time observation of system performance and behaviour. It watches applications, storage, networks, and users — looking for unusual patterns and policy violations.

**Why Monitoring Matters:**

| Purpose | Description |
|---------|-------------|
| **Troubleshooting** | Real-time identification of performance issues |
| **Performance optimisation** | Track utilisation patterns to optimise resource allocation |
| **Preventing failures** | Spot capacity issues and hardware degradation before they cause outages |
| **Security risk mitigation** | Detect unauthorised access and malicious activity in real time |
| **Regulatory compliance** | Demonstrate continuous compliance monitoring to auditors |

---

### Concept 11 — Logging, Monitoring, and Auditing — Comparison

| Dimension | Logging | Monitoring | Auditing |
|-----------|---------|-----------|---------|
| **Definition** | Recording system activities and changes | Real-time observation of system status | Systematic analysis and review of actions |
| **Main function** | Historical record of events | Live system health and performance observation | Compliance verification and control evaluation |
| **When findings occur** | Post-incident or on review | Real-time, at point of anomaly | Periodic or event-triggered |
| **Process type** | Passive and continuous | Active and continuous | Periodic or triggered |
| **Primary uses** | Forensics, debugging, compliance evidence | Operational maintenance, security alerting | Compliance reporting, accountability verification |
| **Key role** | Data gathering and accountability | Preventive and predictive maintenance | Compliance, verification, legal defensibility |
| **Timeliness** | Retrospective analysis | Real-time | Usually retrospective |

**How They Work Together:**
```
System generates event
         ↓
Logging → Event is recorded in log (passive, continuous)
         ↓
Monitoring → Anomaly detected in real time → Alert triggered (active, real-time)
         ↓
SOC investigates → Queries logs for context (logging enables investigation)
         ↓
Auditing → Periodic systematic review of log history and monitoring effectiveness
```

---

## Security Engineer Perspective

### What Security Engineers Do With These Concepts

**Logging Design:**
- Define what events must be logged (map to compliance requirements)
- Ensure logs are forwarded to a SIEM before local logs could be tampered
- Set appropriate retention periods (GDPR, PCI DSS, HIPAA requirements)
- Test log integrity — verify logs cannot be deleted by users being audited

**Monitoring Design:**
- Define alert thresholds (e.g., alert on >5 failed logins in 60 seconds)
- Tune alerts to reduce false positives without missing true positives
- Establish escalation procedures for different alert severities
- Integrate with SOC ticketing systems

**Audit Support:**
- Maintain evidence for auditors: policy documents, configuration screenshots, log samples
- Run internal audits before external audits to identify and fix gaps
- Map controls to framework requirements (ISO 27001 Annex A, PCI DSS requirements)
- Document control exceptions and compensating controls

### Detection Scenarios Using Logs

| Scenario | Log Source | What to Look For |
|----------|-----------|-----------------|
| Brute force attack | auth.log / Event 4625 | Multiple failed logins in short window |
| Successful attack after brute force | auth.log / Event 4624 | 4625 followed by 4624 from same source |
| New backdoor account | Event 4720 + 4728 | Account created and immediately added to admin group |
| Privilege escalation | /var/log/secure, audit.log | sudo commands, SUID binary execution |
| Data exfiltration | File access logs, network flow | Large outbound transfers, unusual access to sensitive files |
| Malware persistence (Windows) | Event 7045 | New service installed outside change window |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **Audit** | Systematic, independent, objective evaluation of compliance with controls and standards |
| **aureport** | Linux command-line tool for generating reports from the audit subsystem |
| **ausearch** | Linux command-line tool for querying audit logs |
| **COBIT** | IT governance framework providing control objectives for managing IT |
| **Compliance** | State of adhering to applicable laws, regulations, and standards |
| **COSO** | Internal control framework for enterprise risk management |
| **Event ID** | Unique numerical identifier for a specific Windows log event type |
| **External audit** | Audit conducted by independent auditors outside the organisation |
| **Internal audit** | Audit conducted by the organisation's own personnel |
| **ISAE 3402** | Assurance standard for service organisation controls over financial reporting |
| **ISO 27001** | International ISMS standard |
| **ITIL** | IT service management best practice framework |
| **Logging** | Recording events as they occur on a system |
| **Monitoring** | Continuous real-time observation of system performance and behaviour |
| **PCI DSS** | Payment card industry security standard |
| **SOX** | Sarbanes-Oxley Act — financial reporting requirements for public companies |
| **Third-party audit** | Assessment of vendors and service providers |
| **Windows Event Forwarding** | Mechanism for forwarding Windows event logs to a central collector |

---

## Exam and Interview Revision

### Must Remember

- Auditing = systematic, **independent**, objective evaluation — independence is what makes it credible
- Internal audit first → external audit second → cheaper and finds issues before paying external auditors
- COBIT = IT governance; ISO 27001 = ISMS; PCI DSS = card payments; SOX = US public companies
- Audit process: Planning → Information Gathering → Risk Assessment → Testing → Analysis → Reporting → Follow-up
- Linux auth logs: `/var/log/auth.log` (Debian) or `/var/log/secure` (RHEL)
- `aureport --failed` = summary of failed events; `aureport --summary` = all events summary
- `ausearch -m USER_LOGIN -sv no -i | grep ct=root | wc -l` = count failed root logins
- Key Windows event IDs: **4624** (logon), **4625** (failed), **4720** (account created), **4728** (group member added), **7045** (new service)
- Logging = passive + historical; Monitoring = active + real-time; Auditing = periodic + systematic
- Centralise logs before anything else — local logs can be deleted by attackers
- Compliance logging retention: PCI DSS = 1 year; HIPAA = 6 years

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is the difference between logging and monitoring? | Logging records events historically (passive). Monitoring observes system state in real time (active). Logging enables investigation after the fact; monitoring triggers alerts as events happen. |
| What is the purpose of an audit? | Systematic, independent evaluation of whether controls, processes, and compliance obligations are working as intended. Finds gaps that internal teams miss. Provides evidence of compliance to regulators. |
| Why should internal audits precede external audits? | Internal audits find and fix issues cheaply. External audits are expensive and finding issues there is more costly. Internal first → fix gaps → external audit confirms everything is in order. |
| What does Event ID 4625 indicate? | Failed logon attempt on a Windows system. Multiple 4625 events in short succession from the same source = brute force or password spray attack. |
| How do you count failed root login attempts on Linux? | `ausearch -m USER_LOGIN -sv no -i | grep ct=root | wc -l` — queries the audit log for failed USER_LOGIN events, filters for root, counts lines. |
| What is the difference between auditing and compliance? | Compliance is the state of meeting requirements. Auditing is the process that verifies you are actually in that state. Compliance without auditing is just attestation — unverified. |
