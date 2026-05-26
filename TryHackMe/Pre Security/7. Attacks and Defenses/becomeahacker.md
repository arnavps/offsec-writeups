# Become a Hacker

## Overview

Offensive security is the practice of proactively testing systems by attempting to break into them — with authorisation — to identify weaknesses before real attackers can exploit them. This room introduces the core terminology, mindset, and approach of ethical hacking. It bridges the foundational knowledge from the Pre Security path into the attacker's perspective.

---

## Topics Covered

- What offensive security is
- Core offensive security terminology
- The hacker mindset
- What attackers target and why
- Key concepts: scope, vulnerabilities, exploits, enumeration

---

## Key Concepts

### What is Offensive Security?

Offensive security involves authorised attempts to compromise systems, applications, and networks to identify vulnerabilities before malicious actors do. The goal is not to cause harm — it's to find weaknesses and report them so they can be fixed.

This is distinct from malicious hacking only by one thing: **authorisation**. Everything an ethical hacker does is within a defined scope and with explicit permission.

---

### Core Offensive Security Approaches

**Penetration Testing**
A structured, time-boxed security assessment where an authorised tester attempts to identify and exploit vulnerabilities within a defined scope. The output is a report detailing findings, their severity, and remediation recommendations.

**Red Teaming**
A more advanced, adversary-simulation exercise. Red teams operate with minimal constraints and simulate a real threat actor — testing not just technical controls but also people and processes. Broader in scope and longer in duration than a standard pentest.

---

### The Hacker Mindset

Thinking like an attacker means approaching systems with curiosity and scepticism:

- **Ask questions** — don't assume a feature works as intended. Ask "What if it doesn't?"
- **Test the unexpected** — try inputs and actions the developers didn't anticipate
- **Chain small weaknesses** — a minor flaw alone may be harmless, but combined with others it can become critical
- **Think like an adversary** — ask "How would a malicious actor approach this target?"

---

### What Attackers Target

Once an attacker gains access to a system or application, they look for:

| Target | Description |
|---|---|
| Credentials | Usernames and passwords that unlock further access |
| Sensitive functionality | Features that modify data, view restricted content, or trigger privileged processes |
| User data | Personal information that can be stolen, sold, or abused |
| Administrative features | High-privilege functionality for managing users, settings, or the entire application |
| Further attack opportunities | Authenticated access often exposes additional vulnerabilities for deeper compromise |

Valid credentials are particularly valuable — they provide legitimate-looking access that is harder to detect and can unlock large portions of an application.

---

## Important Terminology

| Term | Definition |
|---|---|
| Offensive Security | Authorised testing of systems to find vulnerabilities before attackers do |
| Penetration Test | A structured, scoped assessment to identify and exploit vulnerabilities |
| Red Teaming | An adversary simulation exercise testing technical and human defences |
| Vulnerability | A weakness or flaw in a system, application, or configuration |
| Exploit | A technique that takes advantage of a vulnerability to achieve a specific outcome |
| Scope | The defined boundaries of what is permitted during a security engagement |
| Enumeration | Collecting information about a system — users, services, configurations — to identify weak points |
| Credentials | Login details (username + password) that authenticate a user |
| Authentication | The process of verifying identity before granting access |
| Dictionary Attack | Using a predefined wordlist to guess passwords or usernames |

---

## Real-World Relevance

- Penetration testing is a standard requirement in many compliance frameworks (PCI-DSS, ISO 27001, SOC 2)
- Red team engagements test whether an organisation's detection and response capabilities work — not just whether vulnerabilities exist
- Enumeration is the foundation of every offensive engagement — you can't exploit what you haven't found
- Dictionary attacks against weak passwords are one of the most common and successful initial access techniques
- Chaining vulnerabilities is how most real-world breaches work — attackers rarely find a single critical flaw; they combine smaller issues into a full compromise path

---

## Key Learnings

- Offensive security is authorised, structured, and goal-oriented — the only difference from malicious hacking is permission
- Penetration testing and red teaming serve different purposes — pentests find vulnerabilities, red teams test the entire defence
- The hacker mindset is about questioning assumptions and testing edge cases
- Attackers prioritise credentials, sensitive data, and administrative access
- Scope defines what is and isn't permitted — operating outside scope is unethical and potentially illegal

---

## Additional Notes

- Rules of Engagement (RoE) documents define the scope, permitted techniques, and escalation procedures for a security engagement
- Bug bounty programmes are a form of authorised offensive security where researchers are rewarded for responsibly disclosing vulnerabilities
- Common offensive security certifications: OSCP (Offensive Security Certified Professional), CEH, eJPT
- Tools used in offensive security: `nmap`, `burpsuite`, `metasploit`, `gobuster`, `hydra`, `sqlmap`

---

## Conclusion

Offensive security is about understanding how systems fail so they can be made more resilient. The hacker mindset — questioning assumptions, testing the unexpected, and chaining weaknesses — is a skill that develops with practice. The terminology and concepts introduced here form the vocabulary of ethical hacking and will appear throughout every offensive security topic that follows.
