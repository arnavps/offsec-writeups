# Logging for Accountability

## Executive Summary

This room establishes logging as the foundation of accountability in cybersecurity — not merely as a technical data collection mechanism, but as the evidence chain that enables non-repudiation, incident investigation, and regulatory compliance. It covers the IAAA model's final pillar (accountability), SIEM architecture and data ingestion methods, log quality principles, storage strategies (cold/warm/hot), correlation across multiple log sources, and the critical importance of log authenticity.

**Major concepts covered:** Accountability and non-repudiation, SIEM architecture (search head, indexer, forwarder), four data ingestion methods (agent, port-forwarding, syslog, upload), log quality requirements, storage tiers (hot/warm/cold), PCI DSS log retention requirements, log source types (manual, automated, compliance), and log correlation.

**Why these concepts matter:** Without reliable, authentic logs, incident response is effectively blind. Defenders cannot answer "what happened," regulators cannot be satisfied that controls are working, and prosecutors cannot hold threat actors or negligent insiders accountable. Logging is the technical implementation of the accountability pillar — the audit trail that proves what occurred.

**Where these concepts appear in real organisations:** SIEM platforms in every mature SOC; compliance programmes (PCI DSS, GDPR, HIPAA) that mandate log retention; forensic investigations requiring log evidence; insider threat programmes; regulatory audits.

---

## Big Picture Overview

Accountability is the fourth and final pillar of the IAAA (Identification, Authentication, Authorisation, and Accountability) model. The first three pillars establish identity, verify it, and grant permissions. Accountability closes the loop: once a user has been identified, authenticated, and authorised, their actions must be recorded so they can be held responsible.

Without accountability:
- A malicious insider can exfiltrate data with no traceable evidence
- An attacker who compromises an account can act as the account owner without detection
- Regulators cannot be assured that data was handled appropriately
- Incident responders cannot reconstruct what happened or when
- Legal proceedings have no admissible evidence

The technical mechanism for accountability is logging. But logging alone is insufficient — the logs must be:
- **Complete:** covering the relevant systems and actions
- **Authentic:** provably unmodified since collection
- **Retained appropriately:** available for the required period
- **Correlated:** multiple sources telling a coherent story

---

## Core Concepts

---

### Concept 1 — Accountability and Non-Repudiation

#### What Is Accountability?

Accountability holds users and systems responsible for their actions by maintaining a verifiable record of those actions. It is the final pillar of the IAAA model and the last line of defence when preventive controls fail.

In the context of incident response, accountability enables:
- Identifying which account was used during an attack
- Establishing a timeline of attacker activity
- Proving that a specific user performed a specific action
- Supporting legal proceedings and regulatory investigations

#### What Is Non-Repudiation?

Non-repudiation is the property that prevents a user from credibly denying that they performed an action. It is the formal, legal application of accountability.

```
Repudiation:     "I did not send that email / access that file / delete that record"
Non-repudiation: "The log proves you did, and the log's integrity is verified"
```

Non-repudiation is only achievable when logs are:
1. **Collected** from the relevant systems at the time of the action
2. **Authentic** — provably unmodified since collection (cryptographic integrity, chain of custody)
3. **Retained** for a sufficient period to be available when needed

#### The STRIDE Connection

Non-repudiation is the countermeasure to the **R** (Repudiation) threat in the STRIDE threat model. Systems that do not log user actions are inherently vulnerable to repudiation — users can deny their actions with no way to contradict them.

#### Log Integrity — The Foundation of Authenticity

If logs cannot be proven to be in their original, unmodified state, they lose their value for accountability and legal proceedings. Mechanisms to protect log integrity include:
- **Write-once storage:** Logs written to storage that cannot be modified after creation (WORM drives)
- **Cryptographic hashing:** Log files are hashed at collection; any modification changes the hash
- **Centralised, access-controlled storage:** Logs forwarded to a secure SIEM that endpoint users cannot modify
- **Chain of custody documentation:** Every access to log data is recorded

---

### Concept 2 — SIEM Architecture

#### What Is a SIEM?

A Security Information and Event Management (SIEM) system is a platform that:
- Ingests log data from across the organisation's estate
- Indexes and stores the data for fast retrieval
- Provides correlation and alerting capabilities
- Enables investigation of historical events
- Visualises security posture in real-time dashboards

Examples: Splunk, Wazuh, Elastic Stack (ELK), IBM QRadar, Microsoft Sentinel

#### The Three Core Components

```
Data Sources (Endpoints, Servers, Network Devices, Applications)
         ↓
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│   Forwarder   │────▶│    Indexer    │────▶│ Search Head   │
│ (collect/fwd) │     │ (store/index) │     │ (search/alert)│
└───────────────┘     └───────────────┘     └───────────────┘
```

| Component | Function | Analogy |
|-----------|----------|---------|
| **Forwarder** | Lightweight agent on endpoints that collects and transmits logs to the indexer | Mail carrier |
| **Indexer** | Receives, parses, and stores log data in a searchable format | Library with a card catalogue |
| **Search Head** | User-facing interface for searching, creating alerts, and visualising data | Librarian |

#### Accountability Focus: The Indexer

For accountability purposes, the most critical component is the indexer — it determines:
- What data is stored and in what format
- How long data is retained
- How quickly data can be retrieved for investigation
- Whether the data is protected from modification

The path from data source to indexer (data ingestion) is where most accountability failures occur.

---

### Concept 3 — Data Ingestion Methods

How log data travels from its source to the SIEM indexer determines the completeness, timeliness, and reliability of the log record.

#### Method 1: Agent/Forwarder

A lightweight software agent is installed on the endpoint. It collects local log data and forwards it to the SIEM.

**Advantages:**
- Real-time or near-real-time forwarding
- Can parse and filter data before forwarding (reducing ingestion costs)
- Works well across heterogeneous environments

**Challenges:**
- Agent must be installed and maintained on every endpoint
- Agent itself can be compromised by a threat actor (leading to log tampering or suppression)
- Management overhead scales with endpoint count

#### Method 2: Port-Forwarding

The source system sends logs directly to a specific network port on the SIEM (or a log aggregator).

**Advantages:**
- Simple configuration on the source device
- No software installation required

**Challenges:**
- May require network rule changes
- Less flexible than agent-based forwarding

#### Method 3: Syslog

Syslog is a standardised protocol for sending log messages over a network. Widely supported by network devices (routers, switches, firewalls) and Unix/Linux systems.

**Advantages:**
- Universal — almost every network device supports syslog
- Centralises logs from devices where agent installation is impossible

**Challenges:**
- UDP-based syslog (traditional) has no delivery confirmation — logs can be silently dropped
- TCP syslog and syslog-over-TLS are more reliable and secure alternatives

#### Method 4: Upload

Logs are collected and uploaded to the SIEM in batches (manually or via scheduled job).

**Advantages:**
- Works for systems that cannot maintain a continuous connection
- Useful for archival or legacy systems

**Challenges:**
- Not real-time — there is a gap between when events occur and when they are available for investigation
- Higher risk of loss if the upload fails

---

### Concept 4 — Log Storage Strategies

Log data has a lifecycle. The longer it is retained, the more it costs to store. The more frequently it is accessed, the faster the storage needs to be. Storage tiers balance cost and performance across the data lifecycle.

#### Hot Storage

- **Accessed:** Frequently (real-time or recent investigation)
- **Performance:** High — fast retrieval
- **Examples:** Solid-state drives (SSDs), high-performance HDDs
- **Use case:** Current incidents; last 30-90 days of logs

#### Warm Storage

- **Accessed:** Occasionally (historical investigation)
- **Performance:** Moderate
- **Use case:** 3-6 month old data; available but not immediately critical

#### Cold Storage

- **Accessed:** Rarely (regulatory compliance, legal proceedings, long-term archival)
- **Performance:** Low — slow retrieval acceptable
- **Examples:** Low-cost HDDs, tape drives
- **Use case:** Long-term compliance archiving; 6 months to several years

#### A Practical Storage Lifecycle Example

```
Log ingested → Hot storage (first 6 months)
                    ↓
             Warm storage (months 7-9)
                    ↓
             Cold storage (months 10 onwards)
                    ↓
             Indefinite archival (depending on regulatory requirements)
```

#### PCI DSS Log Retention Requirement

PCI DSS (Payment Card Industry Data Security Standard) requires:
- **Audit logs retained for 12 months**
- **Immediately available for the last 90 days**

This means the last 90 days must be in hot storage (immediately accessible), and the full 12 months must exist somewhere in the storage system.

---

### Concept 5 — What Makes a Good Log?

A log source is only valuable if it provides information that can support an investigation or prove accountability. A log that does not contain relevant, actionable information does not uphold non-repudiation.

**Key qualities of a good log:**
- Contains sufficient context to reconstruct what happened (who, what, when, where)
- Is structured in a consistent, parseable format
- Captures timestamps with sufficient precision and timezone accuracy
- Includes the identity of the actor performing the action
- Records the outcome (success/failure)
- Does not contain excessive noise that obscures relevant events

#### Types of Log Sources

**Manual Log Sources:** Logs created by human action.
- **Change logs:** Document changes made to systems, configurations, or processes — created by the person making the change
- Valuable because they document intent, not just effect

**Automated Log Sources:** Logs created automatically by systems, applications, and tools.
- **System logs:** OS-level events — logins, service starts/stops, hardware events
- **Application logs:** Application-level events — errors, transactions, user actions within the application

**Other/Compliance Log Types:** Logs required for specific compliance or investigative purposes.
- **Email logs:** Records of sent, received, and blocked messages — critical for phishing investigations
- **Messaging/communication logs:** Chat, collaboration tools — required in some regulatory environments

---

### Concept 6 — Log Correlation

#### What Is Log Correlation?

Correlation is the process of building relationships between multiple log sources to create a complete picture of an event or incident. A single log source rarely tells the whole story; the combination of multiple sources is what enables accurate scope determination.

**Example: DLL File Created on Disk**

```
Single log observation: System log shows DLL file created at 14:32

Correlated with browser log: User searched for "how to install X plugin" at 14:30
→ Plausible explanation: user was legitimately installing software

Correlated with email log: User received phishing email at 14:28
→ Raises concern: DLL creation may have been triggered by the phishing email

Correlated with network log: Outbound connection to unknown IP at 14:33
→ Confirms: likely malware delivery from phishing → DLL creation → C2 communication
```

The three logs together create a coherent narrative. Each individual log could be dismissed; the correlation establishes the attack chain.

#### Data Enrichment

Correlation is enhanced by data enrichment — adding contextual information to log events from external sources:
- **Threat intelligence feeds:** Is this IP address/domain known malicious?
- **User context:** What is this account's normal working hours and location?
- **Asset context:** Is this host a critical system or a development machine?
- **Geolocation data:** Is this login from an expected geographic location?

#### Mutual Validation

Multiple log sources can validate each other:
- A firewall log showing a connection to an external IP is validated by a host process log showing the specific process that made the connection
- An authentication log showing a login is validated by an application log showing what was accessed after login
- If the two logs are consistent — the story is credible; if inconsistent — investigation is required

#### Alert Noise vs Correlation Quality

More log sources are not always better. If multiple sources are capturing the same events and generating the same alerts, the result is:
- Increased storage complexity and cost
- Higher alert noise
- Analyst confusion about which source is authoritative

Log architecture should be designed so that each source adds unique information, and correlation adds value rather than redundancy.

---

## Architecture and Relationships

### Full Logging Pipeline for Accountability

```
Event occurs on endpoint (login, file creation, network connection)
         ↓
Local system log created (OS, application, audit log)
         ↓
Forwarder/Syslog agent collects the log
         ↓
Log transmitted to SIEM indexer
  (Agent forwarding / Port-forwarding / Syslog / Upload)
         ↓
Indexer stores and indexes the log
  (Hot storage — recent; Warm/Cold — aged data)
         ↓
Log authenticated (hash verified; chain of custody documented)
         ↓
SIEM search head enables:
  - Real-time alerting on rule matches
  - Historical investigation during incidents
  - Correlation across multiple log sources
  - Compliance reporting and audit export
         ↓
Non-repudiation achieved:
  User cannot deny action — log proves it, integrity is verified
```

---

## Security Engineer Perspective

### What Must Be Configured Correctly

| Concern | Requirement |
|---------|------------|
| **Log coverage** | Every system in scope must forward logs to the SIEM; identify and remediate blind spots |
| **Log integrity** | Write-once or cryptographically signed log storage; access controls on log modification |
| **Retention policy** | Meet regulatory minimums (PCI DSS: 12 months; HIPAA: 6 years) with documented hot/cold transitions |
| **Clock synchronisation** | All systems must use NTP to ensure timestamps are consistent across sources for accurate correlation |
| **Ingestion reliability** | Prefer TCP-based syslog over UDP; use acknowledgement-based forwarding where possible |
| **SIEM capacity** | Ensure ingestion capacity meets log volume; over-capacity leads to dropped events |

### Common Misconfigurations

- **Under-logging:** Systems not forwarding logs, or logs being dropped at the forwarder due to volume limits
- **Under-retention:** Logs purged before a regulatory or investigation need arises
- **Clock skew:** Systems with unsynchronised clocks produce log timestamps that cannot be accurately correlated
- **Local-only logs:** Systems logging only to local files, which can be deleted by a threat actor with sufficient access
- **Over-alerting on SIEM:** Too many noisy rules cause analysts to ignore genuine alerts
- **Unprotected log storage:** SIEM indexer accessible to endpoints (allowing log tampering)

### Offensive Perspective — How Attackers Target Logs

| Attack | Description | Mitigation |
|--------|-------------|-----------|
| **Log deletion** | Attacker deletes local logs to hide activity | Forward logs to central SIEM before deletion is possible |
| **Log tampering** | Attacker modifies logs to remove their traces | Cryptographic integrity; write-once storage |
| **Forwarder agent compromise** | Attacker disables or corrupts the log forwarding agent | Monitor forwarder health; alert on agent stoppage |
| **Log flooding** | Attacker generates massive volumes of log noise to obscure their activity | Anomaly detection on log volume; intelligent filtering |
| **Coverage exploitation** | Attacker identifies and uses systems not covered by logging | Regular coverage audits; alert on "no logs received" for known hosts |

---

## Practical Scenarios

### Normal Operation
```
User authenticates to CRM at 09:14 UTC
Log forwarded to SIEM within 30 seconds
Log stored in hot tier; retained 90 days active, 12 months total
No alerting rule triggered
Log available for investigation if needed
```

### Attack Scenario — Insider Threat
```
Insider accesses HR salary database at 22:47 (outside working hours)
→ SIEM alert: access to HR database outside business hours
→ Analyst investigates: retrieves auth log, application log, DLP log
→ Correlation: user logged in from office IP at 22:47; exported 10,000 records
→ Non-repudiation: auth log + application export log + DLP alert = proven action
→ HR, legal, and ISO notified; account suspended
```

### Attack Scenario — Log Deletion Attempt
```
Attacker compromises server at 03:12
→ Attempts to clear Windows Event Logs
→ Local logs deleted — but forwarding agent already sent logs to SIEM at 03:08
→ SIEM alert: log clearing event (Windows Event ID 1102)
→ Incident raised; all logs already preserved in SIEM
→ Log deletion fails to hide attacker activity
```

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **Accountability** | The IAAA pillar that holds users responsible for their actions via a verifiable record |
| **Agent/Forwarder** | Lightweight software installed on endpoints to collect and transmit logs |
| **Cold Storage** | Low-cost, slow-access storage for infrequently accessed archival data |
| **Correlation** | Building relationships between multiple log sources to create a complete event narrative |
| **Data Enrichment** | Adding contextual information to log events from external sources (threat intel, geolocation) |
| **Hot Storage** | High-performance, fast-access storage for frequently accessed recent data |
| **IAAA** | Identification, Authentication, Authorisation, Accountability — four pillars of access security |
| **Indexer** | SIEM component that receives, parses, stores, and makes log data searchable |
| **Non-repudiation** | The property that prevents a user from credibly denying they performed an action |
| **PCI DSS** | Payment Card Industry Data Security Standard — requires 12 months log retention |
| **Repudiation** | An individual disputing or denying an action they performed |
| **Search Head** | SIEM component providing the user-facing search, alert, and visualisation interface |
| **SIEM** | Security Information and Event Management — centralised log collection and analysis platform |
| **Syslog** | Standardised network protocol for sending log messages; widely supported by network devices |
| **Warm Storage** | Intermediate-performance storage between hot and cold for moderately accessed data |
| **WORM** | Write Once Read Many — storage that prevents modification after data is written |

---

## Exam and Interview Revision

### Must Remember

- Accountability is the **4th pillar** of IAAA — the others are Identification, Authentication, Authorisation
- Non-repudiation requires logs to be **collected, authentic, and retained** — all three conditions must be met
- SIEM three core components: **Forwarder → Indexer → Search Head**
- Four ingestion methods: **Agent, Port-forwarding, Syslog, Upload**
- Storage tiers: **Hot** (frequent access, high performance) → **Warm** (occasional) → **Cold** (archival, tape drives)
- PCI DSS: **12 months total retention; 90 days immediately available**
- UDP syslog has no delivery confirmation — logs can be silently dropped; prefer TCP syslog
- Log correlation builds a narrative from multiple sources — individual logs are insufficient
- Attackers delete local logs — centralised SIEM prevents this from hiding their activity
- Clock synchronisation (NTP) is essential for accurate log correlation across sources
- Over-alerting is as dangerous as under-alerting — signal-to-noise ratio determines alert value

### Common Interview Questions

| Question | Key Answer Points |
|----------|-----------------|
| What is non-repudiation and how is it achieved? | The property preventing denial of an action. Achieved through authenticated, tamper-proof logs collected at the time of the action, with verified integrity at time of review. |
| What are the three SIEM components? | Forwarder (collects and transmits), Indexer (stores and makes searchable), Search Head (user interface for queries and alerts). |
| Why is syslog over UDP problematic for accountability? | UDP has no delivery confirmation. Logs can be dropped silently without the sender or receiver knowing. Critical for accountability because missing logs break the non-repudiation chain. |
| What is cold storage and when is it used? | Low-cost, slow-access storage for archival data rarely accessed. Used for compliance retention (e.g., PCI DSS 12 months; HIPAA 6 years). Examples: low-cost HDDs, tape drives. |
| How does log correlation support incident response? | Multiple log sources (auth log + application log + network log) are combined to create a complete picture. Individual logs may be ambiguous; correlation provides context that confirms or refutes suspicious activity. |
| How does an attacker try to defeat logging, and how do defenders counter? | Delete local logs (counter: central SIEM receives logs before deletion); tamper with logs (counter: write-once storage, cryptographic hashing); disable forwarding agent (counter: monitor agent health, alert on absence). |
