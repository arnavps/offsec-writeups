# Become a Defender

## Overview

Defensive security is the practice of protecting systems, detecting threats, and responding to incidents. Where offensive security asks "how can this be broken?", defensive security asks "how do we prevent, detect, and recover from that?". Defenders work to maintain the CIA Triad — confidentiality, integrity, and availability — across the systems they protect. This room introduces the defender mindset, core defensive concepts, and the key principles that guide blue team work.

---

## Topics Covered

- What defensive security is
- The five core defensive concepts: Prevention, Detection, Mitigation, Analysis, Response
- The defender mindset
- Key defender principles
- Blue team terminology

---

## Key Concepts

### What is Defensive Security?

Defensive security involves understanding what needs to be protected, implementing controls to prevent attacks, monitoring for threats, and responding when incidents occur. Defenders work to:

- Gain visibility into systems and network activity
- Identify and address weak points before attackers exploit them
- Ensure systems remain available and protected
- Respond effectively when incidents happen

---

### The Five Core Defensive Concepts

| Concept | Description |
|---|---|
| Prevention | Implementing controls to stop attacks before they happen — firewalls, patching, access controls, antivirus |
| Detection | Monitoring systems and networks to identify suspicious or malicious activity through logs, alerts, and security tools |
| Mitigation | Taking action during an incident to limit damage — blocking traffic, isolating systems, disabling compromised accounts |
| Analysis | Investigating what happened, how it happened, and which systems were affected by reviewing logs and evidence |
| Response and Improvement | Recovering from the incident and improving defences to reduce the risk of recurrence |

These five concepts form a continuous cycle — not a one-time setup.

---

### The Defender Mindset

Effective defence requires understanding the attacker's perspective. Key principles:

**Threat anticipation**
Review the systems you protect and ask "What if?" — imagine realistic attack paths an adversary might take to reach their goal.

**Attack awareness**
Attacks follow recognisable stages. Studying common attack chains and frameworks (such as the MITRE ATT&CK framework) helps defenders anticipate and detect attacker behaviour.

**Risk prioritisation**
Not all systems carry equal risk. Defenders must identify high-value targets — systems that, if compromised, would cause the most damage — and prioritise their protection.

**Continuous adaptation**
Defence is not a one-time configuration. Threats evolve, new vulnerabilities emerge, and attacker techniques change. Defenders must continuously update their knowledge and controls.

---

### Thinking in Chains

Attackers rarely target a single system in isolation. They compromise one asset and pivot to the next, building toward their goal. Defenders must view their environment as an interconnected chain — a weakness in one link can expose the entire system.

This is why visibility across the entire environment matters. A defender who can only see part of the network will miss lateral movement between systems.

---

## Important Terminology

| Term | Definition |
|---|---|
| Defensive Security | The practice of protecting systems, detecting threats, and responding to incidents |
| Blue Team | The group of defenders responsible for protecting systems and responding to threats |
| Prevention | Stopping threats before they cause harm through controls and restrictions |
| Detection | Identifying threats or suspicious activity in networks and systems |
| Mitigation | Reducing or stopping the impact of a threat once identified |
| Analysis | Investigating an incident to understand what happened and how |
| Visibility | The ability to monitor activity across systems to spot potential issues |
| Threat | A potential danger — a hacker, malware, or misconfiguration — that could harm systems |
| Risk | The likelihood and potential impact of a threat successfully causing harm |
| Client Infrastructure | The networks, servers, devices, and applications belonging to an organisation that need protection |
| Lateral Movement | An attacker's technique of moving from one compromised system to others within the network |

---

## Real-World Relevance

- SOC (Security Operations Center) analysts perform detection and analysis as their primary function — monitoring alerts, investigating incidents, and escalating threats
- Incident response teams handle mitigation, analysis, and recovery — following structured playbooks to contain and remediate breaches
- The MITRE ATT&CK framework maps attacker techniques to defensive detections — widely used by blue teams to understand and detect attack behaviour
- Patch management is one of the most impactful prevention controls — the majority of successful attacks exploit known, unpatched vulnerabilities
- Log analysis is the foundation of detection — without logs, defenders are blind to what's happening on their systems
- Threat modelling is the formal practice of threat anticipation — identifying what could go wrong before it does

---

## Key Learnings

- Defensive security is built around five concepts: Prevention, Detection, Mitigation, Analysis, and Response
- Defenders must understand attacker behaviour to anticipate and detect threats effectively
- Risk prioritisation ensures the most critical assets receive the most protection
- Defence is continuous — threats evolve and defences must evolve with them
- Visibility across the entire environment is essential — partial visibility creates blind spots attackers can exploit

---

## Additional Notes

- Common blue team tools: SIEM (Splunk, Microsoft Sentinel), EDR (CrowdStrike, SentinelOne), IDS/IPS, vulnerability scanners
- Common defensive security certifications: CompTIA Security+, Blue Team Level 1 (BTL1), SOC Analyst certifications
- The NIST Cybersecurity Framework (CSF) organises defensive security around five functions: Identify, Protect, Detect, Respond, Recover — closely aligned with the concepts in this room
- Purple teaming combines offensive and defensive teams working together — red team attacks while blue team detects, improving both sides simultaneously

---

## Conclusion

Defensive security is not passive — it requires the same depth of understanding as offensive security, applied from the opposite direction. Knowing how attacks work, where systems are vulnerable, and how to detect and respond to threats is what separates an effective defender from someone who simply has security tools installed. The mindset, the principles, and the continuous cycle of improvement are what make defence work.
