# The CIA Triad

## Overview

The CIA Triad is the foundational framework of cybersecurity. Every security decision, incident assessment, and defensive measure maps back to one or more of its three principles: Confidentiality, Integrity, and Availability. Understanding the triad isn't just about memorising definitions — it's a mindset that shapes how security professionals think about protecting and attacking systems.

---

## Topics Covered

- Confidentiality
- Integrity
- Availability
- Applying the CIA Triad to incident assessment

---

## Key Concepts

### Confidentiality

Confidentiality ensures that sensitive data is accessible only to authorised individuals. Unauthorised access to data is a confidentiality breach.

Consequences of a confidentiality failure:
- Financial loss (stolen payment data, trade secrets)
- Privacy violations (personal data exposure)
- Legal consequences (regulatory fines, lawsuits)

**Attack examples:** data exfiltration, credential theft, eavesdropping, unauthorised file access

---

### Integrity

Integrity ensures that data is not modified by unauthorised parties. Data that has been tampered with can no longer be trusted — and in some contexts, corrupted data can have dangerous real-world consequences.

Consequences of an integrity failure:
- Corrupted records or transactions
- Manipulated logs that hide attacker activity
- Altered configurations that introduce vulnerabilities

**Attack examples:** man-in-the-middle attacks, file tampering, database manipulation, log modification

---

### Availability

Availability ensures that systems and data are accessible to authorised users when needed. Even if data is confidential and intact, it has no value if it can't be accessed.

Consequences of an availability failure:
- Business downtime and revenue loss
- Disruption of critical services (healthcare, infrastructure)
- Reputational damage

**Attack examples:** Denial of Service (DoS), ransomware, hardware destruction, resource exhaustion

---

### The CIA Triad as a Security Mindset

The CIA Triad isn't just a set of definitions — it's a framework for assessing the impact of any security incident. When an incident occurs, security professionals ask:

- Was sensitive data exposed to unauthorised individuals? → **Confidentiality**
- Was data modified without permission? → **Integrity**
- Were systems or services unavailable when needed? → **Availability**

These questions guide the severity assessment and the appropriate response.

---

## Important Terminology

| Term | Definition |
|---|---|
| CIA Triad | The three core principles of cybersecurity: Confidentiality, Integrity, Availability |
| Confidentiality | Ensuring data is accessible only to authorised users |
| Integrity | Ensuring data is not modified by unauthorised parties |
| Availability | Ensuring systems and data are accessible when needed |
| Data Breach | An incident where confidential data is accessed without authorisation |
| Tampering | Unauthorised modification of data |
| DoS | Denial of Service — an attack that makes a system or service unavailable |

---

## Real-World Relevance

- Every attack can be classified by which CIA property it targets — ransomware attacks availability, data theft attacks confidentiality, MITM attacks integrity
- Incident response reports are structured around CIA impact — understanding which pillar was affected determines the response priority
- Security controls are designed to protect specific CIA properties: encryption protects confidentiality, hashing protects integrity, redundancy protects availability
- Compliance frameworks (GDPR, HIPAA, PCI-DSS) are built around CIA requirements — violations often map directly to one of the three pillars

---

## Key Learnings

- The CIA Triad defines what cybersecurity actually protects: data confidentiality, data integrity, and service availability
- All three pillars are equally important — a failure in any one can have serious consequences
- The triad is a practical assessment tool, not just a theoretical framework
- Attacks are designed to break one or more CIA properties; defences are designed to preserve them

---

## Additional Notes

- Some frameworks extend the CIA Triad with additional properties such as non-repudiation (proof that an action occurred) and authenticity
- The Parkerian Hexad expands on the CIA Triad by adding possession, authenticity, and utility
- In practice, security trade-offs often exist between CIA properties — for example, strong encryption improves confidentiality but can impact availability if key management fails

---

## Conclusion

The CIA Triad is the lens through which all of cybersecurity is viewed. Whether you're assessing an incident, designing a defence, or planning an attack simulation, the question always comes back to: what was the impact on confidentiality, integrity, and availability? Internalising this framework is the first step to thinking like a security professional.
