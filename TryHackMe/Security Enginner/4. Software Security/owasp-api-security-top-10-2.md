# OWASP API Security Top 10 - Part 2

## Overview

This room continues the OWASP API Security Top 10 series by covering vulnerabilities six through ten. While the first five focused primarily on broken authorisation and authentication patterns, the second half of the list addresses security misconfigurations, injection flaws, asset management failures, and unsafe third-party integrations. Together these ten categories form the foundational checklist for API security assessments.

---

## Topics Covered

- Mass Assignment
- Security Misconfiguration
- Injection (SQL, Command, XSS via API)
- Improper Assets Management
- Insufficient Logging and Monitoring

---

## Key Concepts

### Vulnerability VI — Mass Assignment

**Definition:** Mass assignment occurs when an API automatically binds user-supplied input to internal object properties without filtering. A developer builds a user registration endpoint that accepts a JSON body and maps it directly to a user model — including fields like `isAdmin`, `role`, or `balance` that should never be settable by a user.

**Why It Happens:** Modern frameworks (Rails, Laravel, Django, Node/Express with body parsers) provide convenient automatic binding of request data to model properties. When developers use these features without explicitly defining which fields are writable, all model properties become writable.

**Likely Impact:** Privilege escalation (a regular user granting themselves admin status), financial manipulation (setting a wallet balance to an arbitrary value), or bypassing business logic controls entirely.

**Example:**
A user registration endpoint accepts:
```json
{
  "username": "attacker",
  "password": "secret123",
  "isAdmin": true
}
```
If the backend binds the entire request body to the user model, the `isAdmin: true` field is accepted and stored — creating an admin account via the public registration endpoint.

**Mitigation:**
- Explicitly define which request fields are bindable (whitelist approach, not blacklist)
- Never auto-bind request payloads to data models that contain privileged fields
- Use separate DTOs (Data Transfer Objects) for user input versus internal models
- Validate request schemas against a strict specification

---

### Vulnerability VII — Security Misconfiguration

**Definition:** Security misconfiguration is a broad category covering any case where a security-relevant setting has been left at an insecure default, disabled inappropriately, or improperly configured. This is the most common vulnerability class across all application types.

**Why It Happens:** APIs are often deployed with default configurations for speed and convenience. CORS policies are set too permissively. Debug endpoints are forgotten in production. Error handlers return full stack traces. HTTP instead of HTTPS is used on internal services.

**Likely Impact:** Information disclosure (stack traces, framework versions, internal paths), unauthorised access via overpermissive CORS, cross-origin attacks, exposure of debug or administrative interfaces.

**Common Misconfiguration Examples:**

| Misconfiguration | Risk |
|-----------------|------|
| CORS: `Access-Control-Allow-Origin: *` | Any origin can read API responses |
| Debug mode enabled in production | Stack traces and internal paths exposed to the client |
| Default admin credentials not changed | Trivial authentication bypass |
| HTTP used instead of HTTPS | Credential and token interception |
| Unnecessary HTTP methods enabled (TRACE, DELETE) | Attack surface expansion |
| Verbose error messages returned to client | Internal structure disclosure |

**CORS misconfigurations specifically:** The `Access-Control-Allow-Origin` header controls which origins can read API responses from a browser. Setting it to `*` or reflecting the `Origin` header without validation allows any website to make authenticated requests to the API on behalf of a visitor — violating the same-origin policy.

**Mitigation:**
- Maintain a hardening checklist for every API deployment
- Disable debug mode and remove debug endpoints before production deployment
- Implement strict CORS policies allowing only trusted domains
- Return generic error messages to the client; log full details server-side
- Enforce TLS/HTTPS across all API communications
- Apply principle of least privilege to API service accounts

---

### Vulnerability VIII — Injection

**Definition:** Injection vulnerabilities occur when user-supplied input is interpreted as part of a query or command — allowing attackers to modify the intended logic. APIs are susceptible to the same injection classes as traditional web applications: SQL injection, command injection, XSS, NoSQL injection, and more.

**Why It Happens:** Input is concatenated directly into queries or commands without sanitisation or parameterisation. Developers trust that API consumers will send clean input, or assume API usage is restricted to trusted clients.

**Likely Impact:** SQL injection can lead to full database read/write access, data exfiltration, or schema destruction. Command injection can lead to remote code execution on the server. XSS through API responses can attack end users consuming API data in a browser context.

**SQL Injection via API Example:**
An endpoint accepts a username parameter and constructs a query:
```
GET /apirule8/user/login?username=admin'--&password=anything
```
If the backend runs:
```sql
SELECT * FROM users WHERE username='admin'--' AND password='anything'
```
The `--` comments out the password check, and the query returns the admin user without password validation.

**Command Injection via API Example:**
An endpoint that runs a system command using user-supplied input:
```
GET /apirule8/ping?ip=127.0.0.1; cat /etc/passwd
```
If the backend executes `ping {ip}` directly, the semicolon allows appending an arbitrary command.

**Mitigation:**
- Use parameterised queries / prepared statements — never concatenate input into SQL
- Validate and sanitise all user input before use in any query or system call
- Use an ORM with built-in parameterisation
- Apply allowlisting for expected input formats (IP addresses, usernames, etc.)
- Run database accounts with minimum required privileges

---

### Vulnerability IX — Improper Assets Management

**Definition:** Improper assets management occurs when old, deprecated, or development API versions remain accessible and are not properly documented, monitored, or decommissioned. Organisations lose track of which API versions exist and which are still active in production.

**Why It Happens:** APIs evolve over time. New versions are deployed; old ones are supposed to be retired but often are not. Development and staging environments may be accessible from the internet. Internal documentation becomes stale. API versioning (v1, v2, beta) creates multiple simultaneous attack surfaces.

**Likely Impact:** Old API versions often lack the security patches and controls added to current versions. An attacker who finds `/api/v1/` (patched in `/api/v2/`) can exploit vulnerabilities that were fixed in the newer version but still present in the forgotten old one. Staging and dev environments often have weaker security postures.

**Example:**
```
/api/v1/account/details    ← old version, unauthenticated access allowed
/api/v2/account/details    ← current version, authentication required
```
An attacker requesting the v1 endpoint bypasses authentication controls that were added in v2.

**Mitigation:**
- Maintain an up-to-date inventory of all API versions, endpoints, and environments
- Decommission old API versions with documented sunset timelines
- Ensure development and staging environments are not publicly accessible
- Apply the same security controls to all active API versions
- Use API gateways with centralised version management
- Include API versioning in the scope of regular security assessments

---

### Vulnerability X — Insufficient Logging and Monitoring

**Definition:** Insufficient logging and monitoring occurs when an API fails to log security-relevant events, or when logs are not reviewed and acted upon in time to detect and respond to attacks. This is the last line of defence.

**Why It Happens:** Logging is often treated as an operational concern rather than a security one. Developers skip logging for error conditions, failed authentication attempts, or unusual request patterns. Even when logs exist, no one reviews them or sets up alerting.

**Likely Impact:** Attacks go undetected for extended periods. Breach investigations are severely hampered or impossible without logs. Regulatory requirements (GDPR, PCI-DSS) for breach notification timelines cannot be met. The OWASP API list notes that most breaches are discovered by third parties, not the affected organisation.

**What Should Be Logged:**

| Event | Why |
|-------|-----|
| Failed authentication attempts | Brute force detection |
| Access to sensitive endpoints | Authorisation audit trail |
| Input validation failures | Injection attempt detection |
| Rate limit violations | Enumeration and scraping detection |
| Unexpected HTTP methods or status codes | Anomaly detection |
| Changes to privileged account settings | Privilege escalation detection |

**Mitigation:**
- Log all authentication events (success and failure) with timestamps, source IPs, and affected accounts
- Log all access to sensitive or privileged endpoints
- Centralise logs in a SIEM or log aggregation platform
- Set up automated alerting for anomalous patterns (10+ failed logins in 60 seconds, etc.)
- Test logging completeness as part of security reviews
- Ensure logs are tamper-resistant (write-only, remote storage)

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Mass Assignment | Automatic binding of request data to model fields without field filtering |
| CORS | Cross-Origin Resource Sharing — HTTP headers controlling cross-origin browser requests |
| DTO | Data Transfer Object — an object used to transfer data between layers without exposing internal models |
| SQL Injection | Injecting SQL syntax into a query to alter its logic |
| Command Injection | Injecting OS commands into a server-side call that runs system commands |
| API Versioning | Maintaining multiple concurrent API versions (v1, v2, beta) |
| SIEM | Security Information and Event Management — centralised log collection and alerting |
| Attack Surface | The total set of accessible endpoints, parameters, and interfaces an attacker can interact with |

---

## Real-World Relevance

- **Mass assignment** has been exploited in real products — GitHub once allowed users to add their SSH key to any organisation by sending an `organisation_id` field in their key submission request
- **Security misconfiguration** and permissive CORS policies are among the most commonly found issues in API security assessments and bug bounty programmes
- **Injection** via API parameters is functionally identical to injection via HTML forms — the delivery mechanism is different but the vulnerability is the same
- **Improper assets management** explains why penetration testers always probe `/api/v1/`, `/v2/`, `/beta/`, and common versioning patterns — forgotten versions are consistently vulnerable
- **Insufficient logging** is why attack dwell times (average time between breach and detection) remain high — median is measured in months, not hours

---

## Key Learnings

- Mass assignment requires explicit field whitelisting — never bind entire request bodies to data models
- CORS `Access-Control-Allow-Origin: *` on authenticated APIs is a vulnerability, not a convenience setting
- Injection attacks against APIs are equally severe as those against traditional web applications — parameterise all queries
- Old API versions with different security postures are a primary penetration testing target
- Logging is a security control, not a debugging tool — treat it accordingly

---

## Additional Notes

- OWASP API Security Top 10 was published in 2019; a 2023 updated edition revised several categories and wording but the core principles remain the same
- The distinction between OWASP Top 10 (web applications) and OWASP API Security Top 10 matters — APIs have unique attack surfaces (mass assignment, asset management) that don't map cleanly to traditional web app vulnerabilities
- API security testing with tools like Postman, Burp Suite, and dedicated API fuzzing tools directly applies all ten of these categories

---

## Conclusion

The second half of the OWASP API Security Top 10 covers the implementation and operational gaps that make APIs vulnerable over their entire lifecycle. Mass assignment and injection arise during development. Security misconfiguration arises during deployment. Improper assets management arises as the API evolves and ages. Insufficient logging fails at detection and response. Addressing all ten principles requires integrating security across every stage of the API development lifecycle — design, implementation, deployment, and operations.
