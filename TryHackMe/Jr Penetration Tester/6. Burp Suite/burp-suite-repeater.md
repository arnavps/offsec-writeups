# Burp Suite: Repeater

## Overview

Burp Suite Repeater is a manual request manipulation tool that allows testers to capture, modify, and resend HTTP requests to a target as many times as needed. It is one of the most frequently used modules in a web application penetration test — sitting between passive observation (Proxy) and automated attacks (Intruder). Repeater gives the tester precise, surgical control over individual requests, making it ideal for vulnerability confirmation, manual injection testing, and endpoint behaviour analysis.

**What this room teaches:**
- The purpose and interface layout of Burp Repeater
- How to send requests from Proxy to Repeater and modify them
- How to use the Inspector for structured request editing
- How Repeater is applied to real vulnerability testing scenarios
- Manual Union-based SQL injection through Repeater

**Main objectives:**
- Understand Repeater's role in the pentesting workflow
- Perform header manipulation via Repeater
- Trigger server errors through endpoint parameter manipulation
- Execute a manual Union SQLi attack to retrieve data from a database

**Skills introduced:** Manual HTTP request manipulation, response analysis, endpoint validation testing, Union SQL injection, structured request editing with Inspector

---

## Concepts Covered

### Concept 1 — Burp Suite Repeater

**Definition:** Repeater is a Burp Suite module that allows testers to manually craft, edit, and resend HTTP/S requests to a target, observing the server's response each time.

**Why It Matters:** Automated scanners identify patterns but cannot reason about context. Repeater fills this gap — a tester can iteratively probe a single endpoint with modified payloads, observe exactly how the server responds, and refine the approach based on each result. This is the core of manual web application testing.

**Key Details:**

The Repeater interface has six sections:

| Section | Location | Purpose |
|---------|---------|---------|
| Request List | Top left | Manages multiple concurrent Repeater tabs |
| Request Controls | Below request list | Send, cancel, navigate request history |
| Request / Response View | Main area | Edit requests; view server responses |
| Layout Options | Top right of view area | Switch between horizontal, vertical, or tabbed layout |
| Inspector | Right panel | Structured breakdown of request/response components |
| Target | Above Inspector | IP/domain to send requests to; auto-populated from Proxy |

**Practical Relevance:** Repeater is used in every web application penetration test. It is the tool of choice for:
- Manually confirming vulnerabilities found by scanners
- Testing injection payloads iteratively
- Bypassing WAF filters through payload variation
- Analysing how changing headers or parameters affects server behaviour

---

### Concept 2 — Response Display Modes

**Definition:** Repeater offers four ways to view server responses.

| Mode | Description | When to Use |
|------|-------------|------------|
| **Pretty** | Formatted output with minor readability enhancements | Default; sufficient for most analysis |
| **Raw** | Exact unmodified server response | When checking precise byte sequences |
| **Hex** | Byte-level hexadecimal representation | Binary files, non-printable character analysis |
| **Render** | Browser-rendered visual representation | Checking how a response looks in a real browser |

The **Show non-printable characters** (`\n`) button reveals carriage returns (`\r\n`) and line feeds that are significant in HTTP header parsing — useful when debugging header-related issues.

**Why It Matters:** Different response views reveal different things. A redirect that looks innocuous in Pretty mode may contain interesting data in Raw; a binary file is only inspectable in Hex.

---

### Concept 3 — The Inspector Panel

**Definition:** Inspector provides a structured, tabular view of request and response components, allowing editing without manually touching raw HTTP syntax.

**Editable sections in Inspector:**

| Section | Contents | Editable |
|---------|---------|---------|
| Request Attributes | Method, URL path, HTTP version | Yes |
| Request Query Parameters | URL query string parameters (`?key=value`) | Yes |
| Request Body Parameters | POST body parameters | Yes |
| Request Cookies | Cookies sent with the request | Yes |
| Request Headers | All request headers | Yes |
| Response Headers | Headers returned by server | No (read-only) |

**Why It Matters:** Inspector prevents typos and formatting errors when modifying requests. Changes made in Inspector are immediately reflected in the raw view and vice versa — both views are always in sync.

---

### Concept 4 — Union-Based SQL Injection

**Definition:** Union SQL injection is a technique that appends a `UNION SELECT` statement to a legitimate SQL query to retrieve data from other tables or columns, delivered back to the user within the normal application response.

**Requirements for Union SQLi:**
- The original query must return at least one row
- The injected UNION SELECT must have the same number of columns as the original query
- The data types must be compatible

**Practical Relevance:** Union SQLi is one of the most impactful injection types — it can retrieve data from any accessible table in the database, including credential tables and internal notes.

---

## Methodology

### Repeater Workflow

```
1. Identify a request of interest via normal browsing (Proxy passively captures all traffic)
         |
         v
2. Send the request to Repeater (Ctrl+R or right-click → Send to Repeater)
         |
         v
3. Observe the baseline response — understand what a normal response looks like
         |
         v
4. Modify specific parameters, headers, or values in the Request view or Inspector
         |
         v
5. Send the modified request — compare the new response to the baseline
         |
         v
6. Iterate: refine payloads based on response differences
         |
         v
7. Confirm a vulnerability or information disclosure — document findings
```

Each iteration builds on the previous. The key discipline is methodical change — modify one variable at a time so that response differences are attributable to specific changes.

---

## Practical Activities

---

### Activity 1 — Header Manipulation with Repeater

**Objective:** Demonstrate basic Repeater usage by adding a custom header to a request and observing the server's response to an unexpected header value.

**Actions:**
1. Browse to `http://TARGET_IP/` with Burp Proxy active
2. Capture the GET request in the Proxy
3. Right-click the request → Send to Repeater (or `Ctrl+R`)
4. Click Send once to confirm the baseline response (HTML source visible in Response view)
5. View the response in Hex mode to observe byte-level representation
6. In Inspector → Request Headers, add a new header:
   - Header name: `FlagAuthorised`
   - Header value: `True`
7. Send the modified request

**Command / Raw Request (modified):**
```http
GET / HTTP/1.1
Host: TARGET_IP
User-Agent: Mozilla/5.0 ...
Connection: close
FlagAuthorised: True
```

**Findings:** The server responded differently when the custom `FlagAuthorised: True` header was present — a flag was returned in the response body.

**Why It Matters:** Web servers and application frameworks sometimes implement access control or feature flags via non-standard HTTP headers. Security misconfigurations like this — where a header alone unlocks privileged content — represent a broken access control vulnerability. In production environments, similar patterns appear with headers like `X-Internal-Request`, `X-Admin`, or `X-Forwarded-For`.

---

### Activity 2 — Endpoint Input Validation Testing

**Objective:** Test whether a numeric endpoint parameter is properly validated by the server — triggering a 500 Internal Server Error through extreme input values.

**Background:** The `/products/ID` endpoint accepts a numeric identifier. Without proper server-side input validation, supplying unexpected values (non-integers, negative numbers, floats, strings) may cause the application to throw an unhandled exception.

**Actions:**
1. Navigate to `http://TARGET_IP/products/` with intercept disabled
2. Click a "See More" link — observe redirect to `/products/3` (numeric endpoint)
3. Capture this request in Proxy → Send to Repeater
4. In Repeater, modify the endpoint path:
   - Try: `/products/0`
   - Try: `/products/10`
   - Try: `/products/-1`
5. Send each variation and observe the HTTP response status code

**Modified Request Example:**
```http
GET /products/-1 HTTP/1.1
Host: TARGET_IP
User-Agent: Mozilla/5.0 ...
Connection: close
```

**Findings:**
- Positive integers returned normal 200 responses with product content
- Setting the ID to `-1` caused a **500 Internal Server Error**
- The error response contained a flag confirming the vulnerability: `THM{N2MzMzFhMTA1MmZiYjA2YWQ4M2ZmMzhl}`

**Why It Matters:** A 500 error triggered by a negative integer confirms absent or inadequate input validation. In production, this type of error can:
- Reveal stack traces and internal application structure
- Indicate that the value reaches a database query or system call without sanitisation
- Be the starting point for more serious injection attacks

Proper input validation should reject non-positive integers, non-integer strings, and out-of-range values before they reach any backend processing.

---

### Activity 3 — Manual Union SQL Injection via Repeater

**Objective:** Identify and exploit a Union SQL injection vulnerability in the `/about/ID` endpoint to retrieve notes stored in the database for a specific user (the CEO).

**Background:** The `/about/2` endpoint retrieves a person's profile from the database using their ID. A Union SQLi vulnerability allows an attacker to append a second SELECT statement to the original query and retrieve data from other columns or tables.

---

#### Step 1 — Confirm the Vulnerability

**Action:** Append a single apostrophe to the ID parameter to break the SQL query syntax.

```http
GET /about/2' HTTP/1.1
Host: TARGET_IP
```

**Findings:** Server returned HTTP `500 Internal Server Error`. The response body contained a critical information disclosure — the raw SQL query was printed in the error:

```html
<h2><code>Invalid statement:
SELECT firstName, lastName, pfpLink, role, bio FROM people WHERE id = 2'
</code></h2>
```

**Why It Matters:** This error message reveals:
- The table name: `people`
- The exact column names: `firstName`, `lastName`, `pfpLink`, `role`, `bio`
- The number of columns: 5

This eliminates the column-counting enumeration phase entirely. Verbose SQL error messages are a serious misconfiguration — they hand the attacker a significant portion of the database schema directly.

---

#### Step 2 — Enumerate Column Names

**Objective:** Identify all columns in the `people` table, specifically looking for a `notes` column.

**Action:** Craft a UNION SELECT query pulling column names from `information_schema.columns`. Set the original ID to `0` to suppress the legitimate row and ensure only injected results are returned.

```
GET /about/0 UNION ALL SELECT column_name,null,null,null,null FROM information_schema.columns WHERE table_name="people" HTTP/1.1
```

**Why ID = 0?** Setting the ID to an invalid value means the original query returns no rows. The first (and only) row returned is from the injected UNION SELECT — this ensures the injected data appears cleanly in the response without being buried under legitimate data.

**Findings:** The page title showed `id` — the first column name. Only one result was visible because the page only rendered the first match.

---

#### Step 3 — Retrieve All Column Names at Once

**Objective:** Use `group_concat()` to amalgamate all column names into a single returned value.

```
GET /about/0 UNION ALL SELECT group_concat(column_name),null,null,null,null FROM information_schema.columns WHERE table_name="people" HTTP/1.1
```

**Findings:** The response revealed all eight columns in the `people` table:
```
id, firstName, lastName, pfpLink, role, shortRole, bio, notes
```

The `notes` column is the target — it was not visible in the original error disclosure, demonstrating why column enumeration is necessary even when some structure is leaked.

---

#### Step 4 — Extract the Target Data

**Objective:** Retrieve the `notes` field for the CEO (ID = 1, confirmed by visiting `/about/` and checking Jameson Wolfe's profile URL).

```
GET /about/0 UNION ALL SELECT notes,null,null,null,null FROM people WHERE id = 1 HTTP/1.1
```

**Findings:** The response contained the CEO's notes, which included the final flag.

**Why It Matters:** This demonstrates a complete Union SQLi exploitation chain:
1. Error-based schema discovery (verbose error messages)
2. Column enumeration via `information_schema`
3. `group_concat()` to handle multi-row results in a single-display context
4. Targeted data extraction from a specific row

In a real assessment, this access level could expose credentials, sensitive internal communications, customer PII, or administrative tokens stored in database fields.

---

## Observations and Analysis

**Verbose SQL Error Messages:** The server returned the raw SQL query in the error response body. This is a critical misconfiguration that should never exist in production. SQL error messages should be suppressed and logged server-side only.

**Missing Input Validation:** The `/products/ID` endpoint accepted negative integers without validation. Any user-controlled value that reaches a database query or system function must be validated against expected type, range, and format.

**SQL Injection via Path Parameter:** The vulnerability existed in a URL path segment (`/about/2'`), not a query string or POST body. Many input validation implementations focus only on form fields and query strings — path parameters are frequently overlooked.

**Column Count Matching:** In Union SQLi, the injected SELECT must have the same number of columns as the original query. Null padding (`null,null,null,null`) fills unused positions. Mismatched column counts cause the query to fail.

**group_concat() as an Extraction Technique:** When a page only renders the first database row, `group_concat()` compresses multiple rows into a single delimited string — a standard technique for efficient enumeration when display slots are limited.

---

## Tools and Technologies Used

### Burp Suite Repeater
- **Purpose:** Manual HTTP request modification and resending
- **Common Usage:** Vulnerability confirmation, injection testing, header manipulation, parameter fuzzing
- **In This Room:** Header manipulation for flag retrieval, endpoint parameter testing, full Union SQLi exploitation chain

### Burp Suite Proxy
- **Purpose:** Intercepting and capturing HTTP/S traffic
- **Common Usage:** Traffic observation, request capture for forwarding to other modules
- **In This Room:** Initial request capture before forwarding to Repeater

---

## Key Learnings

- Repeater's value is in iterative, single-variable testing — change one thing at a time and observe the difference
- The Inspector panel is more reliable than raw editing for structured modifications (headers, cookies, parameters)
- A single apostrophe (`'`) appended to a numeric parameter is the fastest first test for SQL injection
- Verbose SQL error messages expose schema information that should be the attacker's work to discover — suppress them in production
- Setting the original ID to `0` in Union SQLi ensures clean injected output without competing legitimate rows
- `group_concat()` solves the single-result display problem in Union SQLi without requiring multiple separate queries
- Endpoint input validation must cover path parameters, not just form fields and query strings

---

## Real-World Relevance

**Penetration Testing:** Repeater is used in every web application pentest for manual vulnerability confirmation and exploitation. Automated scanners flag potential injection points; Repeater is how they are manually verified and exploited.

**Bug Bounty Hunting:** Union SQLi in path parameters is a commonly missed vulnerability class — it is frequently out of scope for automated scanners that don't test path segments.

**Security Engineering:** The findings from this room directly map to developer training topics — input validation, error handling, parameterised queries, and the principle that user-controlled data must never reach SQL without sanitisation.

---

## Things Worth Remembering

- Send to Repeater: `Ctrl+R` or right-click → Send to Repeater
- First SQLi probe: append `'` to any user-controlled ID parameter
- Union SQLi column count: match with `null` padding — e.g., `UNION SELECT data,null,null,null,null`
- Set ID to `0` (invalid) to suppress legitimate rows and isolate injected output
- `group_concat(column_name)` retrieves all values in one query
- `information_schema.columns WHERE table_name="tablename"` enumerates all columns in a table
- Inspector edits sync instantly with raw view — use whichever is more convenient
- A 500 error from a modified parameter = server-side exception = likely missing validation
- Verbose error messages in production = critical misconfiguration — always document and report
- Response view modes: Pretty (default), Raw (exact), Hex (binary), Render (visual browser-like)
