# Burp Suite: Intruder

## Overview

Burp Suite Intruder is the automated request manipulation module — Burp's built-in fuzzer and brute-forcer. Where Repeater handles single, manually crafted requests, Intruder handles repetitive attacks that require systematically substituting payloads into one or more positions across hundreds or thousands of requests. This room covers Intruder's interface, its four attack types, payload configuration, and two practical attack scenarios: a credential-stuffing attack using Pitchfork and an IDOR fuzzing exercise.

**What this room teaches:**
- The Intruder interface and its four sub-tabs
- The four attack types: Sniper, Battering Ram, Pitchfork, Cluster Bomb
- How to configure payload sets for each attack type
- Credential stuffing vs brute-force — when each is appropriate
- Intruder + Burp Macros for CSRF-protected login forms
- IDOR discovery through parameter fuzzing

**Main objectives:**
- Perform a credential-stuffing attack using Pitchfork against a login form
- Identify a successful login using response length as a differentiator
- Discover an IDOR vulnerability by fuzzing a numeric ticket endpoint
- Bypass CSRF token protection using Burp Macros in a macro-assisted Pitchfork attack

**Skills introduced:** Automated fuzzing, credential stuffing, IDOR discovery, Burp Macros, response analysis, payload set configuration

---

## Concepts Covered

### Concept 1 — Burp Suite Intruder

**Definition:** Intruder is Burp Suite's automated request manipulation module. It takes a captured HTTP request and inserts payloads from user-defined lists into designated positions within the request, sending a modified version for each payload.

**Why It Matters:** Manual testing one payload at a time is impractical for attacks requiring hundreds or thousands of attempts — password lists, endpoint wordlists, ID ranges. Intruder automates this process while maintaining full visibility into each individual request and response.

**Important Limitation:** The Community Edition of Burp Suite rate-limits Intruder significantly. For high-volume fuzzing, practitioners typically use tools like `ffuf`, `wfuzz`, or `hydra`. However, Intruder remains essential for attack types that require tight integration with Burp's session handling (macros, cookies, CSRF tokens).

**Intruder's four sub-tabs:**

| Tab | Purpose |
|-----|---------|
| **Positions** | Define which parts of the request receive payloads; select attack type |
| **Payloads** | Configure payload sets, payload types, processing rules, and encoding |
| **Resource Pool** | Manage concurrent attack threads (Burp Pro only) |
| **Settings** | Control attack behaviour — redirects, grep flags, result handling |

---

### Concept 2 — Positions Tab and Payload Markers

**Definition:** Positions are the locations within the captured request where Intruder will inject payloads. They are marked with section signs (`§`) wrapping the target value.

**Why It Matters:** Precise position marking determines exactly what gets replaced with payloads. Over-marking positions wastes attack iterations; under-marking misses injection points.

**Controls:**
- `Add §` — manually highlight text and click to create a position
- `Clear §` — remove all positions (start fresh)
- `Auto §` — Burp automatically identifies likely injection candidates (form fields, cookies, query parameters)

**Practical Note:** Auto§ is a good starting point but often marks session cookies and CSRF tokens — these usually need to be cleared when attacking a specific parameter.

---

### Concept 3 — Payload Configuration

The Payloads tab has four sections:

| Section | Purpose |
|---------|---------|
| **Payload Sets** | Choose which position's payload set to configure; select payload type |
| **Payload Settings** | Add/load/remove payloads — e.g., load from a wordlist file |
| **Payload Processing** | Define transformations applied to each payload before sending (prefix, suffix, regex filter, capitalisation) |
| **Payload Encoding** | Control URL encoding — disable or customise which characters are encoded |

**Payload Types** (beyond Simple List):
- **Numbers** — sequential or random numeric ranges
- **Dates** — date sequence generation
- **Brute Forcer** — character set combinations up to specified length
- **Null payloads** — repeat the request N times without modification (useful for race conditions)
- **Runtime file** — read from file in real time during attack

---

### Concept 4 — The Four Attack Types

#### Sniper

**Definition:** Tests one payload at a time, cycling each payload through every defined position sequentially.

**How it works:** With N positions and a wordlist of W words, Intruder makes N × W requests. It substitutes the first payload into position 1, leaves all other positions at their original values, then moves to position 2, and so on.

**Example:** Two positions (username, password), three words (burp, suite, intruder):

| Request | username | password |
|---------|---------|---------|
| 1 | burp | Expl01ted (original) |
| 2 | suite | Expl01ted |
| 3 | intruder | Expl01ted |
| 4 | pentester (original) | burp |
| 5 | pentester | suite |
| 6 | pentester | intruder |

**Best for:** Single-position attacks — password brute-force on one field, endpoint fuzzing, testing one parameter at a time.

---

#### Battering Ram

**Definition:** Inserts the same payload into every defined position simultaneously.

**How it works:** For each payload, all positions receive the identical value at the same time.

**Example:** Two positions, three words:

| Request | username | password |
|---------|---------|---------|
| 1 | burp | burp |
| 2 | suite | suite |
| 3 | intruder | intruder |

**Best for:** Scenarios where the same value must appear in multiple places — e.g., testing for username = password (common weak credentials), race condition testing.

---

#### Pitchfork

**Definition:** Tests multiple positions simultaneously using separate payload sets — one set per position. Positions are paired with their respective lists and iterated in lockstep.

**How it works:** Takes the first item from list 1 and the first item from list 2, sends the combined request. Then moves to the second item from each list, and so on. Stops when the shorter list runs out.

**Example:** Two positions, two paired lists:

| Request | username | password |
|---------|---------|---------|
| 1 | joel | J03l |
| 2 | harriet | Emma1815 |
| 3 | alex | Sk1ll |

**Best for:** Credential stuffing (known username:password pairs), any scenario where list items have a known 1:1 correspondence.

**Important:** Pitchfork assumes the lists are aligned — row 1 of list 1 corresponds to row 1 of list 2. If they are different lengths, testing stops when the shorter list ends.

---

#### Cluster Bomb

**Definition:** Tests every combination of every payload from every list against every position. Produces a cartesian product of all payloads.

**How it works:** Iterates through all values of list 2 for each value of list 1.

**Example:** Two positions, three words each:

| Request | username | password |
|---------|---------|---------|
| 1 | joel | J03l |
| 2 | harriet | J03l |
| 3 | alex | J03l |
| 4 | joel | Emma1815 |
| 5 | harriet | Emma1815 |
| 6 | alex | Emma1815 |
| 7 | joel | Sk1ll |
| 8 | harriet | Sk1ll |
| 9 | alex | Sk1ll |

Total requests = list1 length × list2 length (exponential growth with more lists).

**Best for:** Credential brute-force when there is no known username:password mapping — test all combinations.

**Caution:** Generates enormous traffic at scale. A 1000-word username list × 1000-word password list = 1,000,000 requests.

---

### Concept 5 — Credential Stuffing vs Brute-Force

| Attack | Definition | When to Use |
|--------|-----------|------------|
| **Brute-Force** | Try all possible combinations of characters or words | No prior knowledge of valid credentials |
| **Credential Stuffing** | Try known username:password pairs from previous breaches | When a breach dump exists with matched pairs |

Credential stuffing is significantly more efficient — it uses real credentials from real breaches rather than guessing. It exploits password reuse: users who do not change passwords after a breach remain vulnerable at any service using the same credentials.

**Why Pitchfork for credential stuffing:** The usernames and passwords in a breach dump are already paired. Pitchfork's lockstep iteration preserves this pairing exactly.

---

### Concept 6 — IDOR (Insecure Direct Object Reference)

**Definition:** An access control vulnerability where an application exposes internal object identifiers (database IDs, file names) in URLs or parameters without verifying whether the requesting user is authorised to access that specific object.

**Why It Matters:** IDOR allows horizontal privilege escalation — accessing data belonging to other users by simply changing a numeric ID in the URL.

**Example from this room:** After authenticating, support tickets are accessible at `/support/ticket/NUMBER`. If the server does not verify that the ticket belongs to the currently authenticated user, any authenticated user can access any ticket by changing the number.

**Detection approach:** Fuzz the numeric parameter with a sequential number range using Intruder Sniper. Compare response lengths — if some return different lengths (more or less content), they are accessible and the vulnerability is confirmed.

---

### Concept 7 — Burp Macros for CSRF Token Handling

**Definition:** Burp Macros are pre-defined sequences of HTTP requests that Intruder (or other modules) execute before each attack request. They extract dynamic values (CSRF tokens, session cookies) from responses and inject them into subsequent attack requests.

**Why It Matters:** CSRF tokens and session cookies change with each page load. Without refreshing them before each attack request, all requests will be rejected as invalid/expired. Macros automate this token refresh transparently.

**How macros interact with Intruder:**
1. Macro sends a GET request to the login page
2. Extracts the fresh `loginToken` from the form field
3. Extracts the fresh `session` cookie from the response
4. Injects both into the Intruder attack request before sending
5. Intruder sends the attack request with valid, current token values

---

## Methodology

### Credential Stuffing Workflow

```
1. Obtain breach dump (usernames.txt + passwords.txt paired lists)
         |
         v
2. Capture a login POST request in Proxy
         |
         v
3. Send to Intruder (Ctrl+I)
         |
         v
4. Positions tab: mark only username and password fields
   Clear auto-detected positions (session cookies etc.)
   Set attack type: Pitchfork
         |
         v
5. Payloads tab:
   Payload set 1 → Load usernames.txt
   Payload set 2 → Load passwords.txt
         |
         v
6. Start Attack
         |
         v
7. Analyse results: sort by Length column
   Successful login = significantly shorter or longer response
         |
         v
8. Confirm credentials: log in manually with the identified pair
```

### IDOR Fuzzing Workflow

```
1. Authenticate and capture a request to /support/ticket/1
         |
         v
2. Send to Intruder
         |
         v
3. Mark the ticket number as the payload position
   Attack type: Sniper
         |
         v
4. Payload type: Numbers
   From: 1, To: 100, Step: 1
         |
         v
5. Start Attack
         |
         v
6. Sort by Length — different length = different content = potentially accessible ticket
7. Any accessible ticket belonging to another user = IDOR confirmed
```

### CSRF-Protected Login Attack Workflow

```
1. Capture login POST to /admin/login/ — note loginToken + session cookie
         |
         v
2. Send to Intruder → Pitchfork → mark username and password only
         |
         v
3. Load wordlists for both positions
         |
         v
4. Build Macro:
   Settings → Sessions → Macros → Add
   Select GET /admin/login/ from history
   Name the macro
         |
         v
5. Create Session Handling Rule:
   Add rule → Scope: Intruder only → URL scope: target host
   Rule action: Run a Macro → select created macro
   Update only: loginToken (parameter), session (cookie)
         |
         v
6. Start Attack — all requests now receive fresh tokens
7. Sort by Length — find the successful login (shorter response)
```

---

## Practical Activities

### Activity 1 — Credential Stuffing Against Support Login

**Objective:** Use a leaked credential dump to perform a Pitchfork attack and identify valid credentials for the support portal at `/support/login`.

**Background:** Bastion Hosting suffered a breach three months prior. Employee credentials (usernames + passwords) were exposed. Some users may not have changed their passwords.

**Setup:**
```bash
# Download the leaked credentials archive
wget http://TARGET_IP:9999/Credentials/BastionHostingCreds.zip
unzip BastionHostingCreds.zip
# Contains: emails.txt, usernames.txt, passwords.txt, combined.txt
```

**Configuration:**
1. Browse to `http://TARGET_IP/support/login` with Proxy active
2. Submit any credentials — capture the POST request
3. Send to Intruder (`Ctrl+I`)
4. Positions tab:
   - Clear all auto-detected positions
   - Mark only `username=§value§` and `password=§value§`
   - Set attack type: **Pitchfork**
5. Payloads tab:
   - Payload set 1 → Load `usernames.txt`
   - Payload set 2 → Load `passwords.txt`
6. Click Start Attack

**Example marked request:**
```http
POST /support/login HTTP/1.1
Host: TARGET_IP
Content-Type: application/x-www-form-urlencoded

username=§pentester§&password=§Expl01ted§
```

**Findings and Analysis:**
- All responses returned HTTP 302 (redirect) — status code alone does not differentiate success from failure
- Sorting by **response length** reveals one request with a significantly shorter response
- This shorter response indicates a successful login — the redirect target is different (home page vs error page)
- Confirmed credentials by using them to log in manually

**Why It Matters:** Response status codes are not reliable differentiators for login attacks — both successes and failures often redirect. Response length is the correct signal because a failed login returns the login page again (larger), while a successful login redirects to a dashboard (smaller response body). This is a fundamental Intruder analysis technique.

---

### Activity 2 — IDOR Discovery via Ticket Endpoint Fuzzing

**Objective:** After authenticating, fuzz the `/support/ticket/NUMBER` endpoint to determine whether it is properly access-controlled or vulnerable to IDOR.

**Background:** The support ticket system uses sequential integer IDs. If the server doesn't verify ticket ownership, any authenticated user can view any ticket.

**Configuration:**
1. While authenticated, capture a request to `/support/ticket/1`
2. Send to Intruder
3. Mark only the ticket number as the position: `/support/ticket/§1§`
4. Attack type: **Sniper**
5. Payload type: **Numbers** — From: 1, To: 100, Step: 1
6. Start Attack

**Findings:** Responses with different content lengths indicate accessible tickets. Tickets belonging to other users being returned confirms the IDOR vulnerability — the server returns any ticket without verifying ownership against the current session.

**Why It Matters:** IDOR in ticketing systems is a real-world finding. Support tickets frequently contain sensitive information — customer complaints, internal escalations, password reset tokens, account details. Unauthorised access to this data is a significant confidentiality breach.

---

### Activity 3 — CSRF-Protected Admin Login with Burp Macros

**Objective:** Perform a credential-stuffing attack against `/admin/login/` which uses a CSRF token (`loginToken`) and session cookie that change with every page load.

**The Challenge:**
- Every GET to `/admin/login/` returns a new `loginToken` in the form and a new `session` cookie
- Intruder reuses the initial captured values — all subsequent requests fail with HTTP 403

**Macro Configuration:**
1. Settings → Sessions → Macros → Add
2. Select `GET /admin/login/` from request history
3. Name the macro "Get Login Token"
4. Create Session Handling Rule:
   - Scope: Intruder only
   - URL scope: `http://TARGET_IP/`
   - Action: Run a Macro → select "Get Login Token"
   - Update only the following parameters: `loginToken`
   - Update only the following cookies: `session`

**Why Only These Values?** Updating only `loginToken` and `session` preserves the username and password values that Intruder is injecting. Without this restriction, the macro would overwrite all parameters including the attack payloads.

**Findings:**
- All requests return HTTP 302 (same as before)
- Sorting by length reveals the successful credential pair — shorter response indicates successful authentication
- The valid credentials allow access to the admin panel

**Why It Matters:** CSRF tokens are a standard anti-automation mechanism. Burp Macros make them transparent to Intruder — demonstrating that CSRF tokens alone do not prevent brute-force attacks when credentials are the actual target. The real protection against credential attacks is account lockout, rate limiting, and MFA — not CSRF tokens.

---

## Observations and Analysis

**Response Length as the Differentiator:** When status codes are identical across all responses (all 302 redirects), response length is the primary differentiator for identifying successful attempts. A shorter redirect response means the destination page is different — typically a dashboard vs a re-displayed login form with an error message.

**No Lockout on Login Forms:** Neither the support login nor the admin login implemented account lockout or rate limiting (beyond Intruder's Community Edition throttle). Production systems should implement progressive delays, temporary lockout after N failed attempts, and alerting on high-volume failed attempts from a single IP.

**CSRF Tokens Without Rate Limiting:** The admin login used CSRF tokens but had no rate limiting. CSRF tokens prevent cross-site request forgery but offer no protection against a same-origin brute-force attack where an attacker can obtain fresh tokens before each request.

**Sequential IDs as IDOR Indicators:** Any endpoint using sequential integer IDs in its URL is an immediate IDOR candidate. Proper access control requires server-side verification that the requesting user owns or is authorised to access the referenced resource — not just that they are authenticated.

---

## Tools and Technologies Used

### Burp Suite Intruder
- **Purpose:** Automated HTTP request manipulation — fuzzing, brute-forcing, credential stuffing
- **Common Usage:** Login brute-force, IDOR discovery, endpoint fuzzing, parameter testing
- **In This Room:** Pitchfork credential stuffing, numeric IDOR fuzzing, CSRF-bypassed admin login

### Burp Suite Macros
- **Purpose:** Pre-defined request sequences executed before each Intruder attack request
- **Common Usage:** Refreshing CSRF tokens, maintaining session state in complex multi-step attacks
- **In This Room:** Extracting fresh `loginToken` and `session` values before each admin login attempt

---

## Key Learnings

- Pitchfork preserves list alignment — essential for credential stuffing where username:password pairs must stay paired
- Cluster Bomb tests all combinations — exponential request count — use only when mapping is unknown
- Sniper is the right choice for single-position fuzzing (one field, one list)
- Response length, not status code, identifies successful logins when all responses redirect
- Macros solve the CSRF token problem in automated attacks — but CSRF tokens alone are not a brute-force defence
- IDOR manifests when sequential IDs are accessible without ownership verification — always test numeric endpoints
- Rate limiting and account lockout are the actual defences against credential attacks
- Clear the auto-detected positions before marking your own — Burp often includes session tokens that should not be in the attack

---

## Real-World Relevance

**Penetration Testing:** Credential stuffing with leaked credentials is a standard attack scenario in external penetration tests. Intruder + Pitchfork is the Burp-native approach; for large lists, testers switch to `hydra` or custom scripts.

**Bug Bounty Hunting:** IDOR in sequential ticket/order/invoice endpoints is one of the most common high-value findings. Intruder Sniper with a number range is the fastest way to confirm it.

**Red Team Operations:** Credential stuffing using breach data from previous incidents (LinkedIn, Dropbox, etc.) is a realistic initial access technique. Many organisations have employees who reuse passwords across personal and corporate accounts.

**Security Engineering:** This room directly motivates implementing account lockout, rate limiting, CAPTCHA, and MFA — all controls that would defeat the attacks demonstrated here.

---

## Things Worth Remembering

- Send to Intruder: `Ctrl+I` or right-click → Send to Intruder
- Attack type selection:
  - **Sniper** → single position, iterate one payload at a time
  - **Battering Ram** → all positions get the same payload simultaneously
  - **Pitchfork** → multiple positions, paired lists, lockstep iteration
  - **Cluster Bomb** → multiple positions, all combinations (cartesian product)
- Total Sniper requests = words × positions
- Total Cluster Bomb requests = list1.length × list2.length × ...
- Sort results by Length to find the odd-one-out in redirect-heavy attacks
- Clear auto-detected positions before configuring your own
- Macros: Settings → Sessions → Macros → Add
- Restrict macro updates to specific parameters and cookies to avoid overwriting payloads
- CSRF tokens ≠ brute-force protection; account lockout + rate limiting are the actual defences
- Sequential IDs in URLs = immediate IDOR candidate; fuzz with Numbers payload
