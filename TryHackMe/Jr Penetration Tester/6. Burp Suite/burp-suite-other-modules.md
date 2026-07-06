# Burp Suite: Other Modules

## Overview

This room covers four supporting modules within Burp Suite that complement the core Proxy/Repeater/Intruder workflow: Decoder, Comparer, Sequencer, and Organizer. Each module addresses a specific analysis need — data transformation, response comparison, token entropy analysis, and workflow organisation. Understanding these tools rounds out Burp Suite competency and provides capabilities that directly inform vulnerability identification and exploitation decisions.

**What this room teaches:**
- How to encode, decode, and hash data using Decoder
- How to compare two HTTP responses side-by-side using Comparer
- How to analyse the randomness of session tokens and CSRF tokens using Sequencer
- How to store and annotate interesting requests using Organizer

**Main objectives:**
- Use Decoder to manually transform data between URL, HTML, Base64, ASCII Hex, and other encodings
- Generate and verify hash values in Decoder
- Use Comparer to visually diff two login responses
- Capture and analyse a token's entropy using Sequencer's live capture
- Understand what effective entropy means for token security

**Skills introduced:** Data encoding/decoding, hash generation, response diffing, entropy analysis, workflow documentation

---

## Concepts Covered

### Concept 1 — Burp Suite Decoder

**Definition:** Decoder is a data transformation module that encodes, decodes, and hashes data using a wide range of schemes. It allows both manual data manipulation and intelligent auto-detection of encoding formats.

**Why It Matters:** Web applications constantly encode and transform data — URL encoding in query strings, Base64 in cookies and API tokens, HTML entity encoding in responses, and hashing in authentication mechanisms. A pentester needs to quickly transform data in both directions to understand what is being transmitted and to craft valid payloads.

**How It Works:** Input data is entered into the top field. The chosen transformation is applied and the output is shown in a new row below. Multiple transformations can be chained — each output becomes the input for the next operation.

---

### Concept 2 — Encoding and Decoding Schemes

| Scheme | Description | Common Use in Web Security |
|--------|-------------|--------------------------|
| **Plain** | Raw text — no transformation | Baseline reference |
| **URL** | Substitutes characters with `%XX` hex codes (ASCII value in hex) | Query strings, form data, path parameters |
| **HTML** | Replaces special chars with `&code;` or `&#xHH;` entities | Preventing XSS in HTML output; detecting XSS filter bypasses |
| **Base64** | Converts arbitrary binary data to ASCII-safe alphanumeric string | Cookies, API tokens, JWT headers/payloads, basic auth headers |
| **ASCII Hex** | Converts between ASCII text and hexadecimal representation | Encoding payloads for injection attacks; binary analysis |
| **Hex** | Numeric decimal ↔ hexadecimal conversion | Binary data analysis |
| **Octal** | Numeric ↔ base-8 | Unusual encoding in payloads; filter bypass |
| **Binary** | Numeric ↔ binary | Low-level bit analysis |
| **Gzip** | Compresses/decompresses data | Examining compressed HTTP responses; crafting compressed payloads |

**URL Encoding Example:**
- Forward slash `/` → ASCII code 47 → hex 2F → URL encoded: `%2F`
- Used to bypass WAF rules that block `/` in path parameters

**Base64:**
- The most common encoding for tokens, session data, and JWT components
- `base64 -d` (Linux) or Decoder's "Decode as Base64" instantly reveals the plaintext value

**Chaining Encodings:**
Multiple transformations can be stacked — each applied sequentially. For example, converting "Burp Suite Decoder" to ASCII Hex then to Octal produces a double-encoded representation. This is directly applicable to WAF bypass attempts where single-encoding is detected and filtered.

**Smart Decode:**
Smart Decode automatically detects and recursively reverses applied encodings — similar to CyberChef's Magic mode. Useful when encountering a value of unknown encoding. Input `&#x42;&#x75;&#x72;&#x70;&#x20;&#x53;&#x75;&#x69;&#x74;&#x65;` and Smart Decode recognises it as HTML entities and returns the plaintext.

---

### Concept 3 — Hashing in Decoder

**Definition:** Hashing is a one-way transformation of arbitrary data into a fixed-size digest. The same input always produces the same output; different inputs (even by one bit) produce completely different outputs. It is computationally infeasible to reverse a hash to its original input.

**Why It Matters:**
- **Integrity verification:** Any modification to a file or message changes its hash — detect tampering
- **Password storage:** Applications store password hashes, not plaintext — a breach exposes hashes, not passwords
- **Digital signatures:** Documents are signed by hashing and then encrypting the hash with a private key

**Secure Hash Algorithms:**

| Algorithm | Output Size | Status |
|-----------|-------------|--------|
| SHA-224, SHA-256, SHA-384, SHA-512 | 224/256/384/512 bits | Secure |
| RIPEMD-160 | 160 bits | Secure |
| MD5 | 128 bits | **Deprecated — broken** |
| SHA-1 | 160 bits | **Deprecated — broken** |

**Why MD5 and SHA-1 Are Broken:** Collision attacks exist — it is computationally feasible to find two different inputs that produce the same hash. This invalidates integrity checking and allows crafting files that match a known hash.

**Generating Hashes in Decoder:**
1. Enter data in the input field
2. Click "Hash" dropdown → select algorithm
3. Output displays in hex view (standard representation)
4. Apply "ASCII Hex" encoding to produce the familiar hex string format

**Example:** Input "MD5sum" → Hash MD5 → encode result as ASCII Hex → produces `4ae1a02de5bd02a5515f583f4fca5e8c`

---

### Concept 4 — Burp Suite Comparer

**Definition:** Comparer allows side-by-side comparison of two data sets — HTTP responses, request bodies, or any text/binary data — highlighting differences at either the word or byte level.

**Why It Matters:** When performing brute-force or credential-stuffing attacks in Intruder, all responses may have the same status code. Comparer allows visual inspection of two responses to understand exactly where they differ — pinpointing why one response indicates success and another failure.

**Interface Sections:**
1. **Left panel (data sets):** Two rows — each holds one data set to compare. Can be loaded via paste, file, or "Send to Comparer" from other modules.
2. **Upper right:** Paste, Load, Remove, Clear buttons for managing data sets.
3. **Lower right:** "Words" or "Bytes" comparison buttons — trigger the diff view.

**Diff Colour Coding:**
- **Modified** regions are highlighted in one colour
- **Deleted** content in another
- **Added** content in a third

**Comparison Modes:**
- **Words:** Highlights differences at the word level — useful for text responses
- **Bytes:** Highlights byte-level differences — useful for binary data or small character changes

**Sync Views:** When checked, both panels maintain the same format (both text or both hex simultaneously).

---

### Concept 5 — Burp Suite Sequencer

**Definition:** Sequencer analyses the entropy (randomness) of tokens — session cookies, CSRF tokens, password reset tokens — to determine whether they are generated with sufficient cryptographic randomness to be unpredictable.

**Why It Matters:** If a token is predictable or follows a pattern, an attacker can calculate future token values:
- **Session cookies:** Predict another user's session and take it over
- **Password reset tokens:** Predict a reset token sent to another user and hijack their account
- **CSRF tokens:** If predictable, cross-site request forgery protections are ineffective

**Token Analysis Modes:**

| Mode | Description | When to Use |
|------|-------------|------------|
| **Live Capture** | Burp makes the same request thousands of times, collecting token samples automatically | Standard analysis — Sequencer sends the request and collects responses |
| **Manual Load** | Pre-generated token list loaded directly | When you already have a large token sample set |

**Entropy Measurement:**
Entropy is measured in **bits**. A token with N bits of effective entropy means an attacker would need to try on average 2^(N-1) values to guess a valid token.

- **High entropy (>= 100 bits):** Token is practically unpredictable
- **Low entropy (< 64 bits):** Token may be guessable, especially with scripted attacks
- The **significance level** (e.g., 1%) means there is 99% confidence in the entropy estimate

**Report Sections:**
1. **Summary:** Overall result, effective entropy, reliability (significance level), sample details
2. **Character-level analysis:** Distribution of characters across all positions
3. **Bit-level analysis:** Statistical tests on individual bits across all samples

---

### Concept 6 — Burp Suite Organizer

**Definition:** Organizer is a request storage and annotation module — a bookmarking system for interesting HTTP requests discovered during a test.

**Why It Matters:** During a penetration test, testers discover dozens of interesting requests worth revisiting. Without a way to store and annotate them, findings get lost in the Proxy history. Organizer creates an organised, searchable, annotated record.

**Key Characteristics:**
- Requests are **read-only copies** — preserving the exact state at the time they were added
- Send to Organizer: `Ctrl+O` or right-click → Send to Organizer
- Each entry records: index, timestamp, workflow status, source module, HTTP method, host, URL path, query string, parameter count, response status code, response length, and custom notes

**Practical Use Cases:**
- Store requests that confirm a vulnerability — for inclusion in the report
- Bookmark endpoints worth deeper investigation later
- Track which requests have been tested and what the outcome was
- Maintain a clean audit trail across a long engagement

---

## Methodology

### Decoder Workflow

```
Encounter encoded data (cookie, parameter, header value)
         |
         v
Paste into Decoder input field
         |
         v
Try Smart Decode first — auto-detects common encodings
         |
         v
If Smart Decode fails, try likely formats:
  Is it padded alphanumeric? → Try Base64
  Does it start with %? → Try URL decode
  Is it &amp; / &#x? → Try HTML decode
  Is it hex characters only? → Try ASCII Hex decode
         |
         v
Chain transformations if needed (e.g., Base64 → then URL decode)
         |
         v
Use the decoded value in payload construction or analysis
```

### Comparer Workflow for Login Responses

```
Perform two login attempts in Repeater:
  Attempt 1: Known-invalid credentials
  Attempt 2: Potentially valid credentials
         |
         v
Right-click response 1 → Send to Comparer
Right-click response 2 → Send to Comparer
         |
         v
In Comparer: select both rows → click Words (or Bytes)
         |
         v
Identify differences:
  Different redirect destination? → successful login
  Different error message? → different failure reason
  Different body content? → different server response path
```

### Sequencer Workflow

```
Identify a token-generating endpoint (login, admin form, password reset)
         |
         v
Capture the request in Proxy → right-click → Send to Sequencer
         |
         v
In Sequencer: Token Location section
  Select "Form field" if token is in form body
  Select "Cookie" if token is a cookie
  Choose the specific field from dropdown
         |
         v
Click "Start Live Capture"
Wait for ~10,000 tokens (more = more accurate)
         |
         v
Click Pause → Analyze Now
         |
         v
Review Summary:
  Effective entropy >= 100 bits? → Token is secure
  Effective entropy < 64 bits? → Token may be predictable → investigate
  Review character-level and bit-level analysis for patterns
```

---

## Practical Activities

### Activity 1 — Comparing Login Responses with Comparer

**Objective:** Use Comparer to visually identify the differences between a failed and a successful login response.

**Actions:**
1. Navigate to `http://TARGET_IP/support/login`
2. Submit invalid credentials (any username/password) — capture in Proxy
3. Send request to Repeater (`Ctrl+R`)
4. Send the request from Repeater — right-click response → Send to Comparer
5. In Repeater, change credentials to:
   - Username: `support_admin`
   - Password: `w58ySK4W`
6. Send again — right-click new response → Send to Comparer
7. In Comparer: select both rows → click Words

**Findings:** The diff view highlights the exact differences between the two responses — the redirect destination URL, body content, and potentially error message text. The valid credentials produce a different response path, confirming authentication success even if both return HTTP 302.

**Why It Matters:** Understanding exactly how a server differentiates successful from failed authentication informs the selection of the correct differentiator for Intruder analysis. Comparer makes this visual and immediate.

---

### Activity 2 — Token Entropy Analysis with Sequencer

**Objective:** Analyse the `loginToken` CSRF token in the admin login form to determine whether it is generated with sufficient cryptographic randomness.

**Actions:**
1. Browse to `http://TARGET_IP/admin/login/` — capture the GET request in Proxy
2. Right-click → Send to Sequencer
3. In "Token Location Within Response": select **Form field** → choose `loginToken` from dropdown
4. Click **Start Live Capture**
5. Allow ~10,000 tokens to be captured
6. Click Pause → click **Analyze Now**

**Findings from the Analysis Report:**
- **Overall result:** Tokens appear securely generated
- **Effective entropy:** ~117 bits
- **Significance level:** 1% (99% confidence in the result)
- **Sample:** ~10,000 tokens

**Why It Matters:** An effective entropy of 117 bits means an attacker would need to make approximately 2^116 attempts to guess a valid token — computationally infeasible. This confirms the CSRF token implementation is cryptographically sound. If the entropy were low (e.g., 20–30 bits), the token would be practically guessable, allowing CSRF protection bypass.

**Important Caveat:** High entropy from Sequencer is a strong indicator but not absolute proof. The analysis is probabilistic (99% confidence, not 100%). Other factors — predictable seeding, timing-based patterns — may not be detected by entropy analysis alone. Further investigation with character-level and bit-level reports is recommended when summary results raise concerns.

---

## Observations and Analysis

**Encoding is not Encryption:** URL encoding, Base64, and HTML encoding are transformations, not protections. Anyone with knowledge of the scheme can reverse them instantly. Data encoded in Base64 in a cookie or header is not secret — it is simply a formatting choice.

**MD5 and SHA-1 in Production:** If Decoder reveals that a web application is producing MD5 hashes for passwords or integrity checks, this is a directly reportable vulnerability. Both are cryptographically broken and should be replaced with SHA-256 or stronger.

**Chained Encoding for Bypass:** Real-world WAF bypass attempts often use chained encoding — double URL encoding, Base64 inside URL encoding, etc. The ability to chain transformations in Decoder makes payload construction for these scenarios straightforward.

**Low Token Entropy = Predictable Sessions:** If Sequencer reports low effective entropy for session cookies, an attacker with a large sample of cookies could potentially predict other users' session values — enabling session hijacking without credential theft. This is a critical finding.

**Comparer vs Manual Review:** For large responses, manually finding the difference between two 3KB HTML pages is impractical. Comparer's byte-level diff is far more reliable and faster — critical when response differences are minor (e.g., a single word change in an error message).

---

## Tools and Technologies Used

### Decoder
- **Purpose:** Encoding, decoding, and hashing data in multiple formats
- **Common Usage:** Analysing cookies, tokens, and parameters; crafting encoded payloads; hash verification
- **In This Room:** URL encoding demonstration, HTML entity decoding, Base64 operations, MD5/SHA hashing, Smart Decode

### Comparer
- **Purpose:** Side-by-side diff of two HTTP responses or data sets
- **Common Usage:** Differentiating successful from failed login responses; comparing before/after modifications
- **In This Room:** Comparing valid vs invalid login responses to identify the differentiating factor

### Sequencer
- **Purpose:** Statistical entropy analysis of tokens (session cookies, CSRF tokens, reset tokens)
- **Common Usage:** Evaluating whether token generation is cryptographically secure
- **In This Room:** Live capture and entropy analysis of the `loginToken` CSRF token

### Organizer
- **Purpose:** Storing and annotating interesting requests during a test
- **Common Usage:** Bookmarking confirmed vulnerability requests; managing workflow across long engagements
- **In This Room:** Conceptual introduction — request storage, workflow status tracking, note-taking

---

## Key Learnings

- Encoding ≠ encryption — Base64 and URL encoding are trivially reversible and provide no confidentiality
- Smart Decode auto-identifies common encoding schemes — use it as a first pass on unknown values
- MD5 and SHA-1 are broken — any application using them for integrity or password hashing has a reportable vulnerability
- Hash output is binary — it must be ASCII Hex encoded to produce the familiar hexadecimal string representation
- Comparer saves significant time when identifying the single differentiator between two large similar responses
- Sequencer entropy results are probabilistic — 99% confidence, not 100% certainty
- Effective entropy >= 100 bits is the practical threshold for token security
- Low entropy in session tokens or password reset tokens = immediate, critical investigation priority
- Organizer preserves point-in-time copies — changes to the live application after capture don't affect stored requests

---

## Real-World Relevance

**Penetration Testing:** Decoder is used constantly — inspecting JWT tokens (Base64 decode the header and payload), analysing encoded parameters for SQLi bypass, verifying hash algorithms in use. Sequencer is directly relevant when assessing session management security.

**Bug Bounty Hunting:** Finding a low-entropy password reset token is a high-severity finding. If reset tokens follow a predictable pattern (timestamp-based, sequential), an attacker could generate a valid reset link for another user's account without ever having access to their email.

**Security Engineering:** These modules map directly to developer mistakes — using deprecated hash functions, predictable token generation, not using cryptographically secure random number generators (CSPRNG) for tokens.

---

## Things Worth Remembering

- Decoder shortcut: `Ctrl+Shift+D` (or navigate to Decoder tab)
- Send to Comparer: right-click response → Send to Comparer
- Send to Organizer: `Ctrl+O` or right-click → Send to Organizer
- Smart Decode: auto-detects encoding — use as first pass on unknown values
- MD5 deprecated 2023; SHA-1 deprecated — use SHA-256 or stronger
- Base64 = encoding not encryption — provides no security
- URL encoding: `/` → `%2F`, space → `%20`, `+` → `%2B`
- Hash output must be converted to ASCII Hex to produce readable hex string
- Sequencer: select "Form field" for CSRF tokens in form bodies; "Cookie" for session cookies
- Effective entropy >= 100 bits = secure token; < 64 bits = investigate further
- Comparer: Words mode for text analysis; Bytes mode for binary or subtle character differences
- Organizer requests are read-only snapshots — preserved even if the original request is modified
