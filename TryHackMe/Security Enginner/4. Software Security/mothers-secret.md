# Mother's Secret

## Overview

Mother's Secret is a CTF-style application security challenge set aboard the fictional TryHackMe Cargo Star Ship (THMCSS) Nostromo, owned by Weyland-TryHackMe Corps. The challenge centres on the ship's computer system — MU-TH-UR 6000 (a.k.a. "Mother") — which runs a web application with several exploitable security flaws. The core skill tested is code analysis: a source file is provided at the start, and understanding that code is the key to exploiting the vulnerabilities correctly.

The attack chain involves reading and understanding the provided source code, exploiting a route-chaining authentication bypass to elevate from Crew Member to Science Officer role, extracting a flag hidden in obfuscated JavaScript, and finally leveraging a path traversal / LFI vulnerability to read a sensitive file from the server.

**Room URL:** `tryhackme.com/room/codeanalysis`

---

## Topics Covered

- Source code analysis (Express.js / Node.js)
- Authentication bypass via route chaining
- JavaScript obfuscation and base64 encoding
- Path traversal / Local File Inclusion (LFI)
- Role-based access control weaknesses

---

## Key Concepts

### Code Analysis as the Primary Skill

The room begins by providing a source code file (an Express router definition) rather than starting with a blank target. Reading and understanding this code reveals all three vulnerabilities. This is the central lesson: understanding the code tells you exactly where to attack before touching the application.

**The provided code defines:**
- A POST route `/nostromo` — reads a file specified in the `file_path` request body and sets an authentication flag on the session
- A POST route `/nostromo/mother` — also reads a `file_path` from the request body, but first checks that the `/nostromo` authentication flag is set before serving the file content

This design creates a route dependency: visiting `/nostromo/mother` requires the session to have been authenticated via `/nostromo` first.

---

### Vulnerability 1 — Authentication Bypass via Route Chaining

**Definition:** The server uses a session flag to track whether a user has "authenticated" via the `/nostromo` route. This flag-based check can be satisfied by sending a `/nostromo` request in the same session before accessing `/nostromo/mother` — there is no actual credential check.

**Why It Works:** The authentication check is a simple boolean flag in the session object. Any POST request to `/nostromo` sets the flag. A user-controlled `file_path` parameter is also accepted and read — there is no validation of what path is supplied.

**Exploitation Process:**
1. Start the machine and navigate to the MU-TH-UR 6000 web interface
2. Download and read the provided source code / Operating Manual
3. Send a POST request to `/nostromo` with a `file_path` body parameter — this sets the authentication flag
4. Immediately send a POST request to `/nostromo/mother` with a `file_path` body parameter — the flag is now set, so the check passes
5. The response returns the file content specified in `file_path`

**Tool of choice:** Burp Suite (intercept and replay the POST requests), or curl:
```bash
# Step 1 — Set the authentication flag
curl -s -X POST http://MACHINE_IP/nostromo \
  -H "Content-Type: application/json" \
  -d '{"file_path": "/etc/hostname"}' \
  -c cookies.txt

# Step 2 — Access the protected route with the session cookie
curl -s -X POST http://MACHINE_IP/nostromo/mother \
  -H "Content-Type: application/json" \
  -d '{"file_path": "/etc/hostname"}' \
  -b cookies.txt
```

The `-c` and `-b` flags save and reuse the session cookie so the authentication flag persists between requests.

---

### Vulnerability 2 — Flag in Obfuscated JavaScript

**Why It Happens:** One of the flags is embedded in the client-side JavaScript, encoded in base64 and/or obfuscated to make it non-obvious. This is a common hiding technique in CTF web challenges.

**Exploitation Process:**
1. Navigate to the web application in the browser
2. Open Developer Tools → Sources (Debugger) tab
3. Locate the JavaScript files loaded by the application
4. Use the Pretty Print (`{}`) button to deobfuscate minified code
5. Look for base64-encoded strings — identifiable by their padded alphanumeric format
6. Decode the string using a tool or command:

```bash
echo "[ENCODED_STRING]" | base64 -d
```

The decoded output contains the flag.

---

### Vulnerability 3 — Path Traversal / Local File Inclusion (LFI)

**Definition:** The `file_path` parameter accepted by both `/nostromo` and `/nostromo/mother` routes is passed directly to a file-read function with no path sanitisation. An attacker can supply absolute paths or relative `../` traversal sequences to read arbitrary files on the server.

**The Target File:** The challenge objective is to read the contents of `/opt/m0th3r` — a file on the server representing Mother's secret.

**Exploitation Process:**
1. With the authentication flag set (from Vulnerability 1 above), send a POST to `/nostromo/mother`
2. Set `file_path` to the absolute path of the target file:

```bash
curl -s -X POST http://MACHINE_IP/nostromo/mother \
  -H "Content-Type: application/json" \
  -d '{"file_path": "/opt/m0th3r"}' \
  -b cookies.txt
```

3. The server reads and returns the contents of `/opt/m0th3r` — which contains the final flag

**Why It Works:** The route reads `fs.readFile(req.body.file_path, ...)` (or equivalent) without validating that the path is within an allowed directory. No path canonicalisation, no allowlist, no prefix check.

---

## Workflow / Process

```
Download and read the provided source code:
  Understand the /nostromo route — sets auth flag, reads file_path
  Understand the /nostromo/mother route — checks auth flag, reads file_path
  Note: no credential validation — flag is set by any POST to /nostromo
        |
        v
Navigate to the web interface:
  Access http://MACHINE_IP
  Observe the MU-TH-UR 6000 "Mother" UI
  Current role: Crew Member — limited access
        |
        v
Flag 1 — Find flag in obfuscated JavaScript:
  Open DevTools → Debugger/Sources
  Locate JS files → Pretty Print
  Find base64 string → decode it
        |
        v
Flag 2 — Authentication bypass:
  POST to /nostromo with any file_path → sets auth flag in session
  POST to /nostromo/mother with file_path of a readable file
  Session now has elevated access (Science Officer level)
        |
        v
Flag 3 — Path traversal:
  POST to /nostromo/mother with file_path = /opt/m0th3r
  Server reads and returns the file content
  Retrieve the final flag from the response
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Code Analysis | Reading and understanding source code to identify exploitable logic |
| Route Chaining | Exploiting a dependency between two API routes to satisfy an authentication check |
| LFI | Local File Inclusion — reading arbitrary local files via a user-controlled file path parameter |
| Path Traversal | Using `../` or absolute paths to read files outside the intended directory |
| Session Flag | A boolean value stored in a server-side session used (incorrectly here) as an authentication check |
| Base64 Encoding | A text encoding scheme often used to hide data in client-side scripts |
| Express Router | Node.js routing module — defines how the application responds to different HTTP endpoints |

---

## Real-World Relevance

- Path traversal vulnerabilities in file-serving endpoints are consistently found in web application penetration tests — any `file_path`, `filename`, or `document` parameter is a candidate
- Session flag-based authentication (checking a session variable rather than validating a credential) is an anti-pattern seen in legacy and quickly-built applications
- Hardcoded or obfuscated secrets in client-side JavaScript are a common finding — tools like `trufflehog` and manual JS review can find them
- Code review (as the room demonstrates) is often faster than blind enumeration — reading the source reveals the attack chain immediately when code is available

---

## Key Learnings

- Source code analysis reveals attack paths that would take far longer to discover through blind testing
- Authentication checks based on session flags (not credentials) can be satisfied by manipulating the request sequence
- File path parameters with no sanitisation allow reading arbitrary server files — always validate against an allowlist or use a path-canonicalisation check
- Secrets in client-side JavaScript (even obfuscated) are accessible to anyone who opens DevTools

---

## Additional Notes

- The room is based in the Alien franchise universe — MU-TH-UR 6000 is the ship's computer in the original film, and "Mother's Secret" is a reference to the company's hidden orders stored in the computer
- The challenge is categorised under Software Security Training because the primary skill is code analysis — not network exploitation
- This type of challenge is representative of code-assisted penetration testing engagements where developers provide source code to speed up the assessment

---

## Conclusion

Mother's Secret demonstrates the value of source code analysis as an attack planning tool. Reading the provided Express router code reveals all three vulnerabilities before a single request is sent: an auth bypass via route sequencing, a base64 flag in obfuscated JavaScript, and an unsanitised file path parameter enabling path traversal to read `/opt/m0th3r`. The room reinforces that code-level access changes the attacker's speed and precision — understanding what the application does is half the work of exploiting it.
