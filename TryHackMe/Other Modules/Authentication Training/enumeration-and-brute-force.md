# Enumeration and Brute Force

## Overview

Authentication enumeration is the systematic inspection of authentication mechanisms to identify weaknesses that an attacker can exploit. Before brute-forcing credentials, attackers gather as much information as possible about valid usernames, password policies, and account behaviour. This room covers where and how enumeration occurs, how verbose errors aid attackers, password reset vulnerabilities, and how to automate email enumeration and basic authentication brute-forcing.

---

## Topics Covered

- What authentication enumeration is
- Sources of username enumeration: registration, password reset, verbose errors, data breaches
- Password policy identification
- Verbose error analysis and techniques to induce errors
- Automated email enumeration with Python
- Password reset flow vulnerabilities: predictable tokens, token expiration
- HTTP Basic Authentication and brute-forcing
- Google Dorks for exposed authentication surfaces

---

## Key Concepts

### What is Authentication Enumeration?

Authentication enumeration is the process of methodically inspecting authentication components — username validation, password policies, session management — to identify exploitable weaknesses. It is not just a checklist; it is understanding how the system's components connect and what information each reveals.

### Where Enumeration Occurs

| Source | What it reveals |
|--------|----------------|
| **Registration pages** | Whether a username or email is already taken — confirms active accounts |
| **Password reset features** | Different responses for valid vs invalid usernames expose which accounts exist |
| **Verbose error messages** | `"Username not found"` vs `"Incorrect password"` directly confirms valid usernames |
| **Data breach information** | Previously leaked credentials confirm username reuse across platforms |

### Password Policy Identification

An application that returns detailed validation error messages (e.g. "Password must contain at least one uppercase letter, one number, and one symbol") reveals the complexity requirements. An attacker can use this to generate a targeted dictionary that satisfies the policy, reducing the brute-force search space significantly.

---

### Understanding Verbose Errors

Verbose errors are detailed messages intended for debugging that unintentionally expose sensitive information:

| Information Type | What it reveals |
|-----------------|----------------|
| Internal paths | File paths, directory structure, configuration file locations |
| Database details | Table names, column names, query structure |
| User information | Hints at valid usernames or account existence |

#### Techniques to Induce Verbose Errors

| Technique | Description |
|-----------|-------------|
| Invalid login attempts | Intentionally incorrect input to trigger different error messages per case |
| SQL injection | Single quote (`'`) in a login field may cause a database error exposing schema info |
| File inclusion / path traversal | `../../` sequences to provoke errors revealing internal paths |
| Form manipulation | Modifying hidden form fields to trigger validation errors revealing backend logic |
| Application fuzzing | Sending unexpected inputs with tools like Burp Suite Intruder to find weak points |

---

### Automated Email Enumeration

The technique: submit each candidate email to the login endpoint and differentiate valid from invalid based on the server's response.

**Python enumeration script (conceptual structure):**

```python
import requests
import sys

def check_email(email):
    url = 'http://TARGET/labs/verbose_login/functions.php'
    headers = {
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
        'X-Requested-With': 'XMLHttpRequest'
    }
    data = {
        'username': email,
        'password': 'placeholder',
        'function': 'login'
    }
    response = requests.post(url, headers=headers, data=data)
    return response.json()

def enumerate_emails(email_file):
    invalid_error = "Email does not exist"
    valid_emails = []
    with open(email_file, 'r') as f:
        for email in f:
            email = email.strip()
            result = check_email(email)
            if invalid_error in result.get('message', ''):
                print(f"[INVALID] {email}")
            else:
                print(f"[VALID] {email}")
                valid_emails.append(email)
    return valid_emails
```

**Key detection logic:** The script looks for the specific error string `"Email does not exist"` in the response. Any response without this string indicates a valid email.

---

### Password Reset Flow Vulnerabilities

Password reset mechanisms have several common weaknesses:

| Vulnerability | Description |
|---------------|-------------|
| **Predictable tokens** | Short or sequential reset tokens can be brute-forced |
| **Token expiration issues** | Tokens that do not expire quickly provide a long window for exploitation |
| **Insufficient validation** | Security questions with guessable answers; weak email-based verification |
| **Information disclosure** | Error messages confirming whether an email is registered |
| **Insecure transport** | Reset links sent over HTTP can be intercepted |

#### Predictable Token Example

```php
$token = mt_rand(100, 200);  // Only 101 possible values — trivially brute-forceable
$query = $conn->prepare("UPDATE users SET reset_token = ? WHERE email = ?");
$query->bind_param("ss", $token, $email);
$query->execute();
```

A 3-digit numeric token with 101 possible values can be brute-forced in seconds. A secure token should use `random_bytes()` or a cryptographically secure PRNG producing at minimum 128 bits of entropy.

---

### HTTP Basic Authentication

HTTP Basic Authentication transmits credentials as a Base64-encoded string in the `Authorization` header. The format is:

```
Authorization: Basic <base64(username:password)>
```

**Important:** Base64 is encoding, not encryption. The credentials are trivially decodable. Basic Auth over HTTP is completely insecure — over HTTPS it provides confidentiality but not protection from brute force.

**Brute-force approach:** Generate all username:password combinations, Base64-encode each, and submit as the Authorization header until a `200 OK` is received instead of `401 Unauthorized`.

Hydra supports HTTP Basic Auth directly:
```bash
hydra -l admin -P wordlist.txt TARGET http-get /admin
```

---

### Google Dorks

Google Dorks use advanced search operators to find exposed authentication surfaces:

| Goal | Dork |
|------|------|
| Find admin panels | `site:example.com inurl:admin` |
| Find log files with passwords | `filetype:log "password" site:example.com` |
| Discover backup directories | `intitle:"index of" "backup" site:example.com` |
| Exposed configuration files | `filetype:env "DB_PASSWORD" site:example.com` |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Username enumeration | Identifying which usernames exist in an application |
| Verbose error | A detailed error message that unintentionally reveals internal information |
| Brute force | Systematically trying all possible values until the correct one is found |
| Password policy | Rules governing password complexity — reveals minimum viable password structure |
| Predictable token | A reset or session token with insufficient randomness that can be guessed or brute-forced |
| HTTP Basic Auth | Simple authentication scheme encoding credentials as Base64 in the Authorization header |
| Google Dork | A specialised search query using Google's advanced operators to find sensitive exposed content |

---

## Workflow / Process

```
Target web application identified
        |
        v
Map authentication surfaces:
  - Registration page
  - Login page
  - Password reset
  - Any form accepting user input
        |
        v
Test for username enumeration:
  - Register with known and unknown usernames — observe different responses
  - Try password reset with valid and invalid emails — observe response differences
  - Observe login error messages
        |
        v
Identify password policy from validation error messages
        |
        v
Automate enumeration with a wordlist:
  - Python script or Burp Intruder
  - Filter responses by error message difference
        |
        v
Test password reset for predictable tokens:
  - Request multiple reset tokens — check for patterns or short ranges
  - Brute-force token if range is small
        |
        v
Build targeted wordlist based on password policy
        |
        v
Brute-force login with valid username + wordlist
```

---

## Real-World Relevance

- Username enumeration via password reset is one of the most consistently reported HackerOne vulnerabilities — different responses for registered vs unregistered emails are a pervasive issue
- HTTP Basic Auth on network devices (routers, NAS devices) is still common in enterprise environments — brute-forcing it with vendor default credentials is a frequent finding in internal network assessments
- Predictable OTP tokens (as shown in the PHP `mt_rand(100, 200)` example) have been found in production applications and can be brute-forced in under a second
- Google Dorks revealing exposed admin panels and credential files are run continuously by automated scanners — any publicly accessible sensitive page will be indexed

---

## Key Learnings

- Verbose errors are the primary information source for username enumeration — different messages for "username not found" vs "wrong password" confirm account existence
- Registration pages and password reset flows are the two most common enumeration points
- Password policy errors reveal complexity requirements — use them to build targeted wordlists
- Predictable reset tokens are trivially brute-forced — secure tokens require CSPRNG with high entropy
- HTTP Basic Auth provides no protection against brute force — always pair with rate limiting and strong credentials
- Automation (Python scripts, Hydra, Burp Intruder) makes large-scale enumeration practical

---

## Conclusion

Authentication enumeration is the reconnaissance phase of credential attacks. The information gathered — valid usernames, password policies, token entropy, error message differences — directly reduces the effort required for subsequent brute-force attacks. Understanding these techniques is essential both for offensive security testing and for designing authentication systems that resist enumeration in the first place.
