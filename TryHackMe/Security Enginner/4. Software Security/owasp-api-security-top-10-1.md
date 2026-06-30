# OWASP API Security Top 10 - Part 1

## Overview

OWASP (Open Worldwide Application Security Project) is a non-profit community focused on improving application security through open documentation, tools, and research. In 2019, OWASP published a dedicated list of the top 10 API vulnerabilities — distinct from the general OWASP Top 10 — recognising that APIs have unique attack surfaces. This room covers the first five principles, with each vulnerability examined through the lens of how it happens, its likely impact, and how to mitigate it.

Real-world API breaches demonstrate the stakes: the 2021 LinkedIn scrape (700M records), the 2022 Twitter breach (5.4M users via a zero-day in the Twitter API), and the 2021 PIXLR breach (1.9M users) all originated from API security failures.

---

## Topics Covered

- Broken Object Level Authorisation (BOLA)
- Broken User Authentication (BUA)
- Excessive Data Exposure
- Lack of Resources and Rate Limiting
- Broken Function Level Authorisation (BFLA)

---

## Key Concepts

### Vulnerability I — Broken Object Level Authorisation (BOLA)

**Definition:** BOLA (also known as IDOR — Insecure Direct Object Reference) occurs when an API endpoint retrieves or manipulates objects using identifiers without verifying whether the requesting user is authorised to access that specific object.

**Why it happens:** Access controls are often missing at the code level (model layer in MVC architecture). The endpoint accepts any identifier without checking ownership.

**Likely impact:** Unauthorised data access, data leakage, and in severe cases, complete account takeover. Exposure of user or subscriber data can cause significant financial and reputational damage.

**Example:**
```
GET /apirule1/users/{ID}
```
Without authorisation validation, any user can retrieve any employee record by changing the `{ID}` parameter.

**Mitigation:**
- Implement an authorisation mechanism based on user policies and hierarchies
- Verify that the logged-in user has permission to access the specific resource
- Use cryptographically random, unpredictable tokens (not sequential IDs)

---

### Vulnerability II — Broken User Authentication (BUA)

**Definition:** BUA occurs when an API endpoint allows authentication to be bypassed or abused — either because the authentication logic is incorrectly implemented or because security mechanisms (tokens, authorisation headers) are absent.

**Why it happens:** Developers may implement authentication incompletely — for example, validating email but not password in a login query. Attackers can then obtain valid tokens using only partial credentials.

**Likely impact:** Attackers can compromise authenticated sessions, impersonate legitimate users, and perform account takeover.

**Example:**
A login endpoint that queries the database using only the email field and ignores the password allows any attacker who knows a target's email to obtain a valid token:
```sql
SELECT * FROM users WHERE email = '$email'  -- password never checked
```

**Mitigation:**
- Enforce complex passwords with high entropy
- Never expose credentials in GET or POST requests
- Use strong JWT implementations with appropriate authorisation headers
- Implement MFA, account lockout, or CAPTCHA to mitigate brute force
- Never store passwords in plaintext — use strong hashing (bcrypt, Argon2)

---

### Vulnerability III — Excessive Data Exposure

**Definition:** Excessive data exposure occurs when an API returns more data than is necessary for the requested operation. Developers expose entire object properties and delegate filtration to the front-end, leaving sensitive data accessible to anyone intercepting the API response.

**Why it happens:** Generic data retrieval implementations (e.g., `to_json()`, `to_string()`) return all available fields. The assumption that front-end developers will filter the output is a security anti-pattern.

**Likely impact:** Attackers intercepting API traffic can extract sensitive fields — account numbers, phone numbers, access tokens — even if they never appear in the rendered UI. API responses sometimes include authentication tokens that can be reused against other endpoints.

**Example:**
An endpoint for displaying comments returns the entire comment object from the database — including internal metadata, user IDs, and other fields not intended for public display — instead of only the fields needed for the comment display.

**Mitigation:**
- Never rely on front-end developers to filter sensitive data
- Periodically review API responses to confirm only necessary data is returned
- Avoid generic serialisation methods that return all object properties
- Test API endpoints with automated and manual test cases to detect data leakage

---

### Vulnerability IV — Lack of Resources and Rate Limiting

**Definition:** This vulnerability exists when APIs impose no restrictions on the frequency or volume of client requests — allowing unlimited requests per second, unlimited file upload sizes, or unlimited use of resource-intensive endpoints.

**Why it happens:** Rate limiting is often overlooked during development. Developers implement functionality but forget to add controls that prevent abuse.

**Likely impact:** Denial of Service (DoS), financial loss (e.g., exhausting a purchased email quota), and damage to availability and brand reputation.

**Example:**
A password reset endpoint that sends a one-time password (OTP) by email, with no limit on how many OTPs can be requested, allows an attacker to script thousands of requests — exhausting the company's monthly email quota within seconds.

**Mitigation:**
- Use CAPTCHA to block automated requests
- Implement per-client rate limits with cooldown periods (e.g., one OTP request every 2 minutes)
- Define maximum payload sizes for all parameters (max string length, max array size)
- Return appropriate status codes and notify users when limits are exceeded

---

### Vulnerability V — Broken Function Level Authorisation (BFLA)

**Definition:** BFLA occurs when a lower-privileged user can access or execute functionality reserved for higher-privileged users — effectively bypassing function-level access controls.

**Why it happens:** Complex role hierarchies and vague separation between regular and administrative functions create gaps. Header-based access checks that can be set by the client (e.g., `isAdmin: 1`) are not real security controls.

**Likely impact:** Attackers can impersonate administrators, access sensitive data, or perform administrative actions — violating authorisation and non-repudiation principles.

**Example:**
An admin dashboard endpoint checks for a custom header `isAdmin=1` alongside an authorisation token. A non-admin user (HR) can forge this header in their request and receive the full employee dataset that should only be accessible to administrators.

**Mitigation:**
- Implement server-side role validation — check the user's actual role in the database, never trust client-supplied values
- Deny all access by default; explicitly allow only permitted operations
- Regularly audit API endpoints for function-level authorisation flaws, especially considering business logic and group hierarchies

---

## Important Terminology

| Term | Meaning |
|------|---------|
| BOLA | Broken Object Level Authorisation — accessing objects without ownership verification |
| IDOR | Insecure Direct Object Reference — the general vulnerability class underlying BOLA |
| BUA | Broken User Authentication — authentication logic that can be bypassed or abused |
| Rate Limiting | Restricting the number of requests a client can make in a given time period |
| BFLA | Broken Function Level Authorisation — lower-privileged users accessing admin functions |
| JWT | JSON Web Token — a signed token standard used for stateless authentication |
| MFA | Multi-Factor Authentication — requiring multiple proof factors for identity verification |

---

## Real-World Relevance

- **BOLA** is consistently the most reported vulnerability in bug bounty programmes — changing a numeric `id=` parameter is one of the first checks any researcher performs
- **BUA** flaws like incomplete SQL validation lead directly to account takeover — the basis of credential-based attacks
- **Excessive data exposure** is often discovered by intercepting mobile app API traffic with tools like Burp Suite, revealing fields that are hidden in the UI but present in the response
- **Rate limiting failures** are exploited in password spraying, OTP brute-forcing, and API scraping attacks
- **BFLA** is what allows attackers to access admin dashboards — a common finding in web application penetration tests

---

## Key Learnings

- API security requires explicit ownership validation on every object retrieval endpoint — not just authentication
- Never validate only part of the credential set (email without password)
- Filter data server-side — never trust the front-end to hide sensitive fields
- Rate limiting is a first-class security control, not an afterthought
- Access control must be enforced programmatically in the backend — client-supplied headers like `isAdmin` are trivially forgeable

---

## Conclusion

The first five OWASP API Security principles establish the foundation of secure API development. BOLA, BUA, excessive data exposure, lack of rate limiting, and BFLA are all failures of the authorisation and authentication layer — areas where developers commonly make assumptions about trust that attackers systematically test. Addressing these early in the development lifecycle prevents the majority of API-based data breaches documented in real-world incidents.
