# Session Management

## Overview

Session management bridges the gap between stateless HTTP and the stateful experience users expect from web applications. After authentication, a session tracks who the user is and what they are allowed to do across all subsequent requests. Failures in session management — weak session values, fixation, excessive lifetimes, insufficient termination — allow attackers to hijack accounts, escalate privileges, and maintain persistent access. This room covers the session management lifecycle, cookie vs token-based sessions, IAAA model context, and exploitation techniques.

---

## Topics Covered

- Session management lifecycle: creation, tracking, expiry, termination
- Cookie-based vs token-based session management
- IAAA model: Identification, Authentication, Authorisation, Accountability
- Session creation vulnerabilities: weak values, controllable values, session fixation
- Session tracking vulnerabilities: authorisation bypass, insufficient logging
- Session expiry issues: excessive lifetimes, location tracking
- Session termination issues: server-side invalidation
- Enumeration and exploitation methodology

---

## Key Concepts

### Session Management Lifecycle

HTTP is stateless — the server has no inherent way to know that two requests come from the same user. Sessions solve this by issuing a session value after authentication that is submitted with every subsequent request.

| Phase | Description | What goes wrong |
|-------|-------------|----------------|
| **Session Creation** | A session value is generated and issued to the user after authentication | Weak or predictable values, session fixation, insecure transmission |
| **Session Tracking** | The session value is sent with each request; server looks it up to identify the user and their permissions | Authorisation bypass, insufficient logging |
| **Session Expiry** | Sessions have a lifetime — expired sessions are rejected | Excessive lifetimes, no location-change detection |
| **Session Termination** | Logout action explicitly invalidates the session server-side | Sessions not actually invalidated server-side on logout |

---

### IAAA Model and Session Management

| Layer | Description | Session Management Role |
|-------|-------------|------------------------|
| **Identification** | User claims an identity (e.g. submits a username) | Establishes who is being authenticated |
| **Authentication** | Proof of identity (e.g. correct password) | Session creation occurs here — a session value is issued on success |
| **Authorisation** | Determines what the authenticated user can do | Session tracking — each request is checked against the user's permissions |
| **Accountability** | Recording what actions were taken | Logging all actions associated with each session |

---

### Cookie-Based vs Token-Based Session Management

| Aspect | Cookie-Based | Token-Based (JWT) |
|--------|-------------|-------------------|
| Transmission | Browser sends cookie automatically with every request | JavaScript must attach token as `Authorization: Bearer` header |
| Security protections | Cookie attributes: `Secure`, `HttpOnly`, `SameSite`, `Expires` | No automatic protections — must be implemented manually |
| CSRF risk | Vulnerable — browser auto-sends cookies in cross-site requests | Not vulnerable — tokens are not automatically attached cross-site |
| Decentralised apps | Cookies are domain-locked — difficult in multi-domain apps | Works well — token can be verified by any service with the public key |
| Session revocation | Server-side session can be invalidated immediately | Token validity is embedded in the token — revocation requires a blocklist |

#### Cookie Security Attributes

| Attribute | Effect |
|-----------|--------|
| `Secure` | Cookie only transmitted over verified HTTPS — not sent over HTTP or with cert errors |
| `HttpOnly` | Cookie cannot be read by client-side JavaScript — prevents XSS-based theft |
| `Expires` / `Max-Age` | Defines when the cookie becomes invalid |
| `SameSite` | Controls whether cookie is sent in cross-site requests — mitigates CSRF |

---

### Session Creation Vulnerabilities

#### Weak Session Values

If session values are generated using a predictable mechanism (e.g. Base64-encoding the username, using a sequential counter, or seeding from a timestamp), an attacker who understands the pattern can generate valid session values for other users without authenticating.

**Example:** A session that is simply `base64("admin")` = `YWRtaW4=` — trivially reversible.

#### Controllable Session Values (JWT-specific)

Some token formats embed all the information needed to both create and verify them. If signature verification is not enforced or the signing secret is weak, an attacker can forge their own valid token with modified claims (e.g. changing `"admin": 0` to `"admin": 1`).

#### Session Fixation

If a web application assigns a session value before authentication and does not rotate it after successful login, an attacker can:
1. Visit the application and record their pre-authentication session value
2. Trick the victim into authenticating with the same session value (e.g. via a crafted URL)
3. Use the known session value to access the now-authenticated session

**Fix:** Always issue a new session value upon successful authentication — never reuse the pre-authentication session.

#### Insecure Session Transmission

In SSO environments, the session material must travel from the authentication server to the application server via the user's browser. If the redirect URL after authentication can be controlled by an attacker (open redirect), the session token may be redirected to the attacker.

---

### Session Tracking Vulnerabilities

#### Authorisation Bypass

Two types:
- **Vertical bypass** — performing an action reserved for a more privileged role (e.g. a regular user accessing admin functionality)
- **Horizontal bypass** — performing a permitted action on another user's data (e.g. viewing another user's orders by manipulating an ID parameter)

Vertical bypasses are often blocked by path-based access control decorators. Horizontal bypasses require explicit server-side code to verify that the requesting user owns the data being accessed.

**Example:** If a session stores `role=2` and an attacker changes it to `role=3`, this is a vertical bypass. If `id=42` in a session can be changed to `id=43` to access another user's data, this is horizontal.

#### Insufficient Logging

Without adequate logging of both accepted and rejected actions linked to session identifiers, it is impossible to reconstruct what happened during a session hijacking incident. Logging only rejected actions is insufficient — hijacked sessions perform apparently legitimate actions.

---

### Session Expiry Issues

**Excessive session lifetimes:** A session should be treated like a timed ticket. A banking application needs much shorter session lifetimes than a webmail client. If sessions never expire or expire after very long periods, compromised sessions remain valid indefinitely.

**Location-based expiry:** For long-lived sessions (e.g. webmail), the session should be tied to the expected usage location. A sudden change in IP or geolocation should trigger re-authentication.

---

### Session Termination Issues

When a user logs out, the server must invalidate the session server-side. If the session is not invalidated server-side:
- An attacker who previously captured the session value can continue using it even after the legitimate user logs out
- The user has no way to revoke an attacker's access

For token-based sessions where the expiry is embedded in the token: implement a server-side blocklist and check tokens against it. Allow users to view and terminate all active sessions.

**Password reset:** Upon a successful password reset, all existing sessions should be terminated to prevent a hijacker who has a valid session from maintaining access after the password change.

---

## Practical — Session Lifecycle Mapping

When assessing a web application's session management, document each phase:

| Lifecycle Phase | What to Check |
|----------------|--------------|
| Session Creation | Is the session value random and sufficiently long? Is a new value issued after authentication? |
| Session Tracking | Are all endpoints checking both authentication and authorisation? Are actions logged with session IDs? |
| Session Expiry | How long does the session last? Is there location/IP binding for long-lived sessions? |
| Session Termination | After logout, can the old session value still be used? Are all sessions terminated on password reset? |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Session fixation | Attack where an attacker sets a known session value before authentication to hijack the resulting authenticated session |
| Vertical bypass | Accessing functionality reserved for a higher privilege level |
| Horizontal bypass | Performing an allowed action on data belonging to a different user |
| Session hijacking | Using a stolen or forged session value to impersonate a legitimate user |
| `HttpOnly` | Cookie attribute preventing JavaScript access — blocks XSS-based session theft |
| `SameSite` | Cookie attribute controlling cross-site request inclusion — mitigates CSRF |
| Session blocklist | Server-side list of invalidated tokens — required for token-based session revocation |
| CSRF | Cross-Site Request Forgery — tricking a browser into making authenticated requests |

---

## Real-World Relevance

- Oracle's SSO solution had a session transmission vulnerability allowing redirect-based session hijacking — real-world examples of insecure SSO transmission exist in major enterprise products
- Horizontal authorisation bypasses (IDOR) are the most commonly reported class of vulnerability in bug bounty programmes — insufficient session-to-data ownership checks are pervasive
- Session fixation is commonly found in PHP applications that do not call `session_regenerate_id(true)` after successful login
- JWT manipulation attacks (changing `admin: 0` to `admin: 1`) are a standard web assessment finding — covered in depth in the JWT Security room
- Long-lived sessions without geographic binding are a common finding in financial services applications where regulators increasingly require session risk controls

---

## Key Learnings

- Sessions bridge stateless HTTP with stateful user interactions — every authenticated action depends on session management being correct
- Cookie attributes (`Secure`, `HttpOnly`, `SameSite`) are the primary browser-enforced protections for cookie-based sessions
- Token-based sessions provide no automatic protections — all security must be explicitly implemented
- Session fixation requires a fresh session value to be issued upon authentication
- Authorisation bypass (vertical and horizontal) occurs when session tracking fails to verify what the user is allowed to access
- Sessions must be invalidated server-side on logout and on password reset

---

## Conclusion

Session management is the operational thread connecting authentication, authorisation, and accountability across every request a user makes. Weaknesses at any point in the lifecycle — from session creation through to termination — can allow attackers to bypass authentication, hijack accounts, or persist access even after a user logs out. A secure session implementation treats every phase of the lifecycle as a distinct security boundary.
