# SSRF — Server-Side Request Forgery

## Overview

Server-Side Request Forgery (SSRF) is a vulnerability where an attacker manipulates a parameter that causes the application's server to make HTTP requests to a destination of the attacker's choosing. Instead of the attacker's browser making the request, the server makes it — and the server is trusted by internal infrastructure in ways that external users are not.

This room covers SSRF mechanics, the two types (Regular and Blind), SSRF impact, four common vector patterns, identification methodology, and bypass techniques for the three main defences: deny lists, allow lists, and open redirects. A practical lab demonstrates chaining a hidden form field vector with a directory traversal bypass.

**Skills introduced:** SSRF vector identification (full URL parameters, partial URLs, hidden form fields, path traversal), deny list bypass techniques (IP representation variants, DNS rebinding), allow list bypass (subdomain matching, URL credential abuse), open redirect chaining, and base64 decoding exfiltrated content.

---

## Concepts Covered

### SSRF — Core Mechanism

SSRF exploits the trust that internal systems place in the application server. Internal services, databases, and cloud metadata endpoints often accept requests from the server without additional authentication — they assume any request arriving from a trusted internal IP is legitimate. An attacker who controls where the server sends its requests inherits that trust.

```
Normal flow:
  User Browser → Application Server → External Resource

SSRF flow:
  User Browser → Application Server (with SSRF payload) → Internal Service
                                                         → Cloud Metadata
                                                         → Attacker's Server
```

### Two Types of SSRF

| Type | Response Visible? | Exploitation Approach |
|------|-----------------|----------------------|
| **Regular SSRF** | Yes — response returned in HTTP response | Read the output directly; extract data from the response body |
| **Blind SSRF** | No — request made but response not returned | Confirm via callback to attacker-controlled server; infer from timing/errors |

**Blind SSRF confirmation methods:**
- External HTTP logger (requestbin.com) — check for incoming requests
- Burp Collaborator — catches HTTP and DNS callbacks
- Self-hosted listener: `python3 -m http.server`
- Timing analysis — consistent response time difference between reachable and unreachable internal hosts
- Error-based inference — different error messages reveal internal network topology

### SSRF Impact

| Impact | Description |
|--------|-------------|
| **Internal endpoint access** | Admin panels, configuration interfaces not exposed to internet; IP-based access controls bypassed because request originates from the server |
| **Sensitive data exposure** | Backend databases, private APIs, internal tooling that trusts the server's network position |
| **Internal network reconnaissance** | Map internal hosts and services from response differences (status codes, errors, timing) |
| **Cloud metadata theft** | AWS/GCP/Azure expose instance metadata at `169.254.169.254`; retrieve IAM credentials, role details, instance configuration |
| **Credential and token leakage** | Auth tokens between internal services intercepted |

---

### SSRF Vector Patterns

#### 1. Full URL in a Parameter

The application accepts a complete URL as input and makes a server-side request to it:

```
https://website.thm/item/2?server=api
→ Server requests: https://api.website.thm/api/item?id=2

Injected:
https://website.thm/item/2?server=server.website.thm/flag?id=9&x=
→ Server requests: https://server.website.thm/flag?id=9&x=/api/item?id=2
```

The `&x=` technique makes the appended path a harmless extra parameter, neutralising what the application would have appended.

#### 2. Partial URL (Hostname Only)

Application accepts only a hostname and constructs the rest server-side:

```
?server=api.internal → requests https://api.internal/stock/item
?server=attacker.com → requests https://attacker.com/stock/item
```

If the server parameter is not validated against an allowlist, any hostname is redirected to.

#### 3. Path Traversal in URL

User controls only a path segment; directory traversal sequences redirect to other paths:

```
?url=/item/123/details → server requests /item/123/details
?url=/../admin        → server requests /admin
```

Same traversal technique as file inclusion, applied to URL paths.

#### 4. Hidden Form Fields

Not visible in the browser — discoverable only by inspecting page source or intercepting requests:

```html
<input type="hidden" name="avatar" value="/images/avatars/default.png">
```

If the server fetches the resource at whatever path this field contains, changing the value via DevTools or Burp redirects the server's request. This is why inspecting all form fields and POST bodies is essential — the injection point is not always in the URL.

---

### Identifying SSRF

**Strong indicators:**
- Full URL in a query parameter (most obvious)
- Hidden form fields containing paths or URLs
- Partial URL / hostname-only parameter
- Path-only parameters

**Application features frequently containing SSRF vectors:**

| Feature | Why |
|---------|-----|
| Webhook configuration | App makes request to user-supplied URL to verify endpoint |
| PDF/report generation | Server fetches content from user-supplied URL to render |
| URL preview/unfurling | Server retrieves metadata from user-provided link |
| File import by URL | Server downloads file from user-specified remote location |
| Integration settings | Third-party service URLs queried by the server |

---

### Bypass Techniques

#### Deny List Bypass

A deny list blocks specific addresses (localhost, 127.0.0.1) while permitting everything else. These are inherently fragile — the IPv4 loopback has many representations:

| Representation | Value |
|---------------|-------|
| Standard | `127.0.0.1` |
| Decimal | `2130706433` |
| Octal | `017700000001` |
| Shorthand | `127.1` or `0` or `0.0.0.0` |
| IPv6 | `[::1]` |
| DNS-based | `127.0.0.1.nip.io` — resolves to 127.0.0.1 but looks like a normal domain |

**Cloud metadata bypass:** Register a domain with a DNS A record pointing to `169.254.169.254`. The deny list sees a hostname string (no match), permits the request; the server resolves the hostname and connects to the metadata service.

**DNS-based bypass is particularly effective** because the string validation and the actual DNS resolution happen at different points. A string-based check sees a domain name, not a blocked IP.

#### Allow List Bypass

An allow list requires URLs to match an approved pattern (e.g., must begin with `https://website.thm`):

| Bypass Technique | Example | Why It Works |
|-----------------|---------|-------------|
| **Subdomain matching** | `https://website.thm.attackers-domain.thm` | URL string begins with expected prefix; actual hostname is attacker-controlled |
| **URL credentials** | `https://website.thm@attacker.com/` | Some HTTP libraries treat text before `@` as credentials; text after `@` as the hostname. Allow list sees `website.thm`; request goes to `attacker.com` |

Root cause: the application validates the URL string using pattern matching instead of properly parsing it into scheme, credentials, hostname, path components.

#### Open Redirect Chaining

When deny list and allow list bypasses both fail, chain an open redirect on the target domain. An open redirect is an endpoint that forwards visitors to a URL in a parameter:

```
https://website.thm/link?url=https://tryhackme.com
```

**Chain with SSRF allowlist bypass:**
```
https://website.thm/link?url=http://169.254.169.254/latest/meta-data/
```

The allowlist check is satisfied (`website.thm` domain). The server follows the request, hits the open redirect, which forwards to the metadata service. The application's own feature circumvents its own protection.

---

## Methodology

```
1. Map all application inputs that may influence server-side requests:
   URL parameters, POST bodies, hidden form fields, API parameters

2. Test each candidate with an external callback URL:
   Point the parameter at a Burp Collaborator or requestbin URL
   If a callback arrives → SSRF confirmed

3. Identify what internal resources are reachable:
   Try localhost, 127.0.0.1, 169.254.169.254, common internal IPs
   Try common admin paths: /admin, /management, /internal

4. If validation blocks common addresses:
   Try IP encoding alternatives (decimal, octal, DNS-based)
   Try subdomains of the expected domain
   Try URL credential abuse (@hostname)
   Look for open redirects on the target domain to chain

5. Decode exfiltrated content:
   Responses may be base64-encoded (especially when embedded in page source)
   echo 'BASE64' | base64 -d
```

---

## Practical Activity — Hidden Form Field + Deny List Bypass

**Objective:** Access the `/private` endpoint (restricted to server-only access) by exploiting an SSRF vulnerability in the avatar selection feature.

### Step 1: Locate the Vector

After creating an account and navigating to `/customers/new-account-page`, inspect the page source. Avatar options are radio buttons whose `value` attribute contains image paths:

```html
<input type="radio" name="avatar" value="/images/avatars/cat.png">
```

The server uses this path to fetch the resource when "Update Avatar" is submitted.

### Step 2: Observe the Server's Behaviour

Select an avatar and click Update Avatar. Inspect the updated page source. The avatar is rendered as a base64-encoded data URI:

```html
<img src="data:image/png;base64,iVBORw0KGg...">
```

**Key insight:** The server is fetching the path from the form field, reading the response, and embedding it as base64 in the page. If the server can be directed to fetch `/private` instead of an image, the contents of `/private` will appear as base64 data in the page source.

### Step 3: Test Direct Access

Using browser DevTools, change the radio button's `value` attribute from the image path to `private`, then select it and click Update Avatar.

**Result:** Error — "The path cannot start with /private"

A deny list is blocking paths that begin with `/private`.

### Step 4: Bypass with Directory Traversal

Change the radio button's value to: `x/../private`

**Why this bypasses the deny list:**

| Stage | Path | Explanation |
|-------|------|-------------|
| Input validation | `x/../private` | Deny list checks raw string — does not begin with `/private` → passes |
| Path normalisation | `/private` | Web server resolves `x/` then `../` (up one level) → arrives at `/private` |

The validation and the path normalisation happen at different stages. The deny list sees the raw string before normalisation; the file system sees the normalised path after.

**Result:** The request succeeds. The page source now contains base64-encoded content in the avatar `<img>` tag.

### Step 5: Decode the Flag

Copy the base64 string from the `src` attribute and decode it:

```bash
echo "PASTE_BASE64_HERE" | base64 -d
# Output: THM{SSRF_FLAG_VALUE}
```

**Finding:** The base64-encoded response from `/private` contains the flag. The server fetched the restricted endpoint on the attacker's behalf, embedded the content in the page, and the attacker decoded it.

---

## Observations and Analysis

- The avatar form field was a hidden vector — not visible in the URL bar. Thorough SSRF testing requires inspecting all form fields and POST body parameters, not just URL parameters.
- The deny list checked the raw string before the web server normalised the path. This ordering creates an exploitable gap — the same type of discrepancy that makes path traversal in file inclusion vulnerabilities work.
- The fact that the server base64-encodes the fetched resource and embeds it in the page is both the feature and the vulnerability — it is the mechanism that allows reading the `/private` response.
- The SSRF was Regular (not Blind) because the server's response was returned in the page. Blind SSRF would have required a different confirmation approach — an external callback listener.
- IP allow/deny lists are fundamentally weaker than application-level SSRF protections because they can always be bypassed through DNS resolution, IP encoding, or chaining with open redirects.

---

## Tools and Technologies Used

### Browser Developer Tools
- **Purpose:** Inspect and modify hidden form fields before submitting requests
- **How used:** Used "Inspect Element" to change the `value` attribute of the avatar radio button from an image path to the SSRF payload

### base64 / Command Line
- **Purpose:** Decode base64-encoded exfiltrated content
- **Common usage:** `echo 'BASE64_STRING' | base64 -d`
- **How used:** Decoded the base64 data URI embedded in the page source to reveal the `/private` endpoint's contents

### Burp Collaborator / requestbin
- **Purpose:** Receive callbacks for Blind SSRF confirmation
- **Common usage:** Generate unique domain, embed in SSRF payload, check dashboard for callbacks
- **In this room:** Referenced as the confirmation mechanism for Blind SSRF; not required for the Regular SSRF lab

---

## Key Learnings

- SSRF exploits the server's trusted position in the network — the server can reach internal services that external users cannot
- Regular SSRF returns the response; Blind SSRF requires a callback to confirm
- Four vector patterns: full URL parameter, hostname parameter, path-only parameter, hidden form fields
- Deny lists are fragile: 127.0.0.1 has many representations; DNS names can resolve to blocked IPs
- Allow lists can be bypassed: subdomain prefix matching and URL credential abuse (`@hostname`)
- Open redirects can be chained to satisfy an allow list while still redirecting to an arbitrary destination
- Cloud metadata at `169.254.169.254` is a primary SSRF target in cloud environments — yields IAM credentials
- Always inspect hidden form fields — SSRF vectors are not always visible in the URL bar
- Directory traversal (`x/../private`) can bypass path-based deny lists by exploiting validation-normalisation ordering discrepancy

---

## Real-World Relevance

- **Penetration testing:** SSRF is tested on all features that make server-side HTTP requests. PDF generators, webhook configs, and file import features are the primary targets.
- **Bug bounty:** SSRF to cloud metadata endpoint (`169.254.169.254`) yielding AWS credentials is consistently rated Critical due to the potential for full cloud account compromise.
- **Enterprise environments:** Internal admin panels, databases, and monitoring tools that trust server-originating requests are accessible via SSRF, bypassing network-level controls.
- **Red team operations:** SSRF is used for internal network reconnaissance — systematically probing internal IP ranges to discover services, then leveraging cloud metadata for credential escalation.

---

## Things Worth Remembering

- SSRF makes the server send requests on the attacker's behalf — it inherits the server's trusted network position
- Regular SSRF: response visible; Blind SSRF: need callback to confirm
- Vector patterns: full URL param, hostname param, path param, hidden form fields
- Deny list bypass: `127.0.0.1.nip.io`, decimal `2130706433`, octal `017700000001`, IPv6 `[::1]`
- Allow list bypass: `https://expected.thm.attacker.com/` or `https://expected.thm@attacker.com/`
- Open redirect chain: `https://trusted.thm/redirect?url=http://169.254.169.254/metadata`
- Cloud metadata: `http://169.254.169.254/latest/meta-data/iam/security-credentials/` → IAM temp credentials
- Directory traversal bypass: `x/../target` passes string validation, normalises to `/target`
- Decode base64 exfiltrated responses: `echo 'VALUE' | base64 -d`

---

## Conclusion

SSRF demonstrates that trust boundaries in network architecture can be circumvented when an application accepts attacker-controlled URLs and uses them to make server-side requests. The practical lab reinforced two core offensive skills: recognising that hidden form fields are SSRF vectors that require active discovery, and exploiting the discrepancy between input validation and path normalisation to bypass a deny list using directory traversal. The three bypass categories — deny lists, allow lists, and open redirect chaining — illustrate why perimeter controls around SSRF cannot substitute for proper architectural changes, specifically ensuring that server-side request functionality only reaches intended resources through explicit allowlisting with strict URL parsing.
