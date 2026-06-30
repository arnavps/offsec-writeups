# Traverse

## Overview

Traverse is a web exploitation challenge built around a fictional tourism web application. The scenario follows Bob, a security engineer whose team noticed the site was being attacked regularly after it moved from QA to production. The objective is to find the root causes by exploiting the same vulnerabilities an attacker would use — working through a multi-step attack chain from initial enumeration to OS command injection.

The challenge is notable for its layered attack chain: each finding unlocks the next step. Directory listing reveals a hidden admin area, JavaScript in that area requires deobfuscation to extract credentials, an API endpoint is vulnerable to IDOR allowing extraction of admin credentials, the admin panel is vulnerable to command injection, and a hidden file manager allows restoring a defaced page.

---

## Topics Covered

- Directory enumeration with Gobuster
- JavaScript deobfuscation
- IDOR (Insecure Direct Object Reference) on an API endpoint
- OS command injection via admin panel
- Path traversal via a file manager
- Hidden endpoint discovery

---

## Key Concepts

### Scenario Context

Bob's tourism website is being attacked repeatedly after deployment. The challenge simulates discovering the vulnerabilities that the attacker exploited — and in the process, demonstrates a realistic multi-step web attack chain common in real-world penetration testing engagements.

---

### Step 1 — Directory Enumeration

**What to do:** Run Gobuster against the target to discover endpoints not linked from the main navigation.

```bash
gobuster dir -u http://MACHINE_IP \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

**Key Finding:**
- A `/logs` directory is discovered with directory listing enabled
- The logs directory contains email logs and internal communication between team members
- The logs reference a directory named `/planning` — a hidden admin area created by Bob

**Significance:** The email logs contain contextual information — including the name of a hidden planning directory and a reference to a flag used as a password for accessing it. Directory listing on the logs folder exposes internal correspondence that was never intended to be public.

---

### Step 2 — Accessing the Planning Directory

**What to do:** Navigate to the `/planning` endpoint using the flag/password discovered in the email logs.

**Key Finding:**
- The `/planning` page requires a password — the flag value from the logs serves as the password
- Once authenticated, the planning section is accessible
- Inside the planning area, JavaScript files are present that define additional API endpoints not discovered in the initial Gobuster scan

---

### Step 3 — JavaScript Deobfuscation

**What to do:** Examine the JavaScript loaded by the `/planning` page using browser DevTools or by downloading the JS file.

**Why It Matters:** The planning area's JavaScript is obfuscated or minified. It contains references to an internal API endpoint that handles user data and is not visible from outside the planning section.

**Process:**
1. Open Developer Tools → Sources tab
2. Locate the JavaScript files loaded by `/planning`
3. Use Pretty Print (`{}`) or an external deobfuscation tool to make the code readable
4. Identify the internal API endpoint — typically something like `/api/users` with a `custom_id` parameter

---

### Step 4 — IDOR on the API Endpoint

**Definition:** The internal API endpoint exposed in the JavaScript accepts a `custom_id` parameter and returns user data based on that value. There is no authorisation check — any caller can request any ID.

**Exploitation Process:**
1. Identify the API endpoint from the deobfuscated JavaScript
2. Send requests with incrementing `custom_id` values:

```bash
# Manual request
curl http://MACHINE_IP/api/users?custom_id=1
curl http://MACHINE_IP/api/users?custom_id=2
# ... continue until admin credentials are returned
```

**Or automate with Burp Suite Intruder:**
- Intercept an API call in Burp Proxy
- Send to Intruder
- Set `custom_id` as the payload position
- Use a number list payload from 0 to 100 with step 1
- Run the attack and look for responses with different content lengths — indicating valid data

**Key Finding:** One of the IDs returns the admin user's credentials (username and password). These are used to log into the admin panel.

---

### Step 5 — Command Injection on Admin Panel

**Definition:** After logging into the admin panel with the credentials retrieved from the IDOR, a functionality (typically a diagnostic or ping-style feature) is found that executes OS commands server-side using user-supplied input — without sanitisation.

**Exploitation Process:**
1. Log into the admin panel using the credentials extracted from the IDOR
2. Navigate to the vulnerable functionality (a network diagnostics or system command feature)
3. Test for command injection by appending a secondary command:

```
; whoami
; id
; ls /
```

If the response returns system command output, the injection is confirmed.

**Example (conceptual):**
```
Input: 127.0.0.1; id
Output: uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Significance:** Command injection on an admin panel gives arbitrary code execution as the web server process. The flags for this challenge step are retrieved by reading specific files on the server via command injection.

---

### Step 6 — File Manager and Defaced Page Restoration

**What it is:** A hidden file manager endpoint (also discoverable via enumeration or referenced in the source code) allows reading and writing files on the server.

**Purpose in the challenge:** The website's main page has been defaced by the attacker. Using the file manager, the original page content can be restored — demonstrating the impact of the command injection vulnerability and showing how an attacker modified site content.

**Process:**
1. Discover the file manager endpoint (via source code review or Gobuster)
2. Use the file manager to navigate the web root
3. Restore the defaced `index.html` or equivalent file with the correct content

---

## Workflow / Process

```
Initial enumeration:
  gobuster dir against MACHINE_IP
  Discover /logs directory — directory listing enabled
  Read email logs → find reference to /planning and password hint
        |
        v
Access /planning:
  Use flag from logs as password
  Authenticated access to planning section
        |
        v
JavaScript analysis:
  Download/inspect JS files in /planning
  Deobfuscate → identify internal API endpoint with custom_id parameter
        |
        v
IDOR exploitation:
  Enumerate custom_id values (0–100) via curl or Burp Intruder
  Find ID that returns admin credentials
        |
        v
Admin panel access:
  Log in with recovered admin credentials
  Explore admin functionality
  Identify command execution feature
        |
        v
Command injection:
  Inject OS commands via vulnerable input field
  Confirm execution via output (id, whoami, ls)
  Extract flags from server files
        |
        v
File manager:
  Discover hidden file manager endpoint
  Navigate to web root
  Restore defaced page content
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Gobuster | Directory and endpoint enumeration |
| Browser DevTools | JavaScript deobfuscation and source review |
| Burp Suite | Proxy, Intruder for IDOR enumeration |
| curl | Manual API requests and IDOR testing |
| OS commands | Executed via injection to read flags and enumerate the server |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Directory Listing | Server misconfiguration that exposes the contents of a directory as a browsable list |
| IDOR | Insecure Direct Object Reference — accessing objects by changing an identifier without authorisation checks |
| Command Injection | Injecting OS commands into a server-side call that executes system commands |
| Deobfuscation | Reversing obfuscation techniques to make code readable |
| Hidden Endpoint | An API route or page not linked from the UI but accessible if its path is known |
| File Manager | A web-based tool for browsing and managing server files — dangerous if accessible without strict auth |
| Attack Chain | A sequence of vulnerabilities where exploiting one enables exploitation of the next |

---

## Real-World Relevance

- Multi-step attack chains are the norm in real-world web application penetration tests — finding admin credentials via IDOR and using them for RCE via command injection is a realistic scenario
- Directory listing on a logs or development directory exposing internal emails and file paths is a consistently reported finding
- JavaScript deobfuscation to find hidden API endpoints is a standard technique in mobile app and SPA security testing
- IDOR on user ID enumeration endpoints remains one of the highest-frequency bug bounty findings
- Command injection on admin panels is a critical severity finding with direct RCE impact

---

## Key Learnings

- Gobuster is the starting point — enumerate before manual testing
- Directory listing on non-public directories exposes internal structure and sensitive files
- JavaScript in authenticated sections may reference internal APIs not exposed to unauthenticated users — always check
- IDOR via sequential ID enumeration (0–100 with Burp Intruder) is a quick, high-value check on any endpoint with a numeric user identifier
- Command injection on any user-controlled input that reaches a system call is critical — test with semicolons and pipe characters
- Attack chains multiply impact: IDOR alone gets credentials; credentials plus command injection gets RCE

---

## Additional Notes

- The challenge is part of the Software Security Training path, making it appropriate for understanding how developer mistakes (no authorisation checks, no input sanitisation) translate directly to exploitation
- The scenario framing — Bob investigating his own app — is a good model for how security engineers should think: try to break what you build
- Python scripting (as some writeups mention) can also be used to automate the IDOR enumeration instead of Burp Intruder

---

## Conclusion

Traverse demonstrates a complete, multi-stage web exploitation attack chain on a realistic application. Each step uses a different vulnerability class — directory listing, JavaScript analysis, IDOR, command injection — and each finding is the prerequisite for the next. This chained structure reflects real-world web application assessments, where the most impactful findings are often reached through a sequence of lower-severity discoveries. The room reinforces that individually manageable issues (an exposed log directory, a missing authorisation check, unsanitised input) can combine into a critical attack path when left unaddressed.
