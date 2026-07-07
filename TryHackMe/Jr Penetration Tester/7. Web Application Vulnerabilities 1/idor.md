# IDOR — Insecure Direct Object Reference

## Overview

Insecure Direct Object Reference (IDOR) is an access control vulnerability where an application exposes a direct reference to an internal object (user record, order, document, ticket) and retrieves it based on user input without verifying whether the requesting user is authorised to access that specific object. It sits at position one in the OWASP Top 10 under Broken Access Control, and appears as Broken Object Level Authorisation (BOLA) in the OWASP API Security Top 10.

IDOR's significance lies in the gap between its simplicity and its impact. Exploiting it often requires nothing more than changing a number in a URL or decoding and re-encoding a value. Yet the consequences range from mass data disclosure to full account takeover.

**Skills introduced:** Identifying direct object references in URLs, POST bodies, cookies, and AJAX requests; base64 decoding and re-encoding of encoded references; understanding hash-based obfuscation; two-account testing technique; parameter mining; API endpoint identification via the Network tab.

---

## Concepts Covered

### What Makes IDOR Different from Other Vulnerabilities

Authentication says "I know who you are." Authorisation says "I know what you're allowed to do." IDOR is a failure of authorisation specifically. The application correctly authenticates the user but then allows them to access any object by changing an identifier — without checking ownership.

```
Authenticated as user 1305:

http://service.thm/profile?user_id=1305  → your profile (expected)
http://service.thm/profile?user_id=1000  → another user's profile (IDOR)

The authentication layer is working. The authorisation check is absent.
```

### Object Reference Types

#### Plaintext / Numeric IDs (Most Common)

The simplest form: a sequential or non-sequential integer directly in the URL or POST body. Increment or decrement to access other records.

#### Encoded References (Base64)

The application encodes identifiers before including them in requests. Base64 is the most common encoding.

**Recognising base64:** Longer than the value it represents, often ends in `=` or `==`, uses characters `a-z A-Z 0-9 + / =`.

**Examples:**
- `123` → `MTIz`
- `{"user_id": 5}` → `eyJ1c2VyX2lkIjogNX0=`

**Exploit workflow:**
```bash
# 1. Decode the observed value
echo 'MTIz' | base64 -d       # Output: 123

# 2. Modify the decoded output (change ID)
# 123 → 1

# 3. Re-encode
echo '1' | base64             # Output: MQ==

# 4. Substitute in the request and submit
```

**Important:** Base64 is encoding, not encryption. It provides zero security — anyone can decode it. An application relying on encoded identifiers without server-side authorisation is exactly as vulnerable as one using plaintext IDs.

#### Hashed References

Some applications hash identifiers. MD5 of `123` is `202cb962ac59075b964b07152d234b70`. This looks opaque — but for predictable inputs like sequential integers, it provides no real protection.

**Attack approach:**
1. Identify the hash length to determine the algorithm (MD5: 32 hex chars, SHA-1: 40, SHA-256: 64)
2. For sequential integers, compute the hash of 1, 2, 3, ... using the same algorithm
3. Compare computed hashes to the observed value — a match reveals the underlying integer
4. Services like CrackStation maintain precomputed hash-to-value tables; lookup is instantaneous for small integers

**Tools:** `hash-identifier`, `hashid` on Kali Linux can identify the algorithm from the hash format.

**Conclusion:** Hashing adds obfuscation but not security for predictable inputs. The fix is still server-side authorisation, not better identifier formatting.

#### Unpredictable References (UUIDs)

A UUID like `d3b07384-d9a0-4e9b-8b3c-2f1a6c7e4a90` cannot be guessed or enumerated by incrementing. An unpredictable identifier removes the enumeration attack path — but it does **not** eliminate IDOR. If the server still returns the object to anyone who presents the identifier without checking ownership, the vulnerability exists.

**Testing technique — Two-Account Method:**
1. Create Account A and Account B
2. Log into Account A; note all identifiers associated with A's resources
3. Log into Account B; substitute Account A's identifiers into Account B's requests
4. If Account B's session can access Account A's resources → IDOR exists regardless of identifier format

This technique separates two distinct questions: "Can an attacker obtain a valid identifier?" and "Will the server serve the object without checking permissions?" IDOR is present if the answer to the second question is yes, regardless of the first.

**Where unpredictable IDs leak:** Shared URLs, API responses referencing other users' resources, HTML source code, JavaScript files, notification emails, exported data (CSV reports). Once a valid identifier is obtained from any of these sources, exploiting the IDOR is identical to the plaintext case.

---

### Where IDOR Vectors Exist

| Location | How to Find |
|----------|-------------|
| **URL query parameters** | Visible in address bar: `?user_id=1305`, `?order=89543` |
| **POST body fields** | Intercepted in Burp or DevTools Network tab |
| **Cookie values** | Inspect Application tab in DevTools |
| **HTTP request headers** | Less common; inspect all headers |
| **REST API path segments** | `/api/users/123/orders` — the `123` is an IDOR candidate |
| **Background AJAX requests** | Never in the address bar; found in Network tab when page loads |
| **JavaScript files** | API endpoint URLs, parameter names, internal IDs embedded in client-side code |
| **Hidden form fields** | In page source, not visible in rendered page |

**Parameter mining:** Some endpoints accept parameters the front end never sends. Try appending `?user_id=123` to a user profile endpoint that normally returns the current user's data. If the server honours the parameter, it may return a different user's record.

---

## Methodology

```
1. Identify all direct object references:
   URL parameters, POST bodies, cookies, API paths, background AJAX

2. Check whether the reference controls which object is returned:
   If changing the value changes the response object → potential IDOR

3. Test authorisation:
   As User A: access User B's identifier
   If User B's data is returned to User A → IDOR confirmed

4. For encoded references:
   Decode → modify → re-encode → re-submit

5. For hashed references:
   Identify algorithm → compute hash for target IDs → substitute → submit

6. For unpredictable references:
   Use two-account method → check if second account can access first account's resources
   Look for ID leakage in: shared URLs, API responses, JS files, emails

7. Test write operations as well as read:
   IDOR on update/delete endpoints can lead to account takeover
```

---

## Practical Activity — API Endpoint IDOR

**Objective:** Identify a vulnerable background API endpoint and access other users' account data by modifying the `id` parameter.

### Step 1: Set Up

Create an account on the lab application and log in. Navigate to the "Your Account" tab. This page displays username and email pre-filled from the account details.

### Step 2: Identify the Background Request

Open DevTools (F12), select the Network tab, and refresh the page. Among the background requests, find:

```
GET /api/v1/customer?id={your_user_id}
```

This AJAX request fetches account data for the current user. It is never visible in the URL bar — only in the Network tab or a proxy like Burp Suite.

Click the request to inspect its response:
```json
{
  "user_id": 15,
  "username": "youruser",
  "email": "youruser@example.com"
}
```

The `id` parameter in the URL directly controls which user record is returned.

### Step 3: Exploit the IDOR

Right-click the request in the Network tab and select "Edit and Resend" (Firefox), or replay in Burp Suite / curl:

**Testing with id=1:**
```bash
curl 'https://LAB_URL/api/v1/customer?id=1' \
  -H 'Cookie: session=YOUR_SESSION_COOKIE'
```

**Response (different user's data):**
```json
{
  "user_id": 1,
  "username": "admin",
  "email": "admin@company.thm"
}
```

The server returned a completely different user's record. No authorisation check was performed — the `id` parameter was taken at face value.

**Testing with id=3:**
Repeat with `id=3` to retrieve a third user's information.

**Finding:** The `/api/v1/customer?id=` endpoint has no server-side check verifying that the requesting session is authorised to access the requested `id`. Any authenticated user can read any other user's account details by enumerating `id` values.

**Why it matters:** In a real application, this endpoint likely powers the account page for every user. Enumerating all user IDs would exfiltrate every user's email address and username in the system. Write operations (updating email, password) with the same missing authorisation check would enable full account takeover for any user.

---

## Observations and Analysis

- The vulnerable API endpoint (`/api/v1/customer`) was a background AJAX request — completely invisible in the URL bar. Testing only visible URL parameters would have missed it entirely.
- The authentication layer was functioning correctly — the session cookie was valid and the user was authenticated. The missing authorisation check is the only failure.
- The endpoint returned structured JSON containing account details. In a real engagement, this exact response format is what would be included in the proof-of-concept for the finding.
- Integer user IDs starting from 1 are directly enumerable. Even if the application used UUIDs, the two-account method would still confirm the IDOR by swapping identifiers between sessions.
- The difference between IDOR on a read endpoint (data disclosure) and IDOR on a write endpoint (account takeover) is significant from a severity rating perspective. Always test both GET and POST/PUT/DELETE endpoints for missing authorisation.

---

## Tools and Technologies Used

### Browser Developer Tools (Network Tab)
- **Purpose:** Observe background AJAX/XHR requests that never appear in the URL bar
- **Common usage:** F12 → Network tab → filter by XHR/Fetch → refresh page to capture all requests
- **In this room:** Identified the `/api/v1/customer?id=` endpoint that powers the account page

### Burp Suite (Repeater)
- **Purpose:** Intercept, modify, and replay HTTP requests with arbitrary parameters
- **Common usage:** Proxy browser traffic → capture request → send to Repeater → modify parameters → resend
- **How used:** Replay the API request with different `id` values to test authorisation

### `curl`
- **Purpose:** Command-line HTTP client for replaying requests outside the browser
- **Common usage:** `curl 'URL' -H 'Cookie: session=VALUE'`

### CrackStation / hash-identifier
- **Purpose:** Identify hash algorithms and look up precomputed hash values for small integers
- **Common usage:** hash-identifier on Kali: `hash-identifier HASH_VALUE`; CrackStation: paste hash at crackstation.net

### base64 (command line / online tools)
- **Purpose:** Decode and re-encode base64 identifiers for encoded IDOR exploitation
- **Common usage:** `echo 'VALUE' | base64 -d` (decode), `echo 'VALUE' | base64` (encode)

---

## Key Learnings

- IDOR is an authorisation failure, not an authentication failure — the user is correctly identified but not correctly restricted
- Authentication and authorisation are separate controls; both must be implemented
- Base64 encoding provides no security — it is a reversible transformation accessible to any attacker
- Hashing small sequential integers is easily defeated via precomputed lookup tables
- Unpredictable identifiers (UUIDs) remove enumeration as an attack path but do not eliminate IDOR — the two-account method still works
- Background AJAX requests are invisible in the URL bar — always use the Network tab or a proxy to capture all application traffic
- Parameter mining — appending hidden parameters like `?user_id=X` to known endpoints — discovers IDOR vectors the UI never exposes
- IDOR on write endpoints (update, delete, transfer) is typically higher severity than read IDOR

---

## Real-World Relevance

- **Bug bounty:** IDOR is consistently the highest-frequency finding in bug bounty programmes. BOLA (API IDOR) is the number one API security risk. Even small platforms regularly have these on user data endpoints.
- **Penetration testing:** IDOR testing is standard on every web application assessment. API endpoints require explicit testing alongside UI-visible parameters.
- **Enterprise environments:** CRM systems, HR portals, and SaaS platforms frequently have IDOR on their internal APIs because authorisation checks are easy to forget when building data retrieval endpoints.
- **Red team operations:** IDOR enables enumeration of all users in a system, escalation to admin accounts, and extraction of contact data for targeting.

---

## Things Worth Remembering

- IDOR: missing server-side check verifying the requesting user is authorised to access the specific referenced object
- BOLA = IDOR in API context (OWASP API Security Top 10 #1)
- Test all object references: URL params, POST bodies, cookies, headers, API paths, AJAX requests
- Base64 identification: longer than input, ends in `=`, uses `a-zA-Z0-9+/=` — decode with `echo 'V' | base64 -d`
- Hash algorithm identification: 32 hex chars = MD5, 40 = SHA-1, 64 = SHA-256
- Two-account method: Account A's identifiers → Account B's session → if data returned, IDOR confirmed
- Network tab in DevTools captures background AJAX requests never visible in the address bar
- Parameter mining: append `?user_id=X` to endpoints that normally use session context — server may honour explicit parameter
- Severity is higher on write endpoints (account takeover) than read endpoints (data disclosure)

---

## Conclusion

IDOR illustrates that security cannot end at authentication. Knowing who a user is must be followed by verifying what they are allowed to access. The practical lab demonstrated the importance of looking beyond the visible URL — the vulnerable endpoint was a background API call visible only in the Network tab, which reflects how real-world IDOR vulnerabilities are distributed. The vulnerability was trivially exploitable once found: change a number, receive another user's data. Defending against IDOR requires a consistent architectural discipline — every server-side request that retrieves data based on a user-supplied identifier must include an authorisation check verifying that the current session's user owns or is permitted to access that specific object.
