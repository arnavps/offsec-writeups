# Hammer

## Overview

Hammer is a practical authentication challenge room on TryHackMe. It combines multiple authentication security concepts from the module — rate limiting bypass, OTP brute-forcing, session management flaws, and JWT manipulation — into a single hands-on machine. The goal is to chain these weaknesses to achieve full authentication bypass and privilege escalation.

---

## Key Concepts Applied

This room is a practical application of techniques from the Authentication Training module:

- Username/email enumeration (Enumeration and Brute Force)
- OTP brute-forcing with rate limit handling (Multi-Factor Authentication)
- JWT claim manipulation (JWT Security)
- Session management flaws (Session Management)

---

## Investigation Approach

### Step 1 — Reconnaissance

Start by mapping the authentication flow:

```bash
# Identify endpoints
# Look for: /login, /register, /reset, /mfa, /api
# Capture requests with Burp Suite or browser DevTools
```

Key questions:
- Does the login form return different errors for valid vs invalid usernames?
- Is there a password reset or registration flow?
- Does the application use JWT or cookie-based sessions?
- Is there a 2FA/OTP step?

### Step 2 — Username Enumeration

Test the login and password reset endpoints for verbose error messages:
- Valid email: `"Incorrect password"` or redirect to password reset
- Invalid email: `"Email does not exist"` or similar

If different responses are returned, enumerate valid emails from a wordlist.

### Step 3 — Password Reset / OTP Analysis

After identifying a valid email, trigger the password reset flow:
1. Observe the OTP length and character set (numeric only? alphanumeric?)
2. Check whether the OTP appears in any HTTP response (intercept with Burp)
3. Determine if there is rate limiting on OTP submission
4. Check the OTP range — is `mt_rand()` or a similar weak function being used?

If the OTP is numeric with a small range, brute-force it:

```python
import requests

def try_otp(session, otp):
    response = session.post('http://TARGET/mfa', data={'code': otp}, allow_redirects=False)
    return response

for i in range(1000, 9999):
    otp = str(i).zfill(4)
    # If rate-limited: create a new session and re-authenticate before each attempt
    response = try_otp(session, otp)
    if response.status_code == 302 and 'dashboard' in response.headers.get('Location', ''):
        print(f"Valid OTP: {otp}")
        break
```

### Step 4 — JWT Manipulation

If the application issues a JWT after authentication:

1. Decode the JWT header and payload (Base64Url decode, or use jwt.io)
2. Check for:
   - Sensitive data in claims (passwords, flags, API keys)
   - `"alg": "none"` acceptance
   - Weak secret (brute-force with hashcat)
   - Missing `exp` claim
   - Mutable privilege claims (`admin`, `role`, `is_admin`)

**Privilege escalation via JWT:**
```python
import jwt

# After recovering the secret via hashcat or identifying none algorithm acceptance:
payload = {"username": "user", "admin": 1}
token = jwt.encode(payload, secret, algorithm="HS256")
print(token)
```

### Step 5 — Session Analysis

If cookie-based sessions are in use:
- Decode the session cookie (Base64? JSON? JWT?)
- Check if role or privilege values are embedded and modifiable
- Test if old session cookies remain valid after logout (server-side invalidation)

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Burp Suite | Intercept and modify HTTP requests |
| Python requests | Automate OTP brute-forcing with session management |
| hashcat (`-m 16500`) | Crack JWT HS256 secrets offline |
| jwt.io | Decode and inspect JWT structure |
| Hydra | Credential brute-forcing for login forms |
| Browser DevTools | Inspect cookies, localStorage, XHR requests |

---

## Common Findings in This Room Type

| Vulnerability | Indicator |
|---------------|-----------|
| Username enumeration | Different login error messages |
| Weak OTP | Short numeric range, `mt_rand()` pattern |
| OTP in HTTP response | Visible in XHR response body |
| No rate limiting | OTP attempts not throttled |
| JWT algorithm issue | Accepts `none`, weak secret, or allows algorithm switching |
| MFA logic bypass | Dashboard accessible after first auth step |
| Insecure session | Role encoded in modifiable cookie value |

---

## Notes

This is a challenge room — specific flags and step-by-step solutions are not documented here. Apply the authentication techniques from the module writeups systematically, working through each layer of the authentication flow.

---

## Key Learnings

- Real-world authentication systems combine multiple weaknesses — a single room often requires chaining enumeration → OTP brute-force → JWT manipulation
- Automation is essential for OTP brute-forcing — manual attempts are impractical, especially when the application logs you out on failure
- JWT decoding reveals privilege claims that are often the direct path to escalation
- Always check for rate limiting on every OTP and credential submission endpoint — its absence is one of the most common findings in authentication assessments
