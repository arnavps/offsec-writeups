# Risk Management
## Security Engineer Knowledge Base

> **Source Room:** Risk Management — TryHackMe Security Engineer Path
> **Purpose:** Long-term reference for revision, interviews, certifications, blue-team and security engineering work

---

## Overview

### What This Room Teaches

- Precise definitions of threat, vulnerability, asset, and risk and how they interact
- Threat classification: human-made, technical, and natural
- The NIST SP 800-30 four-step risk management process: Frame → Assess → Respond → Monitor
- Qualitative and quantitative risk analysis, including SLE, ALE, and ARO calculations
- Risk response options and when to use each
- Supply chain risk — hardware, software, and service provider attack vectors
- Vulnerability management as a six-phase lifecycle tied to the NIST Cybersecurity Framework (CSF)

### Why It Matters

Risk management is the discipline that turns the abstract concept of "cyber risk" into a structured, measurable, and business-aligned decision-making process. Security engineers who understand risk management can justify controls using financial calculations, communicate risk to executives in business terms, and build programmes that prioritise the right protections in the right places.

### Where These Concepts Are Used in Real Environments

- **CISO and security leadership** — risk registers, risk treatment decisions, board reporting
- **Security engineering** — ALE calculations justify control investment; risk framing defines what to protect
- **Compliance and GRC** — risk assessments are mandatory deliverables for ISO 27001, SOC 2, HIPAA
- **Procurement and legal** — supply chain risk assessments for vendors and service providers
- **Vulnerability management teams** — CVSS scoring, remediation prioritisation, lifecycle tracking
- **Insurance** — quantitative risk metrics (ALE) feed cyber insurance underwriting

### Key Takeaways

- Risk = probability that a threat exploits a vulnerability and causes adverse business impact
- SLE × ARO = ALE — the financial basis for justifying security control spending
- Risk can be mitigated, transferred, accepted, or avoided — the choice depends on cost and tolerance
- Risk monitoring is continuous, not a one-time activity — effectiveness, change, and compliance all shift
- Supply chain risk is real and systematic — SolarWinds demonstrated it at scale
- Vulnerability management is a lifecycle (6 phases), not a periodic scan

---

## Core Concepts

---

### Concept 1 — Core Definitions

| Term | Definition |
|------|-----------|
| **Threat** | An intentional or accidental event that can compromise the security of an information system. Examples: hacking, phishing, human error, natural disasters. |
| **Vulnerability** | A weakness in software, hardware, or processes exploitable by a threat to cause harm or gain unauthorised access. |
| **Asset** | Any valuable resource (tangible or intangible) an organisation depends upon to achieve its objectives. |
| **Risk** | The probability of a threat exploiting a vulnerability resulting in adverse business effects. |
| **Risk Management** | The ongoing process of identifying, assessing, and mitigating risk to maintain acceptable levels. |
| **Risk Management Policy** | A set of procedures designed to minimise the probability and impact of adverse events. |

#### Why These Distinctions Matter

Confusing threat with vulnerability is a common error. A threat cannot cause harm without a vulnerability to exploit. A vulnerability is harmless if no threat actor can reach or is motivated to exploit it. Risk only exists when threat, vulnerability, and asset value all coexist.

---

### Concept 2 — Threat Classification

#### Human-Made Threats

Caused by human activities — intentional or accidental:
- **Cyberattacks** — hacking, phishing, malware, ransomware, DDoS
- **Social engineering** — pretexting, vishing, business email compromise
- **Insider threats** — malicious employees, accidental data disclosure
- **Physical threats** — arson, theft of hardware, tailgating into secure areas
- **Terrorism, civil unrest** — can disrupt facilities and supply chains

#### Technical Threats

Result from technological failures, malfunctions, or vulnerabilities:
- Power outages, UPS failures
- Hardware failures (disk failure, NIC failure, PSU failure)
- Software bugs, zero-day vulnerabilities, unpatched systems
- Network outages, DNS failures, BGP misconfigurations

#### Natural Threats

Caused by natural events — site-dependent risk:
- Earthquakes, floods, hurricanes, tornadoes
- Lightning strikes causing power surges
- Extreme temperature events affecting data centres

**Important:** Natural threat analysis must be site-specific. A data centre in coastal Florida has a materially different flood and hurricane risk profile than one in central Germany.

---

### Concept 3 — Assets in Information Systems

| Asset Type | Examples |
|-----------|---------|
| **Hardware** | Servers, workstations, routers, switches, firewalls, storage arrays |
| **Software** | Operating systems, applications, databases, custom code |
| **Data** | Customer records, PII, financial data, intellectual property, credentials |
| **Documentation** | Policy documents, architecture diagrams, manuals, contracts |
| **Services** | Cloud services, email providers, payment processors, CDNs |
| **Human Capital** | Knowledge, skills, and relationships of key personnel |

Asset identification is the foundation of risk assessment — you cannot assess risk to assets you have not identified.


---

### Concept 4 — Risk Management Frameworks

| Framework | Origin | Approach |
|-----------|--------|---------|
| **NIST SP 800-30** | NIST | Four-step structured process: Frame → Assess → Respond → Monitor |
| **FRAP** (Facilitated Risk Analysis Process) | Industry | Collaborative workshop — stakeholders identify and evaluate risks together |
| **OCTAVE** (Operationally Critical Threat, Asset, and Vulnerability Evaluation) | Carnegie Mellon | Asset-centric; identifies threats to critical operational assets |
| **FMEA** (Failure Modes and Effect Analysis) | Engineering | Identifies failure modes; analyses effects and likelihood |

---

### Concept 5 — NIST SP 800-30: The Four-Step Process

```
FRAME RISK
(Establish context and risk parameters)
         |
         v
ASSESS RISK
(Identify threats, vulnerabilities, likelihood, impact)
         |
         v
RESPOND TO RISK
(Decide: mitigate, transfer, accept, or avoid)
         |
         v
MONITOR RISK
(Continuous: effectiveness, change, compliance)
         |
         (loop back to ASSESS as environment changes)
```

---

#### Step 1: Frame Risk

**Purpose:** Establish the context within which all risk decisions are made. Without framing, risk assessments lack boundaries and prioritisation criteria.

**Four Elements to Define:**

| Element | Questions to Answer |
|---------|-------------------|
| **Risk Assumptions** | What threats are assumed? What is the baseline likelihood? What would be the impact? |
| **Risk Constraints** | What budget, personnel, or regulatory limits constrain risk response options? |
| **Risk Tolerance** | What level of risk is acceptable? What types of risk are intolerable? |
| **Priorities and Trade-offs** | Which business functions are highest priority? What trade-offs exist between risk types? |

**Example — Accounting Firm:**

| Element | Answer |
|---------|--------|
| Risk Assumption | Handles client financial data → high-value target; without controls, attack success likelihood is high and impact is catastrophic |
| Risk Constraint | Budget limits physical security upgrades and hiring additional personnel |
| Risk Tolerance | Data theft is intolerable — it would end the business and violate client trust |
| Priority | Confidentiality and integrity of client financial data is the primary objective |

---

#### Step 2: Assess Risk

**Purpose:** Identify threats, evaluate vulnerabilities, assess impact and likelihood to produce a prioritised risk inventory.

**Two approaches to risk analysis:**

##### Qualitative Risk Analysis

Uses descriptive ratings (High/Medium/Low or Red/Yellow/Green):

| Impact ↓ / Likelihood → | Low | Medium | High |
|------------------------|-----|--------|------|
| **High** | Medium | High | Critical |
| **Medium** | Low | Medium | High |
| **Low** | Low | Low | Medium |

**Advantages:** Fast, communicable to non-technical executives, no financial data required.
**Disadvantages:** Imprecise — cannot directly justify specific spending amounts.

##### Quantitative Risk Analysis

Uses monetary values and numeric frequencies for precise financial decision-making.

**Key Terms and Formulas:**

**Single Loss Expectancy (SLE):**
```
SLE = Asset Value × Exposure Factor (EF)

Where:
  Asset Value  = total monetary value of the asset (hardware + data)
  Exposure Factor = % of asset value lost if the threat is realised (0–100%)
```

**Annualised Rate of Occurrence (ARO):**
- Expected number of times the threat is realised per year
- ARO = 1 → once per year
- ARO = 0.5 → once every two years
- ARO = 2 → twice per year

**Annualised Loss Expectancy (ALE):**
```
ALE = SLE × ARO
```
ALE = the expected annual financial loss from a specific threat to a specific asset.

**Worked Example:**

```
Scenario: Work laptop — ransomware threat

Step 1: Calculate SLE
  Asset Value  = $10,000 ($1,000 hardware + $9,000 data value)
  Exposure Factor = 90% (ransomware encrypts all data; hardware survives)
  SLE = $10,000 × 0.90 = $9,000

Step 2: Calculate ARO
  Based on incident history: ransomware infection expected once every 2 years
  ARO = 0.5

Step 3: Calculate ALE
  ALE = $9,000 × 0.5 = $4,500 per laptop per year

Step 4: Justify control
  If an EDR solution costs $200/year/laptop and prevents ransomware:
  Cost savings = $4,500 - $200 = $4,300/year/laptop → justified
```

**Why ALE Matters:** ALE directly answers the executive question: "How much should we spend on this control?" If the annual cost of a control is less than the ALE it prevents, the investment is financially justified.

**Quantitative vs Qualitative Comparison:**

| Aspect | Qualitative | Quantitative |
|--------|------------|-------------|
| Speed | Fast | Slower |
| Precision | Low | High |
| Data required | Low | High (asset values, ARO history) |
| Communication | Easy for executives | Requires financial framing |
| Use case | Initial triage, strategic decisions | Budget justification, insurance, ROI |


---

#### Step 3: Respond to Risk

**Purpose:** Decide how to treat each identified risk.

**Four Response Options:**

| Response | Description | When to Use |
|----------|------------|-------------|
| **Mitigate / Remediate** | Implement controls to reduce likelihood or impact | Risk exceeds tolerance AND a cost-effective control exists |
| **Transfer** | Shift financial impact to a third party (cyber insurance, contractual liability) | Mitigation too costly or residual risk needs financial coverage |
| **Accept** | Acknowledge risk; choose not to act; document the decision | Risk is within tolerance AND control cost exceeds the ALE |
| **Avoid** | Eliminate the activity or asset that creates the risk | Risk cannot be adequately mitigated AND the activity is optional |

**Residual Risk:** Risk remaining after controls are applied. Must still be within the organisation's risk tolerance. If not, additional controls or risk transfer are required.

**Control Justification Formula:**
```
If (ALE before control) - (ALE after control) > Annual cost of control
  → Implement the control

If (ALE before control) - (ALE after control) < Annual cost of control
  → Accept the risk (control costs more than it saves)
```

---

#### Step 4: Monitor Risk

**Purpose:** Continuously track control effectiveness, respond to environment changes, and ensure continued compliance. Risk monitoring is ongoing — not a one-time review.

**Three Focus Areas:**

| Area | What to Monitor | Why It Can Change |
|------|----------------|-------------------|
| **Effectiveness** | Are implemented controls still working as intended? | User behaviour adapts; new attack techniques emerge; technology drifts |
| **Change** | New systems, business activities, processes, acquisitions, personnel | Any change can introduce new risks or invalidate existing controls |
| **Compliance** | New laws, updated regulations, internal policy updates, audit findings | Regulatory requirements evolve; unaddressed audit findings attract fines |

**Effectiveness Example:**
Password complexity policy was implemented to prevent weak passwords. Monitoring later reveals employees writing complex passwords on sticky notes — the control became ineffective. Without effectiveness monitoring, this drift goes undetected indefinitely.

**Change Example:**
Company acquires a subsidiary with a different IT environment. Acquisition introduces new attack surfaces outside the existing control scope. Change monitoring triggers a risk re-assessment for the acquired systems.

---

### Concept 6 — Supply Chain Risk

#### Definition

Risk introduced by third-party suppliers of hardware, software, and services the organisation depends upon. The organisation's own security controls do not extend into the supplier's environment.

#### Three Risk Categories

| Supply Type | Risk Description | Real-World Example |
|------------|-----------------|-------------------|
| **Hardware** | Hardware Trojans — malicious circuits embedded in components enabling backdoor access or data exfiltration | Nation-state targeting of high-value hardware supply chains |
| **Software** | Trojans in source code or compiled binaries; malicious code injected during the build or update process | SolarWinds 2020 — attacker inserted code into Orion build pipeline |
| **Services** | Provider downtime → availability loss; provider breach → client data exposed; compromised provider → attack vector into client networks | Kaseya VSA 2021 — MSP software compromised, impacting thousands of end clients |

#### SolarWinds 2020 — The Defining Example

APT29 (Cozy Bear) inserted malicious code into the SolarWinds Orion software build process. The attack was delivered via a digitally signed, trusted software update — approximately 18,000 organisations installed it. This bypassed every perimeter control because the attack entered through a trusted update channel. It is the canonical real-world example of software supply chain risk realised at scale.

**Key lesson:** An organisation's own security posture is only as strong as its weakest trusted supplier.

#### Accounting Firm Supply Chain Example

| Supplier | Type | Potential Risk |
|---------|------|---------------|
| Local computer shop | Hardware | Hardware Trojan inserted into purchased equipment |
| Accounting software vendor | Software | Malicious code in a software update |
| Email service provider | Service | Provider breach exposes confidential client communications |

#### Mitigation Approaches

- Third-party risk assessments and security questionnaires before onboarding suppliers
- Contractual security requirements — right-to-audit clauses, breach notification SLAs
- Software Composition Analysis (SCA) for open-source dependency vulnerabilities
- Verify code signing — reject unsigned or improperly signed software packages
- Review supplier security certifications (ISO 27001, SOC 2 Type II)
- Include critical suppliers in incident response planning and tabletop exercises
- NIST 800-53 SR (Supply Chain Risk Management) control family addresses this systematically


---

### Concept 7 — Vulnerability Management

#### Vulnerability Management vs Vulnerability Scanning

| Concept | Definition |
|---------|-----------|
| **Vulnerability Scanning** | Using a software tool to automatically discover vulnerabilities in systems, networks, or applications |
| **Vulnerability Management** | The full lifecycle programme — scanning, risk acceptance, prioritisation, remediation, reporting, tracking, and continuous improvement |

Vulnerability scanning is one activity within vulnerability management — like a medical test is one activity within patient healthcare. The scan alone is not treatment.

#### SCAP — Security Content Automation Protocol

NIST's SCAP standard provides a common, automated language for vulnerability management:

| Component | Full Name | Purpose |
|-----------|-----------|---------|
| **CVE** | Common Vulnerabilities and Exposures | Unique identifier per known vulnerability. Maintained by MITRE. |
| **CCE** | Common Configuration Enumeration | Unique IDs for configuration issues |
| **CPE** | Common Platform Enumeration | Classifies and identifies devices, OSes, and applications |
| **CVSS** | Common Vulnerability Scoring System | Standardised 0–10 severity score |
| **NVD** | National Vulnerability Database | NIST's comprehensive CVE database with CVSS scores and analysis |

**CVE Format:** `CVE-YEAR-SEQUENCE` — e.g., `CVE-2021-44228` (Log4Shell)

**CVSS 3.x Severity Scale:**

| Score | Severity |
|-------|---------|
| 0.0 | None |
| 0.1–3.9 | Low |
| 4.0–6.9 | Medium |
| 7.0–8.9 | High |
| 9.0–10.0 | Critical |

**Important limitation:** CVSS scores are environment-independent. A CVSS 9.8 on an isolated internal system may pose less actual risk than a CVSS 5.0 on an internet-exposed, business-critical server. Always contextualise with environmental factors.

---

#### Vulnerability Management Lifecycle — Six Phases

Mapped from the NIST Cybersecurity Framework:

```
1. DISCOVER → 2. PRIORITISE → 3. ASSESS
     ↑                              ↓
6. VERIFY    5. REMEDIATE ← 4. REPORT
```

**Phase 1 — Discover**
Compile a complete inventory of all assets: systems, services, operating systems, configurations.
- Configure scanner with target scope (IP ranges, subnets)
- Run authenticated scans (sees installed software, patch levels) and unauthenticated scans
- In Greenbone/OpenVAS: `Configuration → Targets → Add Target` then `Scans → Tasks → Add Task → Run`

**Phase 2 — Prioritise**
Group assets by business criticality and assign risk-based remediation priority.
- Internet-facing systems handling PII or financial data → highest priority
- Internal critical servers → high priority
- Isolated, low-value systems → lower priority
- Consider: CVSS score + exploitability + compensating controls + asset criticality

**Phase 3 — Assess**
Establish a risk baseline; evaluate each finding for actual exploitability in context.
- Review CVSS scores and vulnerability descriptions
- Confirm the vulnerability applies (correct OS, version, configuration)
- Filter false positives before proceeding — scanners produce false positives regularly
- Authenticated scans produce significantly fewer false positives than unauthenticated

**Phase 4 — Report**
Document findings for stakeholders; ensure accountability and evidence of compliance.
- Executive summary: risk exposure, critical finding count, trend vs prior assessment
- Detailed findings: CVE ID, affected system, CVSS score, description, evidence, recommended fix
- Flag confirmed false positives in the tool for future reference
- Download reports from scanner (Greenbone: download button at top of report view)

**Phase 5 — Remediate**
Fix identified vulnerabilities, starting with the highest severity.

| Option | Description | When Appropriate |
|--------|------------|-----------------|
| **Full Remediation** | Patch or fix completely | Always preferred when feasible |
| **Mitigation** | Reduce exploitability without full fix (WAF rule, network isolation) | When patching requires downtime or no vendor patch exists |
| **Risk Acceptance** | Document and accept the risk | When risk is low and remediation cost is disproportionate |

In Greenbone/OpenVAS: create remediation tickets from vulnerability detail view → assign to responsible team → track via `Resilience → Remediation Tickets`.

**Phase 6 — Verify and Monitor**
Confirm remediation was successful; maintain ongoing visibility.
- Re-scan after remediation to confirm vulnerability is resolved
- Close the remediation ticket on verification
- Continue scheduled scans for continuous detection
- Track posture trends over time

---

#### NIST CSF Five Functions Mapped to Vulnerability Management

| Function | Vulnerability Management Application |
|----------|--------------------------------------|
| **Identify** | Asset discovery; criticality classification; data flow mapping |
| **Protect** | Deploy vuln management tooling; patch management; security baselines |
| **Detect** | Continuous scanning; prioritisation; risk scoring; anomaly monitoring |
| **Respond** | Assign ownership; report to stakeholders; risk acceptance decisions; rapid patch for active exploits |
| **Recover** | Verify remediation; extend coverage to cloud/IoT assets; record lessons learned |

---

#### Vulnerability Scanning Tools

| Tool | Type | Notes |
|------|------|-------|
| **Nessus** (Tenable) | Commercial | Industry-leading; widely used in enterprise |
| **Nexpose** / InsightVM (Rapid7) | Commercial | Strong asset-risk correlation |
| **Acunetix** (Invicti) | Commercial | Specialises in web application scanning |
| **Greenbone / OpenVAS** | Open-source | Free; community edition of GVM; good for labs |
| **OWASP ZAP** | Open-source | Web application active and passive scanning |
| **Qualys VMDR** | SaaS | Cloud-based continuous monitoring; enterprise standard |

---

## Security Engineer Perspective

- **Quantify risk in business terms** — use ALE to present security investments as financial decisions, not just technical ones
- **Risk framing before any assessment** — understand the organisation's risk tolerance and priorities before scoring vulnerabilities, or the priorities will be wrong
- **ALE justification is your budget argument** — if a control costs less per year than the ALE it prevents, the investment is financially justified
- **Authenticated scans are mandatory** — unauthenticated scans miss patch levels, installed software, and configuration issues; they give a false sense of coverage
- **False positives waste remediation resources** — always verify critical/high findings before creating tickets
- **Supply chain is an undermonitored attack surface** — vendor security assessments, SCA, and code signing are the primary controls
- **CISA KEV (Known Exploited Vulnerabilities)** catalogue — vulnerabilities with confirmed active exploitation in the wild; these supersede CVSS in priority regardless of score

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **ALE** | Annualised Loss Expectancy — expected annual financial loss: SLE × ARO |
| **ARO** | Annualised Rate of Occurrence — expected frequency of a threat per year |
| **Asset** | Valuable resource the organisation depends on |
| **CCE** | Common Configuration Enumeration — unique IDs for configuration issues |
| **CISA KEV** | CISA Known Exploited Vulnerabilities catalogue — confirmed actively exploited CVEs |
| **CPE** | Common Platform Enumeration — standard for identifying devices, OSes, applications |
| **CVE** | Common Vulnerabilities and Exposures — unique identifier per known vulnerability |
| **CVSS** | Common Vulnerability Scoring System — standardised 0–10 severity score |
| **EF** | Exposure Factor — % of asset value lost if a specific threat is realised |
| **False Positive** | Vulnerability flagged by scanner that does not actually exist or is not exploitable |
| **FMEA** | Failure Modes and Effect Analysis — engineering risk framework |
| **FRAP** | Facilitated Risk Analysis Process — collaborative stakeholder risk workshop |
| **NVD** | National Vulnerability Database — NIST's comprehensive CVE + CVSS database |
| **OCTAVE** | Operationally Critical Threat, Asset, and Vulnerability Evaluation |
| **Qualitative Analysis** | Risk assessment using descriptive ratings (High/Medium/Low) |
| **Quantitative Analysis** | Risk assessment using monetary values and numeric frequencies |
| **Residual Risk** | Risk remaining after controls are applied |
| **Risk** | Probability × impact of a threat exploiting a vulnerability |
| **Risk Acceptance** | Choosing not to act on a risk within tolerance; must be documented |
| **Risk Avoidance** | Eliminating an activity or asset that creates a risk |
| **Risk Register** | Living document tracking identified risks, owners, ratings, and treatment status |
| **Risk Tolerance** | The level of risk an organisation is willing to accept |
| **Risk Transfer** | Shifting financial impact to a third party (cyber insurance, contracts) |
| **SCAP** | Security Content Automation Protocol — NIST standard enabling automated vuln management |
| **SLE** | Single Loss Expectancy — financial loss per incident: Asset Value × EF |
| **Supply Chain Risk** | Risk introduced by third-party hardware, software, or service suppliers |
| **Threat** | Potential event or actor that could exploit a vulnerability to cause harm |
| **Vulnerability** | A weakness exploitable by a threat |
| **Vulnerability Management** | Full lifecycle: discover, prioritise, assess, report, remediate, verify |
| **Vulnerability Scanning** | Use of automated tools to detect vulnerabilities in systems and applications |

---

## Exam and Interview Revision

### Must Remember

- Risk = Threat + Vulnerability + Asset — all three must coexist
- NIST 800-30: **Frame → Assess → Respond → Monitor**
- Risk response options: **Mitigate, Transfer, Accept, Avoid**
- **SLE = Asset Value × Exposure Factor**
- **ALE = SLE × ARO**
- ALE justifies control spending: if ALE > annual cost of control → implement it
- Qualitative = descriptive (High/Medium/Low); Quantitative = monetary (SLE, ALE)
- Supply chain risks: Hardware Trojans, software Trojans, service provider breaches
- SolarWinds = canonical supply chain attack — trusted update channel compromise
- Risk monitoring covers: **effectiveness, change, compliance** — all three continuously
- Vulnerability scanning ≠ vulnerability management (scanning is one phase of six)
- CVSS 9.0–10.0 = Critical; 7.0–8.9 = High; 4.0–6.9 = Medium; 0.1–3.9 = Low
- CVE format: `CVE-YEAR-SEQUENCE`
- CISA KEV overrides CVSS for prioritisation — active exploitation confirmed
- Six vulnerability management phases: Discover → Prioritise → Assess → Report → Remediate → Verify

### Common Interview Questions

| Question | Key Points |
|----------|-----------|
| What is the formula for ALE? | ALE = SLE × ARO. SLE = Asset Value × Exposure Factor. |
| How do you justify a security investment using risk management? | Calculate ALE for the risk. If ALE > annual cost of the control, the investment is financially justified. |
| What are the four risk response options? | Mitigate (reduce), Transfer (insurance/contract), Accept (document and do nothing), Avoid (eliminate the activity) |
| What is residual risk? | Risk remaining after controls are applied. Must still be within the organisation's risk tolerance. |
| What are the three areas of risk monitoring? | Effectiveness (controls still working?), Change (new systems/processes?), Compliance (new regulations?) |
| What is supply chain risk? | Risk from third-party hardware, software, or service providers whose security posture you cannot directly control |
| What is the difference between vulnerability scanning and vulnerability management? | Scanning is one automated activity; management is the full lifecycle including prioritisation, remediation, reporting, and tracking |
| What is CVSS and what are its limitations? | Standardised 0–10 vulnerability severity score. Limitation: environment-independent — a high CVSS score on an isolated system may be lower actual risk than a medium score on an internet-exposed critical system |
| What is the CISA KEV catalogue? | CISA's public list of vulnerabilities with confirmed active exploitation in the wild — these should be treated as highest remediation priority regardless of CVSS score |
| Why must you verify before closing a remediation ticket? | Scanners can miss re-introduced vulnerabilities; re-scanning after remediation confirms the fix actually worked and provides audit evidence |
