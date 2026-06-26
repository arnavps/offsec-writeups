# Threat Modelling
## Security Engineer Knowledge Base

> **Source Room:** Threat Modelling — TryHackMe Security Engineer Path
> **Purpose:** Long-term reference for revision, interviews, certifications, blue-team and security engineering work

---

## Overview

### What This Room Teaches

- The definitions of threat, vulnerability, and risk and how they relate
- The high-level threat modelling process and which teams are involved
- Attack trees and attack paths as visual modelling tools
- Four major threat modelling frameworks: MITRE ATT&CK, DREAD, STRIDE, and PASTA
- How to apply each framework to real scenarios and when to use each

### Why It Matters

Threat modelling is how organisations move from vague awareness of cyber risk to a structured, prioritised, actionable security plan. Without it, security spending is reactive and misdirected. With it, you understand your attack surface, know which threats are most likely, and implement controls where they actually matter.

### Where These Concepts Are Used in Real Environments

- **Security architecture reviews** — STRIDE and PASTA are used during design to identify threats before code is written
- **SOC and detection engineering** — MITRE ATT&CK drives detection rule development, threat hunting, and alert triage
- **Red team and purple team exercises** — ATT&CK TTPs define the techniques simulated
- **Penetration test scoping** — threat models define what to test and in what priority order
- **GRC programmes** — threat models feed into risk registers and risk treatment decisions
- **DevSecOps** — threat modelling is integrated into sprint planning and design reviews

### Key Takeaways

- Risk = Threat + Vulnerability + Asset — all three must coexist for risk to exist
- Attack trees map the attacker's decision space hierarchically from goal to specific tactic
- MITRE ATT&CK is grounded in real-world observed adversary behaviour — it is not theoretical
- DREAD scores risk numerically but is subjective — needs standardised rubrics
- STRIDE categorises threats by type against the CIA triad and non-repudiation
- PASTA is the most business-aligned framework — seven steps from objectives to risk treatment

---

## Core Concepts

---

### Concept 1 — Threat, Vulnerability, and Risk

#### Definitions

| Term | Definition |
|------|-----------|
| **Threat** | Any potential occurrence, event, or actor that may exploit a vulnerability to compromise the CIA of information. Can be cyber attacks, human error, or natural events. |
| **Vulnerability** | A weakness or flaw in a system, application, or process exploitable by a threat to cause harm. Arises from bugs, misconfiguration, or design flaws. |
| **Risk** | The probability of a threat exploiting a vulnerability and resulting in adverse business impact. Risk is the intersection of all three elements. |

#### The House Analogy

| Security Term | House Analogy |
|--------------|--------------|
| Threat | A burglar attempting to break into your home |
| Vulnerability | A broken lock or open window |
| Risk | The likelihood of being burglarised based on neighbourhood crime rate and absence of an alarm |

#### Relationship Diagram

```
         ASSET
      (what has value)
           |
    VULNERABILITY
  (weakness in asset)
           |
        THREAT
  (actor/event that exploits)
           |
          RISK
  (probability × impact of
   a successful exploit)
```

#### Why the Distinction Matters

Understanding which of the three elements is present helps focus the response:
- If there is no asset of value → risk is negligible regardless of vulnerability
- If there is no vulnerability → a threat cannot exploit what does not exist
- If threat likelihood is negligible → a high-severity vulnerability may be low actual risk

#### Interview Notes

- "Can you have risk without all three elements?" — No. Risk requires a threat that can exploit a vulnerability affecting an asset. Remove any one element and risk does not exist or approaches zero.
- "What is the difference between a threat and a vulnerability?" — A threat is the actor or event; a vulnerability is the weakness it exploits. A threat exists externally; a vulnerability exists internally.


---

### Concept 2 — The Threat Modelling Process

#### Definition

A systematic, structured approach to identifying, prioritising, and addressing potential security threats to an organisation's systems and applications.

#### High-Level Process

| Step | Activity |
|------|---------|
| 1. **Define Scope** | Identify systems, applications, and networks in scope |
| 2. **Asset Identification** | Build architecture diagrams; classify assets by criticality (PII, financial data, IP) |
| 3. **Identify Threats** | Cyber attacks, physical attacks, social engineering, insider threats |
| 4. **Analyse Vulnerabilities and Prioritise Risks** | Assess existing controls; prioritise by likelihood × impact |
| 5. **Develop and Implement Countermeasures** | Access controls, patching, encryption, WAF, network segmentation |
| 6. **Monitor and Evaluate** | Track mitigation effectiveness; re-evaluate on schedule |

#### Teams Involved

| Team | Role |
|------|------|
| **Security Team** | Lead process; provide threat and vulnerability expertise; validate controls |
| **Development Team** | Ensure security is integrated into code; provide code-level vulnerability insight |
| **IT and Operations** | Provide infrastructure topology, system configs, and integration details |
| **GRC Team** | Align threat modelling with regulatory obligations and risk tolerance |
| **Business Stakeholders** | Define critical assets, acceptable risk, and business process priorities |
| **End Users** | Provide user behaviour insights; surface user-specific attack surfaces |

---

### Concept 3 — Attack Trees

#### Definition

A graphical, hierarchical representation of possible attack scenarios. The root node represents the attacker's ultimate goal; child nodes represent strategies to achieve it, broken down progressively into specific techniques.

#### Why It Exists

Complex attack scenarios become manageable when decomposed into components that can be individually analysed, scored, and mitigated. Attack trees provide a visual map of the attacker's decision space and make attack paths communicable to non-technical stakeholders.

#### Structure

```
Root Node: Attacker's Primary Goal
"Gain unauthorised access to sensitive cloud storage data"
              |
    ┌─────────┴───────────┐
    |                     |
Sub-goal A           Sub-goal B
"Exploit application  "Compromise
 vulnerabilities"      credentials"
    |                     |
  ┌─┴─┐               ┌───┴───┐
SQLi  RCE          Phishing  Password
                              Spray
```

#### Attack Paths

A variant emphasising the *sequential chain* an attacker follows — each step enables the next. The starting node is the entry point; branches represent the specific vulnerabilities/techniques that advance toward the objective. Attack paths are useful for identifying where a single control failure creates a complete attack chain.

#### Security Importance

- **Penetration test planning** — define realistic test scenarios
- **Purple team exercise design** — select technique sequences to simulate
- **Security architecture review** — identify single points of failure in control chains
- **Executive communication** — visually explain attack scenarios without technical jargon

---

### Concept 4 — MITRE ATT&CK Framework

#### Definition

MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) is a globally accessible, publicly maintained knowledge base of real-world adversary behaviours, organised as a matrix of tactics and techniques. Maintained by the MITRE Corporation.

#### Why It Exists

Before ATT&CK, there was no standardised, behaviour-based taxonomy for describing how attackers operated. ATT&CK provides a shared language enabling defenders, threat intelligence analysts, detection engineers, and red teams to communicate precisely about adversary behaviour.

#### Structure

```
Tactic (high-level goal — the WHY)
  e.g., "Lateral Movement"
         |
         v
Technique (method — the HOW)
  e.g., "T1021 - Remote Services"
         |
         v
Sub-Technique (specific variation)
  e.g., "T1021.001 - Remote Desktop Protocol"
```

#### The Three ATT&CK Matrices

| Matrix | Covers |
|--------|--------|
| **Enterprise** | Windows, Linux, macOS, cloud (AWS/GCP/Azure), containers, network |
| **Mobile** | Android and iOS |
| **ICS** | Industrial Control Systems — power plants, water treatment, manufacturing |

#### Technique Page Anatomy

Each technique page contains:
- **Name and description** — what the technique is and how it works
- **Platforms** — which OS/environments are affected
- **Procedure examples** — named real-world threat groups that have used this technique
- **Mitigations** — specific controls to prevent or reduce exposure
- **Detections** — data sources, log events, and analytic logic to detect the technique
- **References** — threat intelligence reports, CVEs, external research

#### Applying ATT&CK in Threat Modelling

```
Standard Threat Modelling Step:
  "Identify Threats"
         |
         v
Insert: "Map to MITRE ATT&CK"
  - Map each identified threat to one or more ATT&CK techniques
  - Use technique pages for:
      - Deeper understanding of how the technique works
      - Detection data sources to build SIEM rules
      - Recommended mitigations to implement
```

#### ATT&CK Use Cases

| Use Case | How ATT&CK Helps |
|----------|----------------|
| Threat modelling | Map identified threats to techniques; use mitigation guidance directly |
| Detection engineering | Write SIEM/EDR rules targeting data sources listed in detection sections |
| Red team / purple team | Select technique sequences relevant to the target environment |
| Threat hunting | Search logs and endpoints for evidence of specific technique indicators |
| Vendor evaluation | Test which ATT&CK techniques a security product detects or blocks |
| Gap analysis | Identify which techniques have zero detection coverage in your environment |
| Threat intelligence mapping | Map known threat group TTPs to your industry and infrastructure type |

#### ATT&CK Navigator

Web-based, open-source tool for visualising and annotating the ATT&CK matrix:
- Create layers for specific threat groups, platforms, or assessment scopes
- Search by threat group (e.g., APT41), software, mitigation, campaign, or data source
- Annotate techniques with severity scores, background colours, comments, and external links
- Filter by platform (e.g., show only Office 365 techniques)
- Export as JSON, Excel, or SVG for reporting and pipeline ingestion

#### Financial Sector Example

Threat groups relevant to financial organisations: APT28, APT29, Carbanak, FIN7, Lazarus Group

Using ATT&CK Navigator:
1. Search APT28 → select all techniques → colour-code by severity
2. Filter by GCP platform → view cloud-specific technique exposure
3. Cross-reference with critical assets (financial data, PII, transaction records)
4. Prioritise: T1190 (Exploit Public-Facing Application), T1068 (Privilege Escalation), T1530 (Data from Cloud Storage), T1498 (Network DoS)

#### Interview Notes

- "What is the difference between a tactic and a technique?" — Tactic = high-level goal (e.g., Persistence). Technique = specific method to achieve it (e.g., T1078 Valid Accounts).
- "Name five ATT&CK tactics" — Initial Access, Execution, Persistence, Privilege Escalation, Defence Evasion, Credential Access, Discovery, Lateral Movement, Collection, Exfiltration, Impact (any five).
- "How would you use ATT&CK for detection?" — Identify techniques relevant to your environment; use the detection section of each technique page to find data sources; write SIEM rules targeting those data sources.


---

### Concept 5 — DREAD Framework

#### Definition

DREAD is a risk assessment model developed by Microsoft for evaluating and prioritising security threats. Each letter is a scoring dimension; the average of all five produces the overall DREAD risk rating.

#### DREAD Components

| Letter | Component | Guiding Question | What It Measures |
|--------|-----------|-----------------|-----------------|
| **D** | Damage | How bad would an attack be? | Data loss, downtime, reputational damage from successful exploitation |
| **R** | Reproducibility | How easy is it to reproduce the attack? | Whether an attacker can reliably re-execute the exploit |
| **E** | Exploitability | How much work is launching the attack? | Skill level, tools, and time required to exploit |
| **A** | Affected Users | How many people are impacted? | Scope of users affected if the vulnerability is exploited |
| **D** | Discoverability | How easy is it to find the vulnerability? | Whether the vulnerability is publicly known or requires deep knowledge |

#### Scoring Formula

```
DREAD Score = (Damage + Reproducibility + Exploitability + Affected Users + Discoverability) / 5
```

Scores are typically 1–10 or use a structured scale (0 / 2.5 / 5 / 7.5 / 10).

Higher score = higher priority for remediation.

#### Example Scoring

| Vulnerability | D | R | E | A | D | Score |
|--------------|---|---|---|---|---|-------|
| Unauthenticated RCE | 10 | 7.5 | 10 | 10 | 2.5 | **8.0** |
| IDOR in user profiles | 2.5 | 7.5 | 7.5 | 10 | 5 | **6.5** |
| Misconfiguration → info disclosure | 0 | 10 | 10 | 0 | 5 | **5.0** |

#### How It Works

After assigning scores to each dimension based on the guiding questions, the average score places the vulnerability in a risk tier that drives remediation priority order.

#### Limitations and Guidelines

**Primary Limitation:** DREAD is **subjective and opinion-based**. Two analysts rating the same vulnerability can produce substantially different scores. This limits repeatability and cross-team consistency.

**Guidelines to improve reliability:**
- Establish a standardised scoring rubric with examples for each score value per dimension
- Use multi-analyst consensus — multiple team members score independently, then reconcile
- Combine with other frameworks (STRIDE, CVSS) to cross-validate
- Define scoring criteria per organisation context — "Affected Users: 10" means different things for a 100-user app vs a 10M-user platform

#### Common Mistakes

- Treating DREAD scores as objective measurements — they are structured opinions
- Scoring without a written rubric — causes inconsistency across time and analysts
- Using DREAD alone without business context — a 7.0 vulnerability on a test server is not the same as a 7.0 on the payment processing server

#### Interview Notes

- "What is the DREAD formula?" — Average of five components: (D+R+E+A+D) / 5
- "What is the main weakness?" — Subjectivity — scores vary between analysts without standardised rubrics
- "DREAD vs CVSS?" — DREAD is opinion-based and context-specific; CVSS is standardised and environment-agnostic. CVSS is better for aligning with industry databases; DREAD is better for internal prioritisation with business context.

---

### Concept 6 — STRIDE Framework

#### Definition

STRIDE is a threat modelling methodology developed by Microsoft for systematically identifying and categorising security threats in software systems. Each letter represents a threat category with a corresponding violated security property.

#### STRIDE Categories

| Letter | Threat | Security Property Violated | Definition |
|--------|--------|---------------------------|-----------|
| **S** | Spoofing | Authentication | Impersonating a legitimate user or system |
| **T** | Tampering | Integrity | Unauthorised modification of data or code |
| **R** | Repudiation | Non-repudiation | Ability to deny having performed an action |
| **I** | Information Disclosure | Confidentiality | Unauthorised access to sensitive information |
| **D** | Denial of Service | Availability | Disrupting availability to legitimate users |
| **E** | Elevation of Privilege | Authorisation | Gaining capabilities beyond what is permitted |

#### Examples Per Category

**Spoofing:** Sending email as another user; phishing site mimicking legitimate login page

**Tampering:** Updating another user's password via IDOR; installing a backdoor using elevated privileges

**Repudiation:** Denying an unauthorised money transfer because the system lacks audit logging; denying sending a message because no delivery record exists

**Information Disclosure:** Unauthenticated access to a misconfigured database; accessing a public S3 bucket with sensitive documents

**Denial of Service:** HTTP flood overwhelming a web server; ransomware encrypting files required by dependent systems

**Elevation of Privilege:** Regular user accessing admin console due to missing authorisation check; local privilege escalation via unpatched vulnerability

#### STRIDE Process

```
1. System Decomposition
   Break down into components, data flows,
   trust boundaries, and attack surfaces
         |
         v
2. Apply STRIDE Categories
   For each component, analyse exposure
   to all six threat categories
         |
         v
3. Threat Assessment
   Evaluate impact and likelihood; prioritise
         |
         v
4. Develop Countermeasures
   Controls specific to each STRIDE category
   Example → Spoofing: implement DMARC, DKIM, SPF
         |
         v
5. Validation and Verification
   Penetration testing, code review, security audit
         |
         v
6. Continuous Improvement
   Revisit as system evolves; update for new threats
```

#### STRIDE Results Table Format

| Scenario | S | T | R | I | D | E |
|----------|---|---|---|---|---|---|
| Spoofed email; mail gateway lacks logging | ✔ | | | | | ✔ |
| Web server flood; no load balancing | | | | | ✔ | |
| SQL injection vulnerability | | ✔ | | ✔ | | |
| Public S3 bucket with customer data | | | | ✔ | | |
| Privilege escalation + persistent backdoor | | ✔ | | | | ✔ |

#### STRIDE vs DREAD Relationship

- **STRIDE** identifies *what* threats exist and categorises them by type
- **DREAD** *scores* those threats by severity and exploitability
- They are complementary — use STRIDE to catalogue, DREAD to prioritise

#### Interview Notes

- "What does STRIDE stand for?" — Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
- "What security property does Repudiation violate?" — Non-repudiation. Primary control: comprehensive audit logging
- "What control addresses email Spoofing?" — DMARC, DKIM, and SPF collectively
- "How does STRIDE connect to the CIA triad?" — I → Confidentiality, T → Integrity, D → Availability. Spoofing/EoP → Authentication/Authorisation. Repudiation → Non-repudiation (extends beyond CIA)


---

### Concept 7 — PASTA Framework

#### Definition

PASTA (Process for Attack Simulation and Threat Analysis) is a structured, **risk-centric**, seven-step threat modelling framework. Created by Tony UcedaVélez and Marco Morana (published 2015). It explicitly incorporates business context, making it the most business-aligned of the major frameworks.

#### Seven-Step Methodology

| Step | Name | Core Activity |
|------|------|--------------|
| 1 | **Define Objectives** | Establish scope, security objectives, and compliance requirements |
| 2 | **Define Technical Scope** | Inventory assets; map architecture, dependencies, and data flows |
| 3 | **Decompose the Application** | Break into components; identify entry points, trust boundaries, attack surfaces, user flows |
| 4 | **Analyse Threats** | Identify threats from all sources (external, insider, accidental); use threat intelligence |
| 5 | **Vulnerabilities and Weaknesses Analysis** | Identify existing vulnerabilities via SAST, DAST, scanning, pen testing |
| 6 | **Analyse Attacks** | Simulate attack scenarios using attack trees; evaluate likelihood and impact |
| 7 | **Risk and Impact Analysis** | Develop countermeasures; align with risk tolerance and business objectives |

#### How PASTA Differs

| Characteristic | STRIDE | DREAD | PASTA |
|---------------|--------|-------|-------|
| Focus | Threat categorisation | Risk prioritisation scoring | Risk-centric, business-aligned analysis |
| Business context | Low | Low | High |
| Suitable for | Software design reviews | Rapid numeric prioritisation | Comprehensive enterprise threat modelling |
| Output | Threat checklist by category | Risk score per vulnerability | Risk-aligned countermeasure plan |
| Complexity | Low–Medium | Low | High |
| Created by | Microsoft | Microsoft | UcedaVélez & Morana |

#### Benefits of PASTA

- Adaptable — applies to diverse organisational contexts and risk tolerances
- Aligns with compliance requirements by mapping controls to identified risks
- Promotes collaboration across stakeholders (developers, architects, security, business)
- Most comprehensive and systematic process of the common frameworks
- Output directly feeds into risk registers and security investment decisions

#### Application Scenario

Online banking platform (APAC region) — PASTA approach:
1. **Objectives** — Protect customer transaction data; comply with regional financial regulations
2. **Technical Scope** — Cloud-hosted banking platform, mobile apps, backend APIs, customer database
3. **Decompose** — Entry points: mobile app, web portal, API gateway; trust boundaries: internet → DMZ → internal services
4. **Threats** — External attackers targeting account takeover; insider threats; fraudulent transaction injection
5. **Vulnerabilities** — Weak MFA implementation, unpatched API libraries, misconfigured cloud storage
6. **Attacks** — Simulate credential stuffing attack chain; simulate API injection path; build attack trees
7. **Risk Treatment** — Implement adaptive MFA, enforce API input validation, apply cloud storage access policies

---

## Framework Comparison Reference

| Framework | Best When | Primary Output | Created By |
|-----------|----------|---------------|-----------|
| **MITRE ATT&CK** | Mapping real adversary TTPs; detection engineering; measuring control coverage | Annotated ATT&CK matrix with prioritised techniques | MITRE Corporation |
| **DREAD** | Quickly scoring a known list of vulnerabilities by relative severity | Prioritised numeric risk score per vulnerability | Microsoft |
| **STRIDE** | Systematically cataloguing threat types in software/system design | Threat checklist mapped to CIA + non-repudiation | Microsoft |
| **PASTA** | Comprehensive, risk-centric, business-aligned threat modelling | Risk-aligned countermeasure plan tied to business objectives | UcedaVélez & Morana |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **Asset** | Valuable resource an organisation depends on |
| **Attack Path** | Sequential chain of vulnerabilities an attacker exploits in order to reach an objective |
| **Attack Tree** | Hierarchical diagram decomposing an attacker's goal into strategies and specific tactics |
| **ATT&CK Navigator** | Open-source web tool for visualising and annotating the MITRE ATT&CK matrix |
| **DREAD** | Risk scoring model: Damage, Reproducibility, Exploitability, Affected Users, Discoverability |
| **MITRE ATT&CK** | Knowledge base of real-world adversary tactics, techniques, and procedures |
| **PASTA** | Process for Attack Simulation and Threat Analysis — 7-step risk-centric framework |
| **Risk** | Probability × Impact of a threat exploiting a vulnerability in an asset |
| **STRIDE** | Threat categorisation: Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation |
| **Sub-technique** | A specific variation of an ATT&CK technique (e.g., T1021.001 = RDP under Remote Services) |
| **Tactic** | ATT&CK high-level goal (the WHY) — e.g., Persistence, Lateral Movement |
| **Technique** | ATT&CK specific method to achieve a tactic (the HOW) — e.g., T1078 Valid Accounts |
| **Threat** | Potential event or actor that could exploit a vulnerability to cause harm |
| **Threat Modelling** | Systematic process of identifying, prioritising, and addressing security threats |
| **TTP** | Tactics, Techniques, and Procedures — the behavioural profile of a threat actor |
| **Trust Boundary** | A point in an architecture where data or control crosses from one trust level to another |
| **Vulnerability** | A weakness that can be exploited by a threat |

---

## Exam and Interview Revision

### Must Remember

- Risk = Threat + Vulnerability + Asset — remove any one element and risk collapses
- Attack tree: root = attacker's goal; child nodes = strategies; leaves = specific techniques
- ATT&CK Tactic = WHY (goal); Technique = HOW (method); Sub-technique = specific variant
- ATT&CK matrices: **Enterprise, Mobile, ICS**
- DREAD = Damage + Reproducibility + Exploitability + Affected Users + Discoverability / 5
- DREAD is **subjective** — needs standardised scoring rubrics to be reliable
- STRIDE letters and violated properties:
  - **S**poofing → Authentication
  - **T**ampering → Integrity
  - **R**epudiation → Non-repudiation
  - **I**nformation Disclosure → Confidentiality
  - **D**enial of Service → Availability
  - **E**levation of Privilege → Authorisation
- PASTA = 7 steps; risk-centric; most business-aligned; best for enterprise threat modelling
- Email anti-spoofing controls: DMARC + DKIM + SPF (all three work together)

### Common Interview Questions

| Question | Key Points |
|----------|-----------|
| What is the difference between a threat and a vulnerability? | Threat = external actor/event; vulnerability = internal weakness. Threat exploits vulnerability. |
| What does STRIDE stand for? | Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege |
| What security property does Repudiation violate? | Non-repudiation. Controls: audit logging, digital signatures, timestamping |
| How does MITRE ATT&CK differ from STRIDE? | ATT&CK maps real-world observed adversary behaviour; STRIDE is a design-time theoretical categorisation. ATT&CK is descriptive; STRIDE is prescriptive. |
| How would you use ATT&CK Navigator for a threat model? | Select relevant threat groups and platforms; highlight techniques; annotate with scores; export for reporting |
| What is the main weakness of DREAD? | Subjectivity — without standardised rubrics, scores vary significantly between analysts |
| When would you use PASTA over STRIDE? | PASTA when you need to align threat modelling with business objectives and risk tolerance; STRIDE for rapid design-time threat categorisation in software development |
| What is a trust boundary? | A point in an architecture where data or control crosses between different trust levels — a primary place to apply controls and conduct threat analysis |
