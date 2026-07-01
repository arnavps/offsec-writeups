# Becoming a First Responder

## Executive Summary

This room addresses the practical reality that security engineers — not just SOC analysts — can find themselves as the first person to discover a security incident. It covers evidence preservation in volatility order, the critical mistakes that destroy forensic value, chain of custody, incident playbooks and call trees, the correct order of containment actions, Business Continuity Planning (BCP), BCP metrics, and the importance of documentation during an incident.

**Major concepts covered:** Evidence volatility hierarchy (IETF order), the three major "don'ts" for first responders, chain of custody, playbooks and call trees, containment methods (network segmentation, physical isolation, virtual isolation, rate limiting), BCP vs DRP, BCP metrics (RPO, RTO, WRT, MTD, MTBF, MTTR), and documentation templates.

**Why these concepts matter:** Every security engineer is a potential first responder. The first few minutes after incident discovery are the most consequential for the investigation. Evidence destroyed in those minutes — through well-intentioned but incorrect actions — cannot be recovered. Understanding what to do (and what not to do) before the blue team arrives determines whether the incident can be properly investigated and attributed.

**Where these concepts apply:** Any security engineer, developer, system administrator, or IT professional who could discover an incident in their environment before the SOC is informed.

---

## Big Picture Overview

The scenario: you are a security engineer. You discover that one of your servers appears to have been compromised. What is the first thing you do?

Most people's instinct is wrong. The instinct is to shut the machine down (remove the threat) or disconnect it from the network (stop the spread). Both of these actions, while intuitive, can:
1. Destroy irreplaceable forensic evidence stored in volatile memory
2. Alert the threat actor that they have been detected, potentially triggering a more destructive action (deploying ransomware, deleting data, covering tracks)

The correct approach is systematic evidence preservation followed by expert notification, not immediate isolation or shutdown.

```
Discovery
    ↓
DO NOT power off  ← Most common destructive mistake
DO NOT disconnect network  ← May alert threat actor
    ↓
Preserve evidence in volatility order (most volatile first)
    ↓
Notify the blue team / SOC
    ↓
Support containment decision (as SME)
    ↓
Document everything
    ↓
Support BCP invocation if required
```

---

## Core Concepts

---

### Concept 1 — Evidence Volatility and the IETF Collection Order

The Internet Engineering Task Force (IETF) published *Guidelines for Evidence Collection and Archiving* (RFC 3227) which establishes the order in which digital evidence should be collected, based on volatility — how quickly the evidence will be lost or change.

#### The Volatility Hierarchy

| Priority | Evidence Type | Volatility | Why It Matters |
|----------|--------------|-----------|----------------|
| 1 | **Registers and Cache** | Extreme — changes every microsecond | Reflects exact state of CPU at time of incident; critical for malware analysis |
| 2 | **Routing Table, ARP Cache, Process Table, Kernel Statistics, RAM** | Very high — TTLs expire; processes change | Network communication paths; what processes were running; malware staged in memory |
| 3 | **Temporary File Systems** | High | Active sessions, temporary application state, staged files |
| 4 | **Disk** | Moderate | File system artefacts, installed malware, modified files; critical for legal proceedings |
| 5 | **Remote Logging and Monitoring** | Low-Moderate | SIEM logs have retention policies; they are not permanent |
| 6 | **Physical Configuration and Network Topology** | Low | Network diagrams, host placement, subnet configuration |
| 7 | **Archival Media (Backups)** | Very low | Historical comparison to determine how far back the incident extends |

#### Why RAM Is Critical

Malware has evolved to use memory-resident techniques specifically to avoid disk-based detection:
- **Fileless malware** exists only in RAM — there is no disk artefact for AV/EDR to scan
- **Staged payloads** — the initial dropper executes from disk, but subsequent payloads are injected into memory only
- **Code injection** — malware injects code into legitimate processes (e.g., `explorer.exe`) that appear clean on disk

Even if the team has a sample of the malware, they cannot fully understand what it was doing on this specific host without analysing it in the memory where it was executing.

#### Why Routing Tables and ARP Cache Matter

The incident may have started on one host but spread to others. Routing and ARP tables contain:
- Active network connections the host was making at the time of the incident
- Evidence of lateral movement to other hosts
- Timestamps that will expire as TTLs (Time-to-Live values) count down

This data exists only in volatile memory and will be lost within minutes to hours if not captured.

---

### Concept 2 — The Three Major DON'Ts

#### DON'T 1: Do Not Power Off the Device

This is the single most common destructive mistake. Powering off:
- Destroys all RAM contents (process table, ARP cache, network connections, memory-resident malware)
- May trigger malware anti-forensics routines designed to activate on shutdown
- Prevents the forensic team from capturing the running state

**Exception:** If the device is actively destroying evidence (e.g., wiping its own disk), the calculus changes. But this is a decision for the forensic team, not the first responder.

#### DON'T 2: Do Not Trust Programs on the Compromised System

The threat actor may have replaced legitimate system utilities with modified versions that:
- Report incorrect information (e.g., `ps` shows no suspicious processes, but they exist)
- Delete or modify logs during execution
- Taint the evidence you are trying to capture

Always use trusted tools from a verified forensic environment (e.g., a forensic boot drive or trusted external tools) rather than the native utilities on the compromised system.

#### DON'T 3: Do Not Run Programs That Modify File Access Times

File access timestamps are evidence. When and by whom a file was last accessed is forensically relevant. A simple copy-paste operation updates the "last accessed" timestamp, contaminating the evidence.

Forensic evidence collection uses:
- **Hardware write blockers** — physically prevent writes to the evidence drive during imaging
- **Forensic imaging tools** (dd, FTK Imager, dc3dd) that capture bit-for-bit copies without modifying the source

---

### Concept 3 — Chain of Custody

#### What Is Chain of Custody?

Chain of custody is the documented record of who handled evidence, when, how, and what was done with it. It is required for evidence to be admissible in legal proceedings.

The chain must prove:
1. Evidence was collected intact from the original source
2. Only authorised personnel handled it
3. Every access was documented
4. The current copy matches the original (verified via cryptographic hash)

#### Digital Evidence Chain of Custody Process

```
1. Evidence identified on compromised system
         ↓
2. Forensic copy (image) created
   → Hash computed (MD5, SHA-256) of original
         ↓
3. Original sealed and stored securely (never analysed directly)
         ↓
4. Analysis performed only on copy
         ↓
5. After analysis, copy's hash verified against original
   → Match = evidence integrity maintained
   → Mismatch = evidence has been modified; integrity compromised
         ↓
6. Every access to evidence documented:
   - Who accessed it
   - When
   - For what purpose
   - What was found
```

**Why this matters:** If the evidence chain is broken — if it cannot be proven that the evidence is in its original state — it cannot be used in court. Threat actors can argue that evidence was fabricated or tampered with if chain of custody documentation is absent.

---

### Concept 4 — Incident Playbooks

#### What Is a Playbook?

A playbook is a predefined, documented set of steps for responding to a specific type of incident. It ensures the response process is:
- **Repeatable** — every incident of this type is handled consistently
- **Complete** — no steps are forgotten under the stress of an active incident
- **Efficient** — the team does not waste time deciding what to do; they follow the process

#### Playbook Integration

Playbooks are designed to chain together. A phishing playbook, upon determining that the user clicked a malicious link, would reference and invoke the account compromise playbook. The team follows both simultaneously.

**Example phishing playbook flow:**
```
1. Receive phishing email report
2. Investigate email: identify sender, subject, links, attachments
3. Check if other users received the same email → IF YES: escalate to CERT
4. Check if any users clicked links or opened attachments → IF YES: invoke account compromise playbook
5. Block sender at mail gateway
6. Update spam filter signatures
7. Notify affected users
8. Document actions and close ticket
```

#### Security Engineer's Playbook Responsibility

Security engineers are not typically responsible for creating the full incident playbooks — that is the SOC's responsibility. However, they may be responsible for:
- Creating a playbook for incidents within their specific division
- Documenting how and where to raise an incident from their area
- Defining the escalation path from their division to the blue team

---

### Concept 5 — Call Trees

#### What Is a Call Tree?

A call tree documents who must be notified during an incident, who is responsible for notifying them, and who the backup contact is if the primary is unavailable.

```
Example Call Tree Structure:

SOC Manager
    ├── Analyst Team Lead (if SOC Manager unavailable: Deputy SOC Manager)
    │       ├── Analyst 1
    │       ├── Analyst 2
    │       └── Analyst 3
    └── Incident Manager
            ├── Legal Counsel
            ├── Communications Lead
            └── ISO (Information Security Officer)
                    └── Security Engineer (SME for specific division)
```

In large organisations, call trees are automated through ticketing systems (Jira, ServiceNow) that automatically notify stakeholders based on incident severity classification. In smaller organisations, manual call trees are standard.

**Security engineer responsibility:** As a security engineer, you may need to:
- Create the call tree for your division
- Document the escalation path to the blue team
- Identify your replacement in the tree if you are unavailable

---

### Concept 6 — Containment Methods

Once the blue team has been notified, the next question is how to contain the incident. As a first responder, you will not typically perform containment unilaterally — but you will be consulted as an SME on what is feasible in your environment.

#### Why Order Matters

```
WRONG ORDER:
  Eradicate (reset passwords) while threat actor has AD access
  → Threat actor uses existing access to re-harvest credentials
  → Eradication wasted; have to redo it

CORRECT ORDER:
  Contain → Eradicate → Recover
  (Cut off access → Remove malware/clean → Restore to BAU)
```

#### Containment Methods

| Method | Description | Pros | Cons |
|--------|-------------|------|------|
| **Network Segmentation** | Host is moved to an isolated network segment via VLAN/firewall rule | Prevents lateral spread; host remains powered; less disruptive than physical isolation | Requires network access; may not be fast enough for active spread |
| **Physical Isolation** | Host is physically confiscated and disconnected | Complete isolation; evidence preserved | Disruptive; requires physical access; takes user offline |
| **Virtual Isolation (EDR Jailing)** | EDR software restricts the host to only communicate with specific endpoints | Can be performed remotely; preserves forensic state | If the EDR agent is compromised, this may not work |
| **Rate Limiting** | Network speed to/from the host is dramatically reduced | Does not alert the threat actor (everything still works, just slowly) | Not full isolation; only buys time for investigation |

#### The Rate Limiting Technique

Rate limiting is a sophisticated technique used when full isolation might tip off the threat actor. By slowing the network connection:
- The threat actor does not realise they have been detected (everything still "works")
- Their command-and-control channel is severely impaired (they cannot effectively issue commands or exfiltrate data at scale)
- The blue team gains time to perform in-depth analysis of C2 traffic and lateral movement patterns
- Once the scope is understood, full isolation can be performed with confidence

This technique is colloquially referred to as "sending the threat actor back to the dial-up days."

---

### Concept 7 — Business Continuity Planning (BCP)

#### What Is a BCP?

A Business Continuity Plan (BCP) is a pre-documented plan that enables an organisation to continue critical operations and recover from an incident. It covers the full breadth of recovery — from technical system restoration to stakeholder communication and legal obligations.

#### BCP vs DRP

| Aspect | BCP | DRP |
|--------|-----|-----|
| **Scope** | Entire organisation — operations, communications, legal, technical | Technical recovery of systems and data |
| **Focus** | Maintaining business continuity and managing all aspects of recovery | Restoring technology to operational state |
| **Relationship** | BCP is more encompassing; DRP is typically included within the BCP |

#### Why BCP Must Come After Containment

BCP invocation gives senior management the authority to bypass normal change management processes (emergency changes without full approval chains). This is powerful but only makes sense once containment has stopped the threat actor from undoing recovery actions. Recovering without containment is futile.

#### Creating a BCP — Key Steps

| Step | Activity |
|------|----------|
| **Business Impact Analysis (BIA)** | Identify worst-case scenarios; determine what would happen to the organisation and customers; use qualitative and quantitative measures |
| **Define Recovery Actions** | For each BIA scenario, document the available recovery options (DR failover, backup restoration, manual workaround) |
| **Plan the BCP Team Structure** | Document who is responsible for what during BCP invocation |
| **Test the BCP** | Tabletop exercises; simulated recovery drills; verify that recovery actions work as documented |

#### BCP Metrics

These quantitative metrics are the core outputs of the Business Impact Analysis and determine the technical requirements for backup and recovery infrastructure:

| Metric | Definition | Practical Meaning |
|--------|-----------|------------------|
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss | If RPO = 1 hour, backups must run every hour |
| **RTO** (Recovery Time Objective) | Maximum acceptable time to restore hardware/infrastructure | Time to get servers online after failure |
| **WRT** (Work Recovery Time) | Time to restore software, applications, and data after hardware is available | Time to restore data, reinstall apps, reconfigure |
| **MTD** (Maximum Tolerable Downtime) | Maximum total downtime the organisation can survive | Must be ≥ RTO + WRT |
| **MTBF** (Mean Time Between Failures) | Average time the system operates between incidents | Longer is better — indicates reliability |
| **MTTR** (Mean Time To Repair) | Average time to recover from a failure | Shorter is better — indicates recovery efficiency |

**Critical relationship:** RTO + WRT must be less than or equal to MTD. If recovering the hardware takes 2 hours and restoring software takes 3 hours, but the business can only tolerate 4 hours of downtime, the plan fails.

---

### Concept 8 — Documentation During an Incident

#### Why Documentation Is Critical When BCP Is Invoked

BCP invocation bypasses normal change management processes. This means:
- Changes are made without the normal approval and documentation workflow
- Without alternative documentation, there is no record of what was changed, why, or by whom
- Retracing actions becomes impossible
- Legal and regulatory obligations cannot be met

#### Documentation Template Requirements

Every action during an incident should be documented with:

| Field | Purpose |
|-------|---------|
| **Time action was requested** | Establish the timeline; use UTC to enable cross-source correlation |
| **Description of the action** | What was done |
| **Reasoning for the action** | Why it was done; what problem it was addressing |
| **Individual approving the action** | Who authorised this step |
| **Individual responsible for performing it** | Who is accountable for execution |
| **Time action was performed** | When it was actually done (may differ from when it was requested) |
| **Description of changes observed** | Did the action have the expected effect? |

**The two-timestamp requirement:** Both when the action was *requested* and when it was *performed* must be recorded. A requested action that was never performed creates the illusion of progress while the incident continues to worsen. Documenting both fields ensures accountability for execution, not just discussion.

#### Lessons Learned

Documentation is not just for the current incident. The post-incident review uses this documentation to:
- Rebuild the exact timeline of events
- Identify decisions that could have been made faster or better
- Find gaps in playbooks, tools, or team capabilities
- Drive improvements in preparation for the next incident

The better the documentation, the more useful the lessons learned process — and the better the organisation will handle the next incident.

---

## Architecture and Relationships

### First Responder Decision Tree

```
Discover potential incident
         ↓
Assess: is this a security incident?
  → YES: continue
  → UNSURE: treat as yes until confirmed otherwise
         ↓
DON'T power off
DON'T disconnect from network
DON'T run programs on the system
         ↓
Document the current state (what you see, when you saw it)
         ↓
Notify the SOC / blue team
  → Use call tree if direct contact unavailable
         ↓
Preserve volatile evidence in order (if forensic tools available):
  Registers → ARP/Routing → Processes/RAM → Temp files → Disk → Logs
         ↓
Await blue team instruction
  → Provide SME input on containment feasibility
  → Support containment as instructed
         ↓
Document all actions taken and their effects
         ↓
Continue documentation through BCP invocation if required
```

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **BCP** | Business Continuity Plan — comprehensive plan for maintaining operations and recovering from incidents |
| **BIA** | Business Impact Analysis — assessment of potential incident scenarios and their organisational impact |
| **Call Tree** | Documented notification structure showing who must be contacted and who is responsible for contacting them |
| **Chain of Custody** | Documentation proving that evidence has been collected, handled, and analysed without tampering |
| **DRP** | Disaster Recovery Plan — technical plan for restoring systems after a failure (subset of BCP) |
| **EDR Jailing** | Virtual isolation of a host through EDR agent that restricts its network communication |
| **MTD** | Maximum Tolerable Downtime — the maximum outage the organisation can survive |
| **MTBF** | Mean Time Between Failures — average operational time between incidents |
| **MTTR** | Mean Time To Repair — average time to recover from a failure |
| **Network Segmentation** | Isolating a host into a separate network segment to prevent lateral spread |
| **Physical Isolation** | Physically confiscating and disconnecting a host |
| **Playbook** | Predefined response steps for a specific incident type |
| **Rate Limiting** | Throttling network speed to impair attacker communications without alerting them |
| **RFC 3227** | IETF document defining evidence collection order based on volatility |
| **RPO** | Recovery Point Objective — maximum acceptable data loss |
| **RTO** | Recovery Time Objective — maximum acceptable time to restore hardware |
| **Virtual Isolation** | Software-based host isolation through EDR or network controls |
| **Volatile Evidence** | Evidence that will be lost if the device loses power (RAM, ARP cache, process table) |
| **WRT** | Work Recovery Time — time to restore software and data after hardware is available |

---

## Exam and Interview Revision

### Must Remember

- The biggest first responder mistake: **powering off the device** — destroys volatile evidence
- Second biggest mistake: **disconnecting from the network immediately** — may alert the threat actor
- IETF evidence volatility order: Registers → ARP/Routing/RAM → Temp files → Disk → Remote logs → Physical config → Backups
- **RAM is critical** — fileless malware exists only in memory; stages payloads only in RAM
- **Chain of custody** requires: collect intact, document all access, analyse only copies, verify hash match
- Playbooks provide: repeatable, complete, efficient response to known incident types
- Call trees document: who to notify, who notifies them, and who the backup is
- Containment order: **Contain → Eradicate → Recover** — never skip containment
- Rate limiting = "dial-up technique" — impairs C2 without alerting threat actor; buys investigation time
- BCP vs DRP: BCP = entire organisation recovery (including comms, legal); DRP = technical recovery only (subset of BCP)
- BCP metrics: RPO (data loss tolerance), RTO (hardware restore time), WRT (software restore time), MTD (max downtime), MTBF (reliability), MTTR (recovery speed)
- **RTO + WRT ≤ MTD** — if recovery takes longer than the organisation can tolerate, the BCP fails
- Documentation must record: when requested, who approved, who performed, when performed, what changed

### Common Interview Questions

| Question | Key Answer Points |
|----------|-----------------|
| Why shouldn't you power off a compromised host? | Destroys volatile evidence (RAM, process table, ARP cache) that may be the only record of malware behaviour and network activity. Forensic analysis requires capturing the live state first. |
| What is the IETF evidence volatility order? | Registers → Routing/ARP/Processes/RAM → Temp files → Disk → Remote logs → Physical config → Backups |
| What is chain of custody and why does it matter? | Documentation proving evidence has not been tampered with. Required for legal proceedings. Must show: who collected, who accessed, that analysis was on a copy, that hash matches original. |
| What is the difference between RPO and RTO? | RPO = maximum data loss the organisation can accept (determines backup frequency). RTO = maximum time to restore hardware/infrastructure after failure. Both must be under MTD. |
| Why must containment precede eradication? | Without containment, the threat actor retains access and can undo eradication actions. Example: resetting passwords while the attacker still has domain admin access is immediately reversible. |
| What is the BCP "superpower" and why does it require careful documentation? | BCP allows bypassing normal change management approvals for emergency actions. Without alternative documentation, there is no record of changes made — making post-incident review and legal accountability impossible. |
