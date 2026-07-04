# Web Server Attacks - Part 1

## Overview

Web server attacks target the underlying infrastructure serving a web application — the web server software itself (Apache, Nginx, IIS), misconfigurations in how it is deployed, and exposed administrative interfaces. This room introduces the foundational techniques for attacking web servers directly, as opposed to attacking the application running on top of them. Understanding how web servers work and how they are commonly misconfigured is the prerequisite for identifying and exploiting server-level vulnerabilities.

Topics in this room span enumeration of web server technologies, exploitation of common misconfigurations, directory traversal in web server contexts, and attacks against exposed server management interfaces.

---

## Topics Covered

- Web server enumeration and fingerprinting
- Common web server misconfigurations
- Directory traversal on web servers
- HTTP verb tampering
- Web server management interface attacks
- File upload vulnerabilities
- Default credentials and exposed panels

---

## Key Concepts

### Web Server Enumeration and Fingerprinting

Before attacking a web server, identify what it is running. Every piece of version information is a lead for CVE research.

**Banner grabbing:**
```bash
# Retrieve headers including Server and X-Powered-By
curl -I http://TARGET_IP

# Verbose — shows full request and response headers
curl -v http://TARGET_IP

# Using Netcat — raw TCP banner grab
nc TARGET_IP 80
HEAD / HTTP/1.0

# Using Nmap's HTTP scripts
nmap -sV -p 80,443 --script=http-headers TARGET_IP
nmap -p 80 --script=http-server-header TARGET_IP
```

**What to look for:**
- `Server:` header — web server software and version (e.g., `Apache/2.4.51`, `nginx/1.18.0`, `Microsoft-IIS/10.0`)
- `X-Powered-By:` header — backend language/framework (e.g., `PHP/7.4.3`, `ASP.NET`)
- `X-AspNet-Version:` — specific .NET version
- Error page formatting — Apache, Nginx, and IIS all have distinct default error page styles
- Default files (`/index.php`, `/index.aspx`, `/default.aspx`) indicate the technology stack

**Nmap service version detection:**
```bash
# Version detection on web ports
nmap -sV -p 80,443,8080,8443 TARGET_IP

# Use HTTP enumeration scripts
nmap -p 80 --script http-enum TARGET_IP
```

---

### Common Web Server Misconfigurations

**1. Directory Listing Enabled**

When the web server is configured to list directory contents when no index file is present, browsing to a directory returns a file listing.

**Risk:** Exposes unlinked files — backups, configuration files, source code, credential files, logs.

**Detection:**
```bash
gobuster dir -u http://TARGET_IP -w wordlist.txt
# Look for directories returning 200 with directory listing indicators
```

**Indicators:** Response body contains `Index of /`, `[DIR]`, or `[PARENTDIR]`.

---

**2. Exposed Configuration Files**

Web servers and applications leave configuration files in web-accessible locations. Common paths:

| File | Potential Contents |
|------|-------------------|
| `.htaccess` | Apache rewrite rules, access controls, password protection |
| `web.config` | IIS configuration, connection strings, sometimes credentials |
| `.env` | Environment variables — database credentials, API keys, secret keys |
| `config.php`, `settings.py`, `application.yml` | Database credentials, secret keys |
| `wp-config.php` | WordPress database credentials |

```bash
# Check for common sensitive files
curl http://TARGET_IP/.env
curl http://TARGET_IP/web.config
curl http://TARGET_IP/.htaccess
```

---

**3. Backup Files**

Developers leave backup copies of files with modified extensions that the web server serves as plaintext instead of executing.

| Original File | Backup Variants |
|--------------|----------------|
| `index.php` | `index.php.bak`, `index.php~`, `index.php.old` |
| `config.php` | `config.php.bak`, `config.bak` |
| `database.sql` | Left in web root during development |

```bash
# Include extensions in Gobuster scan
gobuster dir -u http://TARGET_IP -w wordlist.txt -x .bak,.old,.txt,.sql,.zip
```

**Risk:** Backup `.php` files served as plaintext expose full source code including hardcoded credentials.

---

**4. HTTP Methods Misconfiguration**

Web servers may allow HTTP methods that should be disabled in production.

**Testing allowed methods:**
```bash
# Check allowed methods via OPTIONS
curl -X OPTIONS http://TARGET_IP -v

# Test specific methods
curl -X TRACE http://TARGET_IP -v   # Should be disabled — enables XST attacks
curl -X DELETE http://TARGET_IP/file.txt -v
curl -X PUT http://TARGET_IP/test.txt -d "test data" -v
```

**Dangerous enabled methods:**
- `TRACE` — enables Cross-Site Tracing (XST) attacks to steal cookies
- `PUT` — may allow direct file upload to the server
- `DELETE` — may allow arbitrary file deletion

---

### Directory Traversal

**Definition:** Directory traversal (also path traversal) occurs when user-supplied input that specifies a file path is not properly sanitised, allowing an attacker to navigate outside the intended directory by using `../` sequences or absolute paths.

**This differs from application-level path traversal** (like LFI) in that it can occur directly in the web server layer — not just in application code.

**Basic Traversal:**
```
http://TARGET_IP/download?file=../../../../etc/passwd
http://TARGET_IP/view.php?page=../../../etc/shadow
http://TARGET_IP/include?document=../../../../../windows/win.ini
```

**URL-encoded variants (to bypass simple filters):**
```
../  →  %2e%2e%2f
..\  →  %2e%2e%5c
..%2f → double-encoded
```

**Double-encoded:**
```
%252e%252e%252f  (% is encoded as %25, giving double encoding)
```

**Common target files:**

| OS | Target File | Contents |
|----|------------|---------|
| Linux | `/etc/passwd` | User accounts list |
| Linux | `/etc/shadow` | Hashed passwords (if readable) |
| Linux | `/proc/self/environ` | Process environment variables |
| Linux | `/var/log/apache2/access.log` | Apache access log |
| Windows | `C:\Windows\win.ini` | Basic Windows config — confirms Windows target |
| Windows | `C:\Windows\System32\drivers\etc\hosts` | Host file |
| Windows | `C:\inetpub\wwwroot\web.config` | IIS config with potential credentials |

---

### HTTP Verb Tampering

**Definition:** HTTP verb tampering exploits web applications or servers that check access control based on the HTTP method but fail to enforce it consistently. Changing from `GET` to `POST` (or an arbitrary method) can bypass authentication or access controls.

**Why it works:** Some frameworks apply access controls per HTTP method — a developer might restrict `GET /admin` but forget to restrict `POST /admin`, or an `.htaccess` rule only `Limit` certain verbs.

**Testing:**
```bash
# Try different HTTP methods on a restricted endpoint
curl -X GET http://TARGET_IP/admin     # 403 Forbidden
curl -X POST http://TARGET_IP/admin    # 200 OK?
curl -X HEAD http://TARGET_IP/admin    # Reveals if endpoint exists
curl -X RANDOM http://TARGET_IP/admin  # Random method — some servers accept any verb
```

---

### File Upload Vulnerabilities

**Definition:** Web servers and applications that allow file uploads are often misconfigured in ways that allow uploading and executing malicious files.

**Types of file upload vulnerabilities:**

**1. No file type validation:**
The server accepts any file type. Uploading a `.php` webshell and accessing it via HTTP gives remote code execution.

**2. Client-side validation only:**
The file type is checked by JavaScript in the browser. Intercepting the upload with Burp Suite and changing the `Content-Type` header or filename bypasses the check.

**3. Extension blacklisting (incomplete):**
The server blocks `.php` but not `.php5`, `.phtml`, `.phar`, `.PHP` (case variation), or other executable extensions.

**4. MIME type spoofing:**
The server checks the `Content-Type` header, not the actual file content. Setting `Content-Type: image/jpeg` on a PHP file may bypass the check.

**Basic webshell (for educational purposes — this is what gets uploaded):**
The content of a file upload attack is redacted — the concept is a PHP file that executes OS commands via a parameter. In a real assessment, this would be agreed upon with the client.

**Testing file uploads with Burp Suite:**
1. Intercept the upload request
2. Modify filename extension (e.g., `shell.jpg` → `shell.php`)
3. Modify `Content-Type` header
4. Forward the request
5. Find the uploaded file's URL (often in the response or the uploads directory)
6. Access the file URL — if the server executes it, RCE is confirmed

---

### Exposed Management Interfaces

**Definition:** Web servers often ship with administrative interfaces that are either left accessible or inadequately protected.

**Common exposed interfaces:**

| Interface | Default Path | Risk |
|-----------|-------------|------|
| Apache server-status | `/server-status` | Reveals active requests, client IPs, server version |
| Apache server-info | `/server-info` | Detailed module and configuration info |
| Nginx status | `/nginx_status`, `/stub_status` | Request counts, connections |
| IIS web.config | `/web.config` | May contain credentials, connection strings |
| phpMyAdmin | `/phpmyadmin`, `/pma` | Full database access if default/weak credentials |
| Tomcat Manager | `/manager/html` | Deploy WAR files — RCE via WAR upload |
| WordPress admin | `/wp-admin` | CMS admin — credential brute force target |

**Tomcat Manager Attack:**
If the Tomcat Manager interface is accessible and default or weak credentials work (`tomcat:tomcat`, `admin:admin`, `tomcat:s3cr3t`), a WAR file containing a webshell can be deployed:

```bash
# With Metasploit
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS TARGET_IP
set RPORT 8080
set HttpUsername tomcat
set HttpPassword tomcat
run
```

---

## Workflow / Process

```
Enumerate the web server:
  curl -I TARGET to read headers
  Identify Server, X-Powered-By headers
  Run nmap -sV for version detection
  Run nmap --script http-enum for interesting paths
        |
        v
Directory enumeration:
  gobuster dir with common wordlist
  Include extensions: -x .bak,.old,.txt,.sql,.zip,.php,.config
  Note directories with listing enabled
  Check common sensitive file paths (.env, web.config, .htaccess)
        |
        v
Test for misconfigurations:
  Try OPTIONS on key endpoints
  Test TRACE method
  Check for PUT/DELETE on file paths
  Test path traversal on file parameters (?file=, ?page=, ?document=)
        |
        v
Management interfaces:
  Check /server-status, /server-info, /nginx_status
  Check /phpmyadmin, /manager, /wp-admin
  Try default credentials on any login pages found
        |
        v
File upload testing:
  If upload functionality exists:
    Upload legitimate file first — observe response and storage location
    Try extension bypass (double extension, alternative extensions)
    Try content-type spoofing via Burp
    Attempt to access uploaded file
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Banner Grabbing | Retrieving server software and version information from HTTP headers |
| Directory Listing | Server feature that exposes directory contents when no index file exists |
| Path Traversal | Using `../` to navigate outside the intended file access directory |
| HTTP Verb Tampering | Changing HTTP methods to bypass per-method access controls |
| File Upload Bypass | Circumventing file type restrictions to upload malicious files |
| Webshell | A script uploaded to a web server that executes OS commands via HTTP requests |
| WAR File | Web Application Archive — deployable Java web application package (exploitable via Tomcat Manager) |
| phpMyAdmin | Web-based MySQL management interface — dangerous if publicly accessible |
| Tomcat Manager | Apache Tomcat's web admin interface — allows WAR deployment → RCE |

---

## Real-World Relevance

- Banner grabbing and version identification are first steps in any web server assessment — exact versions enable targeted CVE research
- `.env` files with database credentials publicly accessible are a consistent high-severity bug bounty finding
- Path traversal vulnerabilities in file download parameters are found regularly in enterprise applications
- Tomcat Manager with default credentials is a classic high-severity finding — often leads directly to RCE
- File upload vulnerabilities remain in the OWASP Top 10 (as part of Broken Access Control and Security Misconfiguration)

---

## Key Learnings

- HTTP headers are the fastest source of technology fingerprinting — always run `curl -I` first
- Directory listing on sensitive directories is a finding, not just a curiosity — it exposes files that were never meant to be public
- Path traversal requires iterative testing with different encoding variants — simple `../` is often filtered but encoded variants are not
- Management interfaces with default credentials are critical findings — always test common defaults before moving to brute force
- File upload vulnerabilities require testing multiple bypass techniques — extension, content-type, double extension, null byte

---

## Conclusion

Web server attacks target the infrastructure layer beneath the application — version-exposed software, misconfigured directories, improperly restricted HTTP methods, path traversal in file handlers, and accessible management interfaces. These are findings that exist independently of the application code and are often overlooked when testing focuses purely on application logic. Systematic enumeration of the web server — headers, directories, methods, management paths — before testing the application layer ensures that server-level vulnerabilities are not missed.
