# Security Principles

## Overview

Before a system can be called secure, it needs to be evaluated against a defined set of principles. Security is not absolute — it is always measured in context: who is the adversary, what are they after, and what controls are appropriate to counter them? This room covers the foundational security models, principles, and frameworks that underpin how the industry thinks about and designs secure systems — from the CIA triad through formal security models to modern approaches like Zero Trust.

---

## Topics Covered

- The CIA Triad: Confidentiality, Integrity, Availability
- Extended properties: Authenticity and Nonrepudiation
- The Parkerian Hexad
- The DAD Triad: Disclosure, Alteration, Destruction
- Security models: Bell-LaPadula, Biba, Clark-Wilson
- Defence-in-Depth
- ISO/IEC 19249 architectural and design principles
- Trust models: Trust but Verify vs Zero Trust
- Vulnerability, Threat, and Risk definitions

---

## Key Concepts

### Knowing Your Adversary

Security is always relative to a threat model. The controls appropriate for protecting a personal laptop differ entirely from those protecting a server containing classified government data. Before designing security controls, you must understand:

- Who the adversary is and what their capabilities are
- What assets are being protected and their value
- What attack methods the adversary is likely to use

Security without a defined adversary and threat model produces arbitrary controls that may be over- or under-engineered for the actual risk.

---

### The CIA Triad

The CIA Triad is the foundational model for evaluating the security of any system. Every security control maps back to protecting one or more of these three properties.

| Property | Definition |
|----------|-----------|
| **Confidentiality** | Only authorised persons or recipients can access the data |
| **Integrity** | Data cannot be altered without detection; any alteration is detectable |
| **Availability** | Systems and services are accessible and functional when needed |

All three must be balanced. Locking data behind 50 layers of access control preserves confidentiality and integrity but may destroy availability. Security design requires trade-offs between these properties.

---

### Extended Properties: Authenticity and Nonrepudiation

Beyond the core triad, two additional properties are essential for many real-world systems:

| Property | Definition |
|----------|-----------|
| **Authenticity** | The data, document, or message genuinely originates from the claimed source — it is not forged or counterfeit |
| **Nonrepudiation** | The originating party cannot later deny that they created or sent the data |

These two are tightly coupled. Authenticity confirms who created something; nonrepudiation prevents them from denying it later. Both are critical in legal, financial, medical, and transactional contexts — digital signatures are a primary technical mechanism implementing both.

---

### The Parkerian Hexad

Proposed by Donn Parker in 1998, the Parkerian Hexad extends the CIA Triad to six security elements:

| Element | Description |
|---------|-------------|
| **Availability** | System or data is accessible when needed |
| **Utility** | The information is in a usable form — availability alone is insufficient if the data cannot be processed |
| **Integrity** | Data has not been altered |
| **Authenticity** | Data is from the claimed source |
| **Confidentiality** | Data is only accessible to authorised parties |
| **Possession** | The authorised party retains control over the data — it has not been stolen, copied, or seized |

**Utility vs Availability distinction:**
A user who has their encrypted laptop but has lost the decryption key still has *availability* (the device is physically present) but has lost *utility* (the data is inaccessible in a usable form).

**Possession vs Confidentiality distinction:**
An attacker who copies a backup drive has violated *possession* even if the data remains encrypted (confidentiality maintained). A ransomware attack that encrypts files also violates possession — the owner can no longer control their own data.

---

### The DAD Triad

The DAD Triad is the adversarial counterpart to CIA — each element represents an attack against the corresponding CIA property:

| DAD | Opposes | Example |
|-----|---------|---------|
| **Disclosure** | Confidentiality | Attacker steals and publicly dumps patient medical records |
| **Alteration** | Integrity | Attacker modifies patient records, causing wrong treatment to be administered |
| **Destruction / Denial** | Availability | Attacker takes a paperless medical system's database offline — the entire facility cannot operate |

Understanding DAD helps frame security objectives: preventing disclosure = protecting confidentiality, preventing alteration = protecting integrity, preventing denial = protecting availability.

---

### Security Models

Formal security models provide mathematical and logical frameworks for enforcing security properties in systems.

#### Bell-LaPadula Model

**Goal: Confidentiality**

Designed for environments with formal security clearance levels (e.g. government/military). Subjects and objects are assigned security levels.

| Rule | Name | Description |
|------|------|-------------|
| Simple Security Property | "No read up" | A subject at a lower security level cannot **read** an object at a higher security level |
| Star Security Property | "No write down" | A subject at a higher security level cannot **write** to an object at a lower security level |
| Discretionary-Security Property | Access matrix | An access control matrix defines explicit read/write permissions |

**Summary:** "Write up, read down" — you can share data upward (to higher clearance) and receive data from below (lower clearance), but you cannot read above your level or write to levels below yours.

**Limitation:** Not designed for file-sharing scenarios or systems requiring bidirectional data flow between levels.

#### Biba Model

**Goal: Integrity**

Biba is the integrity counterpart to Bell-LaPadula. Subjects and objects are assigned integrity levels.

| Rule | Name | Description |
|------|------|-------------|
| Simple Integrity Property | "No read down" | A higher-integrity subject should not **read** from a lower-integrity object (contamination risk) |
| Star Integrity Property | "No write up" | A lower-integrity subject should not **write** to a higher-integrity object (corruption risk) |

**Summary:** "Read up, write down" — this is the inverse of Bell-LaPadula, which should be expected since they protect different properties.

**Limitation:** Does not address insider threats — a trusted subject can still damage integrity through authorised actions.

#### Clark-Wilson Model

**Goal: Integrity**

Clark-Wilson uses a different approach to integrity, focusing on well-formed transactions and separation of duties:

| Concept | Acronym | Description |
|---------|---------|-------------|
| Constrained Data Item | CDI | Data whose integrity must be preserved |
| Unconstrained Data Item | UDI | All other data — user input, external data — not under integrity control |
| Transformation Procedure | TP | Programmed operations that maintain CDI integrity (the only authorised way to modify a CDI) |
| Integrity Verification Procedure | IVP | Procedures that verify the validity and consistency of CDIs |

The model ensures that data can only be modified through controlled, validated procedures — not directly. It enforces **separation of duties** and **well-formed transactions**, making it well-suited to commercial environments (banking, accounting) where both confidentiality and integrity matter.

---

### Defence-in-Depth

Defence-in-Depth (also called Multi-Level Security) is the principle of layering multiple independent security controls so that a failure or bypass of any single control does not result in a complete compromise.

**Analogy:** A locked drawer in a locked room, in a locked apartment, in a gated building with security cameras. Each layer is independent — an attacker must defeat all of them.

In practice this means:
- Firewall at the network perimeter
- IDS/IPS monitoring internal traffic
- Endpoint protection on each host
- Application-layer security controls
- Data encryption at rest and in transit
- User authentication and access controls
- Audit logging and monitoring

No single layer is expected to stop everything. The combination slows attackers down, raises the cost of attack, and increases the chance of detection.

---

### ISO/IEC 19249 Principles

ISO/IEC 19249:2017 is an international standard providing a catalogue of architectural and design principles for building secure products and systems.

#### Five Architectural Principles

| Principle | Description |
|-----------|-------------|
| **Domain Separation** | Related components are grouped into domains, each with its own security attributes. Example: OS kernel runs in ring 0 (privileged); user applications run in ring 3 (unprivileged) — they cannot directly access kernel space |
| **Layering** | Systems are structured into abstract layers; security policies can be enforced independently at each layer. Example: OSI model layers each providing services to the layer above with defined interfaces |
| **Encapsulation** | Internal implementation details are hidden; interaction occurs only through defined interfaces (APIs, methods). Prevents direct data manipulation and enforces valid operations only |
| **Redundancy** | Duplicate systems or data ensure availability and integrity. Example: RAID 5 — if one drive fails, data remains available; parity detects corruption |
| **Virtualization** | Hardware shared among multiple isolated OS instances. Provides sandboxing, security boundaries, and safe malware analysis environments |

#### Five Design Principles

| Principle | Description |
|-----------|-------------|
| **Least Privilege** | Grant the minimum permissions necessary to perform a task — nothing more. A user who needs to view a document gets read access only, not write |
| **Attack Surface Minimisation** | Reduce the number of vulnerabilities and entry points available to an attacker. Example: disable unused services, close unnecessary ports, remove unneeded software |
| **Centralised Parameter Validation** | All user and external input must be validated. Centralise this validation in a single library or system to ensure consistency and prevent bypass |
| **Centralised General Security Services** | Security services (authentication, authorisation, logging) should be centralised. Prevents inconsistent implementations and makes policy enforcement manageable. Requires high availability to avoid becoming a single point of failure |
| **Preparing for Error and Exception Handling** | Systems must be designed to **fail safe**. If a firewall crashes, it should block all traffic — not allow all traffic. Error messages must not leak sensitive information (no stack traces, no memory dumps visible to users) |

---

### Trust Models

#### Trust but Verify

Traditional security posture. Entities that have been granted trust (e.g. an authenticated internal user) are allowed to operate, but their activity is **logged and monitored**. Verification happens through log review and automated detection systems (IDS, proxy, SIEM).

**Limitation:** Relies on logging being comprehensive and reviewed. It is not feasible to manually verify every action — automated monitoring is required. Trust is still extended first; verification is reactive.

#### Zero Trust

Modern security model. Trust is treated as a vulnerability — it is never granted automatically, regardless of network location or device ownership.

- No distinction between internal and external network
- Every access request is authenticated and authorised, every time
- Assumes breach — limits lateral movement by containing damage
- **Microsegmentation** is a key implementation: network segments are reduced to individual hosts; communication between any two hosts requires explicit authentication and access control checks

**Comparison:**

| Model | Trust Granted | Verification | Insider Threat Handling |
|-------|--------------|-------------|------------------------|
| Trust but Verify | After authentication | Retrospective (log review) | Partial — logs catch it post-facto |
| Zero Trust | Never automatically | Continuous, real-time | Strong — every action requires authorisation |

---

### Vulnerability, Threat, and Risk

These three terms are related but distinct — using them interchangeably is inaccurate.

| Term | Definition |
|------|-----------|
| **Vulnerability** | A weakness in a system, software, or process that can be exploited |
| **Threat** | A potential danger associated with a vulnerability — what could happen if it is exploited |
| **Risk** | The likelihood that a threat actor will exploit a vulnerability, combined with the impact on the business if they do |

**Example:**
- **Vulnerability:** An unpatched web server running an old version of Apache with a known RCE bug
- **Threat:** An attacker could exploit this bug to gain shell access to the server
- **Risk:** High — the server is internet-facing, the exploit is publicly available, and the server hosts customer payment data

Understanding this distinction is fundamental to risk-based security decision making — you cannot prioritise what to fix without understanding both the likelihood and the impact.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| CIA Triad | Confidentiality, Integrity, Availability — the three core security properties |
| DAD Triad | Disclosure, Alteration, Destruction — attacks against CIA |
| Nonrepudiation | The originating party cannot deny creating or sending data |
| Parkerian Hexad | Six-element security model: CIA + Utility, Authenticity, Possession |
| Bell-LaPadula | Formal model for confidentiality — "no read up, no write down" |
| Biba | Formal model for integrity — "no read down, no write up" |
| Clark-Wilson | Integrity model using CDIs, TPs, and IVPs — enforces well-formed transactions |
| Defence-in-Depth | Multiple independent security layers |
| Least Privilege | Grant only the minimum permissions required for a task |
| Zero Trust | Never trust, always verify — no implicit trust based on location or ownership |
| Microsegmentation | Zero Trust implementation where each host is its own network segment |
| Attack Surface | The total set of points where an attacker could attempt to enter or extract data |
| Fail Safe | A system design where failure defaults to a secure state (e.g. firewall blocks all on crash) |

---

## Workflow / Process

Security principle application follows a consistent pattern:

```
Define the adversary and threat model
        |
        v
Identify assets and their required security properties (CIA + extensions)
        |
        v
Identify vulnerabilities and map to threats and risks
        |
        v
Select appropriate security model(s) for the system type:
  Confidentiality focus --> Bell-LaPadula
  Integrity focus       --> Biba or Clark-Wilson
        |
        v
Apply architectural principles (ISO/IEC 19249):
  Domain separation, layering, encapsulation, redundancy, virtualisation
        |
        v
Apply design principles:
  Least privilege, attack surface minimisation, centralised validation,
  centralised security services, fail-safe error handling
        |
        v
Determine trust model:
  Trust but Verify (with logging and automated monitoring)
  or Zero Trust (continuous authentication and authorisation)
        |
        v
Implement Defence-in-Depth:
  No single control is the last line of defence
```

---

## Real-World Relevance

- The CIA Triad is the lens through which every security assessment, pentest finding, and security control is evaluated — every risk has a CIA impact
- Bell-LaPadula directly models the access control requirements in classified government and military systems
- Clark-Wilson's separation of duties concept is implemented in financial systems, SOC workflows (analyst cannot both detect and close an alert without review), and change management processes
- Zero Trust is the dominant enterprise security architecture model — major cloud providers (AWS, Azure, Google) and NIST (SP 800-207) have published Zero Trust frameworks and implementation guides
- Least Privilege is one of the most consistently violated principles in practice — over-privileged accounts and service accounts are a primary attack vector in lateral movement and privilege escalation
- Attack Surface Minimisation is directly relevant in every penetration test — reducing exposed services, ports, and interfaces limits the attack options available to the tester
- Fail-safe error handling is a specific check in secure code review — improper error handling that leaks stack traces, credentials, or internal paths is a common finding in web application assessments
- Understanding Vulnerability vs Threat vs Risk is mandatory for communicating findings to management — technical staff focus on vulnerabilities, but business decisions are made on risk

---

## Key Learnings

- Security must be designed against a defined adversary — context determines appropriate controls
- CIA (Confidentiality, Integrity, Availability) is the foundation; DAD (Disclosure, Alteration, Destruction) are the attacks against it
- Parkerian Hexad adds Utility, Authenticity, and Possession to the CIA framework
- Bell-LaPadula = confidentiality model ("no read up, no write down"); Biba = integrity model ("no read down, no write up")
- Clark-Wilson enforces integrity through controlled transformation procedures rather than access level rules
- Defence-in-Depth means no single security control is the last line of defence
- ISO/IEC 19249 provides five architectural principles (domain separation, layering, encapsulation, redundancy, virtualisation) and five design principles (least privilege, attack surface minimisation, centralised validation, centralised security services, fail-safe handling)
- Zero Trust treats trust as a vulnerability — every access request requires authentication and authorisation regardless of origin
- Vulnerability = weakness; Threat = potential harm; Risk = likelihood × impact

---

## Additional Notes

- The Bell-LaPadula and Biba models are complementary — a complete system often needs to address both confidentiality and integrity, requiring elements of both
- "Fail safe" vs "fail open" is a critical design decision: fail safe defaults to denying access on failure; fail open defaults to allowing — always prefer fail safe in security-critical systems
- Microsegmentation is increasingly implemented through software-defined networking (SDN) and cloud security groups rather than physical network hardware
- Nonrepudiation is technically achieved through digital signatures using asymmetric cryptography — the private key signs the data, proving origin and preventing denial
- The distinction between Availability (CIA) and Utility (Parkerian Hexad) is subtle but matters: an encrypted backup with a lost key is available but has no utility

---

## Conclusion

Security principles are the conceptual backbone of every security control, architecture decision, and risk assessment. The CIA Triad defines what needs protecting; the DAD Triad defines what attacks look like; formal models like Bell-LaPadula, Biba, and Clark-Wilson provide structured frameworks for enforcing confidentiality and integrity; ISO/IEC 19249 translates these into practical design and architectural guidance; and trust models determine how access decisions are made. Together, these principles form the vocabulary and logic that security professionals use to reason about, design, and evaluate secure systems.
