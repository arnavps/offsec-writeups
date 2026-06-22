# Protocols and Servers 1

## Overview

This room provides a foundational understanding of the core application-layer protocols that form the backbone of network services: HTTP, FTP, SMTP, POP3, and IMAP, along with Telnet as a teaching tool. The approach is deliberately low-level — rather than working through a GUI, you interact with each protocol manually using Telnet to understand exactly what is happening beneath the surface. This reveals how data is transmitted, why cleartext protocols are insecure, and what an attacker can observe when traffic is unencrypted.

**Main objectives:**
- Understand what Telnet is and why it was replaced by SSH
- Manually construct HTTP requests and read server responses
- Understand FTP's dual-connection architecture and active/passive modes
- Understand SMTP's email routing role and manual email submission
- Understand POP3's download-and-delete model
- Understand IMAP's server-side synchronisation model
- Recognise the security implications of cleartext credential transmission

**Skills introduced:** Manual protocol interaction, banner grabbing, understanding email delivery components, recognising cleartext credential risk

---

## Concepts Covered

### Why Learn Legacy Protocols?

**Definition:** Legacy protocols are application-layer protocols designed in the 1970s–1990s for trusted academic networks where security was not a design priority. They transmit commands, data, and credentials in cleartext.

**Why It Matters:**
1. These protocols are still in use — legacy systems, internal networks, IoT devices, and misconfigured services still run unencrypted versions
2. The protocol mechanics are identical to their encrypted successors — HTTP commands are the same whether transported over HTTP or HTTPS
3. Understanding the protocol helps understand the attacks — knowing SMTP explains email spoofing; knowing HTTP explains web application vulnerabilities
4. Finding these protocols during a pentest represents an immediate finding: credentials and data are exposed to anyone on the network path

**Key Details:**
- The follow-up room (Protocols and Servers 2) covers attacks (sniffing, MITM, password attacks) and encrypted replacements (TLS, SSH)
- This room focuses on the "what and how" — the next room focuses on "what goes wrong and how to fix it"

---

### Telnet

**Definition:** Telnet (Teletype Network, 1969) is an application-layer protocol for remote terminal access. It provides a text-based command-line interface to a remote system. Servers listen on TCP port 23.

**Why It Matters:** Telnet demonstrates the fundamental insecurity of cleartext protocols. All data including credentials is transmitted without encryption, making it trivially readable to anyone on the network path.

**Key Details:**
- Authentication: prompts for username and password at login
- All data including the authentication exchange is transmitted in cleartext
- A network capture during Telnet login shows the username and password in plain ASCII
- The password is not displayed on screen during typing (visual protection only — the password still travels across the network in cleartext)
- Completely replaced by SSH for interactive remote access
- You may still encounter Telnet on: legacy systems, older network equipment (routers/switches), IoT devices with limited resources, misconfigured systems

**Telnet Client as a Testing Tool:** The Telnet *client* remains useful for connecting to any TCP port and manually interacting with text-based protocols. This technique is banner grabbing — connecting to a port and reading the initial response.

**Practical Relevance:** Finding an open Telnet port (23) during a penetration test is a significant finding. It indicates either a legacy system or a security misconfiguration. During an assessment, it means both a method of access and an information disclosure risk for anyone on the network.

---

### HTTP (Hypertext Transfer Protocol)

**Definition:** HTTP is the application-layer protocol used to transfer web pages and related resources between web browsers (clients) and web servers.

**Why It Matters:** Understanding HTTP at the command level reveals exactly what browsers send and receive on every page load. It explains how authentication, sessions, and data transfer work — and why web application vulnerabilities like injection and authentication bypass are possible.

**Key Details:**
- Plain HTTP transmits all data including login credentials in cleartext
- Nearly all modern websites use HTTPS (HTTP over TLS) — same commands, encrypted transport
- HTTP/1.1 is text-based and human-readable; HTTP/2 is binary; HTTP/3 uses QUIC/UDP
- Default ports: 80 (HTTP), 443 (HTTPS)
- The `Server:` response header often reveals web server software and version (e.g., `Server: nginx/1.18.0 (Ubuntu)`)
- Manually constructing HTTP requests: `GET / HTTP/1.1` followed by `host: value` and two enters

**HTTP Versions:**
- HTTP/1.1 (1997): text-based, persistent connections, still widely used
- HTTP/2 (2015): binary, multiplexed, header compression — not manually interactive via Telnet
- HTTP/3 (2022): QUIC/UDP-based, improved performance — requires specialised clients

**Web Servers in Common Use:**
- Nginx: most widely deployed globally, known for performance
- Apache: historically dominant, highly configurable
- IIS: Microsoft's server, found in Windows enterprise environments
- Caddy: modern server with automatic HTTPS built in

**Practical Relevance:** The `Server:` header is a key reconnaissance target. Version-specific information enables CVE research. Security-conscious administrators suppress version details — finding a full version string is a reportable finding.

---

### FTP (File Transfer Protocol)

**Definition:** FTP is an application-layer protocol designed for efficient file transfer between systems. Servers listen on TCP port 21 for the control channel.

**Why It Matters:** FTP transmits credentials and file data in cleartext. It also exposes directory structure and operational information that attackers can use for reconnaissance.

**Key Details:**

**Dual-connection architecture:** FTP uses two separate TCP connections:
- **Control channel (port 21):** Commands and responses
- **Data channel:** File transfers — established separately for each transfer

**Active vs Passive mode:**
- **Active mode:** Data connection originates from the FTP server's port 20 back to the client. Often fails through firewalls because the server initiates back to the client.
- **Passive mode:** Data connection originates from the client's port above 1023. More firewall-friendly; default for most modern FTP clients.

**FTP commands:**
- `USER username` — identify the user
- `PASS password` — authenticate
- `SYST` — show system type
- `STAT` — show server status (reveals encryption status)
- `PASV` — switch to passive mode
- `TYPE A` — ASCII transfer mode
- `TYPE I` — binary transfer mode
- `ls` / `get filename` — list files / download a file (via FTP client)

**Anonymous FTP:** Some servers allow login with username `anonymous` (or `ftp`) and any email as password. Finding anonymous FTP access during a pentest is a significant finding — it may expose sensitive files or allow uploads.

**Secure alternatives:**
- SFTP (SSH File Transfer Protocol, port 22): runs over SSH — most common replacement
- FTPS (FTP + TLS, port 990): TLS-encrypted FTP
- SCP (port 22): also SSH-based, being deprecated in favour of SFTP

**Practical Relevance:** Finding an FTP server during a pentest prompts immediate anonymous login testing. If credentials are required but obtained, the directory listing reveals server structure. All credential and file data is capturable without decryption if the session is sniffed.

---

### SMTP (Simple Mail Transfer Protocol)

**Definition:** SMTP is the protocol used to send email between mail servers (MTA to MTA) and from mail clients to their submission server (MUA to MSA).

**Why It Matters:** SMTP was designed without authentication for sender identity — this is why email spoofing is possible. Understanding SMTP is essential for phishing assessments, email header analysis, and identifying open relay misconfigurations.

**Key Details:**

**Email delivery components:**
- **MUA (Mail User Agent):** Email client (Thunderbird, Outlook, Gmail browser)
- **MSA (Mail Submission Agent):** Receives mail from MUA, checks errors
- **MTA (Mail Transfer Agent):** Routes and delivers between servers
- **MDA (Mail Delivery Agent):** Stores email in recipient's mailbox

**SMTP ports and encryption:**
- **Port 25:** Traditional server-to-server (MTA-MTA). Often blocked by ISPs for residential connections. Encryption optional via STARTTLS.
- **Port 587:** Submission port for mail clients (MUA-MSA). Requires authentication. STARTTLS negotiated. **Recommended for sending email.**
- **Port 465:** SMTPS (implicit TLS). TLS begins immediately upon connection.

**SMTP commands (in order):**
1. `helo hostname` (or `ehlo` for extended SMTP) — introduce client
2. `mail from: sender@example.com` — specify sender
3. `rcpt to: recipient@example.com` — specify recipient
4. `data` — begin message body
5. `.` (period on its own line + Enter) — end message
6. `quit` — end session

**Email spoofing:** SMTP accepts any value in the `mail from:` field without verifying the sender controls that address. This is the mechanism behind phishing emails appearing to come from legitimate sources.

**Practical Relevance:** Testing SMTP includes checking for open relay (accepting mail from external senders to external recipients — allows spam), verifying sender authentication requirements, and understanding how email headers build up during routing.

---

### POP3 (Post Office Protocol version 3)

**Definition:** POP3 is a protocol used to retrieve email from a mail server (MDA). It follows a download-and-delete model by default.

**Why It Matters:** POP3 sends credentials in cleartext. Understanding its behaviour explains why it is being replaced and why capturing POP3 traffic during a pentest is a high-value activity.

**Key Details:**

**POP3 ports:**
- Port 110: Cleartext POP3 (some servers support STLS upgrade)
- Port 995: POP3S (implicit TLS, recommended)

**POP3 commands:**
- `USER username` — identify user
- `PASS password` — authenticate
- `STAT` — returns `+OK nn mm` where nn = message count, mm = mailbox size in bytes
- `LIST` — lists messages with sizes
- `RETR n` — retrieves message n
- `DELE n` — marks message n for deletion
- `RSET` — cancels pending deletions
- `QUIT` — ends session and deletes marked messages

**STAT response format:** `+OK nn mm` — per RFC 1939, nn is the number of messages, mm is the total size in octets.

**Download and delete model:**
- Emails are downloaded to the local device and (by default) removed from the server
- Only accessible from the device that downloaded them
- Not synchronised across multiple devices
- Suitable for single-device access; unsuitable for modern multi-device email usage

**Practical Relevance:** POP3 captures during a pentest immediately yield valid credentials. Successful mailbox access is high-value — it may contain credentials for other systems, password reset links, and sensitive business communications.

---

### IMAP (Internet Message Access Protocol)

**Definition:** IMAP is a protocol for accessing email stored on a mail server. Unlike POP3, emails remain on the server and are synchronised across all connected clients.

**Why It Matters:** IMAP is the dominant email access protocol because it supports multi-device synchronisation. Its server-side storage model means a compromised IMAP account gives persistent, multi-session access to the entire mailbox history.

**Key Details:**

**IMAP ports:**
- Port 143: Cleartext IMAP (STARTTLS upgrade available)
- Port 993: IMAPS (implicit TLS, recommended)

**IMAP characteristics:**
- Emails remain on the server — accessible from any device
- Read/unread status, folders, and flags synchronised across clients
- Server-side search without downloading all messages
- Full folder structure management

**IMAP commands (each preceded by a unique tag, e.g., c1, c2):**
- `LOGIN username password`
- `LIST "" "*"` — list all mailbox folders
- `SELECT folder` — open folder for read/write
- `EXAMINE folder` — open folder for read-only
- `FETCH n BODY[]` — retrieve message n
- `SEARCH criteria` — search for messages matching criteria
- `STORE n +FLAGS (\Seen)` — mark message as read
- `LOGOUT` — end session

**Practical Relevance from a Security Perspective:**
- Persistent access: Unlike POP3, emails remain on the server — an attacker retains access for new email indefinitely
- Historical data: The full mailbox history is accessible — potentially years of sensitive communications
- Password reset abuse: Searching for reset emails enables account takeover on other services
- Lateral movement intelligence: Internal business emails contain credentials, documentation, and pivot opportunities
- Business email compromise: Access to corporate email enables invoice fraud and impersonation

---

## Methodology

The learning methodology in this room is deliberate protocol demystification:

```
Select a protocol (HTTP, FTP, SMTP, POP3, IMAP)
        |
        v
Understand its purpose and architecture:
  - What problem does it solve?
  - What components interact?
  - What ports does it use?
        |
        v
Connect with Telnet to observe the raw protocol:
  telnet TARGET_IP PORT
        |
        v
Authenticate (if required) using protocol commands
        |
        v
Execute protocol-specific commands to perform operations
        |
        v
Observe:
  - What information does the server reveal?
  - What is transmitted in cleartext?
  - What does an attacker see in a network capture?
        |
        v
Identify security implications:
  - Cleartext credentials
  - Version disclosure
  - Open relay / misconfiguration
  - Sensitive data in transit
```

This approach — understanding the protocol before the attack — builds the foundational knowledge that makes later attack techniques (sniffing, MITM, brute force) genuinely comprehensible rather than just command execution.

---

## Practical Activities

### Activity 1 — Telnet: Understanding Cleartext Authentication

**Objective:** Observe what a Telnet authentication session looks like and understand why it is insecure.

**Commands:**
```bash
telnet TARGET_IP 23
```

Then when prompted:
- Login: `frank`
- Password: `[REDACTED]`

**Command Explanation:** `telnet TARGET_IP 23` opens a TCP connection to the Telnet server on port 23, which prompts for credentials.

**Findings:**
- The server presents a login prompt
- The username is echoed back visibly
- The password is not displayed on screen (only visual protection)
- Once authenticated, a remote shell prompt appears (e.g., `frank@host:~$`)
- In a network capture, the password appears in plain ASCII — the visual suppression provides no real security

**Why It Matters:** This demonstrates the critical flaw of cleartext protocols — visual suppression of passwords does not prevent network capture. The same credentials that a user sees suppressed are visible to anyone analysing network traffic.

---

### Activity 2 — HTTP: Manual Web Request

**Objective:** Construct an HTTP request manually and extract server information from the response headers.

**Commands:**
```bash
telnet TARGET_IP 80
```

Then type:
```
GET /index.html HTTP/1.1
host: telnet

[press Enter twice]
```

**Command Explanation:**
- `telnet TARGET_IP 80` opens a TCP connection to the web server on port 80
- `GET /index.html HTTP/1.1` is a valid HTTP/1.1 request for the index page
- `host: telnet` satisfies the required Host header in HTTP/1.1
- Two Enter presses terminate the headers and signal the request is complete

**Findings:**
- HTTP 200 OK response with the page content
- `Server: nginx/1.18.0 (Ubuntu)` — web server software, version, and OS
- Additional headers may reveal `X-Powered-By:`, `Content-Type:`, `Date:`

**Why It Matters:** Version information in the `Server:` header enables immediate CVE research. Both the web server type and version are confirmed without requiring any specialised scanner.

---

### Activity 3 — FTP: Authentication and Enumeration

**Objective:** Authenticate to an FTP server manually using Telnet and enumerate its properties.

**Commands:**
```bash
# Manual FTP via Telnet (control channel)
telnet TARGET_IP 21

# FTP client for actual file transfer
ftp TARGET_IP
```

In the Telnet session:
```
USER frank
PASS [REDACTED]
SYST
STAT
PASV
```

In the FTP client:
```
ls
ascii
get filename.txt
bye
```

**Command Explanation:**
- `telnet TARGET_IP 21` connects to the FTP control channel
- `USER frank` identifies the user; `PASS [REDACTED]` authenticates
- `SYST` reveals system type (UNIX)
- `STAT` shows server status — explicitly states "Control connection is plain text"
- `PASV` switches to passive mode for data connections
- The FTP client is required for actual file download because file transfers use a separate data channel

**Findings:**
- Server system type and version from SYST
- STAT confirms cleartext control and data connections
- Directory listing reveals files available for download
- Files retrieved contain further intelligence for the engagement

**Why It Matters:** The STAT output explicitly confirming plaintext connections is a clear statement of the security risk. In a real assessment, FTP credentials captured from the control channel can often be reused on other services.

---

### Activity 4 — SMTP: Manual Email Submission

**Objective:** Observe the SMTP authentication and mail submission process and understand why email spoofing is structurally possible.

**Commands:**
```bash
telnet TARGET_IP 25
```

Then:
```
helo attacker
mail from: <attacker@example.com>
rcpt to: <frank@target.com>
data
Subject: Test email
This is a test message.
.
quit
```

**Command Explanation:**
- `helo` introduces the sending server to the receiving MTA
- `mail from:` specifies the envelope sender — no verification required
- `rcpt to:` specifies the recipient
- `data` begins the message body
- A single period on its own line terminates the message
- The server accepts and queues the message

**Findings:**
- The SMTP server accepts any value in `mail from:` without verifying ownership
- This is the structural basis for email spoofing — SMTP was designed without sender authentication

**Why It Matters:** This demonstration makes explicit why phishing emails can appear to come from legitimate addresses. Modern mitigations (SPF, DKIM, DMARC) exist to address this but require explicit deployment and configuration — a finding worth checking in assessments.

---

### Activity 5 — POP3: Email Retrieval

**Objective:** Authenticate to a POP3 server and retrieve email messages manually.

**Commands:**
```bash
telnet TARGET_IP 110
```

Then:
```
USER frank
PASS [REDACTED]
STAT
LIST
RETR 1
QUIT
```

**Command Explanation:**
- `telnet TARGET_IP 110` connects to the POP3 server
- `USER` and `PASS` authenticate in sequence
- `STAT` returns `+OK nn mm` — message count and total size in bytes
- `LIST` enumerates messages with their sizes
- `RETR 1` retrieves the full content of message 1
- `QUIT` ends the session and deletes messages marked with DELE (none in this case)

**Findings:**
- Successful authentication confirms valid credentials
- STAT shows message count and mailbox size
- RETR reveals full email content including headers and body

**Why It Matters:** Every POP3 command and every email retrieved is transmitted in cleartext. Network capture of a POP3 session yields valid credentials and potentially sensitive email content.

---

### Activity 6 — IMAP: Mailbox Enumeration

**Objective:** Authenticate to an IMAP server, enumerate the mailbox structure, and understand the protocol's synchronisation model.

**Commands:**
```bash
telnet TARGET_IP 143
```

Then (each command preceded by a unique tag):
```
c1 LOGIN frank [REDACTED]
c2 LIST "" "*"
c3 EXAMINE INBOX
c4 LOGOUT
```

**Command Explanation:**
- IMAP requires every command to be preceded by a unique client-assigned tag (c1, c2, etc.) that appears in the server's response, enabling the client to match responses to commands
- `LOGIN frank [REDACTED]` authenticates
- `LIST "" "*"` enumerates all mailbox folders
- `EXAMINE INBOX` opens the inbox for read-only inspection, reporting message count

**Findings:**
- Server capabilities revealed in initial CAPABILITY response (IMAP version, supported extensions)
- Folder structure: INBOX, Trash, Drafts, Templates, Sent
- INBOX message count and recent message flag from EXAMINE

**Why It Matters:** The folder structure provides an immediate roadmap for data exfiltration priorities. Credentials sent in `LOGIN frank password` are transmitted in cleartext — visible in any network capture.

---

## Observations and Analysis

- **STAT output from FTP explicitly states cleartext:** The FTP server's response to `STAT` included the text "Control connection is plain text" and "Data connections will be plain text". This kind of explicit disclosure is useful evidence in a penetration test report — the server itself acknowledges the security weakness.

- **SMTP sender field is unrestricted:** The mail from: field accepts any address without verification. This is the structural root cause of email spoofing. SPF, DKIM, and DMARC are mitigations, not fixes — they depend on receiving servers checking them and on senders configuring them.

- **POP3 STAT response format:** The `+OK nn mm` format is defined in RFC 1939. Understanding this specification is useful when building automated tools or parsing POP3 responses in scripts.

- **IMAP tag requirement:** IMAP's command tagging system (c1, c2, c3) is unusual among protocols and trips up many people interacting with it manually for the first time. Each server response includes the original tag, allowing multiplexed command/response correlation. This is a design feature for asynchronous operation.

- **FTP passive vs active mode:** The active/passive distinction matters in real assessments. Clients behind NAT or firewalls (almost all of them) will fail active mode FTP. Passive mode is the practical default. This is also relevant when exploiting FTP servers — passive mode connections are easier to capture and analyse.

---

## Tools and Technologies Used

### telnet (client)
- **Purpose:** Raw TCP connection for manual protocol interaction
- **Common Usage:** `telnet TARGET_IP PORT`
- **In This Room:** Used to manually interact with all five protocols — demonstrates exactly what each protocol transmits

### ftp (FTP client)
- **Purpose:** Full FTP client with data channel support for file transfer
- **Common Usage:** `ftp TARGET_IP`
- **In This Room:** Required for actual file downloads (Telnet alone cannot establish the FTP data channel)

---

## Protocol Reference

| Protocol | Port | Purpose | Data Security | Secure Alternative | Secure Port |
|----------|------|---------|--------------|-------------------|-------------|
| Telnet | 23 | Remote access | Cleartext | SSH | 22 |
| HTTP | 80 | Web transfer | Cleartext | HTTPS | 443 |
| FTP | 21 | File transfer | Cleartext | FTPS / SFTP | 990 / 22 |
| SMTP | 25 | Email sending (MTA) | Cleartext | SMTPS / SMTP+STARTTLS | 465 / 587 |
| POP3 | 110 | Email retrieval | Cleartext | POP3S | 995 |
| IMAP | 143 | Email access + sync | Cleartext | IMAPS | 993 |

---

## Key Learnings

- All five protocols transmit credentials in cleartext — anyone capturing network traffic between client and server can read usernames and passwords
- Telnet's visual password suppression provides zero security against network capture
- HTTP's `Server:` header is a direct information disclosure vector — version-specific server software enables targeted CVE research
- FTP's dual-connection (control + data) architecture means file content and credentials are captured in separate but equally readable streams
- SMTP's unrestricted `mail from:` field is the structural basis for email spoofing
- POP3's download-and-delete model makes multi-device email access problematic and is why IMAP became dominant
- IMAP's server-side storage model makes it far more valuable to compromise than POP3 — persistent, history-complete mailbox access

---

## Real-World Relevance

**Penetration Testing:** Finding cleartext protocols running on internal networks is a common finding. Demonstrating captured credentials from a network sniff is one of the most compelling findings in a report — it requires no exploitation, just network position.

**Security Assessments:** Checking for cleartext email protocols (POP3 on 110, IMAP on 143, SMTP on 25) is standard in any network assessment. Finding these open is typically rated medium-to-high severity depending on the sensitivity of the data they carry.

**Enterprise Environments:** Legacy mail servers, IoT devices, and embedded systems in corporate environments frequently still run cleartext protocols. Internal network sniffing during a pentest regularly yields email credentials that work on other internal systems.

**Bug Bounty:** Cleartext protocol exposure in cloud environments (e.g., unencrypted mail submission endpoints) is a valid finding, though scope restrictions vary by programme.

---

## Things Worth Remembering

**Protocol Ports (cleartext):**
- Telnet: 23
- HTTP: 80
- FTP: 21 (control), 20 (active data)
- SMTP: 25 (MTA-MTA), 587 (submission/STARTTLS)
- POP3: 110
- IMAP: 143

**Quick reference commands:**
```bash
# Telnet to any TCP service
telnet TARGET_IP PORT

# HTTP banner grab via telnet
telnet TARGET_IP 80
# then: GET / HTTP/1.1 [Enter] host: x [Enter][Enter]

# FTP control channel via telnet
telnet TARGET_IP 21
# then: USER username [Enter] PASS password [Enter]

# FTP with actual file transfer
ftp TARGET_IP

# SMTP session
telnet TARGET_IP 25
# then: helo x → mail from: → rcpt to: → data → . → quit

# POP3 session
telnet TARGET_IP 110
# then: USER frank → PASS password → STAT → LIST → RETR 1

# IMAP session
telnet TARGET_IP 143
# then: c1 LOGIN frank password → c2 LIST "" "*" → c3 EXAMINE INBOX
```

**Key concepts:**
- Visual password suppression ≠ network security
- FTP: control channel + data channel = two separate TCP connections
- SMTP `mail from:` accepts any address = email spoofing is structurally possible
- POP3 STAT format: `+OK message_count total_bytes`
- IMAP requires unique tags (c1, c2) on every command

---

## Conclusion

Protocols and Servers 1 builds the foundational understanding of how network services communicate at the protocol level. By interacting with HTTP, FTP, SMTP, POP3, and IMAP manually through Telnet, you observe exactly what is transmitted across the wire and develop an intuitive sense for why cleartext protocols create security risk. Every credential entered, every file transferred, and every email read is plainly visible to anyone in a position to capture the traffic. This understanding is the prerequisite for the next room, where these same protocols become the targets of sniffing, MITM, and brute force attacks, and where TLS and SSH emerge as the necessary mitigations.
