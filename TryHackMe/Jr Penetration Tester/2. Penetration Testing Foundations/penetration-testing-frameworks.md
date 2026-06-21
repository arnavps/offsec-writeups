# Penetration Testing Frameworks

## Overview

This room establishes why penetration testing frameworks exist and provides an in-depth examination of the major frameworks used in professional security assessments today. It covers OSSTMM, OWASP WSTG, NIST SP 800-115, PTES, ISSAF, MITRE ATT&CK, and five specialised frameworks. The room concludes with a practical framework selection guide that maps real engagement scenarios to the appropriate methodology.

**Main objectives:**
- Understand why structured methodologies are necessary for penetration testing
- Know the philosophy, structure, strengths, and limitations of each major framework
- Understand MITRE ATT&CK as a complementary knowledge base rather than a testing methodology
- Apply selection criteria to choose the right framework for a given engagement
- Recognise specialised frameworks for specific domains and regulatory environments

---

## Concepts Covered

### Why Penetration Testing Frameworks Exist

**Definition:** A penetration testing framework is a structured methodology that guides security professionals through every stage of an engagement — from planning and scoping through exploitation, reporting, and remediation validation.

**Why it matters:** Without a structured approach, a penetration test becomes a disorganised collection of random checks. Critical attack surfaces get missed, documentation is inconsistent, and findings cannot be prioritised or actioned reliably.

**Benefits of a structured methodology:**
- **Thoroughness** — ensures no critical areas are overlooked
- **Consistency** — different testers on the same team produce comparable results
- **Compliance support** — aligns assessments with regulatory requirements
- **Communication** — clients, auditors, and stakeholders can understand and trust a recognisable standard

**Key analogy:** A building inspector follows a code-compliance checklist — they do not wander through the building hoping to notice problems. Penetration testing frameworks serve the same purpose.

---

## The Major Frameworks

### OSSTMM — Open Source Security Testing Methodology Manual

**Developer:** ISECOM (Institute for Security and Open Methodologies)
**Current version:** v3
**Philosophy:** Scientific, metrics-driven security testing — quantifiable results over subjective opinions

**Why it matters:** OSSTMM challenges the norm of subjective penetration test reports. It produces quantifiable, verifiable, and repeatable results that two different testers assessing the same target should arrive at similarly.

**Five security channels:**

| Channel | Scope |
|---------|-------|
| Human Security (HUMSEC) | Social engineering and human-factor vulnerabilities |
| Physical Security (PHYSSEC) | Physical access controls — badge readers, tailgating |
| Wireless Communications (SPECSEC) | Wi-Fi, Bluetooth, RFID, other electromagnetic signals |
| Telecommunications (COMSEC) | Phone systems, VoIP, fax, modems |
| Data Networks (DATASEC) | Network services, firewalls, application-layer protocols |

**Why five channels matter:** An organisation may have perfect firewall rules but be vulnerable to tailgating (PHYSSEC) or credential phishing (HUMSEC). OSSTMM ensures nothing is overlooked by covering the full security surface.

**Risk Assessment Values (RAVs):** OSSTMM's quantitative core. A RAV measures the balance between the total attack surface (exposure) and the controls protecting it. A positive RAV = residual risk; near-zero RAV = controls well-matched to exposure.

**Four testing phases:**

| Phase | Description | Example (FinVault Corp) |
|-------|-------------|------------------------|
| Induction | Enumeration and verification — map what exists | DNS queries, certificate transparency logs, confirming live assets |
| Interaction | Qualification and quantification — probe verified assets | 12 externally reachable services, 4 accepting unauthenticated connections |
| Inquiry | Privilege escalation and verification escalation | IDOR vulnerability — read access to 12,000 accounts confirmed |
| Intervention | Quarantine, audit, and enticement | Endpoint restricted; audit for similar flaws; canary token deployed |

**Report format:** STAR (Security Test Audit Report) — enforces consistency.

**Strengths:** Results are auditable, comparable, and cross-team reproducible
**Limitations:** Steep learning curve; full implementation is time-consuming; fewer trained practitioners than OWASP/NIST

**Best for:** Organisations needing repeatable, auditable security measurements

---

### OWASP WSTG — Web Security Testing Guide

**Developer:** Open Web Application Security Project (OWASP)
**Philosophy:** Comprehensive, community-driven web application testing across over 90 discrete test cases

**Why it matters:** OWASP WSTG answers the problem of scope creep and missed coverage in web application testing. With 90+ numbered test cases organised across 12 categories, it provides a concrete roadmap.

**12 test categories:** Information gathering, configuration and deployment management, identity management, authentication, authorisation, session management, input validation, error handling, cryptography, business logic, client-side testing, API testing.

**Test case naming:** `WSTG-INPV-01` (Input Validation, test case 1 — reflected XSS)

**SDLC integration — 5 phases:**

| Phase | When | Action |
|-------|------|--------|
| 1 | Before development | Define security requirements and regulatory obligations |
| 2 | During design | Review architecture for security flaws; create threat models |
| 3 | During development | Code review against WSTG test cases |
| 4 | During deployment | Penetration test of staged application |
| 5 | During maintenance | Re-test after updates and new features |

**Example (ShopSecure Inc.):** During design, threat models for the payment flow identified the checkout API as high-value. During development, a code review identified non-expiring password reset tokens against `WSTG-ATHN` test cases.

**Risk-based approach:** Vulnerabilities are prioritised by exploitability and impact — not simply catalogued.

**Strengths:** Exhaustive, practical coverage; continuously updated by a global community; covers modern architectures (SPAs, microservices, APIs)

**Limitations:** Full implementation of all 90+ test cases is impractical for resource-constrained teams; risk of checklist mentality

**Best for:** Web application penetration testing

---

### NIST SP 800-115 — Technical Guide to Information Security Testing

**Developer:** National Institute of Standards and Technology (NIST)
**Full title:** Technical Guide to Information Security Testing and Assessment
**Philosophy:** Flexible, structured framework for security testing in government and enterprise contexts

**Why it matters:** NIST SP 800-115 carries institutional credibility in U.S. government, defence, and regulated industries. It treats penetration testing as one technique within a broader assessment toolset.

**Three core objectives:**
1. Identify vulnerabilities in systems, networks, and applications
2. Validate security controls by testing whether they perform under adversarial conditions
3. Assess exploitability — determine whether identified weaknesses can actually be leveraged

**Three phases:**

| Phase | Description |
|-------|-------------|
| Planning | Define objectives, scope, rules of engagement — all formally documented and signed off |
| Execution | Four technique categories: review, target identification, vulnerability validation, penetration testing |
| Post-Testing | Analyse results, prioritise risks, deliver actionable remediation |

**Execution technique categories (ordered from passive to active):**

| Technique | Description | Example (GovNet) |
|-----------|-------------|-----------------|
| Review | Examine documentation, policies, configurations | Review firewall rules for misconfigurations |
| Target identification | Discover and fingerprint live hosts and services | Scan portal infrastructure, identify 12 internet-facing services |
| Vulnerability validation | Confirm weaknesses are real, not false positives | Manually validate SQL injection on portal search |
| Penetration testing | Simulate adversarial attacks | Chain SQLi + DB privilege escalation to demonstrate data access |

**Key insight:** Not every engagement needs to reach the penetration testing stage. Sometimes review and vulnerability validation are sufficient for the assessment objectives.

**Findings must be actionable:** "You have a SQL injection" is insufficient. "The `search` parameter at `/portal/search` is injectable via UNION-based technique; affected data: citizen records table; remediation: parameterise the query" is the standard.

**Strengths:** Flexible across environments (data centres, cloud, IoT); institutional credibility; promotes standardisation

**Limitations:** Guidance only — no enforcement, unlike PCI DSS; requires broad skill set across all technique categories

**Best for:** Government and enterprise security assessments; regulated industries requiring standardised methodology

---

### PTES — Penetration Testing Execution Standard

**Source:** pentest-standard.org
**Developer:** Experienced security practitioners
**Philosophy:** End-to-end practical engagement workflow — how a real penetration test flows from first client call to final report

**Why it matters:** PTES answers the question: "I have a signed contract; now what?" It maps the entire lifecycle of an engagement in seven explicit phases.

**Seven phases:**

| Phase | Description | Example (MedGuard Health) |
|-------|-------------|--------------------------|
| Pre-Engagement Interactions | Define scope, rules of engagement, legal authorisation | Scope: corporate LAN, patient portal, wireless; testing windows: weeknights only |
| Intelligence Gathering | Passive and active reconnaissance | LinkedIn email harvesting, subdomain discovery, job postings reveal Oracle 19c |
| Threat Modeling | Identify high-value targets and likely attack paths | Patient records DB = highest value; two primary attack paths identified |
| Vulnerability Analysis | Systematically identify and verify weaknesses | Outdated Tomcat version (deserialization CVE); unpatched workstations |
| Exploitation | Exploit confirmed vulnerabilities with purpose | Tomcat deserialization → portal shell; phishing → employee workstation |
| Post-Exploitation | Determine real-world impact from gained access | Pivot to database → read 50,000 patient records; lateral movement to file server |
| Reporting | Structured report with executive and technical sections | HIPAA regulatory exposure communicated to leadership; technical details for IT team |

**Pre-engagement interactions** — PTES is uniquely detailed here because scope ambiguity is the primary source of legal and professional disputes in penetration testing.

**Post-exploitation** — Translates technical access into business risk: "we accessed 50,000 patient records" carries more client weight than "we got a shell."

**Strengths:** Most closely mirrors real engagement workflows; end-to-end structure; excellent for learning workflow instincts

**Limitations:** Not formally updated recently; tool-specific guidance is outdated; lacks quantitative metrics of OSSTMM

**Best for:** Standard network and application penetration tests; practical framework for junior testers

---

### ISSAF — Information Systems Security Assessment Framework

**Developer:** OISSG (Open Information Systems Security Group)
**Version:** v0.2.1 (2006) — no longer actively maintained
**Philosophy:** Adversarial nine-step assessment model mirroring real attack progression

**Why it matters:** ISSAF's nine-step model provides one of the clearest representations of how an attacker progresses through a target environment. It mirrors kill-chain thinking applied to structured assessment.

**Note:** ISSAF is studied for its methodology, not its tool guidance — the latter references decade-old software.

**Three phases:**

**Phase 1: Planning and Preparation** — Define scope, constraints, escalation protocols, toolset

**Phase 2: Assessment — Nine Steps:**

| Step | Activity | Example (TechBridge Solutions) |
|------|----------|-------------------------------|
| 1 | Information gathering | DNS, WHOIS, LinkedIn, job postings ("experience with Jenkins and GitLab required") |
| 2 | Network mapping | External: project portal, VPN, mail server; Internal: Git, Jenkins, workstations |
| 3 | Vulnerability identification | Outdated CMS with auth bypass; Jenkins admin console unauthenticated |
| 4 | Penetration | Unauthenticated Jenkins console → system command execution |
| 5 | Gaining access and privilege escalation | Jenkins service account credentials → admin rights on Git server |
| 6 | Enumerating further | Git repos contain API keys, DB connection strings, client source code |
| 7 | Lateral movement | Harvested credentials → developer workstations and internal mail server |
| 8 | Maintaining access | Document how a CI/CD backdoor could persist across reboots (conceptual) |
| 9 | Covering tracks | Identify which logs captured activity; document logging gaps |

**Phase 3: Reporting and Cleanup** — Structure findings by business impact; remove all test artifacts

**Key insight:** Steps 1–3 are reconnaissance and analysis; steps 4–7 are active compromise; steps 8–9 address persistence and stealth. This mirrors the adversarial mindset precisely.

**Strengths:** Clear adversarial progression model; excellent educational tool for understanding attack methodology

**Limitations:** Unmaintained — tool guidance is obsolete; no community updates

**Best for:** Educational reference for attack methodology; supplement with current tool documentation

---

### MITRE ATT&CK

**Developer:** MITRE Corporation
**Type:** Adversary behaviour knowledge base (not a testing framework)
**Philosophy:** Universal language for describing adversary tactics, techniques, and common knowledge — grounded in real-world threat intelligence

**Why it matters:** ATT&CK fills the gap that penetration testing frameworks leave: they tell you how to conduct a test, but not how your findings compare to what real threat actors actually do in the wild.

**The Matrix structure:**
- **Columns = Tactics** — the adversary's high-level objectives (the *why*)
- **Rows = Techniques** — the specific methods used to achieve each tactic (the *how*)
- **Sub-techniques** — further specificity within a technique (e.g. T1566 Phishing → T1566.001 Spearphishing Attachment)

**14 Enterprise tactics:** Initial Access → Execution → Persistence → Privilege Escalation → Defense Evasion → Credential Access → Discovery → Lateral Movement → Collection → Command and Control → Exfiltration → Impact

**Each technique entry includes:**
- Description and definition
- Real-world examples from known threat groups
- Detection recommendations
- Mitigations

**ATT&CK as complement, not replacement:** ATT&CK does not define phases or reporting formats — it provides standardised naming for what you find. Use PTES to structure the engagement; use ATT&CK to label and contextualise findings.

**Analogy:** PTES is the diagnostic procedure; ATT&CK is the medical dictionary providing standardised terminology for findings.

**Practical mapping example (MedGuard Health from PTES):**

| Engagement Finding | ATT&CK Tactic | ATT&CK Technique |
|-------------------|---------------|-----------------|
| Phishing email delivered payload | Initial Access | T1566.001 — Spearphishing Attachment |
| Tomcat deserialization exploit | Initial Access | T1190 — Exploit Public-Facing Application |
| Cached domain credential extraction | Credential Access | T1003 — OS Credential Dumping |
| Workstation → file server lateral movement | Lateral Movement | T1550 — Use Alternate Authentication Material |
| Patient records database access | Collection | T1213 — Data from Information Repositories |

**Value of mapping:** Enables the client to look up each technique in ATT&CK, review detection guidance, and build detection rules for those specific adversary behaviours. Shifts the conversation from "fix this bug" to "can we detect this class of adversary behaviour?"

**Best for:** Enriching findings from any framework with real-world threat intelligence context; enabling detection engineering based on test findings

---

## Specialised Frameworks

| Framework | Domain | Type | Active | When to Use |
|-----------|--------|------|--------|-------------|
| **WASC Threat Classification** | Web applications | Threat taxonomy | No (superseded by OWASP) | Legacy references; historical context |
| **CSA Cloud Controls Matrix (CCM)** | Cloud environments | Governance/compliance | Yes | Cloud security posture assessments |
| **OWASP MASTG** | Mobile apps (Android/iOS) | Testing guide | Yes | Mobile app penetration testing |
| **PCI DSS Guidelines** | Payment card environments | Regulatory mandate | Yes (v4.0) | Any engagement involving cardholder data |
| **CBEST** | UK financial institutions | Threat-intel-led pentest | Yes | UK financial institution engagements |

**OWASP MASTG** is the mobile counterpart to WSTG — used with MASVS (Mobile Application Security Verification Standard) which defines the security requirements the MASTG tests against.

**PCI DSS Requirement 11.4** mandates annual penetration tests of external perimeter and internal network, after significant infrastructure changes, with validation of network segmentation controls.

**CBEST** is unique: it begins with bespoke threat intelligence identifying the most relevant threat actors for the specific UK financial institution, then simulates those exact threat scenarios.

---

## Framework Selection

### Selection Criteria

| Criterion | Impact on selection |
|-----------|-------------------|
| **Engagement scope and target type** | Web → OWASP WSTG; Mobile → OWASP MASTG; Network → PTES or OSSTMM; Multi-channel → OSSTMM |
| **Regulatory requirements** | PCI DSS → PCI guidelines mandatory; UK bank → CBEST; U.S. federal → NIST SP 800-115 |
| **Need for quantifiable results** | Cross-team comparison over time → OSSTMM RAVs |
| **Team expertise and resources** | Standard team without specialised training → PTES |

### Scenario Practice

**Scenario 1:** Regional hospital, patient-facing web portal and internal network, HIPAA compliance required, needs executive summary for board.

**Selection:** PTES (end-to-end structure, dual executive/technical reporting) + OWASP WSTG (web portal supplement) + MITRE ATT&CK (findings enrichment)

---

**Scenario 2:** Multinational bank in London, UK regulatory obligations, threat intelligence assessment required.

**Selection:** CBEST — designed for UK financial institutions, mandates bespoke threat intelligence phase, recognised by the Bank of England.

---

**Scenario 3:** SaaS startup wants two different firms to test independently and compare results year-over-year. Platform is entirely web-based.

**Selection:** OSSTMM (RAV metrics enable direct cross-team comparison) + OWASP WSTG (web-specific test cases)

---

**Scenario 4:** Fintech company, Android and iOS mobile banking apps, handles credit card transactions.

**Selection:** OWASP MASTG (mobile test cases) + PCI DSS guidelines (mandatory for cardholder data). Both frameworks must be applied simultaneously.

**Key insight:** Real engagements often require a primary framework plus supplementary frameworks for regulatory, platform, or reporting requirements.

---

## Master Framework Comparison Table

| Framework | Scope | Key Strength | Limitation | Best For |
|-----------|-------|-------------|-----------|---------|
| OSSTMM | Multi-channel (5) | Quantifiable metrics (RAVs) | Steep learning curve | Repeatable, auditable assessments |
| OWASP WSTG | Web applications | 90+ structured test cases | Checklist mentality risk | Web app penetration testing |
| NIST SP 800-115 | Systems, networks, apps | Federal credibility; flexible | Guidance only — not enforced | Government and enterprise assessments |
| PTES | End-to-end engagements | Practical, phase-driven workflow | Not recently updated | Standard network/application pentests |
| ISSAF | Networks, systems, apps | Clear nine-step adversarial model | No longer maintained | Educational reference for attack methodology |
| MITRE ATT&CK | Adversary behaviour | Universal threat behaviour language | Not a testing methodology | Enriching findings from any framework |
| OWASP MASTG | Mobile apps (Android/iOS) | Platform-specific mobile test cases | Mobile-only scope | Mobile app security testing |
| PCI DSS Guidelines | Payment card environments | Regulatory mandate | Narrow applicability | Engagements involving cardholder data |
| CSA CCM | Cloud environments | Governance controls mapped to standards | Not a pentest methodology | Cloud security posture assessments |
| CBEST | UK financial sector | Threat-intel-led, regulatory backing | UK financial sector only | UK financial institution engagements |
| WASC Threat Classification | Web applications | Historical taxonomy | Superseded by OWASP | Legacy reference only |

---

## Key Learnings

- Frameworks provide thoroughness, consistency, compliance alignment, and communication quality — without them, testing is unstructured
- OSSTMM's five channels ensure physical and human attack surfaces are not overlooked alongside network testing
- OWASP WSTG's 90+ test cases span 12 categories covering the full web attack surface across the SDLC
- NIST SP 800-115 treats pentesting as one technique within a broader assessment toolkit — planning and post-testing phases are equally important
- PTES is the most practically structured framework for standard engagements — seven phases mirror real engagement workflow
- ISSAF's nine-step model is a valuable adversarial thinking tool even though the framework is unmaintained
- MITRE ATT&CK is not a testing framework — it is a vocabulary for findings that elevates reports to real-world threat context
- Framework selection is driven by scope, regulation, measurement requirements, and team capability — often requires combining frameworks
- PCI DSS and CBEST are regulatory mandates — they override personal framework preference when applicable

---

## Things Worth Remembering

- OSSTMM RAV = measure of residual risk (exposure minus controls); near-zero = well-protected
- OSSTMM testing phases: Induction → Interaction → Inquiry → Intervention
- WSTG test case naming: `WSTG-[CATEGORY]-[NUMBER]` (e.g. `WSTG-INPV-01` = reflected XSS)
- NIST SP 800-115 phases: Planning → Execution (4 technique categories) → Post-Testing
- PTES phases: Pre-Engagement → Intelligence Gathering → Threat Modelling → Vulnerability Analysis → Exploitation → Post-Exploitation → Reporting
- ISSAF nine steps: Info gathering → Network mapping → Vuln ID → Penetration → Access/PrivEsc → Enumerate → Lateral movement → Persistence → Cover tracks
- ATT&CK format: `T[Technique ID].[Sub-technique ID]` (e.g. T1566.001)
- ATT&CK 14 tactics: Initial Access, Execution, Persistence, Privilege Escalation, Defence Evasion, Credential Access, Discovery, Lateral Movement, Collection, C2, Exfiltration, Impact
- Regulatory frameworks that mandate pentests: PCI DSS (annually), CBEST (UK banks), NIST SP 800-115 (U.S. federal guidance)

---

## Conclusion

Penetration testing frameworks transform a security assessment from a collection of tool outputs into a professional, structured engagement that produces actionable, comparable, and credible results. Each framework has a distinct philosophy and optimal context — OSSTMM for quantifiable rigour, OWASP WSTG for web coverage, NIST SP 800-115 for institutional credibility, PTES for practical workflow, ISSAF for adversarial thinking, and MITRE ATT&CK for real-world threat context. The skill of framework selection — knowing which methodology applies to which engagement type and when to combine frameworks — is a practical competency that distinguishes methodical penetration testers from those who simply run tools.
