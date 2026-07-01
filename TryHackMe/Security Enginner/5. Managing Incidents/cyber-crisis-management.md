# Cyber Crisis Management

## Executive Summary

This room covers the Crisis Management Team (CMT) — the highest level of incident response, invoked when a cyber incident has escalated beyond what the CSIRT can manage alone and requires executive authority to authorise disruptive "nuclear" actions. It covers the CMT structure and roles, the Golden Hour process, the cyclic CMT operational model (Information Updates → Triage → Action Discussions → Action Approvals), communication strategy, legal obligations, the role of Subject Matter Experts, and the distinction between jack-of-all-trades executives and master-of-one SMEs.

**Major concepts covered:** CMT roles and governance model (autocracy vs democracy), Golden Hour steps (Assembly → Information Gathering → Crisis Triage → Notifications), CMT operational cycle, holding statements, nuclear actions, internal/external communications, regulator notification, law enforcement engagement, and SME responsibilities.

**Why these concepts matter:** A cyber crisis — ransomware being deployed at scale, critical infrastructure under attack, mass customer data exfiltration — is an existential threat to an organisation. The CMT is the last line of defence. Its ability to make rapid, informed decisions with imperfect information determines whether the organisation survives. Understanding the CMT is essential for any security professional who may serve as an SME, provide briefings, or contribute to crisis response plans.

**Where these concepts apply:** Large enterprises with a defined CMT in their incident response framework; regulated industries (financial services, healthcare, critical infrastructure) where regulatory notification is mandatory; any organisation large enough that executives must be involved in major security decisions.

---

## Big Picture Overview

The cyber crisis is a different beast from a cyber incident. A cyber incident is managed within the security operations function — analysts, forensic teams, incident managers. A cyber crisis requires the organisation's most senior decision-makers to authorise actions that a security team alone cannot take: taking the entire organisation offline, deciding whether to pay a ransom, notifying regulators, managing public communications.

The fundamental challenge of the CMT is making critical, consequential decisions with:
- Incomplete information (the investigation is still ongoing)
- Time pressure (every minute of delay may mean more encrypted files, more exfiltrated data)
- Non-technical decision-makers (executives understand business impact, not technical mechanisms)
- High accountability (executives personally face legal and reputational consequences)

The mental model: think of the CMT as a wartime command structure. Technical experts (SMEs) provide intelligence and options. The command structure (CMT) makes decisions. Action is taken immediately.

```
CSIRT investigating →  CSIRT Chair invokes CMT
                              ↓
                     CMT assembles (Golden Hour)
                              ↓
          ┌─────────────────────────────────────────┐
          │         CMT Operational Cycle           │
          │                                         │
          │  SME Briefing                           │
          │      ↓                                  │
          │  Information Update                     │
          │      ↓                                  │
          │  Triage (severity, additional SMEs)     │
          │      ↓                                  │
          │  Action Discussions (limited time)      │
          │      ↓                                  │
          │  Action Approval (CEO/CMT Chair)        │
          │      ↓                                  │
          │  Implementation + Documentation         │
          │      ↓                                  │
          │  (repeat cycle at defined intervals)    │
          └─────────────────────────────────────────┘
                              ↓
                     Crisis resolved
                              ↓
                     Crisis documentation
```

---

## Core Concepts

---

### Concept 1 — What Is a Cyber Crisis?

A cyber crisis is a Level 4 incident — the highest severity classification — where the scope and impact require executive authority to manage. The CMT is invoked when the CSIRT needs to authorise "nuclear" actions that are beyond the authority of the security team.

#### Incident Severity Classification

The decision to invoke the CMT is made using an incident severity classification matrix. Factors typically include:
- Number of systems affected
- Type of data at risk (PII, financial, intellectual property)
- Operational impact (systems unavailable, revenue impact)
- Regulatory implications (breach notification required)
- Public/media attention risk
- Whether the threat actor has achieved privileged access (domain admin)

#### Example: Ransomware Deployment

A Level 3 CSIRT incident becomes a Level 4 CMT incident when:
- Ransomware is detected actively spreading across the estate
- The CSIRT determines that stopping the spread may require taking entire systems offline
- The impact of going offline would directly affect business operations and customers
- Ransomware has encrypted critical systems or is moving toward them

The CSIRT cannot independently authorise taking the environment offline — this requires CMT approval because it is a business decision, not just a technical one.

---

### Concept 2 — CMT Governance — Autocracy Over Democracy

#### Why Not Democracy?

In most organisational contexts, democratic decision-making is appropriate. For CMT, it is counter-productive:
- Time pressure makes prolonged debate lethal — every minute of discussion may mean thousands more files encrypted
- Consensus-building wastes critical time
- Inaction is often more damaging than imperfect action taken decisively

#### The Autocratic CMT Model

Decision authority is concentrated in a single individual — typically the CEO. However, given the weight of these decisions, many organisations distribute voting rights to a small group (typically no more than five individuals) including:
- CEO
- COO
- CIO or CISO
- Legal counsel
- Operations lead

This limited voting model maintains decisiveness while distributing accountability.

**Key principle:** The CMT Chair's primary duty is not to deliberate endlessly but to receive information, consider options, make decisions, and drive action. Inaction during a cyber crisis is a decision — usually the worst one.

---

### Concept 3 — CMT Roles and Responsibilities

| Role | Description | Primary Function |
|------|-------------|-----------------|
| **CMT Chair** | CEO or COO; leads the CMT | Final decision authority; sets pace and direction of CMT actions |
| **Executives** | CEO, COO, CIO, CTO, CFO, CISO | Strategic oversight; voting rights; ultimate accountability |
| **Communications** | PR/Comms team lead | Internal employee communications; external customer and media communications; control the narrative |
| **Legal** | General Counsel or external counsel | Ensure all CMT actions are legally permissible; advise on ransom payment legality; manage liability |
| **Operations** | COO or Operations specialist team | Identify ways to maintain business operations during the crisis; minimise business disruption |
| **Subject Matter Experts (SMEs)** | SOC Head, Incident Manager, System Experts | Provide technical intelligence and feasibility assessments; answer "what is possible and what would it cost?" |
| **Scribe** | Dedicated note-taker | Complete documentation of all events, discussions, decisions, and actions during the CMT session |

---

### Concept 4 — The Golden Hour

The Golden Hour is the first and most critical phase of CMT activation. Actions taken or not taken in this window significantly affect the outcome of the crisis.

#### Step 1: Assembly

The CSIRT Chair invokes the CMT and begins the notification process. Challenges:
- Key members may be unavailable (travel, personal emergencies)
- Primary communication channels may have been compromised by the threat actor

**Out-of-band communication planning:** If the organisation's email, Slack, Teams, or phone systems are compromised or suspected to be monitored by the threat actor, the CMT must have pre-established out-of-band channels. This typically means:
- Pre-distributed personal mobile numbers
- Designated secure messaging platforms not connected to the corporate infrastructure
- Physical assembly location (if remote communication is unavailable)

**Backup chains:** Every CMT role must have a pre-documented backup (and the backup's backup). The call tree for CMT assembly cannot have single points of failure.

#### Step 2: Information Gathering — The CSIRT Briefing

The CMT's first action is a formal briefing from the CSIRT. The briefing must cover:
1. **Summary of what has happened** — what was discovered, when, and by whom
2. **Summary of actions already taken** — what the CSIRT has done and the effects observed
3. **Recommendations for immediate nuclear actions** — what the CSIRT believes the CMT should authorise immediately

The briefing must be accessible to non-technical executives. Technical details are abstracted; the focus is on **business impact** — how many systems are affected, what data is at risk, what operations are impacted — not the technical mechanics of the attack.

#### Step 3: Crisis Triage

After the briefing, the CMT evaluates:
- Is the severity assessment accurate? Should it be raised or lowered?
- Which additional stakeholders should be brought into the CMT?
- Are the CSIRT's recommendations reasonable given the business impact?
- What immediate actions must be taken in the next 30-60 minutes?

#### Step 4: Notifications and Holding Statements

Even in the first hour, the CMT begins preparing communications. The priority is controlling the narrative before external parties (media, employees, customers) fill the vacuum with speculation.

**Holding statements:** Pre-drafted messages that:
- Acknowledge that the team is aware of an issue and investigating
- Provide reassurance that the team is actively working on it
- Do not disclose the specific nature of the incident (to avoid providing intelligence to the threat actor or unnecessarily alarming stakeholders)
- Commit to providing further updates

Example:
> "We are aware of a technical issue affecting some of our systems and our teams are working to resolve it as quickly as possible. We will provide further updates as more information becomes available. We apologise for any inconvenience this may cause."

---

### Concept 5 — The CMT Operational Cycle

After the Golden Hour, the CMT enters a repeating cycle:

```
SME Briefing (new intelligence from CSIRT/forensics)
         ↓
Information Update to CMT
         ↓
Triage (update severity; add/remove SMEs; communication decisions)
         ↓
Action Discussions (time-limited)
         ↓
Action Approvals (CEO/CMT Chair decides)
         ↓
Actions implemented; effects documented
         ↓
(repeat at defined interval — initially every 30-60 minutes;
 less frequently as crisis stabilises)
```

#### The Static CMT Principle

A critical operational principle: **the CMT remains assembled as a unit**. Individual members do not break away to gather information. Instead, SMEs are brought to the CMT. This ensures:
- All CMT members receive the same information simultaneously
- No CMT member is absent when a critical decision must be made
- Time is not wasted reassembling the team after breaks

#### Action Discussions — Why They Are Time-Limited

The CMT Chair enforces time limits on action discussions. Reasons:
1. **Ransomware spreads continuously** — Group Policy Object-based ransomware can encrypt an entire Windows environment in 120 minutes. Every minute of discussion is a minute of encryption.
2. **Inaction is a decision** — Choosing not to act has consequences, often worse than imperfect action
3. **Perfect information is impossible** — The investigation will never be complete; decisions must be made on best available information

The Chair's responsibility is to gather enough information, allow limited discussion, and then make a decision. Waiting for certainty is a luxury the CMT does not have.

#### Nuclear Actions — Examples

"Nuclear" actions are high-impact, potentially irreversible decisions that the CMT authorises:

| Nuclear Action | Business Impact | When Appropriate |
|---------------|----------------|-----------------|
| **Halt all VPN access** | Remote employees cannot work; all remote operations cease | Active lateral movement via VPN; threat actor using VPN for access |
| **Domain takeback (Active Directory)** | Disrupts all Windows authentication; potential widespread outage | Threat actor has domain admin access and is using it actively |
| **Switch to DR environment** | Potential data loss; requires verification of DR state | Primary environment is so compromised it cannot be recovered quickly |
| **Take entire environment offline** | Complete operational shutdown | Ransomware actively deploying; no other way to stop spread |

These are not taken lightly. The CMT's role is to weigh **impact vs effectiveness** — is this action sufficient to stop the crisis, and is the business disruption it causes proportionate?

---

### Concept 6 — Communications Strategy

#### Internal Communications

**Employees:** The CMT must manage what employees are told and when. Too little information creates fear and speculation. Too much information may compromise the investigation or violate legal obligations.

**Help desk:** If the CMT takes a nuclear action (e.g., disabling VPN, forcing password resets), the help desk will be overwhelmed with support requests. Pre-briefing the help desk with:
- What is happening (at an appropriate level)
- What to tell employees
- What actions they should or should not take
...prevents the help desk from inadvertently spreading panic or providing incorrect information.

#### External Communications

**Customers:** Communication about incidents affecting customers must be carefully timed and worded. Under GDPR, if personal data has been compromised, notification is legally required within 72 hours of discovery.

**Media:** In the social media era, news of a cyber crisis spreads rapidly regardless of whether the organisation makes a statement. The choice is not whether to communicate but whether to communicate proactively with a controlled message or reactively after the narrative is set by others.

Many large organisations employ dedicated crisis communications firms that specialise in managing organisational reputation during incidents. These firms prepare:
- Approved statement templates
- Media spokesperson briefings
- Social media response protocols
- Press release drafts

---

### Concept 7 — Legal and Regulatory Obligations

#### Regulator Notification

Depending on the organisation's sector and geography, notification to regulators may be legally mandated:

| Regulator/Framework | Trigger | Timeline |
|--------------------|---------|---------| 
| **GDPR (EU)** | Personal data breach | 72 hours from discovery (to supervisory authority) |
| **Financial regulators** (e.g., FCA, SEC, FINRA) | Material cyber incident | Varies; often within 72 hours to 1 business day |
| **Information Commissioner's Office (ICO - UK)** | Personal data breach | 72 hours |
| **Sector-specific regulators** (energy, healthcare, water) | Incidents affecting critical infrastructure | Varies by jurisdiction and severity |

Failure to notify within the required timeframe is itself a regulatory violation and can result in additional penalties on top of those from the breach itself.

#### Law Enforcement Engagement

Engaging law enforcement (FBI, NCSC, NCA) is a CMT-level decision. Benefits:
- Access to threat intelligence about the specific threat actor group
- Technical assistance with investigation
- Formal process for establishing chain of custody for prosecution
- In some cases, law enforcement may be able to assist with decryption (if they have obtained decryption keys from prior cases)

**Pre-decision:** Law enforcement contacts and engagement decision criteria should be documented in the CMT playbook before a crisis occurs, not decided under pressure during one.

#### Ransom Payment Legality

Paying a ransom is a CMT-level decision — not a technical one. Critical considerations:
- **Legality:** Paying ransoms may be illegal or restricted depending on the threat actor (if they are on sanctions lists — OFAC in the US — paying them is illegal regardless of circumstances)
- **Effectiveness:** Paying does not guarantee decryption; many organisations pay and receive non-functional decryption tools
- **Encouragement:** Paying funds future ransomware operations
- **Insurance:** Cyber insurance policies have specific terms about ransom payments

Legal counsel is essential for this decision. This is why legal representation is a core CMT role, not an optional one.

---

### Concept 8 — The SME's Role in the CMT

#### Jack of All Trades vs Master of One

CMT members (executives, legal, communications) have broad scope across the organisation. They are generalists who can lead, decide, and communicate — but they cannot be technical experts in every system.

SMEs provide the depth that CMT members lack. As a security engineer, if a CMT crisis involves your systems, you will be the SME. Your value to the CMT is specific:

1. **Accurate scope assessment:** Only you fully understand your system's dependencies, what is and is not affected, and what the attacker would be able to do with the access they have
2. **Recovery options:** What backup and recovery options exist? How long would they take? What would be lost?
3. **Action feasibility:** Can this specific nuclear action be implemented? What are the side effects?
4. **Impact assessment:** If we take system X offline, what business processes are affected? For how long?

#### How SMEs Communicate With the CMT

The CMT does not have time for deep technical explanations. SME briefings must:
- **Abstract technical details** — focus on business impact, not mechanisms
- **Be definitive where possible** — "taking this system offline would affect X, Y, Z functions for approximately N hours"
- **Provide options with clear trade-offs** — "Option A: partial isolation (lower disruption, higher risk of spread); Option B: full isolation (higher disruption, stops spread)"
- **Be honest about uncertainty** — "we estimate X but we do not yet know Y"

Providing unclear or technically dense information to a non-technical CMT delays decisions and ultimately makes the crisis worse.

---

### Concept 9 — Documentation and Crisis Closure

#### The Scribe's Responsibility

The scribe documents the complete timeline of the CMT session:
- All information received from SME briefings
- All options discussed
- All decisions made and who made them
- All actions authorised and who was assigned to execute them
- All actions completed and their observed effects
- All communications sent and to whom

This documentation is:
- **Legal evidence** — in regulatory investigations or prosecutions, the CMT's decisions and rationale must be documented
- **Insurance evidence** — cyber insurance claims require documentation of the incident and response
- **Lessons learned input** — the post-crisis review uses this document to improve future CMT processes

#### Crisis Closure

A cyber crisis is formally closed when:
1. The threat actor has been eradicated from the environment
2. All affected systems have been recovered or are in active recovery
3. Business operations have returned to BAU (or a defined acceptable level)
4. All required external notifications have been made
5. A crisis closure document has been prepared

The crisis closure document includes:
- Summary of what happened and the root cause
- Timeline of events and CMT actions
- Lessons learned and recommended improvements
- Follow-up actions (hardening, policy changes, process improvements) with owners and timelines

---

## Architecture and Relationships

### CMT Information Flow

```
Technical Investigation
  (CSIRT, Forensics, SOC)
           ↓
    SME Briefings (technical intelligence
    translated into business impact terms)
           ↓
         CMT
    ┌─────────────────────────────────┐
    │  Chair (CEO/COO)                │
    │  Executives (decision voters)   │
    │  Legal (compliance/legality)    │
    │  Communications (narrative)     │
    │  Operations (business impact)   │
    │  Scribe (documentation)         │
    └─────────────────────────────────┘
           ↓
    Decisions and Authorisations
           ↓
    ┌──────────────────────────────────┐
    │  Nuclear Actions Implemented     │
    │  (by CSIRT/technical teams)      │
    └──────────────────────────────────┘
           ↓
    Communications
    ┌──────────────────────────────────┐
    │  Internal: employees, help desk  │
    │  External: customers, media      │
    │  Regulatory: ICO, FCA, FBI       │
    └──────────────────────────────────┘
```

---

## Real-World Security Operations

### How the CMT Operates in Practice

**Pre-Crisis:**
- CMT composition, call tree, and roles are documented in advance
- Holding statement templates are pre-drafted for common crisis types
- Out-of-band communication methods are established and tested
- Tabletop exercises test the CMT's ability to make rapid decisions under simulated crisis conditions
- Nuclear action playbooks are defined with clear criteria for invocation

**During Crisis:**
- First briefing within Golden Hour — CSIRT Chair presents initial assessment
- Update cadence established (every 30 minutes in acute phase)
- Legal counsel continuously reviewing all actions
- Communications team monitoring social media and preparing statements
- Scribe maintaining real-time documentation

**Post-Crisis:**
- Crisis closure document prepared by Scribe
- Lessons learned session with CMT, CSIRT, and relevant SMEs
- Action items assigned for process and security improvements
- Regulatory filings completed
- Insurance claim submitted

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **CMT** | Crisis Management Team — executive-level team that manages Level 4 cyber crises |
| **CMT Chair** | Leader of the CMT; typically CEO or COO; has final decision authority |
| **Crisis Closure Document** | Post-crisis summary of what happened, actions taken, and lessons learned |
| **Golden Hour** | The first and most critical period after CMT invocation |
| **Holding Statement** | Pre-drafted communication that acknowledges an issue without disclosing specifics |
| **NCSC** | National Cyber Security Centre — UK government cybersecurity authority |
| **Nuclear Actions** | High-impact, potentially disruptive actions that only the CMT can authorise |
| **Out-of-Band Communication** | Communication through channels separate from potentially compromised corporate infrastructure |
| **Ransom Payment** | Decision to pay a ransomware threat actor; a CMT-level decision with legal implications |
| **Scribe** | CMT member responsible for complete documentation of all events and decisions |
| **SME** | Subject Matter Expert — technical expert who provides intelligence to the CMT |
| **Static CMT Principle** | CMT remains assembled as a unit; SMEs are brought to the CMT rather than CMT members dispersing |

---

## Exam and Interview Revision

### Must Remember

- CMT is invoked at **Level 4 incidents** — when CSIRT needs authority for nuclear actions
- CMT governance is **autocratic**, not democratic — speed and decisiveness over consensus
- CMT Chair is typically CEO or COO; has final decision authority; some CMTs distribute voting to up to 5 members
- Core CMT roles: Chair, Executives, Communications, Legal, Operations, SMEs, Scribe
- **Golden Hour steps:** Assembly → Information Gathering (CSIRT briefing) → Crisis Triage → Notifications
- **CSIRT briefing contents:** What happened, actions taken and their effects, recommendations for immediate nuclear actions
- Briefings to CMT must be **business impact focused** — abstract technical details
- **Static CMT principle:** CMT stays assembled; SMEs are brought to them
- Action discussions are **time-limited** — inaction is often worse than imperfect action
- Holding statements: acknowledge the issue without disclosing specifics; control the narrative
- GDPR requires notification within **72 hours** of discovering a personal data breach
- Ransom payment may be **illegal** if the threat actor is on OFAC sanctions list — legal counsel is essential
- SMEs provide: scope accuracy, recovery options, action feasibility, impact assessment
- **Scribe documentation** is the legal record of CMT decisions — every action, discussion, and decision must be captured
- Post-crisis: crisis closure document → lessons learned → improvements → regulatory filings

### Common Interview Questions

| Question | Key Answer Points |
|----------|-----------------|
| What triggers a CMT invocation? | Level 4 incident requiring executive authority for nuclear actions (e.g., taking environment offline, domain takeback, halting all remote access). CSIRT cannot authorise these independently. |
| Why is the CMT autocratic rather than democratic? | Time pressure — ransomware can encrypt an entire environment in 120 minutes. Prolonged debate is lethal. The CMT Chair must be able to make decisions decisively with limited information. |
| What happens during the Golden Hour? | Assembly (CMT convened via call tree); Information Gathering (CSIRT briefing: what happened, what was done, recommendations); Crisis Triage (severity assessment, stakeholder decisions); Notifications (holding statements prepared and distributed). |
| What is a holding statement? | A pre-drafted communication acknowledging an issue without disclosing specifics. Provides reassurance that the team is investigating. Prevents narrative vacuum that leads to speculation and panic. |
| What is the SME's role in the CMT? | Provide technical intelligence translated into business impact terms. Assess scope accuracy, recovery options, and action feasibility. Enable the non-technical CMT to make informed decisions. |
| What must be documented during a CMT session? | All information received, all discussions, all decisions and who made them, all actions authorised and assigned, all actions completed and their effects, all communications sent. |
| When is regulator notification required? | GDPR: 72 hours for personal data breaches. Financial regulators: varies (often 72 hours to 1 business day). Requirements vary by jurisdiction and sector — legal counsel must advise. |
