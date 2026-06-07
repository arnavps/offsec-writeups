# Hydra

## Overview

Hydra is an online password brute-forcing tool used to attack authentication services by systematically trying credentials from a wordlist. It supports a wide range of protocols, making it one of the most versatile credential-stuffing tools in a penetration tester's toolkit. This room introduces Hydra's core functionality, its command structure, and how to use it against SSH and web login forms.

---

## Topics Covered

- What Hydra is and how it works
- Supported protocols
- Brute-forcing SSH
- Brute-forcing HTTP POST login forms
- Key flags and options

---

## Key Concepts

### What is Hydra?

Hydra is an online brute-force tool — meaning it attacks live services over the network rather than cracking offline hashes. It takes a username (or list of usernames) and a password wordlist, then tries every combination against the target service until a valid login is found.

It supports a large number of protocols including:

`SSH`, `FTP`, `HTTP-FORM-GET`, `HTTP-FORM-POST`, `HTTPS-GET`, `HTTPS-POST`, `SMB`, `RDP`, `SMTP`, `POP3`, `IMAP`, `MYSQL`, `POSTGRES`, `MONGODB`, `VNC`, `LDAP`, `SNMP`, `Telnet`, `RTSP`, and many more.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Brute force | Trying every possible value from a list until a match is found |
| Wordlist | A file containing candidate passwords, one per line |
| Thread | An independent execution path — more threads means faster attacks |
| `^USER^` | Hydra placeholder — replaced with the username during execution |
| `^PASS^` | Hydra placeholder — replaced with each password from the wordlist |
| `F=` | Failure string — a substring from the server response that indicates a failed login |
| Online attack | Attacking a live, running service over the network |

---

## Practical Examples / Demonstrations

### General Syntax

```bash
hydra -l <username> -P <wordlist> <target_ip> <service>
```

Use `-l` (lowercase) for a single username, `-L` (uppercase) for a username list.
Use `-p` (lowercase) for a single password, `-P` (uppercase) for a password list.

---

### Brute-Forcing FTP

```bash
hydra -l user -P passlist.txt ftp://MACHINE_IP
```

---

### Brute-Forcing SSH

```bash
hydra -l root -P passwords.txt MACHINE_IP -t 4 ssh
```

| Flag | Description |
|------|-------------|
| `-l root` | Target username |
| `-P passwords.txt` | Path to the password wordlist |
| `-t 4` | Number of parallel threads (connections) to run |
| `ssh` | Protocol to attack |

Keeping threads low for SSH (`-t 4`) is recommended — too many concurrent connections can lock out the account or crash the service.

---

### Brute-Forcing an HTTP POST Login Form

Hydra can brute-force web login forms. You need to know:
1. The login page path
2. The form field names for username and password
3. A string from the server's response that appears when a login **fails**

Use the browser's DevTools (Network tab) or view the page source to identify the field names and request structure.

**Syntax:**
```bash
hydra -l <username> -P <wordlist> MACHINE_IP http-post-form "<path>:<login_credentials>:<invalid_response>"
```

**Concrete example:**
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt MACHINE_IP \
  http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

Breaking down the form string `"/:username=^USER^&password=^PASS^:F=incorrect"`:

| Part | Meaning |
|------|---------|
| `/` | Login page path (root in this case) |
| `username=^USER^` | Form field name for username; `^USER^` is replaced by Hydra |
| `password=^PASS^` | Form field name for password; `^PASS^` is replaced by Hydra |
| `F=incorrect` | Failure indicator — `F=` means this string appears in the response on a **failed** login |

Use `S=` instead of `F=` to specify a **success** indicator string if that's easier to identify.

| Flag | Description |
|------|-------------|
| `-V` | Verbose — prints every attempt to the terminal |
| `-s <port>` | Specify a non-default port (e.g. if the web app runs on port 8080) |

**With a custom port:**
```bash
hydra -l admin -P wordlist.txt MACHINE_IP \
  http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -s 8080 -V
```

---

## Workflow / Process

```
Identify the target service and authentication endpoint
        |
        v
Determine the username (or use a username list)
        |
        v
Select a password wordlist (e.g. rockyou.txt)
        |
        v
For web forms: identify field names and failure response string
        |
        v
Run Hydra with the appropriate module and flags
        |
        v
Hydra iterates through the wordlist, sending one login attempt per password
        |
        v
Valid credentials are printed when a successful login is detected
```

---

## Real-World Relevance

- Hydra is used in penetration tests to check whether services are vulnerable to credential brute-forcing — particularly when default credentials or weak passwords are suspected
- It's commonly used after obtaining a username through OSINT or enumeration, to test that specific account
- SSH brute-forcing is a real attack vector — exposed SSH ports on port 22 are constantly targeted by automated scanners in the wild
- Web form attacks are relevant for testing login pages that lack rate limiting, account lockout, or CAPTCHA protections
- Hydra is frequently used in CTFs to recover credentials from intentionally vulnerable machines

---

## Key Learnings

- Use `-l` for a single username, `-L` for a username list; `-p` for a single password, `-P` for a wordlist
- `-t` controls thread count — balance speed against service stability
- For HTTP POST forms, the form string has three colon-separated parts: path, field params, and failure indicator
- `^USER^` and `^PASS^` are Hydra's placeholders in the form string
- Use `-V` for verbose output to see every attempt
- Use `-s` to specify a non-default port

---

## Additional Notes

- Hydra is an **online** brute-force tool — it generates live network traffic. This means it can trigger account lockouts, IDS/IPS alerts, and rate limiting
- Using known ports (443, 80, 445) for reverse shell listeners follows the same principle as using well-known services — to blend traffic. Hydra attacks can be throttled with `--delay` to reduce detection risk
- `rockyou.txt` (located at `/usr/share/wordlists/rockyou.txt` on Kali) is the most common wordlist used for general-purpose password attacks
- Always ensure you have explicit authorisation before running Hydra against any target

---

## Conclusion

Hydra is a straightforward but powerful tool for testing authentication weaknesses across a wide range of services. Understanding its flag structure — particularly the HTTP POST form syntax — is essential for web application testing. The key principle behind it is simple: if a service has no rate limiting, lockout policy, or monitoring, a well-chosen wordlist will eventually find the password.
