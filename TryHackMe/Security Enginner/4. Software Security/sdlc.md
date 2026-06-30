# SDLC — Secure Software Development Lifecycle

## Overview

The Software Development Lifecycle (SDLC) is the structured process organisations use to plan, create, test, and deliver software. A Secure SDLC (S-SDLC) integrates security practices and checkpoints into every phase of that process — rather than treating security as a final gate before release. This room introduces the phases of SDLC, the most widely used development methodologies, and the principle that shifting security left (earlier in the process) dramatically reduces the cost and impact of vulnerabilities.

The core argument is simple: a vulnerability caught during design costs almost nothing to fix; the same vulnerability discovered after a production breach can cost millions and cause lasting reputational damage.

---

## Topics Covered

- SDLC phases and their security considerations
- Waterfall vs Agile development methodologies
- Secure SDLC frameworks and models
- The concept of "Shift Left" security
- Security activities mapped to each development phase

---

## Key Concepts

### SDLC Phases

The traditional SDLC is divided into six phases. In a secure SDLC, each phase has specific security activities integrated into it.

| Phase | Description | Security Activities |
|-------|-------------|-------------------|
| Planning | Define scope, resources, timelines | Risk assessment, security requirements definition |
| Requirements | Gather functional and non-functional requirements | Document security requirements, define compliance needs |
| Design | Architecture decisions, data flows, technology choices | Threat modelling, security architecture review |
| Development | Writing and building the actual software | Secure coding standards, peer code review, SAST |
| Testing | Verifying the software works and is secure | Penetration testing, DAST, vulnerability scanning |
| Deployment & Maintenance | Release and ongoing operations | Hardening, patch management, incident response planning |

---

### Development Methodologies

**Waterfall:**
- Linear, sequential approach — each phase must be completed before the next begins
- Security is typically addressed only in the Testing phase, near the end
- Problems: vulnerabilities discovered late are expensive and slow to fix; no iteration possible
- Best suited to projects with fixed, well-understood requirements

**Agile:**
- Iterative approach — work is divided into short sprints (typically 2–4 weeks)
- Continuous delivery and continuous feedback throughout the lifecycle
- Security can be built into each sprint: security user stories, sprint-level security reviews
- The dominant methodology in modern software development

**DevOps and DevSecOps:**
- DevOps integrates development and operations to enable continuous delivery (CI/CD pipelines)
- DevSecOps extends this by integrating security tooling directly into the CI/CD pipeline — automated SAST on every commit, automated DAST on every build, dependency scanning on every package update
- The goal: security checks happen automatically and continuously, not as a final manual gate

---

### Shift Left Security

**Definition:** "Shift Left" refers to moving security activities earlier (to the left) in the development timeline. Earlier is cheaper, faster, and less disruptive.

**Why It Matters:** IBM's Systems Science Institute research demonstrated that the relative cost to fix a vulnerability rises dramatically at each stage:

| Stage Found | Relative Fix Cost |
|------------|-----------------|
| Design | 1x |
| Development | 5–10x |
| Testing | 10–25x |
| Production | 50–200x |

Integrating security into design and development (shift left) eliminates the majority of vulnerabilities before they become expensive to fix.

**Practical Shift Left Activities:**
- Threat modelling during architecture review (before any code is written)
- Security requirements in the product backlog alongside functional requirements
- Developer security training to prevent common mistakes at the source
- SAST tools integrated into the IDE (developers see vulnerabilities as they type)
- Pre-commit hooks that block commits containing hardcoded secrets or known vulnerability patterns

---

### Threat Modelling

**Definition:** A structured analysis performed during the design phase to identify potential threats, the assets they could affect, and the mitigations that should be built in.

**Common Frameworks:**

**STRIDE** (used by Microsoft):
- **S**poofing — can an attacker impersonate a user or system?
- **T**ampering — can data be modified in transit or at rest?
- **R**epudiation — can a user deny performing an action?
- **I**nformation Disclosure — can sensitive data be accessed by unauthorised parties?
- **D**enial of Service — can the service be made unavailable?
- **E**levation of Privilege — can a user gain capabilities they shouldn't have?

**DREAD** (risk rating model):
- **D**amage — how severe is the potential impact?
- **R**eproducibility — how reliably can the attack be repeated?
- **E**xploitability — how much skill/tooling is required?
- **A**ffected users — how many users are impacted?
- **D**iscoverability — how easy is the vulnerability to find?

**PASTA** (Process for Attack Simulation and Threat Analysis):
- A seven-stage risk-centric methodology for identifying and evaluating threats in the context of business objectives

---

### Security Requirements

Security requirements define what the software must do (or not do) from a security perspective. They should be captured alongside functional requirements.

**Examples of security requirements:**
- Passwords must be stored using bcrypt with a minimum cost factor of 12
- All API responses must not include sensitive fields not relevant to the request
- Session tokens must expire after 30 minutes of inactivity
- All user input must be validated server-side before processing
- All database queries must use parameterised statements

**Sources for security requirements:**
- OWASP Application Security Verification Standard (ASVS) — a comprehensive checklist of security requirements by level
- Industry standards: PCI-DSS (payment card data), HIPAA (healthcare data), GDPR (personal data)
- Threat model outputs — each identified threat generates a corresponding mitigation requirement

---

### Secure Coding Standards

**Definition:** Agreed rules for how code should be written to avoid introducing vulnerabilities. Applied during the development phase.

**Common elements:**
- Never hardcode credentials or API keys in source code
- Validate all user input on the server side (not just client side)
- Use prepared statements for all database interactions
- Apply the principle of least privilege to all service accounts and database connections
- Handle errors without exposing internal details to end users
- Use established cryptographic libraries — never implement custom encryption

---

## Workflow / Process

```
Planning:
  Define security requirements alongside functional requirements
  Assess regulatory compliance obligations (PCI-DSS, GDPR, HIPAA)
        |
        v
Design:
  Threat modelling (STRIDE/DREAD/PASTA)
  Data flow diagrams — identify where sensitive data moves
  Security architecture review — auth mechanisms, encryption, boundaries
        |
        v
Development:
  Developers follow secure coding standards
  IDE-integrated SAST (e.g., SonarLint, Semgrep)
  Peer code review with security checklist
  Pre-commit hooks blocking secrets and known patterns
        |
        v
Testing:
  SAST scan of full codebase
  DAST scan against running application
  Penetration test (manual + automated)
  Dependency vulnerability scanning (SCA)
        |
        v
Deployment:
  Hardening of server and container configurations
  Secrets managed via vault (not environment variables in plaintext)
  Security-focused deployment checklist
        |
        v
Maintenance:
  Continuous monitoring for new vulnerabilities
  Regular dependency updates and patch management
  Incident response plan in place
  Periodic re-assessment (annual penetration tests, code audits)
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SDLC | Software Development Lifecycle — the structured phases of software creation |
| S-SDLC | Secure SDLC — SDLC with integrated security activities |
| Shift Left | Moving security activities earlier in the development process |
| Threat Modelling | Structured identification of threats, assets, and mitigations during design |
| STRIDE | Microsoft threat modelling framework — Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation |
| SAST | Static Application Security Testing — analysing source code for vulnerabilities without running it |
| DAST | Dynamic Application Security Testing — testing a running application from the outside |
| SCA | Software Composition Analysis — scanning third-party libraries and dependencies for known vulnerabilities |
| ASVS | Application Security Verification Standard — OWASP's security requirements checklist |
| DevSecOps | Integrating security tooling into DevOps CI/CD pipelines |
| CI/CD | Continuous Integration / Continuous Deployment — automated build, test, and deployment pipeline |

---

## Real-World Relevance

- Most significant data breaches trace back to vulnerabilities that could have been caught during design or development — threat modelling would have identified them before any code was written
- GDPR Article 25 mandates "Data Protection by Design and by Default" — a legal requirement for S-SDLC practices in organisations handling EU citizen data
- DevSecOps adoption has grown significantly as organisations recognise that manual, end-stage security reviews cannot keep pace with rapid release cycles
- The 2020 SolarWinds supply chain attack highlighted the importance of securing the build pipeline itself — not just the application code

---

## Key Learnings

- Security integrated at design phase costs a fraction of security applied at production stage
- Threat modelling (STRIDE, DREAD, PASTA) is the primary security activity during design
- Waterfall relegates security to the end; Agile and DevSecOps distribute it throughout
- Security requirements should be specific, measurable, and verifiable — not vague aspirations
- SAST, DAST, and SCA are the three primary automated security testing categories in a secure SDLC

---

## Additional Notes

- OWASP SAMM (Software Assurance Maturity Model) provides a framework for assessing and improving an organisation's S-SDLC maturity
- BSIMM (Building Security In Maturity Model) tracks how real software security programmes are implemented across industries
- Microsoft Security Development Lifecycle (SDL) is one of the most documented and publicly available S-SDLC frameworks

---

## Conclusion

The Secure SDLC is not a product or a tool — it is a discipline applied consistently across every phase of software development. The return on investment is clear: shifting security left by integrating threat modelling, security requirements, secure coding standards, and automated testing into the development process fundamentally reduces the number of vulnerabilities that reach production. The organisations with the strongest security posture are those that treat security as a development practice, not a final audit.
