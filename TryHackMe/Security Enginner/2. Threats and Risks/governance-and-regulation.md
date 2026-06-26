# Governance & Regulation
## Security Engineer Knowledge Base

> **Source Room:** Governance & Regulation — TryHackMe Security Engineer Path
> **Purpose:** Long-term reference for revision, interviews, certifications, blue-team and security engineering work

---

## Overview

### What This Room Teaches

- The meaning and relationship between governance, regulation, and compliance
- How information security governance is structured inside an organisation
- The landscape of key regulations and standards (GDPR, HIPAA, PCI DSS, GLBA, ISO 27001, NIST 800-53, SOC 2)
- How governance documents (policies, standards, guidelines, procedures, baselines) are developed and used
- The GRC (Governance, Risk, Compliance) framework and how it integrates all three disciplines

### Why It Matters

Every security control, policy, and tool exists within a governance and compliance context. A security engineer who only understands technical controls but not governance operates blind — building tools without understanding why they exist, for whom, or to what standard. This topic is the foundation for CISSP, CISM, CISA, ISO 27001 Lead Implementer, and CompTIA Security+ certifications.

### Where These Concepts Are Used in Real Environments

- **Enterprise security programmes** — governance frameworks drive security strategy and policy
- **Compliance teams** — map controls to regulatory requirements (GDPR, PCI DSS, HIPAA)
- **Security engineering** — controls must be designed to satisfy governance requirements and pass audits
- **Cloud environments** — shared responsibility models depend on clearly defined governance boundaries
- **Regulated industries** — finance, healthcare, government have mandatory compliance obligations

### Key Takeaways

- Governance sets direction; regulation enforces minimums; compliance demonstrates adherence
- Policy > Standard > Guideline > Procedure > Baseline — each has a distinct role and binding level
- GDPR, PCI DSS, HIPAA, and ISO 27001 are the most commonly encountered frameworks in enterprise environments
- GRC ties governance, risk, and compliance into a single integrated programme
- Privacy by design (GDPR Article 25) means security engineers must build data protection in from the start


---

## Core Concepts

---

### Concept 1 — Governance, Regulation, and Compliance

#### Definitions

| Term | Definition |
|------|-----------|
| **Governance** | The structure, policies, and processes by which an organisation directs and controls itself to achieve objectives and ensure legal and regulatory compliance. |
| **Regulation** | A rule or law enforced by a governing body (government agency, industry body) to mandate minimum standards and protect against harm. |
| **Compliance** | The state of adhering to applicable laws, regulations, internal policies, and standards. Compliance is measured, audited, and enforced. |

#### Why They Exist

Without governance, security is reactive and fragmented. Without regulation, industries apply inconsistent protections to sensitive data. Without compliance monitoring, neither governance nor regulation is verifiable.

#### How They Relate

```
Regulation (external — governments, industry bodies)
   Sets minimum legal requirements
         |
         v
Governance (internal — board, CISO, management)
   Sets strategy, policies, and oversight beyond minimum requirements
         |
         v
Compliance (measurement — auditors, assessors)
   Verifies that governance satisfies both internal and regulatory requirements
```

#### Interview Notes

- "Who owns information security governance?" — Top management / board level. The CISO executes it but ultimate accountability sits at the board.
- "What is the difference between governance and management?" — Governance sets direction and oversight; management executes within that direction.
- "Is compliance the same as security?" — No. Compliance demonstrates minimum adherence to standards; security is the actual protection of assets. An organisation can be compliant but insecure (and vice versa).

---

### Concept 2 — Information Security Governance

#### Definition

Information security governance is the established structure, policies, methods, and guidelines designed to guarantee the confidentiality, integrity, and availability of an organisation's information assets. It is a top-management responsibility.

#### Components

| Component | Description |
|-----------|------------|
| **Strategy** | Comprehensive information security strategy aligned with business objectives |
| **Policies and Procedures** | Formal documents governing how information assets are used and protected |
| **Risk Management** | Identify threats, assess vulnerabilities, implement mitigation |
| **Performance Measurement** | KPIs and metrics measuring governance programme effectiveness |
| **Compliance** | Ensuring adherence to regulations and industry best practices |

#### Key Performance Indicators (KPIs) Examples

- Mean time to patch critical vulnerabilities
- Percentage of systems with current vulnerability scans
- Policy exception rate
- Phishing simulation click rate
- Percentage of staff completing annual security awareness training
- Number of open audit findings

#### Security Importance

Governance provides the "why" behind every security control. It establishes ownership, accountability, and measurable outcomes — defining who is responsible when controls fail and what success looks like.

---

### Concept 3 — Information Security Framework Documents

#### Definition

The framework is the collection of documents that define how security is implemented, managed, and enforced within an organisation.

#### Document Hierarchy

| Type | Binding | Purpose | Example |
|------|---------|---------|---------|
| **Policy** | Mandatory | High-level formal statement of goals, principles, and requirements | "All user accounts must use MFA" |
| **Standard** | Mandatory | Specific, measurable requirements for processes or products | "MFA must use TOTP or FIDO2; SMS OTP is not permitted" |
| **Guideline** | Recommended | Non-mandatory best practice | "Consider using a hardware key for privileged accounts" |
| **Procedure** | Mandatory | Step-by-step instructions for a specific task | "MFA enrolment: Step 1 — user logs in with password → Step 2 — prompted for MFA setup → Step 3 — scan QR code..." |
| **Baseline** | Mandatory | Minimum security configuration requirements for a system type | "All Windows servers must have auto-updates enabled, local firewall active, RDP disabled unless approved" |

#### Policy vs Standard vs Procedure — Flow

```
Policy (WHAT — goal/requirement)
   "We will protect all customer data from unauthorised access."
         |
         v
Standard (HOW WELL — specific criteria)
   "Customer data at rest must be encrypted with AES-256-GCM."
         |
         v
Procedure (HOW TO — step-by-step)
   "1. Configure disk encryption using BitLocker...
    2. Store encryption keys in Azure Key Vault...
    3. Verify with: manage-bde -status..."
```

#### Developing Governance Documents — Process

1. **Identify scope and purpose** — What does this document govern and why?
2. **Research and review** — Laws, regulations, existing internal documents
3. **Draft** — Specific, actionable, aligned with organisational goals
4. **Review and approval** — Stakeholders, legal, compliance, senior management
5. **Implement and communicate** — Train affected employees; define roles and responsibilities
6. **Review and update** — Periodic review; update when threat landscape or regulation changes

#### Common Mistakes

- Confusing guidelines (non-mandatory) with policies (mandatory)
- Writing policies so vague they are unenforceable ("systems must be secure")
- Creating procedures without referencing the policy they implement
- Never reviewing documents after initial publication


---

### Concept 4 — Key Regulations and Standards

---

#### GDPR — General Data Protection Regulation

**Definition:** EU regulation (effective May 2018) governing collection, storage, processing, and protection of personal data of EU citizens and residents.

**Personal Data:** Any data that can directly or indirectly identify a natural person — name, email, IP address, location, biometrics, device identifiers.

**Key Principles:**
- **Lawfulness, fairness, transparency** — must have a legal basis for processing
- **Purpose limitation** — collect data only for stated, specific purposes
- **Data minimisation** — collect only what is necessary
- **Accuracy** — keep data accurate and up to date
- **Storage limitation** — retain only as long as necessary
- **Integrity and confidentiality** — implement appropriate security measures
- **Accountability** — demonstrate compliance with all principles

**Breach Notification:** Notify supervisory authority within **72 hours** of becoming aware of a breach. Notify affected individuals if there is high risk to their rights.

**Penalty Tiers:**

| Tier | Violation Type | Maximum Fine |
|------|---------------|-------------|
| **Tier 1 (severe)** | Unlawful processing, sharing without consent, core principle violations | 4% of global annual revenue **or** €20 million — whichever is **higher** |
| **Tier 2 (less severe)** | Breach notification failures, data processing records, DPO obligation failures | 2% of global annual revenue **or** €10 million — whichever is **higher** |

**Article 25 — Privacy by Design and by Default:**
- Technical and organisational measures for data protection must be integrated at design time
- By default, only the minimum necessary personal data must be processed
- **Security Engineer implication:** Privacy controls are architectural decisions, not add-ons

**Where It Applies:** Any organisation worldwide that processes personal data of EU residents.

---

#### HIPAA — Health Insurance Portability and Accountability Act

**Definition:** US federal law protecting the privacy and security of Protected Health Information (PHI).

**Key Rules:**
- **Privacy Rule** — standards for use and disclosure of PHI; patient rights
- **Security Rule** — administrative, physical, and technical safeguards for electronic PHI (ePHI)
- **Breach Notification Rule** — notification to individuals, HHS, and media (if large breach)

**Three Safeguard Categories:**

| Safeguard | Examples |
|-----------|---------|
| Administrative | Risk analysis, workforce training, access management policies |
| Physical | Facility access controls, workstation security, media disposal |
| Technical | Access controls, audit controls, encryption, automatic logoff |

**Who Must Comply:** Covered entities (healthcare providers, health plans, clearinghouses) and their business associates.

---

#### PCI DSS — Payment Card Industry Data Security Standard

**Definition:** Technical and operational standard established by major card brands (Visa, Mastercard, American Express) to protect cardholder data.

**12 Requirements across 6 Objectives:**

| Objective | Requirements |
|-----------|-------------|
| Secure network and systems | 1. Firewall configuration; 2. No vendor defaults |
| Protect cardholder data | 3. Protect stored data; 4. Encrypt data in transit |
| Vulnerability management | 5. Anti-malware; 6. Secure development |
| Strong access control | 7. Restrict access by need-to-know; 8. Identify and authenticate; 9. Restrict physical access |
| Monitor and test | 10. Track and monitor all access; 11. Test security systems |
| Information security policy | 12. Maintain security policy for all personnel |

**Compliance Levels:** Based on annual transaction volume. Level 1 (largest) requires annual on-site QSA audit. Lower levels may use Self-Assessment Questionnaires (SAQ).

---

#### GLBA — Gramm-Leach-Bliley Act

**Definition:** US financial services law requiring financial companies to protect customers' Nonpublic Personal Information (NPI).

**Requirements:**
- Implement an information security programme
- Provide privacy notices to customers
- Disclose information-sharing practices

---

#### NIST 800-53 — Security and Privacy Controls

**Definition:** NIST publication providing a catalogue of security and privacy controls for US federal information systems, widely adopted by enterprises globally.

**20 Control Families (Revision 5):**

| ID | Family | ID | Family |
|----|--------|----|--------|
| AC | Access Control | PE | Physical & Environmental Protection |
| AT | Awareness & Training | PL | Planning |
| AU | Audit & Accountability | PM | Programme Management |
| CA | Assessment, Authorisation & Monitoring | PS | Personnel Security |
| CM | Configuration Management | PT | PII Processing & Transparency |
| CP | Contingency Planning | RA | Risk Assessment |
| IA | Identification & Authentication | SA | System & Services Acquisition |
| IR | Incident Response | SC | System & Communications Protection |
| MA | Maintenance | SI | System & Information Integrity |
| MP | Media Protection | SR | Supply Chain Risk Management |

**Programme Management (PM)** is a critical family — mandates establishing, implementing, and monitoring organisation-wide security and privacy programmes.

---

#### NIST 800-63B — Digital Identity Guidelines

**Definition:** NIST guidelines for authenticating and verifying identities accessing digital services.

**Key Guidance:**
- Minimum password length: **8 characters** (15+ recommended for privileged accounts)
- **No mandatory periodic rotation** — rotate only if compromise is suspected
- Check passwords against known breached password lists at time of creation/change
- Multi-factor authentication required for AAL2 and AAL3
- SMS OTP is acceptable at AAL2 but hardware tokens or TOTP are preferred

**Interview Note:** NIST 800-63B explicitly discourages forced periodic rotation — this directly contradicts many legacy policies and is a common interview topic.


---

### Concept 5 — ISO/IEC 27001 — ISMS

#### Definition

Internationally recognised standard specifying requirements for establishing, implementing, maintaining, and continually improving an Information Security Management System (ISMS). Developed by ISO and IEC. Certifiable by accredited third-party auditors.

#### Why It Exists

Organisations need a systematic, documented, and independently auditable approach to managing information security. ISO 27001 provides this and gives customers, partners, and regulators assurance that security is managed to an internationally agreed standard.

#### Core Components

| Component | Description |
|-----------|------------|
| **Scope** | Boundaries of the ISMS — which assets, processes, locations, departments |
| **Information Security Policy** | High-level management commitment document |
| **Risk Assessment** | Systematic identification and evaluation of risks to CIA triad |
| **Risk Treatment** | Selecting and implementing Annex A controls to reduce identified risks |
| **Statement of Applicability (SoA)** | Declares which Annex A controls are implemented; justifies exclusions |
| **Internal Audit** | Periodic self-assessment of ISMS effectiveness |
| **Management Review** | Board/executive review of ISMS performance at regular intervals |

#### PDCA Cycle

```
PLAN  →  DO  →  CHECK  →  ACT
 |                           |
 └───────────────────────────┘
         (continuous loop)

Plan:  Establish scope, policy, risk assessment, risk treatment plan
Do:    Implement selected controls, training, procedures
Check: Monitor, measure performance, conduct internal audits
Act:   Take corrective actions, drive continual improvement
```

#### ISO 27001 vs ISO 27002

| Standard | Purpose |
|----------|---------|
| **ISO 27001** | Defines ISMS requirements — certifiable |
| **ISO 27002** | Provides implementation guidance for Annex A controls — reference only, not certifiable |

#### Statement of Applicability (SoA)

The SoA is a mandatory document that:
- Lists all Annex A controls
- States whether each is implemented or excluded
- Provides justification for each inclusion and exclusion
- Is a primary audit evidence document

---

### Concept 6 — SOC 2

#### Definition

SOC 2 (Service Organisation Control 2) is an auditing framework developed by AICPA for service organisations. It evaluates controls related to the Trust Services Criteria (TSC).

#### Trust Services Criteria

| Criterion | Description | Mandatory? |
|-----------|------------|-----------|
| **Security** | Systems protected against unauthorised access | Yes — always required |
| **Availability** | Systems available as committed | Optional |
| **Processing Integrity** | Processing is complete, accurate, timely, authorised | Optional |
| **Confidentiality** | Confidential information is protected | Optional |
| **Privacy** | PII handled correctly throughout its lifecycle | Optional |

#### SOC 2 Type I vs Type II

| | Type I | Type II |
|-|--------|---------|
| **Tests** | Design of controls at a point in time | Design AND operating effectiveness over a period |
| **Period** | Single date | Typically 6–12 months |
| **Rigour** | Lower | Higher — more trusted by customers |
| **Cost** | Lower | Higher |

**Type II is strongly preferred** in enterprise procurement — it proves controls work consistently, not just on audit day.

#### Who Needs It

Cloud providers, SaaS companies, managed security service providers, data centres — any organisation that handles customer data and needs to demonstrate security assurance to enterprise clients. Often a contractual requirement.

---

### Concept 7 — GRC Framework

#### Definition

GRC (Governance, Risk, and Compliance) is an integrated framework unifying governance, enterprise risk management, and compliance into a single coherent programme rather than managing them as separate silos.

#### Three Components

```
┌──────────────────────────────────────────────────────────────┐
│  GOVERNANCE              RISK MANAGEMENT         COMPLIANCE  │
│  ──────────              ─────────────────        ─────────  │
│  Set direction           Identify risks           Meet legal  │
│  Define policies         Assess impact            obligations │
│  Establish KPIs          Prioritise threats       Audit       │
│  Monitor outcomes        Implement controls       Report      │
│                          Monitor residual risk    Remediate   │
└──────────────────────────────────────────────────────────────┘
              All three inform and constrain each other
```

#### Developing a GRC Programme — Steps

1. **Define scope and objectives** — What systems, processes, and data? What is the target risk reduction?
2. **Conduct risk assessment** — Identify threats, vulnerabilities, and impacts
3. **Develop policies and procedures** — Codify required security behaviours
4. **Establish governance processes** — Security steering committee, defined roles, decision authority
5. **Implement controls** — Technical (SIEM, IPS, firewalls) and non-technical (training, background checks)
6. **Monitor and measure** — Track KPIs, audit compliance, assess control effectiveness
7. **Continuously improve** — Post-incident reviews, updated risk profiles, revised controls

#### GRC in Financial Sector — Example

| Component | Example Activities |
|-----------|------------------|
| **Governance** | Anti-money laundering policy, financial audit policies, financial reporting standards, crisis management |
| **Risk Management** | Financial fraud risk, phishing-based credential theft, fake ATM cards, fraudulent transaction vectors |
| **Compliance** | PCI DSS implementation, GLBA adherence, SSL/TLS deployment to prevent MITM, patch management, phishing awareness campaigns |


---

## Security Engineer Perspective

- **Design to compliance requirements** — know which regulations govern the data your systems handle; build controls that satisfy them from day one
- **Privacy by Design (GDPR Art. 25)** — data minimisation, encryption, access control, and audit logging are architectural decisions, not afterthoughts
- **Audit evidence** — governance documents (policies, SoA, risk assessments) are evidence in audits; keep them current and accessible
- **Hardening to baselines** — CIS Benchmarks map directly to NIST 800-53 control families; implementing benchmarks satisfies multiple regulatory requirements simultaneously
- **Detection opportunity** — SIEM rules can detect policy violations (unauthorised data access, failed MFA, out-of-hours access) and feed directly into compliance reporting

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **Accountability** | GDPR principle requiring organisations to demonstrate compliance with all data protection principles |
| **Baseline** | Minimum security configuration standards a system must meet |
| **Compliance** | State of adhering to applicable laws, regulations, and standards |
| **GDPR** | EU data privacy regulation; 72-hour breach notification; two penalty tiers |
| **Governance** | Organisation's direction-setting and oversight structure for achieving objectives |
| **GLBA** | US financial services law protecting nonpublic personal information |
| **GRC** | Governance, Risk, and Compliance — integrated management framework |
| **Guideline** | Non-mandatory recommended best practice document |
| **HIPAA** | US healthcare data privacy law with administrative, physical, and technical safeguards |
| **ISMS** | Information Security Management System — systematic approach per ISO 27001 |
| **ISO 27001** | International certifiable standard for ISMS requirements |
| **ISO 27002** | Implementation guidance for ISO 27001 Annex A controls — not certifiable |
| **NPI** | Nonpublic Personal Information — protected under GLBA for financial customers |
| **NIST 800-53** | NIST catalogue of 20 security control families for information systems |
| **NIST 800-63B** | NIST digital identity guidelines; no mandatory rotation; min 8-char passwords |
| **PCI DSS** | Payment card data security standard; 12 requirements across 6 objectives |
| **PHI** | Protected Health Information — protected under HIPAA |
| **Policy** | Mandatory formal statement of organisational goals and requirements |
| **Privacy by Design** | GDPR Article 25 — integrating data protection controls at design time |
| **Procedure** | Mandatory step-by-step instructions for a specific task |
| **QSA** | Qualified Security Assessor — PCI DSS audit professional |
| **Regulation** | Rule or law enforced by a governing body to mandate minimum standards |
| **SoA** | Statement of Applicability — ISO 27001 document declaring applicable Annex A controls |
| **SOC 2** | AICPA service organisation auditing framework based on Trust Services Criteria |
| **Standard** | Mandatory specific requirements for a process or product |
| **TSC** | Trust Services Criteria — the five criteria (Security, Availability, PI, Confidentiality, Privacy) in SOC 2 |

---

## Exam and Interview Revision

### Must Remember

- Governance = direction + oversight; Management = execution within that direction
- Policy (mandatory) > Standard (mandatory, specific) > Guideline (recommended) > Procedure (steps) > Baseline (min config)
- GDPR Tier 1: **4% global revenue or €20M** (higher). Tier 2: **2% or €10M** (higher)
- GDPR breach notification to supervisory authority: **72 hours**
- GDPR Article 25: Privacy by Design and by Default — security engineers must build this in
- ISO 27001 = certifiable ISMS requirements. ISO 27002 = implementation guidance only
- SoA = Statement of Applicability — mandatory ISO 27001 document declaring which controls apply
- SOC 2 Type II > Type I — tests operating effectiveness over time (6–12 months)
- NIST 800-53 = **20 control families** for federal information systems
- NIST 800-63B: **no mandatory password rotation** (rotate only if compromised); min 8 chars
- GRC = Governance + Risk Management + Compliance — integrated, not siloed
- PCI DSS has **12 requirements** across 6 control objectives; Level 1 requires annual QSA audit

### Common Interview Questions

| Question | Key Points |
|----------|-----------|
| What is the difference between governance and compliance? | Governance sets internal direction; compliance demonstrates adherence to external requirements. One is proactive (direction); one is reactive (measurement). |
| What are GDPR's two penalty tiers? | Tier 1: 4% revenue/€20M for severe violations. Tier 2: 2%/€10M for less severe. Always whichever is higher. |
| What is an ISMS? | Information Security Management System — documented, systematic approach to managing information security per ISO 27001 |
| What is the SoA in ISO 27001? | Statement of Applicability — declares which Annex A controls are implemented and why others are excluded |
| What is the difference between ISO 27001 and ISO 27002? | 27001 defines ISMS requirements and is certifiable; 27002 provides guidance on implementing Annex A controls and is not certifiable |
| What is SOC 2 Type II? | Auditing report testing both the design and operating effectiveness of controls over 6–12 months — more rigorous and more trusted than Type I |
| Why does NIST 800-63B say no mandatory password rotation? | Forced rotation causes predictable patterns (Password1! → Password2!). Rotation only when compromise is suspected is more effective. |
| What is Privacy by Design? | GDPR Article 25 requirement to integrate data protection controls at design time by default — not added after deployment |
