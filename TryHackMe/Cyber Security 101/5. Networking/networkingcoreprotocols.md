# Networking Core Protocols

## Overview

This room covers the application-layer protocols that power everyday internet activity — DNS, HTTP/HTTPS, FTP, SMTP, POP3, and IMAP. Understanding how these protocols work at the command level is essential for web security testing, email security analysis, and manual protocol interaction using tools like telnet.

---

## Topics Covered

- DNS — domain resolution and record types
- WHOIS
- HTTP and HTTPS — methods and manual interaction
- FTP — file transfer
- SMTP — email sending
- POP3 and IMAP — email retrieval
- Protocol port reference

---

## Key Concepts

### DNS (Domain Name System)

DNS resolves domain names to IP addresses. It operates at Layer 7 and uses UDP port 53 by default (TCP port 53 as fallback for large responses).

**Key DNS record types:**

| Record | Description | Example |
|---|---|---|
| A | Maps hostname to IPv4 address | `example.com → 93.184.216.34` |
| AAAA | Maps hostname to IPv6 address | `example.com → 2606:2800:...` |
| CNAME | Aliases one domain to another | `www.example.com → example.com` |
| MX | Specifies mail server for a domain | `example.com → mail.example.com` |

```bash
nslookup example.com
nslookup example.com 1.1.1.1    # Use specific DNS server
```

### WHOIS

WHOIS records provide registration information for a domain — registrant name, address, phone, email, registration and expiry dates.

```bash
whois example.com
```

---

### HTTP / HTTPS

HTTP (HyperText Transfer Protocol) is the protocol for web communication. HTTPS is HTTP over TLS — encrypted. Both use TCP.

**Common HTTP methods:**

| Method | Purpose |
|---|---|
| GET | Retrieve a resource |
| POST | Submit data to the server |
| PUT | Create or overwrite a resource |
| DELETE | Remove a resource |

**Default ports:** HTTP → 80, HTTPS → 443 (also 8080, 8443)

**Manual HTTP request via telnet:**

```bash
telnet 10.10.10.10 80
GET /flag.html HTTP/1.1
Host: anything
```

---

### FTP (File Transfer Protocol)

FTP is designed for file transfer. It is more efficient than HTTP for large files. Default port: TCP 21.

**Key FTP commands:**

| Command | Purpose |
|---|---|
| `USER` | Provide username |
| `PASS` | Provide password |
| `RETR` | Download a file from server |
| `STOR` | Upload a file to server |

**FTP session example:**

```bash
ftp 10.10.10.10
# Username: anonymous
# Password: (blank or email)
ls
type ascii
get flag.txt
quit
cat flag.txt
```

---

### SMTP (Simple Mail Transfer Protocol)

SMTP handles sending email. Default port: TCP 25.

**Key SMTP commands:**

| Command | Purpose |
|---|---|
| `HELO` / `EHLO` | Initiate SMTP session |
| `MAIL FROM` | Specify sender address |
| `RCPT TO` | Specify recipient address |
| `DATA` | Begin email body |
| `.` | End email body (sent on its own line) |

---

### POP3 (Post Office Protocol 3)

POP3 retrieves email from a server. Default port: TCP 110. Downloads and typically deletes messages from the server — suited for single-device use.

**Key POP3 commands:**

| Command | Purpose |
|---|---|
| `USER <username>` | Identify user |
| `PASS <password>` | Authenticate |
| `STAT` | Get message count and total size |
| `LIST` | List messages and sizes |
| `RETR <n>` | Retrieve message number n |
| `DELE <n>` | Mark message n for deletion |
| `QUIT` | End session and apply deletions |

```bash
telnet 10.10.10.10 110
```

---

### IMAP (Internet Message Access Protocol)

IMAP synchronises email across multiple devices. Default port: TCP 143. Messages stay on the server and are synced — suited for multi-device use.

**Key IMAP commands:**

| Command | Purpose |
|---|---|
| `LOGIN <user> <pass>` | Authenticate |
| `SELECT <mailbox>` | Select a mailbox folder |
| `FETCH <n> body[]` | Retrieve message n (header + body) |
| `MOVE <set> <mailbox>` | Move messages to another folder |
| `COPY <set> <mailbox>` | Copy messages to another folder |
| `LOGOUT` | End session |

---

### Protocol Port Reference

| Protocol | Transport | Default Port |
|---|---|---|
| TELNET | TCP | 23 |
| DNS | UDP / TCP | 53 |
| HTTP | TCP | 80 |
| HTTPS | TCP | 443 |
| FTP | TCP | 21 |
| SMTP | TCP | 25 |
| POP3 | TCP | 110 |
| IMAP | TCP | 143 |

---

## Practical Examples / Demonstrations

### Retrieve a file via HTTP using telnet

```bash
telnet 10.10.10.10 80
GET /flag.html HTTP/1.1
Host: anything
```

### FTP anonymous login and file retrieval

```bash
ftp 10.10.10.10
# anonymous / (blank)
ls
type ascii
get flag.txt
quit
cat flag.txt
```

### DNS lookup

```bash
nslookup tryhackme.com
nslookup -type=MX tryhackme.com
```

---

## Real-World Relevance

- DNS enumeration (A, MX, CNAME records) is a standard recon step — reveals infrastructure, mail providers, and third-party services
- HTTP methods (PUT, DELETE) on unprotected endpoints are a common API vulnerability
- FTP anonymous login is a frequent misconfiguration finding — exposes files without authentication
- SMTP is used in phishing infrastructure — understanding SMTP commands helps analyse malicious email headers
- POP3 and IMAP credentials transmitted over unencrypted connections are trivially intercepted — always use POP3S/IMAPS
- WHOIS data can reveal registrant information useful in OSINT investigations

---

## Key Learnings

- DNS uses A, AAAA, CNAME, and MX records — each serves a different resolution purpose
- HTTP methods define the action of a request — GET retrieves, POST submits, PUT creates/overwrites, DELETE removes
- FTP uses TCP 21 for control; anonymous login is a common misconfiguration
- SMTP sends email; POP3 downloads it; IMAP synchronises it across devices
- Telnet can be used to manually interact with any plaintext TCP service for learning and testing

---

## Conclusion

Application-layer protocols are the language of the internet. Understanding how DNS resolves names, how HTTP requests are structured, and how email flows through SMTP/POP3/IMAP is foundational for web security testing, email security analysis, and protocol-level investigation. The ability to interact with these protocols manually using telnet is a practical skill that reveals exactly what's happening under the hood.
