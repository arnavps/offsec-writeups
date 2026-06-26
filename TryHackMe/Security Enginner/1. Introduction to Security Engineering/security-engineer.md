# Security Engineer

## Overview

This room defines the security engineer role — what organisations expect from the position, the core responsibilities it carries, and how a security engineer contributes to an organisation's ongoing security posture. Unlike purely technical roles, a security engineer sits at the intersection of technology, business, and risk — translating security principles into practical, organisation-wide protections.

**Why it matters:** Understanding the scope and expectations of the role is essential before building the technical skills to fill it. A security engineer who only understands tools but not responsibilities will struggle to prioritise, communicate, and deliver value in a real organisation.

**Where these concepts apply:** Every organisation with a security function — from startups to enterprises — expects some version of this role. The skills translate directly to blue team positions, security architecture, GRC, and DevSecOps.

---

## Why Organisations Need Security

The digital transformation of business has created enormous opportunity — and enormous risk. Organisations face a fundamental choice: retreat from technology (and lose competitive advantage) or embrace it while managing the security implications. The latter is the only viable path.

Just as organisations protect physical assets with dedicated departments, digital assets require dedicated security functions. Cyber incidents — ransomware, data breaches, credential theft, business email compromise — carry consequences including:

- **Operational disruption** — systems taken offline, operations halted
- **Data loss** — customer PII, intellectual property, financial records exposed
- **Regulatory penalties** — GDPR, HIPAA, PCI DSS fines for inadequate data protection
- **Reputational damage** — loss of customer and partner trust
- **Legal liability** — lawsuits from affected parties

The security engineer is the primary technical resource an organisation relies on to prevent, detect, and respond to these outcomes.

---

## The Security Engineer Role

### How Organisations Define the Role

Organisations typically view the security engineer as:

| Expectation | Description |
|-------------|-------------|
| **Owns overall security** | Primary person responsible for securing the organisation's digital assets |
| **Minimises cyber risk** | Ensures cyber security risk is reduced at all times, not just during incidents |
| **Devises strategies** | Creates systems and strategies that reduce risk from cyber threats |
| **Conducts periodic tests** | Ensures the robustness of security posture; identifies weaknesses; prepares mitigations |
| **Develops secure solutions** | Implements secure network solutions and secure architectures |
| **Engineers trustworthy systems** | Architects reliable, secure systems from the ground up |
| **Coordinates across teams** | Establishes security protocols across the entire organisation |

### Why the Role Is Broad

The security engineer role is intentionally loosely defined — it varies significantly between organisations. An engineer's job is to take large problems, break them into manageable components, and solve them systematically. For security engineers, that means a different set of problems every day — from a new vulnerability discovered in a critical system to a policy gap identified in an audit.

### Entry-Level Requirements

| Requirement | Detail |
|------------|--------|
| Experience | 0–2 years in IT administration, helpdesk, networks, or security operations |
| Technical foundations | Computer networks, operating systems, basic programming |
| Security foundations | Governance, Risk and Compliance (GRC) concepts |

---

## Core Responsibilities

### 1. Asset Management / Asset Inventory

**Definition:** Maintaining a complete, accurate, and up-to-date record of all digital assets the organisation owns or operates.

**Why it matters:** Security engineers can only protect assets they know about. Untracked assets — shadow IT, forgotten servers, unmanaged cloud instances — are the first targets attackers find because they are the least protected.

**What a good asset inventory includes:**

| Field | Purpose |
|-------|---------|
| Asset name and type | Identify the asset |
| IP addresses (internal/external) | Locate the asset on the network |
| Physical location | Know where hardware is housed |
| Network position | Understand what the asset can reach and what can reach it |
| Applications running | Know the software attack surface |
| Access permissions | Is it internal only or public-facing? |
| Asset owner | Who is responsible for this asset? |

**Security importance:** Asset inventory is the foundation of every other security function — vulnerability management, patch management, access control, incident response, and compliance all depend on knowing what exists.

---

### 2. Security Policies

**Definition:** Formal documented rules governing how an organisation's information and systems must be used and protected.

**The security engineer's role:**
- Help create security policies based on established security principles
- Ensure implementation follows both the letter and spirit of the policies
- Handle policy exceptions — when business needs require deviation, the engineer consults security principles to allow or deny the exception and suggests mitigating controls
- Policy exceptions are a real operational challenge — the engineer must balance security rigour with business practicality

---

### 3. Secure by Design

**Definition:** Building security into systems from the start rather than adding it after the fact.

**Why it delivers the best ROI:** Security controls are cheapest and most effective when embedded in the design. Retrofitting security onto an insecure design is expensive and often incomplete.

**What "secure by design" covers:**
- **Secure Network Architecture** — segmentation, least privilege network access, DMZs, defence in depth
- **Hardened endpoints** — Windows, Linux, and Active Directory hardened to baseline standards (CIS Benchmarks, NIST 800-53)
- **Secure Software Development Lifecycle (SSDLC)** — security requirements, threat modelling, SAST/DAST, security code review built into every development sprint

---

### 4. Security Assessment and Assurance

**Definition:** Continuous testing and validation of the security posture through assessments, audits, and adversarial exercises.

**Key insight:** Securing the design is not enough. Security is a continuous effort — one unpatched vulnerability or misconfiguration is all an attacker needs. Continuous assessment catches what implementation misses.

**Types of assessments:**

| Activity | What It Tests |
|----------|--------------|
| Vulnerability assessment | Automated discovery of known vulnerabilities |
| Penetration testing | Manual exploitation testing by skilled testers |
| Red team exercise | Full adversary simulation — test people, processes, and technology |
| Purple team exercise | Collaborative red + blue team exercise to improve detection and response |
| Security audit | Compliance and control effectiveness review |

**The security engineer's role in assessments:**
- Schedule activities and create RFQs (Requests for Quotations) for external parties
- Prioritise and track findings from assessments
- Drive remediation of findings and verify fixes
- The engineer typically does not perform all assessments personally but owns the process

---

### 5. Ensuring Awareness

**Definition:** Maintaining a baseline level of security awareness across the entire organisation, with targeted training for specific teams.

**Why humans are the focus:** Humans are consistently the weakest link in security. Phishing attacks, social engineering, and accidental data exposure are enabled by people — not technology failures. Awareness training directly reduces this risk.

**Types of awareness activities:**
- Organisation-wide anti-phishing simulations and training
- Targeted sessions for developers on secure coding practices
- Targeted sessions for network engineers on secure network architecture
- Periodic refreshers as the threat landscape evolves

---

### 6. Managing Risks

**Definition:** Identifying security risks, determining likelihood and impact, and finding solutions to reduce them to acceptable levels.

**Key principle:** Not all risks can be eliminated. Business operations require accepting some level of risk. The security engineer acts as a trusted advisor to management — providing subject matter expertise to inform accept vs. mitigate decisions.

**Real-world example from the room:**
> An organisation runs a supply chain database on a vulnerable Linux version. Updating would take over a year — vendor has not tested the software on the patched OS version, and deploying without testing risks operational disruption.

The security engineer's response:
- **Cannot eliminate the risk** — patching would break operations
- **Mitigation 1:** Harden the OS through additional controls (disable unused services, apply available patches for non-conflicting components)
- **Mitigation 2:** Place a reverse proxy in front of the vulnerable system to prevent direct internet exposure
- **Outcome:** Risk is significantly reduced without disrupting operations — an accepted, documented residual risk

This is the practical reality of risk management in organisations.

---

### 7. Change Management

**Definition:** Tracking changes to the organisation's digital environment and ensuring they do not introduce new security vulnerabilities.

**Why it matters:** Organisations constantly evolve — new modules, new integrations, new vendors, new cloud services. Each change is a potential new attack surface if not reviewed.

**Example from the room:**
> Organisation wants to upgrade its e-commerce module. The security engineer ensures: risk assessment completed, penetration testing and vulnerability assessment performed before integration, security policies followed, no new vulnerabilities introduced.

**Security engineer's role:** Gate-keeping new changes — nothing goes live without a security review.

---

### 8. Vulnerability Management

**Definition:** Continuously monitoring for new vulnerabilities across the organisation's asset inventory and planning remediation or risk reduction.

**Key point:** The threat landscape evolves constantly — new CVEs are published daily. Vulnerability management is an ongoing operational discipline, not a one-time exercise. Vulnerabilities are prioritised by severity and business impact.

---

### 9. Compliance and Audits

**Definition:** Ensuring the organisation meets all applicable regulatory and industry compliance requirements and maintaining necessary certifications.

**Common frameworks a security engineer works with:**

| Framework | Industry |
|-----------|---------|
| PCI DSS | Payment card processing |
| HIPAA | Healthcare |
| SOC 2 | Technology service providers |
| ISO 27001 | All industries (global) |
| NIST 800-53 | US federal / widely adopted |
| GDPR | EU data handling |

**The security engineer's role:**
- Work with internal and external auditors
- Identify and remediate non-compliance gaps
- Maintain certifications (ISO 27001, SOC 2 Type II)
- Evidence collection and audit preparation

---

## Additional Responsibilities (Organisation-Dependent)

### Managing Security Tooling

Configuring, fine-tuning, and making procurement decisions for security tools:
- **SIEM** (Security Information and Event Management) — log aggregation, correlation, alerting
- **Firewalls** — network perimeter and internal segmentation controls
- **WAF** (Web Application Firewall) — application-layer HTTP/S filtering
- **EDR** (Endpoint Detection and Response) — endpoint threat detection and containment
- Input on tool procurement decisions — evaluating competitive tools against organisational requirements

### Tabletop Exercises

Simulations of security scenarios to test operational readiness:
- Scenarios designed around realistic incident types (e.g., endpoint compromise via phishing)
- Team members explain their actions per the organisation's playbooks
- Identifies gaps in procedures, communication, and tooling before a real incident
- Security engineer often designs or facilitates these exercises

### Disaster Recovery and Crisis Management

Planning for untoward incidents to maintain business continuity:
- **Business Continuity Planning (BCP)** — ensuring critical operations continue during disruption
- **Disaster Recovery Planning (DRP)** — restoring systems after an incident
- **Crisis Management** — coordinating the organisation's response to severe incidents
- Required by many compliance frameworks (ISO 27001, SOC 2)
- Specific involvement varies significantly by organisation

---

## Security Engineer Perspective

### What Makes This Role Different from Pure Technical Roles

A security engineer is not just a technical practitioner — they are a business enabler. Every decision they make must balance security rigour with operational feasibility. Key competencies beyond technical skills:

- **Risk communication** — translate technical risk into business impact language for executives
- **Prioritisation** — not everything can be fixed at once; triage based on actual risk
- **Cross-functional collaboration** — work with development, IT, operations, legal, HR, and leadership
- **Policy and process design** — write controls that are enforceable, not just theoretically correct
- **Continuous improvement mindset** — security is a journey, not a destination

### Common Mistakes New Security Engineers Make

- Treating security as binary — "secure" or "not secure" — rather than as a risk spectrum
- Prioritising technical elegance over operational practicality
- Failing to communicate risk in business terms — executives need impact and cost, not CVE IDs
- Neglecting asset inventory — trying to secure systems they do not fully know exist
- Treating compliance as equivalent to security — compliance is a floor, not a ceiling

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **Asset Inventory** | Comprehensive record of all digital assets including type, location, network position, and ownership |
| **BCP** | Business Continuity Plan — ensures critical operations continue during disruption |
| **Change Management** | Process of reviewing and approving changes to the environment to prevent security regressions |
| **DRP** | Disaster Recovery Plan — procedures for restoring systems after an incident |
| **EDR** | Endpoint Detection and Response — endpoint security tool with detection and containment capability |
| **GRC** | Governance, Risk, and Compliance — integrated framework |
| **Penetration Testing** | Authorised manual exploitation testing to find real-world attack paths |
| **Purple Team** | Collaborative red + blue team exercise to improve detection and response capability |
| **Red Team** | Adversary simulation exercise testing people, process, and technology holistically |
| **RFQ** | Request for Quotation — formal document soliciting bids from external security service providers |
| **ROI** | Return on Investment — used here in context of security-by-design delivering more value per dollar spent |
| **SIEM** | Security Information and Event Management — log aggregation and threat detection platform |
| **SSDLC** | Secure Software Development Lifecycle — security integrated into every stage of development |
| **Tabletop Exercise** | Simulation of a security scenario to test team readiness using discussion rather than live response |
| **WAF** | Web Application Firewall — filters malicious HTTP/S requests at the application layer |

---

## Exam and Interview Revision

### Must Remember

- The security engineer **owns** the organisation's security posture — broad, not narrow
- Security by design delivers the best ROI — cheapest to fix at design, most expensive at production
- Risk management = advising management on accept vs. mitigate; not all risks can be eliminated
- Change management ensures nothing goes into production without a security review
- Compliance ≠ security — compliance is minimum standards; security aims higher
- Asset inventory is the foundation of every other security function
- Humans are the weakest link — awareness training directly reduces this risk

### Common Interview Questions

| Question | Key Points |
|----------|-----------|
| What does a security engineer do? | Owns organisation-wide security posture; manages risk, policies, assets, assessments, compliance, and security tooling |
| What is the difference between red team and purple team? | Red team = adversary simulation, finds gaps. Purple team = collaborative exercise, red and blue work together to improve detection and response |
| Why is security by design important? | Most cost-effective: vulnerabilities found at design cost a fraction of those found in production |
| What is a tabletop exercise? | Discussion-based simulation of a security incident scenario to test team readiness per playbooks |
| How does a security engineer handle a risk that cannot be eliminated? | Mitigate via compensating controls; document the residual risk; obtain management sign-off on acceptance |
| What is change management in security context? | Process of reviewing and approving environmental changes to ensure they do not introduce new vulnerabilities |
