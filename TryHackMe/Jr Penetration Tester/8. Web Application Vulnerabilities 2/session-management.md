# Session Management

## Overview

Session management is the mechanism web applications use to maintain state across the inherently stateless HTTP protocol. After a user authenticates, a session value is issued and used to track their identity and permissions on every subsequent request. This room teaches how sessions are created, tracked, expired, and terminated — and what happens when each of these phases is implemented incorrectly.

**Main objectives:** Understand the complete session management lifecycle, identify vulnerabilities at each lifecycle phase, enumerate session behaviour in a live application, and exploit a real misconfiguration involving LocalStorage token manipulation and IDOR.

**Skills introduced:** Session lifecycle analysis, cookie attribute auditing, cookie vs token-based session management, browser DevTools enumeration, LocalStorage manipulation, IDOR exploitation via token modification, and identifying excessive session lifetimes.

---

## Concepts Covered

### The IAAA Model and Sessions

**Definition:** IAAA (Identification, Authentication, Authorisation, Accountability) is the four-stage model that defines how web applications handle user identity and access.

**Why it matters:** Session management touches every IAAA stage — authentication triggers session creation, authorisation relies on session tracking, and accountability depends on sessions being logged.

| Stage | Role in Session Management |
|-------|---------------------------|
| **Identification** | User claims identity via username/email |
| **Authentication** | Identity is verified (password match → session created) |
| **Authorisation** | Session is checked against permissions on every request |
| **Accountability** | Actions are logged and tied to the session for audit trail |

---

### Session Management Lifecycle

**Definition:** A four-phase cycle describing how sessions are born, used, aged, and destroyed.

#### Phase 1: Session Creation
Sessions may be created before authentication (to track unauthenticated behaviour) or upon successful login. The mechanism of how session values are generated and transmitted is the highest-risk phase.

**Vulnerabilities at this phase:**
- **Weak session values:** Predictable or guessable values (e.g., base64 of the username). An attacker who reverse-engineers the creation logic can generate valid sessions for other accounts.
- **Controllable session values:** Tokens like JWTs that contain all information needed to create and verify themselves. If signature verification is absent or bypassed, the attacker forges their own tokens.
- **Session fixation:** Applications that create a session before login but do not rotate the session value post-authentication. An attacker who captures the pre-auth session ID can wait for the user to log in and gain access to their session.
- **Insecure session transmission:** In SSO flows, session material is transmitted between servers via browser redirects. Open redirect vulnerabilities can expose session tokens to attacker-controlled URLs.

#### Phase 2: Session Tracking
The session value is submitted with every request. The server performs a lookup to identify the user and their permissions.

**Vulnerabilities at this phase:**
- **Authorisation bypass (vertical):** A lower-privileged user accesses functionality reserved for admins.
- **Authorisation bypass (horizontal):** A user accesses the correct type of resource but for a different user (IDOR). The user is allowed to do the action — just not on that specific data object.
- **Insufficient logging:** Actions performed by a session are not logged, or only rejected actions are logged. In a session hijacking scenario, the actions appear legitimate — logging only failures misses the attack entirely.

#### Phase 3: Session Expiry
Sessions must have a finite lifetime. A session without expiry persists indefinitely after the user has stopped interacting.

**Vulnerabilities at this phase:**
- **Excessive session lifetime:** A banking app with a 30-day session is a significant risk. Session expiry should reflect the application's risk profile.
- **Missing geolocation binding:** Long-lived sessions should verify that the session is being used from a consistent location. A location change mid-session can indicate hijacking and should trigger re-authentication.

#### Phase 4: Session Termination
When a user logs out, the session must be invalidated both client-side (cookie/token deleted) and server-side (session record invalidated).

**Vulnerabilities at this phase:**
- **Missing server-side invalidation:** The client deletes the cookie, but the server still honours the old session value. An attacker who captured the session token retains access even after the victim logs out.
- **Token-based complexity:** For JWTs with embedded expiry, server-side invalidation is harder because the server is stateless. A blocklist approach is needed to reject unexpired but logged-out tokens.

---

### Cookie-Based vs Token-Based Session Management

**Cookie-based (traditional):**
- Session value stored in a browser cookie set via `Set-Cookie` response header
- Browser automatically attaches cookie to every request to the matching domain
- Security is enforced via cookie attributes

**Cookie Security Attributes:**

| Attribute | Effect | Security Benefit |
|-----------|--------|-----------------|
| `Secure` | Cookie only sent over HTTPS | Prevents interception over HTTP |
| `HttpOnly` | JavaScript cannot read the cookie | Prevents XSS-based cookie theft |
| `Expires` | Sets when the cookie is removed | Controls session lifetime |
| `SameSite` | Controls cross-site transmission | Mitigates CSRF attacks |

**Token-based (modern):**
- Token (commonly JWT) returned in the response body after login
- Stored in the browser's `LocalStorage` by client-side JavaScript
- Must be manually attached to requests as `Authorization: Bearer <token>`
- No automatic security protections — all responsibility on the developer

**Comparison:**

| Property | Cookie-Based | Token-Based |
|----------|-------------|------------|
| Automatically sent | Yes (by browser) | No (by JS code) |
| CSRF risk | Higher (auto-send) | Lower (CORS + no auto-send) |
| XSS risk | Lower (`HttpOnly`) | Higher (accessible via JS) |
| Cross-domain use | Harder (domain-locked) | Easier (JS can send anywhere) |
| Decentralised auth (SSO) | Complex | Natural fit |

---

## Methodology

```
1. Visit the application unauthenticated — observe if a session is already created
2. Create an account (lowest-privilege available)
3. Log in — capture the Set-Cookie headers and observe session format
4. Review cookie attributes (Secure, HttpOnly, SameSite, Expires)
5. Browse authenticated functionality — observe which cookies/tokens are sent with each request
6. Check LocalStorage and SessionStorage for token data
7. Test session expiry — remove cookie and retry requests
8. Test logout — note if old session values are honoured server-side
9. Attempt to modify LocalStorage values and observe access changes
10. Map the full lifecycle and document findings
```

---

## Practical Activities

### Activity 1 — Session Lifecycle Enumeration

**Objective:** Map how the application creates, uses, and terminates sessions before identifying exploitable weaknesses.

**Actions:**
1. Navigate to the application — no cookies present at this stage (unauthenticated tracking is not used)
2. Click Sign-Up — two account types observed: Student (open) and Lecturer (requires verification code)
3. Create a Student account and log in while monitoring browser DevTools → Network tab

**Findings from login response:**

```
Set-Cookie: session=<value>; HttpOnly
```

- Cookies are used for session management
- `HttpOnly` is set — JavaScript cannot read the cookie value
- Authenticated users gain access to additional functionality (modules, enrolled counts, tests)

**Session tracking observation:**
- The cookie is transmitted with every authenticated request
- The `Set-Cookie` header refreshes the same session value with an extended expiry — this indicates a persistent session
- Removing the cookie and retesting some endpoints: certain module data still loads (showing that not all content requires authentication)

**Session termination test:**
- Logout removes the cookie client-side
- Re-using the old session value results in HTTP 500 (Internal Server Error) — the session is invalidated server-side but the error handling is incorrect; an invalid session should return 401, not 500

**Why this matters:** The 500 error reveals that server-side termination is working but error handling exposes internal errors — a security misconfiguration worth reporting.

---

### Activity 2 — LocalStorage Analysis and Token Manipulation

**Objective:** Investigate whether application behaviour can be altered by modifying client-side token data.

**Actions:**
1. Open DevTools → Application → Local Storage
2. Observe the stored token data post-login

**LocalStorage contents observed:**
```json
{
  "userRole": "student",
  "id": 2,
  "username": "testuser"
}
```

**Manipulation 1 — Change `userRole` to `lecturer`:**
- Refresh the page
- Result: Additional navigation tabs appear (Lecturer tabs now visible)
- However, the actual data within those tabs is not returned — just the navigation changed

**Manipulation 2 — Change `role` value in `user` from `2` to `3`:**
- Same result — visual navigation change but no additional data returned

**Observation:** The navigation is driven by the client-side token values in LocalStorage, but the underlying API data still appears to be gated by the server-side session. This confirms a hybrid session model — **both cookie (for server auth) and LocalStorage token (for client-side UI decisions) are in use**.

---

### Activity 3 — IDOR via Token Manipulation (Exploitation)

**Objective:** Combine the LocalStorage token manipulation with the IDOR pattern to access data belonging to other users (Lecturer information accessible to a Student account).

**Finding:** The `id` field in LocalStorage directly controls which user's data is requested from the API. By changing the `id` to match a Lecturer account's ID, the API returns that lecturer's data — including information a Student should not be able to see.

**Why this works:** The server-side API does not verify that the authenticated student session matches the `id` being requested. The LocalStorage `id` is trusted as user input without an ownership check — a horizontal authorisation bypass (IDOR).

**Why it matters:** This is the distinction between vertical and horizontal bypasses in practice:
- The student is performing an action they are allowed to do (view profile data)
- But they are performing it on someone else's data (the lecturer's record)
- The server validates the action type but not the data ownership

---

## Observations and Analysis

**Mapped Lifecycle Summary:**

| Phase | Observation | Security Implication |
|-------|-------------|---------------------|
| Session Creation | Hybrid cookie + token model | Both components must be secured |
| Session Creation | New session per login | Not reusing old values — good |
| Session Creation | Session value appears random | No obvious guessing/enumeration risk |
| Session Tracking | Unauthenticated actions not tracked | Limited anonymous exposure |
| Session Tracking | Cookie sent with each request | Standard behaviour |
| Session Tracking | Some requests succeed without cookie | Incomplete authentication enforcement |
| Session Tracking | LocalStorage token values drive UI and API requests | Client-side token is a direct IDOR vector |
| Session Expiry | Client and server expiry do not match | Risk of phantom sessions |
| Session Expiry | Expiry time is excessively long | Extended hijack window |
| Session Termination | Logout removes cookie client-side | Client-side termination works |
| Session Termination | Old session returns 500 | Server-side termination works but error handling is wrong |

**Reportable vulnerabilities from this engagement:**
- Excessive session lifetime
- Horizontal authorisation bypass (IDOR) via LocalStorage `id` manipulation

**Requires further investigation:**
- Access controls on all API endpoints
- Accountability: are actions per session properly logged?

---

## Tools and Technologies Used

### Browser DevTools
- **Purpose:** Inspect cookies, LocalStorage, network traffic, and response headers
- **Common usage:** F12 → Application tab (cookies, LocalStorage), Network tab (requests/responses)
- **In this room:** Primary tool for session enumeration — observing Set-Cookie headers, reviewing LocalStorage token values, testing cookie removal

---

## Key Learnings

- The session management lifecycle has four distinct phases; vulnerabilities can exist at any or all of them
- Cookie-based and token-based management have fundamentally different security models — cookie security is enforced by the browser; token security relies entirely on developer discipline
- `HttpOnly` prevents XSS-based cookie theft but does not stop token manipulation in LocalStorage
- Horizontal authorisation bypass (IDOR) is harder to catch than vertical bypass because the user is performing a permitted action — just on the wrong data
- Client-side filtering (UI changes based on LocalStorage) is not a security control — the server must enforce all authorisation decisions
- Excessive session lifetime amplifies the impact of any session hijacking — the longer the window, the more damage is possible
- An invalid session returning 500 instead of 401 reveals server internals — a security misconfiguration to report

---

## Real-World Relevance

- **Penetration testing:** Session management is tested on every web application assessment. Cookie attribute auditing, session fixation testing, and post-logout session validity checks are standard items.
- **Bug bounty:** IDOR via client-side token manipulation is a frequent medium-to-high severity finding. Applications that trust LocalStorage data for API lookups without server-side ownership checks are consistently found.
- **Enterprise environments:** SSO implementations are particularly vulnerable to session fixation and insecure redirect vulnerabilities — the Oracle SSO example in the room is a real-world precedent.
- **Red team operations:** Persistent sessions with excessive lifetimes are ideal for maintaining access after initial compromise — capturing a long-lived session token provides access that survives password resets in some implementations.

---

## Things Worth Remembering

- Session lifecycle: Create → Track → Expire → Terminate (vulnerabilities at each stage)
- `HttpOnly`: protects cookie from JavaScript; `Secure`: HTTPS only; `SameSite`: CSRF mitigation
- Session fixation: pre-auth session not rotated post-auth = fixation risk
- LocalStorage values driven UI and API calls = client-side trust model = IDOR risk
- Vertical bypass = access higher privilege actions; Horizontal bypass = access same actions on other users' data
- Post-logout session reuse should return 401 (not 200, not 500)
- Log both accepted AND rejected actions — legitimate-looking hijacked sessions only appear in accepted action logs
- Excessive session lifetime = extended window for hijacking; bind long-lived sessions to location

---

## Conclusion

This room established session management as a multi-phase security problem that spans authentication, authorisation, and accountability. The practical work demonstrated that a hybrid cookie/LocalStorage model creates an IDOR attack path when the server trusts client-supplied identity values without verifying data ownership. The key takeaway for penetration testers is that session security cannot be evaluated by looking at authentication alone — every phase of the lifecycle must be tested, and client-side storage must always be treated as attacker-controllable input.
