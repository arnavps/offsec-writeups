# Multi-Factor Authentication

## Overview

Multi-Factor Authentication (MFA) significantly raises the barrier for account compromise by requiring proof from multiple independent categories. While MFA is a powerful control, its implementation can contain vulnerabilities — weak OTP generation, token leakage in HTTP responses, rate limiting failures, logic flaws that allow MFA bypass, and automated brute-forcing that handles re-authentication loops. This room covers MFA types, authentication factors, real-world use cases, and the key attack techniques used to defeat MFA implementations.

---

## Topics Covered

- What MFA is and how it differs from 2FA
- Five authentication factor categories
- Common 2FA mechanisms: TOTP, push notifications, SMS, hardware tokens
- Conditional access and adaptive authentication
- Weak OTP generation algorithms
- OTP leakage in HTTP responses
- Brute-forcing OTPs without rate limiting
- MFA bypass via logic flaws
- Automated brute-forcing with session re-creation
- Evilginx: MFA phishing proxy

---

## Key Concepts

### MFA vs 2FA

- **MFA** — any authentication requiring two or more verification factors from different categories
- **2FA** — a specific subset of MFA that requires exactly two factors

All 2FA is MFA, but not all MFA is 2FA. An authentication requiring a password, fingerprint, and hardware token is MFA but not 2FA.

---

### Authentication Factor Categories

| Category | Description | Examples |
|----------|-------------|---------|
| **Something you know** | Memorised secret | Password, PIN, security question |
| **Something you have** | Physical possession | Phone with auth app, YubiKey, smart card, client certificate |
| **Something you are** | Biometric characteristic | Fingerprint, facial recognition, iris scan |
| **Somewhere you are** | Location context | IP address, geolocation |
| **Something you do** | Behavioural pattern | Typing rhythm, mouse movement (primarily anti-bot) |

Biometrics are probabilistic — fingerprints and face scans never match 100%. They should always supplement, never solely replace, other factors.

---

### Common 2FA Mechanisms

| Mechanism | Description | Security Level |
|-----------|-------------|---------------|
| **TOTP** | Time-based one-time password — changes every 30 seconds. Apps: Google Authenticator, Authy, Microsoft Authenticator | High — time-limited, requires physical possession |
| **Push notifications** | Login approval sent to registered device. Apps: Duo, Google Prompt | High — but vulnerable to MFA fatigue attacks |
| **SMS** | One-time code via text message | Medium — vulnerable to SIM swapping and SS7 interception |
| **Hardware tokens** | Physical device generating OTP or using NFC (e.g. YubiKey) | Very high — works offline, tamper-resistant |

**MFA fatigue attack:** Repeatedly sending push notification approval requests hoping the user approves one out of frustration. This technique was used in a real Uber corporate account compromise.

---

### Conditional Access

Conditional access adjusts authentication requirements based on context:

| Condition | Trigger | Response |
|-----------|---------|---------|
| Location-based | Login from unfamiliar IP or country | Require additional OTP or biometric |
| Time-based | Login outside business hours | Require additional verification |
| Behavioural analysis | Unusual data access patterns | Step-up authentication |
| Device-specific | Login from unapproved device | Block after first factor |

---

## MFA Vulnerabilities

### 1. Weak OTP Generation Algorithm

If the OTP is generated using a weak or predictable algorithm (non-cryptographic PRNG, limited range), it can be brute-forced.

**Vulnerable PHP implementation:**
```php
$token = mt_rand(100, 200);   // Only 101 possible values
```

A 3-digit OTP with only 101 possible values can be brute-forced in at most 101 requests — trivial even with basic rate limiting.

**Fix:** Use a CSPRNG to generate OTPs with sufficient entropy. Standard TOTP implementations (RFC 6238) use HMAC-SHA1 over time-based values — use an established library rather than rolling your own.

---

### 2. OTP Leakage in HTTP Response

When a user reaches the 2FA page, some applications make an XHR request to generate and send the OTP. If the server returns the OTP in the HTTP response body (for debugging or poor implementation), the user's browser receives the OTP directly — making MFA trivially bypassable.

**How to identify:** Use browser DevTools (Network tab) or Burp Suite to intercept all XHR requests after reaching the 2FA page. Look for responses containing a numeric value matching the OTP length.

**Why it happens:**
- Debug code left in production
- Developer mistakenly returns OTP for client-side validation (should never happen)
- Poor response handling practices

**Fix:** The OTP is generated and stored server-side only. The server sends it to the user's device (email/SMS/authenticator app) — never in the HTTP response.

---

### 3. Brute-Forcing OTPs Without Rate Limiting

Without rate limiting on the OTP submission endpoint, an attacker can iterate through all possible OTP values. Standard 6-digit TOTP has 1,000,000 possible values, but shorter or numeric-only OTPs are far smaller.

**Real-world example:** Reported HackerOne bug — application did not rate-limit the 2FA code submission endpoint, allowing unlimited OTP attempts.

**Fix:**
- Implement rate limiting (e.g. 5 attempts per minute per user)
- Implement account lockout after N consecutive failures
- Implement exponential backoff between attempts

---

### 4. Logic Flaw — MFA Bypass

A logic flaw occurs when the authenticated flag is set prematurely — before MFA is completed — allowing an attacker with knowledge of the application's URL structure to skip the 2FA page entirely.

**Vulnerable PHP implementation:**
```php
// In the login handler:
if (authenticate($email, $password)) {
    $_SESSION['authenticated'] = true;  // Set BEFORE MFA
    $_SESSION['email'] = $_POST['email'];
    header('Location: /mfa');
    return;
}

// On the dashboard:
if ($_SESSION['authenticated'] !== true) {
    redirect('/login');
}
```

The `$_SESSION['authenticated']` flag is set when credentials are verified — before the OTP step. An attacker who knows the dashboard URL can navigate directly to it after the first step, bypassing 2FA entirely.

**Fix:** Split authentication state into two separate flags:
- `$_SESSION['credentials_verified']` — set after username/password check (only allows access to the 2FA page)
- `$_SESSION['authenticated']` — set only after successful OTP verification (grants access to the application)

---

### 5. Automated Brute-Force With Session Re-Creation

Some applications log the user out after a failed 2FA attempt, forcing re-authentication. Manual brute-forcing becomes impractical, but automation handles this seamlessly.

**Python script structure for handling logouts:**

```python
import requests

login_url = 'http://TARGET/labs/login'
otp_url = 'http://TARGET/labs/mfa'
credentials = {'email': 'user@example.com', 'password': 'password'}

def login(session):
    return session.post(login_url, data=credentials)

def submit_otp(session, otp):
    otp_data = {
        'code-1': otp[0], 'code-2': otp[1],
        'code-3': otp[2], 'code-4': otp[3]
    }
    return session.post(otp_url, data=otp_data, allow_redirects=False)

def brute_force():
    for i in range(1250, 1351):   # Try all values in range
        otp_str = str(i)
        session = requests.Session()   # Fresh session per attempt
        login_response = login(session)
        if "User Verification" not in login_response.text:
            continue
        response = submit_otp(session, otp_str)
        if response.status_code == 302:
            location = response.headers.get('Location', '')
            if '/dashboard' in location:
                print(f"Success: OTP = {otp_str}")
                return session.cookies.get_dict()
```

**Key insight:** Creating a new `requests.Session()` for each attempt handles the logout — the session is fresh and the login step is automatically repeated.

---

### 6. Evilginx — MFA Phishing Proxy

Evilginx operates as a man-in-the-middle reverse proxy between the victim and the real authentication service. When a victim enters credentials and an OTP on what appears to be a legitimate login page:

1. Evilginx forwards the credentials and OTP to the real service
2. The real service authenticates the user and returns a session cookie
3. Evilginx captures the session cookie before passing it to the victim's browser

The attacker obtains the authenticated session cookie — bypassing MFA entirely because the authentication was completed legitimately, just observed.

**Defence:** Phishing-resistant authentication factors (hardware security keys with FIDO2/WebAuthn) cannot be proxied — the credential is cryptographically bound to the legitimate domain.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| MFA | Multi-Factor Authentication — requires two or more factor categories |
| 2FA | Two-Factor Authentication — subset of MFA using exactly two factors |
| TOTP | Time-Based One-Time Password — RFC 6238 standard for time-based OTPs |
| OTP | One-Time Password — a credential valid for a single use or time window |
| MFA fatigue | Attack sending repeated push notifications until the victim approves one |
| Rate limiting | Restricting the number of authentication attempts in a time window |
| Logic flaw | Implementation error allowing authentication state to be bypassed |
| Evilginx | Reverse proxy phishing tool that captures session cookies by proxying real authentication |
| FIDO2/WebAuthn | Phishing-resistant hardware-bound authentication standard |
| SIM swapping | Social engineering mobile carriers to redirect a victim's phone number to the attacker |

---

## Real-World Relevance

- The MFA fatigue attack was used to compromise an Uber corporate account — the attacker repeatedly sent push notifications until the employee approved one
- OTP leakage in HTTP responses has been reported on major platforms — developers exposing OTPs in debug responses that were accidentally shipped to production
- Logic flaw bypasses (authenticated flag set before OTP completion) are a consistent finding in custom authentication implementations
- Evilginx is used in real targeted phishing campaigns against high-value accounts where standard password MFA provides no protection
- SMS-based 2FA weakness via SIM swapping has been exploited in high-profile cryptocurrency thefts and account takeovers

---

## Key Learnings

- MFA requires factors from multiple independent categories — two passwords do not constitute MFA
- TOTP and hardware tokens are stronger than SMS (which is vulnerable to SIM swapping and SS7 attacks)
- OTP generation must use a CSPRNG — predictable ranges are trivially brute-forced
- The OTP must never appear in the HTTP response — it should only be delivered via the out-of-band channel
- Rate limiting and lockout on OTP submission endpoints are mandatory — their absence makes brute force practical
- Logic flaws allowing dashboard access before MFA completion are a common finding in custom implementations
- Automation handles logout-on-failure gracefully — manual rate limits without server-side session invalidation can be defeated

---

## Conclusion

MFA significantly raises the cost of account compromise, but its implementation must be correct. Weak OTP algorithms, leaked tokens, missing rate limits, and logic flaws all defeat MFA's purpose. The strongest MFA implementations use phishing-resistant mechanisms (FIDO2/WebAuthn hardware keys), cryptographic OTPs, enforced rate limiting, and correctly sequenced authentication state management.
