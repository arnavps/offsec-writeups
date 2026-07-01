# Introduction to Incident Response and Incident Management

## Executive Summary

This room establishes the foundational framework for how organisations detect, respond to, and manage cybersecurity incidents. It introduces the distinction between incident response (the technical investigation) and incident management (the process and decision-making layer), the four-level escalation model, the NIST incident management lifecycle, team roles, and the most common pitfalls that cause incidents to spiral beyond control.

**Major concepts covered:** SOC as the detection filter, event-to-incident triage pipeline, Incident Response vs Incident Management, four escalation levels (SOC → CERT → CSIRT → CMT), the NIST Incident Management framework (Prepare → Detect & Analyse → Contain/Eradicate/Recover → Post-Incident), team roles and responsibilities, and six common incident pitfalls.

**Why these concepts matter:** Organisations no longer ask if a breach will happen — they plan for when. A structured incident response and management programme is the difference between a contained, documented, recoverable event and an existential threat to the business. According to the National Cybersecurity Alliance, approximately 60% of small companies that experience a cyberattack close within six months. The ability to detect, scope, contain, and recover is as critical as any preventive security control.

**Where they appear in real organisations:** Every enterprise with a SOC, every cloud-native organisation with a DevSecOps programme, every regulated company subject to GDPR, HIPAA, or PCI DSS, and every organisation large enough to have an executive team that must answer to regulators, insurers, and the public.

---

## Big Picture Overview

The security problem being addressed here is not technical — it is operational. Organisations have invested heavily in preventive controls: firewalls, EDR, WAF, MFA. Yet breaches still occur. The question is no longer how to prevent every incident, but how to respond when prevention fails.

Without a structured incident response and management programme:
- Incidents are handled ad hoc, with no repeatable process
- Evidence is destroyed accidentally before it can be analysed
- The scope of a breach is misunderstood — leading to either over-reaction (unnecessary disruption) or under-reaction (the threat actor persists)
- Regulatory notification deadlines are missed
- Post-incident learning does not happen, so the same mistakes repeat

The mental model: think of the security programme as a funnel.

```
All Events (thousands per day)
         ↓
Automated filtering (spam filter, AV, EDR) removes most
         ↓
Remaining events → SOC Analysts
         ↓
Triage: is this an anomaly? Is it confirmed? What is the severity?
         ↓
Incident declared → Incident Response (what happened?) runs in parallel with
                    Incident Management (how do we respond?)
         ↓
Escalation based on severity → SOC → CERT → CSIRT → CMT
         ↓
Post-Incident Activity → Lessons learned → Improve preparation
```

The SOC is the filter. Not all events become incidents. Not all incidents reach the CMT. But every incident, regardless of level, requires both a technical response and a management response.

---

## Core Concepts

---

### Concept 1 — What Is a Cyber Incident?

#### What Is It?

A cyber incident is a security event where, after triage, the severity is sufficient to require structured response and the full scope is not yet understood. It is not simply any alert or anomaly — most alerts are resolved at the SOC level without becoming incidents.

#### The Event → Alert → Incident Pipeline

| Stage | Description | Example |
|-------|-------------|---------|
| **Event** | Any observable occurrence in a system | A user login at 3am |
| **Alert** | A flagged event that requires attention | SIEM rule triggers on unusual login time |
| **Triage** | Analyst investigates to determine if the alert is real and its severity | Analyst checks: whose account? From where? Normal pattern? |
| **Incident** | Alert is real, severity is sufficient, and scope is not fully understood | Account shows logins from two countries simultaneously |

#### Why the Scope Question Is Central

An incident is raised when there are unanswered questions about the scope of impact:
- Whose account was used?
- Where did the logon originate?
- What was the account doing before this logon?
- Has any other anomalous activity been seen with this account?

If these questions can be answered quickly and the scope is clearly limited, the event is resolved at triage. If they cannot be answered — or the answers suggest broader impact — an incident is raised.

#### Common Misconception

Not all security alerts are incidents. Organisations receive thousands of phishing emails daily. Most are blocked by spam filters. Of those that get through, most are blocked by AV/EDR. Only events that survive automated and first-level analyst filtering — with unresolved scope questions — become incidents.

---

### Concept 2 — Incident Response vs Incident Management

These two disciplines are complementary but distinct. Confusing them leads to either a technically thorough investigation that takes no action, or rapid action taken on a misunderstood scope.

#### Incident Response

**Primary question:** What happened?

Incident Response is the technical discipline that investigates the incident to understand its scope. It uses:

| Tool/Technique | What It Provides |
|---------------|-----------------|
| **EDR/AV alerts** | Activity on specific hosts — keyloggers, suspicious processes, lateral movement |
| **Network tap alerts** | Network-level anomalies — port scans, C2 communications, data exfiltration |
| **SIEM alerts** | Cross-system correlation — impossible travel, privilege escalation patterns |
| **Digital forensics** | Deep investigation when alert data is insufficient — disk imaging, RAM capture, log recovery |

Digital forensics techniques include:
- **Hard disk recovery:** How did the malware arrive? What files were created or modified?
- **Volatile memory capture:** What was the malware doing in memory? What processes were running?
- **Log recovery from multiple devices:** How did the malware spread? What network paths were used?

**The scope accuracy imperative:** Incident response must accurately determine the scope. Underestimating scope leads to insufficient containment (the threat actor persists). Overestimating scope leads to unnecessarily disruptive actions (business disruption without cause).

#### Incident Management

**Primary question:** How do we respond to what happened?

Incident Management is the process discipline that ensures the right actions are taken, by the right people, in the right order, with full accountability.

Key responsibilities:
- **Triage:** Update incident severity as new information arrives; escalate to appropriate stakeholders
- **Playbook guidance:** Ensure the team follows predefined processes for known incident types
- **Containment/Eradication/Recovery decisions:** Decide which actions will be taken and in what order
- **Internal/External communication:** Control the narrative; notify regulators and customers as required
- **Documentation:** Record all actions, their rationale, their effects, and who was responsible
- **Post-incident review:** Learn from the incident to improve future processes

**The key insight:** Incident management is not less important than incident response. A technically excellent investigation that is poorly managed — with no accountability for action items, no communication to affected stakeholders, and no documentation — is a failed incident response.

---

### Concept 3 — Four Levels of Incident Response

Not all incidents are equal, and not all incidents require the same level of response. The four-level model ensures that the response is proportional to the severity and scope.

| Level | Team Invoked | Description | Example | Trigger to Next Level |
|-------|-------------|-------------|---------|----------------------|
| **Level 1** | SOC Incident | Technical resolution by single analyst | User reports phishing email; analyst updates mail filter | Multiple users received the same email |
| **Level 2** | CERT Incident | Multiple SOC analysts involved; more investigation needed | Investigating whether users interacted with the phishing email | Users interacted with the email and malware was delivered |
| **Level 3** | CSIRT Incident | Entire SOC focused on one incident; forensics and containment begin | Malware has spread; containment and eradication in progress | Malware is spreading to critical systems; possible Active Directory compromise |
| **Level 4** | CMT Incident | Full crisis; executives, legal, comms, regulators involved | Ransomware is being deployed across the estate | N/A — highest level |

**Key principle:** Severity is not static. New information from the incident response process can escalate OR de-escalate an incident. A Level 3 incident where containment is successful may never need to invoke CMT. A Level 2 investigation that discovers more widespread compromise than initially assessed may jump directly to Level 4.

**"Nuclear" actions:** Only the CMT can authorise the most disruptive actions — taking entire systems or the organisation offline. These actions are disproportionately disruptive but may be necessary to prevent catastrophic damage. Level 1-3 actions are generally targeted and reversible; Level 4 actions may be broad and operationally impactful.

---

### Concept 4 — Team Roles and Responsibilities

Effective incident response requires a team with diverse skills. The table below maps roles to their responsibilities:

| Role | Primary Function | When Involved |
|------|-----------------|--------------|
| **SOC Analyst** | Monitor events, investigate alerts, perform initial triage | All incidents (levels 1-4) |
| **SOC Lead / SOC Manager** | Divide tasks, decide on escalation, manage SOC operations | All incidents |
| **Forensic Analyst** | Digital forensics — disk, memory, network artefact analysis | Level 2+ (when alert data is insufficient) |
| **Malware Analyst** | Reverse-engineer malware, discover IoCs, understand attack mechanics | Level 2+ (when malware is involved) |
| **Threat Hunter** | Proactively search for unknown threats; create new detection rules | Pre-incident preparation; post-incident improvement |
| **First Responder** | Initial discovery outside the SOC; preserve evidence; notify SOC | Any incident discovered outside SOC |
| **Security Engineer** | SME for their system/division; ensure SOC receives logs; assist investigation | As SME when their systems are involved |
| **Information Security Officer (ISO)** | Bridge between IR team and the division being impacted | Level 2+ when organisational units are affected |
| **Incident Manager** | Manage the incident management process; note-taking; accountability | Level 2+ |
| **Product/Project Owner** | SME for their application; understand system behaviour | When their application is involved |
| **Subject Matter Expert (SME)** | Deep expertise in a specific technology or system | Called in as needed based on incident scope |
| **Crisis Manager** | Lead the CMT; usually CEO or COO | Level 4 only |
| **Executive (CEO, COO, CIO, CISO)** | Ultimate accountability; decision authority for nuclear actions | Level 4 only |

**Security Engineer's position in IR:** Security engineers are not part of the blue team, but they are critical to incident response. They:
1. Ensure their systems forward logs to the SIEM (enabling detection)
2. Act as SMEs during investigations involving their systems
3. Understand what containment actions are feasible without causing disproportionate business disruption
4. May be first responders if they discover an incident in their own system

---

### Concept 5 — The NIST Incident Management Framework

The NIST framework is the most widely adopted standard for incident management. Most organisations' internal processes are derived from or aligned with it.

```
┌────────────────────────────────────────────────────────────────┐
│                         PREPARATION                            │
│   Playbooks, call trees, tabletop exercises, threat hunting    │
└────────────────────┬───────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────────────────┐
│               DETECTION & ANALYSIS (Triage)                   │
│   Alert review, forensics, malware analysis, scope assessment  │
└────────────────────┬───────────────────────────────────────────┘
                     │ (cyclic — scope evolves as investigation proceeds)
                     ▼
┌────────────────────────────────────────────────────────────────┐
│       CONTAINMENT → ERADICATION → RECOVERY                    │
│  Stop the bleed → Remove threat actor → Return to BAU         │
└────────────────────┬───────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────────────────┐
│                  POST-INCIDENT ACTIVITY                        │
│   Lessons learned, process improvement, updated playbooks      │
└────────────────────────────────────────────────────────────────┘
```

#### Phase 1: Preparation

The most important phase — the one that determines how well the team handles the chaos of an actual incident.

**Activities:**
- Document key stakeholder contacts and call trees
- Create and maintain incident playbooks for known incident types
- Conduct tabletop exercises and cyber war games
- Continuously perform threat hunting to create new alert rules
- Ensure adequate logging infrastructure is in place

**Why preparation is underinvested:** Preparation has no immediate, visible output. It is easy to deprioritise in favour of projects with measurable short-term value. Yet inadequate preparation is the single biggest contributor to poor incident outcomes.

#### Phase 2: Detection and Analysis

**Primary question:** What has happened and what is the scope?

This phase is iterative — new information from investigation continuously updates the scope picture. Key activities:
- Review alerts across AV, EDR, and SIEM dashboards
- Forensic investigation of host and network artefacts
- Malware analysis to understand attack mechanics and generate IoCs
- Triage to determine severity and appropriate escalation level

**The cyclic relationship with Phase 3:** Phases 2 and 3 are shown as cyclic in the NIST diagram because it is impossible to wait for complete scope understanding before taking any action. The team starts initial containment while investigation continues. New forensic findings may require additional containment actions.

#### Phase 3: Containment, Eradication, and Recovery

**Why order matters — this is critical:**

```
Wrong order:
  Eradicate (change all passwords)
  → Threat actor uses current Active Directory access to re-harvest credentials
  → Eradication was pointless; team must start over

Correct order:
  1. Contain (cut off threat actor's access)
  2. Eradicate (remove malware, reset credentials, close vulnerabilities)
  3. Recover (restore systems to BAU)
```

If eradication or recovery begins before containment, the threat actor can simply undo those actions using their existing access.

| Sub-phase | Goal | Example Actions |
|-----------|------|----------------|
| **Containment** | Stop the bleed — prevent further spread | Network segmentation, EDR jailing, rate limiting, host isolation |
| **Eradication** | Remove the threat actor completely | Delete malware, reset compromised credentials, patch exploited vulnerabilities |
| **Recovery** | Restore systems to normal operation | Restore from clean backups, rebuild compromised systems, validate clean state |

#### Phase 4: Post-Incident Activity

The final phase is often rushed or skipped entirely — which means the same incidents recur.

**Activities:**
- Conduct a post-incident review (or "lessons learned") meeting
- Document what happened, what actions were taken, and what their effects were
- Identify what could have been done better at each phase
- Update playbooks, alert rules, and preparation based on findings
- Brief relevant stakeholders on findings and improvements

---

### Concept 6 — Common Incident Pitfalls

#### 1. Insufficient Hardening

Security is often deprioritised in favour of speed-to-market. Systems are deployed with configurations that do not adhere to security best practices — and the hardening step is never completed. This directly increases the frequency and severity of incidents.

**The Shift Left principle:** Hardening should be integrated during development (not applied after deployment). This reduces the cost and difficulty of hardening and ensures security is embedded from the start.

#### 2. Insufficient Logging

The blue team cannot detect what it cannot see. Common causes of insufficient logging:
- SIEM licensing costs incentivise organisations to ingest less data
- Remote devices (ATMs, IoT) face network cost constraints for log forwarding
- Log retention policies are too short

**Consequences:** Incidents are detected later (after impact), or the scope cannot be accurately determined because the historical data needed for investigation is missing.

#### 3. Insufficient and Over-Alerting

Two failure modes of the same problem:

| Failure Mode | Cause | Consequence |
|-------------|-------|-------------|
| **Too few alerts** | Not enough threat hunting; generic rules; poor log coverage | Incidents go undetected until there is significant impact |
| **Too many alerts (alert fatigue)** | Rules with poor signal-to-noise; too many false positives | Analysts ignore alerts; real incidents are missed in the noise |

**The "cry wolf" effect:** If an alert generates constant false positives, analysts begin to treat it as noise. When a real incident triggers the same alert, it is dismissed. Alert quality (signal-to-noise ratio) is as important as alert quantity.

#### 4. Insufficient Determination of Incident Scope

Inaccurate scope assessment has two failure modes, both costly:

| Error | Consequence |
|-------|-------------|
| **Underestimated scope** | Actions taken are insufficient; threat actor is not fully eradicated; incident resumes |
| **Overestimated scope** | Unnecessarily disruptive actions are taken; business operations are impacted without justification |

**There is no quick fix:** This is a preparation and skill-building problem. Regular tabletop exercises, threat intelligence awareness, and forensic capability development are the long-term mitigations.

#### 5. Insufficient Accountability

A common pattern during incidents: actions are discussed and decided, but no specific individual is assigned responsibility for executing them. Everyone assumes someone else is doing it. Critical time passes. The incident worsens.

**The fix:** The incident manager explicitly assigns a responsible individual for every action item, with a required completion time and a reporting obligation back to the incident manager. The documentation serves as the accountability mechanism.

#### 6. Insufficient Backups

In ransomware scenarios, backups are often the only path to recovery. Two common backup failures:

| Failure | Description |
|---------|-------------|
| **No backup policy** | Backups were never established or are not kept current |
| **Insufficient isolation** | High Availability / DR replication means ransomware encrypting the primary environment replicates directly to the DR environment |

**Critical requirement:** Offline or air-gapped backups that cannot be reached by ransomware propagating through the network. Modern organisations that rely solely on HA/DR replication have effectively lost their backup capability to ransomware.

---

## Architecture and Relationships

### The Event-to-Incident Pipeline

```
Organisation's Digital Estate (thousands of events/day)
           ↓
Automated Controls (Spam filter, AV, EDR, WAF)
  → Block the majority automatically
           ↓
Remaining Events → SIEM
  → Correlation rules generate alerts
           ↓
SOC Analyst → Triage
  → False positive? → Resolve; tune rules
  → Real but contained? → Level 1 SOC Incident
  → Uncertain scope? → Escalate
           ↓
Incident Declared
  ┌────────────────┐    ┌─────────────────────────────┐
  │ Incident       │    │ Incident Management          │
  │ Response       │    │                              │
  │ (What          │    │ (How to respond; process;    │
  │  happened?)    │    │  accountability; decisions)  │
  └────────────────┘    └─────────────────────────────┘
           ↓
  Scope determines escalation level:
  Level 1 (SOC) → Level 2 (CERT) → Level 3 (CSIRT) → Level 4 (CMT)
           ↓
  NIST Phases:
  Contain → Eradicate → Recover → Post-Incident Review
```

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **Alert** | A flagged event that requires analyst attention |
| **BAU** | Business as Usual — normal organisational operations |
| **CERT** | Computer Emergency Readiness Team — handles Level 2 incidents |
| **CMT** | Crisis Management Team — handles Level 4 cyber crises |
| **CSIRT** | Computer Security Incident Response Team — handles Level 3 incidents |
| **Digital Forensics** | Deep investigation of digital artefacts (disk, memory, logs) to understand an incident |
| **EDR** | Endpoint Detection and Response — monitors and responds to suspicious endpoint activity |
| **Event** | Any observable occurrence in a system or network |
| **Incident** | A security event where severity is sufficient and scope is not yet fully understood |
| **Incident Management** | The process layer of incident response — decisions, communication, documentation |
| **Incident Response** | The technical layer — investigating what happened and determining scope |
| **IoC** | Indicator of Compromise — artefact (hash, IP, domain) identifying a specific threat |
| **NIST** | National Institute of Standards and Technology — publishes the Incident Management framework |
| **Nuclear Actions** | Highly disruptive actions (e.g., taking the entire organisation offline) authorised only by CMT |
| **Playbook** | Predefined set of steps for responding to a known incident type |
| **SIEM** | Security Information and Event Management — aggregates logs and generates alerts |
| **SME** | Subject Matter Expert — called in for specific technical knowledge during incidents |
| **SOC** | Security Operations Centre — monitors the organisation's security posture |
| **Triage** | Process of evaluating an alert to determine severity and appropriate response level |

---

## Exam and Interview Revision

### Must Remember

- Incident response answers: "What happened?" — Incident management answers: "How do we respond?"
- Both are required; technical investigation without process management leads to inaction; process without technical investigation leads to incorrect decisions
- Four escalation levels: SOC (Level 1) → CERT (Level 2) → CSIRT (Level 3) → CMT (Level 4)
- NIST framework: **Preparation → Detection & Analysis → Containment/Eradication/Recovery → Post-Incident**
- Containment must come before eradication — otherwise the threat actor can undo eradication actions
- Alert fatigue (too many false positives) is as dangerous as insufficient alerting (too few rules)
- Scope assessment accuracy is the most critical IR skill — underestimate = insufficient response; overestimate = unnecessary disruption
- Insufficient accountability during incidents = actions discussed but never performed = incident grows
- Offline/air-gapped backups are the only reliable ransomware recovery mechanism — HA/DR replication is not a backup
- ~60% of small businesses close within 6 months of a cyberattack — the business case for IR is clear

### Common Interview Questions

| Question | Key Answer Points |
|----------|-----------------|
| What is the difference between incident response and incident management? | IR = technical (what happened, scope determination). IM = process (how to respond, decisions, communication, documentation, accountability). Both required; neither sufficient alone. |
| What are the four NIST incident management phases? | Preparation; Detection & Analysis; Containment, Eradication & Recovery; Post-Incident Activity |
| Why must containment precede eradication? | If the threat actor still has access (not contained), any eradication actions can be undone by the attacker. Example: resetting passwords while the attacker still has domain admin access is pointless. |
| What is the difference between Level 3 (CSIRT) and Level 4 (CMT) incidents? | Level 3 = entire SOC focused on the incident, active containment/eradication. Level 4 = business crisis requiring executive involvement, legal, communications, and potentially nuclear actions. |
| What is alert fatigue and why is it dangerous? | Too many false-positive alerts causes analysts to ignore them. When a real incident occurs, the alert is dismissed. Balancing signal-to-noise in alert rules is critical. |
| What are the six common incident pitfalls? | Insufficient hardening; insufficient logging; insufficient/over-alerting; insufficient scope determination; insufficient accountability; insufficient backups |
