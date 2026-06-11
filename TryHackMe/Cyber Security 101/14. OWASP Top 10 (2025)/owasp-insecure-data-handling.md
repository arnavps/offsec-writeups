# OWASP Top 10 2025: Insecure Data Handling

## Overview

This room covers three OWASP Top 10:2025 categories that relate to how applications process, protect, and validate data. These vulnerability classes have been present across multiple OWASP Top 10 releases — injection notably appears twice in the 2025 list — reflecting how persistently they are found and exploited in real applications. The categories covered are:

- **A04: Cryptographic Failures**
- **A05: Injection**
- **A08: Software or Data Integrity Failures**

---

## Topics Covered

- Cryptographic failures: what they are, what "rolling your own crypto" means, prevention
- Injection: SQL injection, command injection, SSTI, AI prompt injection, prevention
- Software and data integrity failures: what they are, trust boundaries, prevention

---

## Key Concepts

### A04: Cryptographic Failures

#### What It Is

Cryptographic failures occur when sensitive data is inadequately protected due to a lack of encryption, incorrect implementation, or weak security measures applied to cryptographic controls. This is not limited to absent encryption — it includes every case where encryption is used incorrectly.

Common manifestations:
- Passwords stored without hashing, or hashed with weak algorithms (MD5, SHA-1)
- Outdated or broken encryption algorithms (DES, 3DES, AES-ECB)
- Encryption keys hardcoded in source code, configuration files, or repositories
- Sensitive data transmitted over HTTP instead of HTTPS
- Self-signed or expired TLS certificates
- "Rolling your own cryptography" — implementing custom encryption instead of using established algorithms

**"Rolling your own crypto"** is a particularly dangerous pattern. Custom cryptographic implementations almost always contain subtle flaws that are not apparent without deep cryptographic expertise. Established algorithms (AES-GCM, ChaCha20-Poly1305, bcrypt) have been peer-reviewed, formally analysed, and battle-tested. Custom implementations have not.

#### Prevention

- Use strong, modern algorithms: AES-GCM or ChaCha20-Poly1305 for symmetric encryption; TLS 1.3 for transport
- Hash passwords with slow, adaptive functions: **bcrypt**, **scrypt**, or **Argon2** — not MD5 or SHA-1
- Never implement custom cryptographic algorithms
- Never hardcode credentials, API keys, or encryption keys in source code or config files
- Use dedicated key management services: AWS KMS, Azure Key Vault, HashiCorp Vault
- Rotate secrets and keys regularly following a documented key lifecycle policy
- Maintain an inventory of all certificates, keys, and their expiry dates
- Ensure AI models and automation pipelines never expose unencrypted secrets

---

### A05: Injection

#### What It Is

Injection occurs when an application takes user input and passes it, without proper handling, into a system that can interpret and execute it — such as a database, operating system shell, templating engine, or AI system prompt. The input escapes its intended context and is executed as a command or query.

Injection has appeared in every OWASP Top 10 since the list began. In 2025, it appears **twice** — once under A05 and reflected in AI-specific contexts — which underlines how persistent and impactful this class remains.

#### Types of Injection

| Type | Description |
|------|-------------|
| **SQL Injection** | User input inserted into an SQL query, altering its logic — login bypass, data extraction, data deletion |
| **Command Injection** | User input passed to an OS shell command, executing arbitrary system commands |
| **Server-Side Template Injection (SSTI)** | Input inserted into a server-side template engine, allowing code execution on the server |
| **AI / Prompt Injection** | User input blended into an AI model's prompt context, hijacking the system's instructions or extracting hidden data |

#### SQL Injection — Core Mechanism

SQL injection exploits string concatenation when building queries. A login query might be:

```sql
SELECT * FROM users WHERE username = 'john' AND password = 'pass';
```

If the password field is injected with `' OR '1'='1`, the query becomes:

```sql
SELECT * FROM users WHERE username = 'john' AND password = '' OR '1'='1';
```

`'1'='1'` is always true, so the query returns the user record and authentication succeeds without a valid password.

This works because the input broke out of the string context (via the single quote) and injected SQL logic into the query.

#### Prevention

- Use **prepared statements and parameterised queries** for all database interactions — the query structure is defined separately from the data, making injection structurally impossible
- Never build SQL queries through string concatenation with user input
- Avoid OS shell functions that pass user input directly to the system (e.g. `system()`, `exec()` with raw input)
- Use safe APIs that do not invoke the shell
- **Validate input** — enforce expected types, lengths, and formats before processing
- **Sanitise and escape** dangerous characters where parameterised queries are not available
- For AI systems: separate system prompts from user-controlled content; validate and filter all model inputs and outputs

---

### A08: Software or Data Integrity Failures

#### What It Is

Software or data integrity failures occur when an application processes code, updates, or data that it assumes to be safe without verifying its authenticity, integrity, or origin.

This category covers:
- Accepting software updates without cryptographic verification
- Loading scripts, plugins, or configuration files from untrusted sources without integrity checks
- Trusting data that influences application logic without validation (binaries, JSON payloads, templates, AI model outputs)
- Insecure CI/CD pipelines that allow unauthorised modification of code before it reaches production

#### How It Differs from Supply Chain Failures (A03)

Both A03 and A08 relate to trusting unverified external content. The distinction:
- **A03 (Supply Chain)** — focuses on the components and dependencies the application is built with
- **A08 (Integrity)** — focuses on the data and updates the application processes at runtime

In practice, they are closely related and often overlap.

#### Prevention

- Never assume that code, update packages, or key data are legitimate without verification
- Use **cryptographic checksums and signatures** to verify the integrity of software updates before applying them
- Implement **Subresource Integrity (SRI)** for externally loaded scripts and assets in web pages
- Secure CI/CD pipelines — define who and what can modify build artefacts
- Validate all data that influences application logic — do not blindly trust external JSON, configurations, or AI model outputs
- Use signed commits and code review processes to protect source code integrity
- Monitor deployed application behaviour for anomalies that could indicate tampered components

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Cryptographic failure | Sensitive data not adequately protected due to absent, weak, or incorrectly applied cryptography |
| Rolling your own crypto | Implementing custom encryption instead of using established, vetted algorithms |
| bcrypt / scrypt / Argon2 | Slow, adaptive password hashing functions designed to resist brute force |
| Injection | User input escaping its intended context and being executed as a command or query |
| SQL Injection | Injecting SQL logic into a database query via unsanitised user input |
| Parameterised query | Query where the structure is fixed and data is passed separately — prevents SQL injection |
| SSTI | Server-Side Template Injection — code execution via injection into a server-side template |
| Prompt injection | Attacker input manipulating an AI model's prompt context |
| Integrity failure | Processing code, updates, or data without verifying it has not been tampered with |
| SRI | Subresource Integrity — browser mechanism to verify the integrity of externally loaded scripts |
| CI/CD | Continuous Integration / Continuous Deployment — automated build and release pipeline |

---

## Practical Examples

### SQL Injection — Authentication Bypass

Vulnerable login query:
```sql
SELECT * FROM users WHERE username = '$username' AND password = '$password';
```

Injected password input: `' OR '1'='1`

Resulting query:
```sql
SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1';
```

The `OR '1'='1'` condition is always true — the query returns the user record and the login succeeds.

**Fix — Parameterised query (Python/MySQL example):**
```python
cursor.execute(
    "SELECT * FROM users WHERE username = %s AND password = %s",
    (username, password)
)
```

The query structure is defined first; the user-supplied values are passed separately and cannot alter the query logic.

---

### Cryptographic Failure — Weak Password Hashing

```python
# Vulnerable: MD5 with no salt
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()

# Secure: bcrypt with automatic salting
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

MD5 hashes are fast to compute and precomputed rainbow tables exist for common passwords. bcrypt is slow by design and automatically salts each hash.

---

### Integrity Failure — Missing Update Verification

```python
# Vulnerable: download and execute without verification
import urllib.request
urllib.request.urlretrieve("https://updates.example.com/update.pkg", "update.pkg")
os.system("update.pkg")

# Secure: verify checksum before execution
expected_hash = "abc123..."
actual_hash = hashlib.sha256(open("update.pkg", "rb").read()).hexdigest()
if actual_hash != expected_hash:
    raise ValueError("Update package integrity check failed")
```

---

## Workflow / Process

```
User submits input to the application

Injection Prevention:
  Is this input going into a database query?
    Yes --> Use parameterised query
  Is this input going to an OS command?
    Yes --> Use safe API, avoid shell invocation
  Is this input going into a template?
    Yes --> Use context-aware escaping
  Is this input going into an AI prompt?
    Yes --> Separate system context from user content; validate output

Cryptographic Controls:
  Is this password being stored?
    Yes --> Hash with bcrypt, scrypt, or Argon2
  Is this sensitive data being transmitted?
    Yes --> Use TLS 1.3
  Does this application handle encryption keys?
    Yes --> Use a key management service, never hardcode

Integrity Checks:
  Is this update/package/script from an external source?
    Yes --> Verify cryptographic signature or checksum before using
  Does this data influence application logic?
    Yes --> Validate format, type, and source before processing
```

---

## Real-World Relevance

- SQL injection remains one of the most common and easily exploitable vulnerabilities in web applications — automated tools like `sqlmap` can exploit injectable parameters in seconds
- The 2012 LinkedIn breach exposed 117 million unsalted SHA-1 password hashes — most were cracked within days using publicly available tools and wordlists
- Command injection in web applications (e.g. via shell-calling functions in PHP or Python) regularly leads to full server compromise
- Prompt injection is an emerging and under-mitigated attack class — applications integrating LLMs without input/output filtering are vulnerable to data exfiltration and instruction hijacking
- The XZ Utils backdoor (2024) — a supply chain / integrity failure where a trusted open-source compression library was compromised — came within days of being deployed across major Linux distributions
- SRI adoption on the web remains low despite being a straightforward browser-enforced control — CDN-hosted scripts without SRI represent an active integrity risk

---

## Key Learnings

- Cryptographic failures include both missing encryption and incorrectly applied encryption — MD5 hashing and AES-ECB are as dangerous as no protection
- Never implement custom cryptography — use established, vetted algorithms
- Password hashing must use slow, adaptive functions (bcrypt, Argon2) — fast hashing functions (MD5, SHA-1) are unsuitable for passwords
- Injection occurs when user input escapes its context and is interpreted as a command or query — parameterised queries structurally prevent SQL injection
- Input validation and sanitisation are necessary across all injection types
- Integrity failures occur when software, updates, or data are trusted without verification — cryptographic checksums and signatures are the solution
- CI/CD pipelines are part of the integrity attack surface and must be secured

---

## Additional Notes

- The difference between A04 (Cryptographic Failures) in this room and A04 in the design flaws room is context: the design flaws room covers cryptography as a design-phase failure; this room addresses it as a data handling failure at the implementation level. They are the same OWASP category viewed from different angles
- Parameterised queries do not prevent all injection — they prevent SQL injection specifically. SSTI, command injection, and prompt injection require separate mitigations
- bcrypt has a maximum input length of 72 bytes — for very long passwords, a pre-hash step (e.g. SHA-256 the password first) is sometimes used before passing to bcrypt
- `LIMIT 1` in a SQL query is not a security control — it limits result rows but does not prevent injection from executing

---

## Conclusion

Insecure data handling vulnerabilities — cryptographic failures, injection, and integrity failures — have been consistently present across OWASP Top 10 releases for years. Injection appearing twice in the 2025 list reflects both its persistence and its evolution into new contexts (AI prompts, template engines). The common thread across all three categories is a failure to treat data as untrusted: inadequate protection of stored data, insufficient validation of input before processing it, and blind trust in externally sourced code and updates. Addressing these categories requires discipline in implementation — parameterised queries, established cryptographic primitives, and cryptographic integrity verification — rather than novel security mechanisms.
