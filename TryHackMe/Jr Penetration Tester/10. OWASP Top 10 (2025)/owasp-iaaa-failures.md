# OWASP Top 10 2025: IAAA Failures

## Overview

This room covers three OWASP Top 10:2025 categories that relate to failures in how Identity, Authentication, Authorisation, and Accountability (IAAA) are implemented in web applications. These are among the most impactful vulnerability classes — failures here allow attackers to access other users' data, escalate privileges, or operate completely undetected.

Categories covered:
- **A01: Broken Access Control**
- **A07: Authentication Failures**
- **A09: Logging and Alerting Failures**

---

## Topics Covered

- The IAAA model and why each layer depends on the previous
- Authentication failures: causes and common patterns
- Broken access control: what it is and why it matters
- Logging and alerting failures: accountability and detection gaps

---

## Key Concepts

### The IAAA Model

IAAA is a layered model for how applications verify users and their actions. Each layer depends on the one before it — if a prior layer is absent or broken, the layers that follow cannot function correctly.

| Layer | What It Does |
|-------|-------------|
| **Identity** | A unique account (user ID, email) that represents a person or service |
| **Authentication** | Proving that identity — via password, OTP, passkey, certificate |
| **Authorisation** | Defining what that verified identity is allowed to do |
| **Accountability** | Recording and alerting on who did what, when, and from where |

**Why the order matters:** You cannot authorise someone you haven't authenticated. You cannot hold someone accountable if you have no record of what they did. A failure at any layer propagates forward — broken authentication means authorisation decisions are made on an unverified identity.

---

### A07: Authentication Failures

Authentication failures occur when an application cannot reliably verify or bind a user's identity. This gives an attacker a path to log in as someone else, forge a session, or bind a session to the wrong account.

#### Common Authentication Failure Patterns

| Pattern | Description |
|---------|-------------|
| **Username enumeration** | Application reveals whether a username exists (different error messages for wrong username vs wrong password) |
| **Weak or guessable passwords** | No complexity requirements, default credentials accepted, no account lockout or rate limiting |
| **Logic flaws in login/registration flow** | Flawed multi-step login, broken "forgot password" flows, registration bypass |
| **Insecure session or cookie handling** | Session tokens predictable, not invalidated on logout, transmitted over HTTP, not scoped with `HttpOnly`/`Secure` flags |

If any of these are present, an attacker can often authenticate as a different user or take over an active session without ever knowing the target's password.

#### Prevention

- Enforce strong password policies and implement account lockout after repeated failures
- Use rate limiting on login endpoints to prevent brute force
- Implement multi-factor authentication (MFA)
- Use secure session management: generate cryptographically random tokens, invalidate on logout, set `HttpOnly` and `Secure` cookie flags
- Return identical error messages for wrong username and wrong password — prevent username enumeration

---

### A01: Broken Access Control

Access control enforces what an authenticated user is **allowed to do**. Broken access control means the authorisation layer has failed — users can access resources or perform actions they should not be permitted to.

This is the **top-ranked** vulnerability in OWASP Top 10:2025, reflecting how widespread and impactful it is.

#### Common Access Control Failure Patterns

- Accessing another user's data by modifying a URL parameter or ID (Insecure Direct Object Reference — IDOR)
- Accessing admin functionality by navigating directly to an admin URL without a role check
- Elevation of privilege — acting as a higher-privileged role without authorisation
- Missing function-level access control — API endpoints enforce access at the UI but not server-side
- Forcing browsing to authenticated pages without being authenticated (missing session checks)

#### Prevention

- Enforce access control server-side, not just in the UI
- Deny access by default — only permit what is explicitly allowed
- Log and alert on access control failures
- Apply least privilege — users should only be able to access resources required for their role
- Validate ownership before returning or modifying data based on user-supplied IDs

---

### A09: Logging and Alerting Failures

Logging and alerting failures represent a breakdown in **accountability** — the final layer of IAAA. Without adequate logging and monitoring, attacks go undetected, breaches are not discovered in time, and forensic investigation is impossible after the fact.

This category matters not because it enables an attack directly, but because it removes the visibility needed to detect, contain, and recover from one.

#### Common Logging Failures

- Login failures, access control failures, and input validation errors are not logged
- Log data is not monitored or does not feed into a SIEM
- Logs do not contain enough context — missing timestamps, user IDs, IP addresses, or action details
- Alerts are not configured for suspicious patterns (repeated failures, privilege escalation attempts)
- Log files are not protected — an attacker who gains access can clear or modify them

#### Prevention

- Log all authentication events (successes and failures), access control decisions, and input validation failures
- Include sufficient context per log entry: timestamp, user identity, source IP, action, result
- Feed logs into a centralised SIEM for real-time monitoring and correlation
- Set up alerts for anomalous patterns — repeated login failures, access to admin endpoints, unusual data access volumes
- Protect log integrity — restrict write access to log files; use append-only or remote logging where possible

---

## Important Terminology

| Term | Meaning |
|------|---------|
| IAAA | Identity, Authentication, Authorisation, Accountability — layered security model |
| Authentication | Proving who you are |
| Authorisation | Defining what you are allowed to do |
| Broken Access Control | Authorisation failures that allow users to exceed their permitted access |
| IDOR | Insecure Direct Object Reference — accessing another user's data by manipulating an ID |
| Session token | A credential issued after authentication, used to identify a user across requests |
| Username enumeration | Ability to determine whether a username exists based on application responses |
| Rate limiting | Restricting the number of requests from a source in a time window |
| SIEM | Security Information and Event Management — centralised log collection and alerting |

---

## Workflow / Process

```
User sends a request to the application
        |
        v
Identity: Is there an account associated with this request?
        |
        v
Authentication: Has this identity been proven?
  No --> Reject with a generic error (do not reveal which part failed)
        |
        v
Authorisation: Does this identity have permission for this action?
  No --> Deny with a 403 response; log the attempt
        |
        v
Accountability: Log the action (who, what, when, from where, result)
        |
        v
Monitoring: SIEM processes the log; alert if pattern matches a detection rule
```

---

## Real-World Relevance

- Broken Access Control (A01) is the most commonly found vulnerability in web application security assessments — IDOR is a top finding in bug bounty programmes
- Authentication failures enable credential stuffing, account takeover, and session hijacking — major attack vectors in both targeted attacks and mass compromise campaigns
- Logging failures are consistently exploited by sophisticated attackers who rely on dwell time — staying undetected for weeks or months before the breach is discovered
- The average time to detect a breach remains measured in days to weeks in many organisations — adequate logging and alerting directly reduces this
- MFA bypasses (e.g. SIM swapping, OTP interception) attack the authentication layer specifically — demonstrating that even MFA can fail if poorly implemented
- IDOR vulnerabilities are common in APIs — REST APIs that expose numeric IDs in URLs without ownership checks are routinely tested in bug bounties

---

## Key Learnings

- IAAA is a dependency chain — each layer requires the previous one to function
- Authentication failures allow attackers to impersonate users or forge sessions
- Broken Access Control allows authenticated users to exceed their permitted access — most commonly via IDOR or missing server-side checks
- Logging and alerting failures eliminate the ability to detect and respond to attacks in progress
- Access control must be enforced server-side — never rely on the UI to restrict access
- Logs must contain enough context to reconstruct who did what and when

---

## Additional Notes

- Username enumeration is a subtle but significant weakness — timing attacks can also reveal usernames even when error messages are identical (different server response times for valid vs invalid accounts)
- Session tokens must be regenerated after successful login to prevent session fixation attacks
- A01 Broken Access Control and A07 Authentication Failures together form the most common pathway in web application compromise — broken authentication gets the attacker in, broken access control lets them go beyond their intended scope
- OWASP recommends that access control violations should return a `403 Forbidden` response, not `404 Not Found` — returning 404 for unauthorised resources is a form of security through obscurity that also makes logging harder

---

## Conclusion

The IAAA failure categories represent the most foundational web application security concerns. Getting identity, authentication, authorisation, and accountability right is not optional — each layer protects against a different class of attack. Broken Access Control being the top-ranked OWASP category for 2025 reflects a consistent industry-wide gap: applications frequently enforce access restrictions in the UI but forget to enforce them where it actually matters — the server.
