# Protocols and Servers 2

## Overview

This room builds directly on Protocols and Servers 1 by examining the attacks that become possible when cleartext protocols are in use, and the defences that modern deployments employ to counter them. The three primary attack categories are sniffing (network packet capture), Man-in-the-Middle (MITM), and password attacks. The room then covers Transport Layer Security (TLS) as the universal encryption layer that protects these protocols, SSH as the secure replacement for Telnet, and Hydra as the standard tool for testing authentication security.

**Main objectives:**
- Understand the CIA triad and how it maps to the DAD attack model
- Perform and understand sniffing attacks using tcpdump and Wireshark
- Understand MITM attack mechanics and the tools used to execute them
- Understand how TLS protects protocols and how the TLS handshake works
- Use SSH securely with key-based authentication and file transfer
- Use Hydra to perform password attacks against network services

**Skills introduced:** tcpdump filtering, Wireshark analysis, TLS concepts, SSH key generation, SFTP, Hydra password attacks

---

## Concepts Covered

### The CIA Triad and DAD Attack Model

**Definition:** The CIA triad (Confidentiality, Integrity, Availability) is the foundational security framework for evaluating what needs protecting. The DAD model (Disclosure, Alteration, Destruction) represents the corresponding attack objectives.

**Why It Matters:** Security decisions, vulnerability severity ratings, and attack impact assessments all reference CIA. Understanding the mapping between CIA and DAD clarifies why certain attacks matter.

**Key Details:**

| CIA Property | Definition | Attack (DAD) | Attack Type |
|-------------|-----------|-------------|-------------|
| Confidentiality | Data accessible only to intended parties | Disclosure | Sniffing, credential capture |
| Integrity | Data is accurate and unaltered | Alteration | MITM attack, data modification |
| Availability | Services accessible when needed | Destruction/Denial | DoS, ransomware |

- Network packet capture (sniffing) violates Confidentiality → causes Disclosure
- MITM attacks violate Integrity → cause Alteration
- Password attacks, if successful, violate Confidentiality → cause Disclosure

**Practical Relevance:** When writing pentest findings, framing impact in CIA terms makes findings more meaningful to stakeholders. "Allows disclosure of sensitive data" maps directly to Confidentiality and has regulatory implications (GDPR, PCI-DSS).

---

### Sniffing Attacks

**Definition:** A sniffing attack uses a network packet capture tool to intercept and read data transmitted across a network. When protocols communicate in cleartext, the data (including credentials) is readable in the captured packets.

**Why It Matters:** Sniffing remains highly relevant despite TLS adoption. Internal corporate networks, legacy systems, IoT devices, and misconfigured services still use cleartext protocols. Network access (via compromised host, MITM position, or switch port mirroring) enables credential capture without any exploitation.

**Key Details:**

**Where sniffing attacks still apply:**
- Internal networks where inter-service traffic is unencrypted
- Legacy mail servers, embedded devices, industrial control systems
- Misconfigured services where TLS is available but not enforced
- IoT devices using cleartext protocols
- After a successful MITM attack that downgrades or strips encryption

**Packet capture tools:**
- **Tcpdump:** CLI-based, lightweight, available on most Linux systems by default, excellent for scripting
- **Wireshark:** GUI-based, powerful filtering, protocol dissection, visualisation
- **Tshark:** CLI version of Wireshark's dissection engine, useful for automation
- Others: tcpflow (TCP stream reassembly), ngrep (pattern matching), NetworkMiner (file extraction)

**Access requirements:** Packet capture requires root/administrator privileges. The attacker must have access to network traffic between the client and server — achieved via ARP spoofing, switch port mirroring, or a compromised system on the same network segment.

**Practical Relevance:** During internal penetration tests, demonstrating credential capture from sniffing a POP3 or FTP session is one of the most impactful findings possible — no exploitation required, just network position.

---

### Man-in-the-Middle (MITM) Attacks

**Definition:** A MITM attack occurs when an attacker positions themselves between two communicating parties, allowing them to intercept, read, and potentially modify traffic without either party realising.

**Why It Matters:** MITM attacks violate both Confidentiality (attacker reads traffic) and Integrity (attacker can modify traffic). They can affect both cleartext and encrypted protocols when implemented incorrectly.

**Key Details:**

**How MITM attacks achieve network position:**
- **ARP Spoofing:** Sends forged ARP messages to associate the attacker's MAC address with the gateway/target IP. Effective on local networks.
- **DNS Spoofing:** False DNS responses redirect victims to attacker-controlled servers.
- **Rogue Access Points:** Fake Wi-Fi access points that route all victim traffic through the attacker.
- **BGP Hijacking:** Internet routing-level attack announcing false BGP routes.

**MITM tools:**
- **Bettercap:** Modern, actively maintained; supports ARP spoofing, DNS spoofing, HTTP/HTTPS proxying
- **Ettercap:** Classic LAN MITM tool; functional but Bettercap is generally preferred
- **mitmproxy:** Interactive HTTPS proxy for inspecting and modifying encrypted traffic
- **Responder:** Exploits LLMNR/NBT-NS name resolution on Windows networks; common in Active Directory internal pentests

**MITM against encrypted protocols:**
- **SSL Stripping:** Downgrades HTTPS to HTTP. Attacker connects to real server over HTTPS, serves victim over HTTP. Victim may not notice the missing padlock.
- **Fake Certificates:** Attacker presents their own certificate — works if the victim accepts invalid certificate warnings.

**Modern MITM defences:**
- **HSTS (HTTP Strict Transport Security):** Tells browsers to only use HTTPS for a specified period. Prevents SSL stripping for sites with valid HSTS headers.
- **Certificate Transparency (CT):** CAs must log all certificates publicly. Makes fraudulent certificate issuance detectable.
- **Certificate Pinning:** Applications specify exactly which certificates are valid — prevents attacks even with compromised CAs.
- **DANE:** Uses DNSSEC to publish certificate information, providing an alternative trust path.

**Practical Relevance:** ARP spoofing + Bettercap is a standard internal pentest technique for positioning to capture credentials and demonstrate the impact of unencrypted protocols. Responder is one of the most commonly used tools in Active Directory internal assessments for capturing NTLMv2 hashes.

---

### TLS (Transport Layer Security)

**Definition:** TLS is the cryptographic protocol that provides confidentiality, integrity, and authentication for application-layer protocols by encrypting the transport layer.

**Why It Matters:** TLS is the solution to sniffing and MITM attacks against cleartext protocols. Understanding how TLS works explains why HTTPS is secure, why expired or invalid certificates matter, and why legacy TLS versions are dangerous.

**Key Details:**

**History:**
- SSL 2.0/3.0: Deprecated, insecure — never use
- TLS 1.0/1.1: Deprecated 2021 — major browsers no longer support
- TLS 1.2 (2008): Widely used, secure with modern cipher suites
- TLS 1.3 (2018): Current standard, faster, forward secrecy by default, simplified cipher suites

**How TLS fits in the protocol stack:** TLS sits between the transport layer (TCP) and the application layer. Application protocol data (HTTP, SMTP, etc.) passes through TLS for encryption before being handed to TCP for transmission.

**Encrypted protocol ports:**

| Protocol | Port | Secured As | Port with TLS |
|----------|------|-----------|--------------|
| HTTP | 80 | HTTPS | 443 |
| FTP | 21 | FTPS | 990 |
| SMTP | 25 | SMTPS | 465 |
| SMTP submission | 587 | SMTP+STARTTLS | 587 |
| POP3 | 110 | POP3S | 995 |
| IMAP | 143 | IMAPS | 993 |

**Implicit TLS vs STARTTLS:**
- **Implicit TLS:** Connection encrypted from the first packet (ports 443, 993, 995, 465)
- **STARTTLS:** Connection starts cleartext; client issues STARTTLS command to upgrade to TLS (common on ports 587, 143, 110)
- STARTTLS can be vulnerable to downgrade attacks if an attacker strips the STARTTLS command before the client sees it

**TLS Handshake (TLS 1.2 simplified):**
1. ClientHello: client sends supported TLS versions, cipher suites, and a random value
2. ServerHello: server responds with selected parameters and its certificate
3. Key Exchange: both sides exchange information to derive a shared secret key
4. Finished: both sides confirm the handshake and switch to encrypted communication

**TLS 1.3 improvements over 1.2:**
- 1-RTT handshake (vs 2-RTT for TLS 1.2) — faster connection establishment
- 0-RTT resumption for returning clients (with some security trade-offs)
- Forward secrecy by default for all cipher suites
- Encrypted handshake — more of the negotiation is hidden from observers
- Removed insecure/deprecated algorithms

**Certificates and Trust:** TLS relies on a PKI (Public Key Infrastructure) chain: certificates are signed by Certificate Authorities (CAs) trusted by the client's operating system. A certificate proves the server is who it claims to be — MITM attacks are prevented because an attacker cannot obtain a valid certificate for the target domain (assuming the CA system is not compromised).

**Let's Encrypt:** Free, automated certificate issuance. HTTPS adoption grew from under 50% of web traffic in 2015 to over 95% today largely because of Let's Encrypt removing the cost barrier.

**Testing TLS configurations:**
- **testssl.sh:** CLI tool for detailed TLS configuration assessment — best for internal systems
- **sslyze:** Python tool for automation and CI/CD integration
- **SSL Labs (ssllabs.com):** Web-based, detailed analysis of public HTTPS servers
- **nmap ssl-enum-ciphers:** Enumerates supported cipher suites as part of a port scan

**Practical Relevance:** Finding support for deprecated TLS versions (TLS 1.0/1.1) or weak cipher suites is a reportable finding in any web or network assessment. Certificate validation failures (expired, self-signed, mismatched hostname) indicate potential MITM exposure.

---

### SSH (Secure Shell)

**Definition:** SSH is a cryptographic protocol for secure remote system administration, providing confidentiality, integrity, and server/client authentication. It replaced Telnet as the universal standard for remote CLI access.

**Why It Matters:** SSH is used for remote access, file transfer (SFTP/SCP), tunnelling, and port forwarding. Understanding its authentication methods and configuration is essential for both using it effectively and for identifying misconfigurations during assessments.

**Key Details:**

**SSH authentication methods:**
- **Password authentication:** Username + password over encrypted connection; vulnerable to brute force if weak passwords are used
- **Public key authentication:** Cryptographic key pair (private key on client, public key on server); recommended for regular use
- **Certificate-based authentication:** SSH CA signs user and host keys — scales well in enterprise environments
- **MFA:** Combines key + one-time password for high-security access

**Connecting via SSH:**
```bash
ssh username@TARGET_IP
ssh -p 2222 username@TARGET_IP         # non-standard port
ssh -i ~/.ssh/custom_key user@TARGET_IP # specific private key
ssh -J bastion.example.com user@internal-server  # jump host
```

**Host key verification:** First connection to a new server shows the server's public key fingerprint. Verify this out-of-band before accepting — accepting without verification is a MITM risk. Accepted keys are stored in `~/.ssh/known_hosts`.

**Key generation:**
```bash
# Ed25519 (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# RSA (use if Ed25519 not available)
ssh-keygen -t rsa -b 4096

# Copy public key to server
ssh-copy-id username@TARGET_IP
```

**Secure file transfer:**
```bash
# SFTP (recommended, interactive)
sftp username@TARGET_IP

# SCP (deprecated in OpenSSH, still functional)
scp file.txt username@TARGET_IP:/remote/path/

# rsync over SSH (preferred for large transfers)
rsync -avz -e ssh /local/dir/ username@TARGET_IP:/remote/dir/
```

**SSH vs FTPS vs SFTP:**
- SFTP: runs over SSH (port 22) — most common and recommended
- FTPS: FTP + TLS (port 990) — different protocol despite similar name
- SCP: also over SSH but being deprecated in favour of SFTP

**SSH hardening considerations:**
- Disable password authentication after setting up key-based auth: `PasswordAuthentication no`
- Disable root login: `PermitRootLogin no`
- Restrict allowed users: `AllowUsers` or `AllowGroups`
- Use fail2ban or equivalent to block repeated failures
- Configure modern key exchange algorithms: `KexAlgorithms`, `Ciphers`, `MACs`

**Practical Relevance:** SSH key management weaknesses (private keys without passphrases, keys stored in accessible locations, authorised_keys files with overly broad access) are common findings in enterprise assessments. Finding SSH host key mismatches triggers a MITM investigation.

---

### Password Attacks with Hydra

**Definition:** Password attacks attempt to gain authenticated access by testing credentials against a service. Hydra is the standard tool for testing password strength against network services.

**Why It Matters:** Weak passwords remain the most prevalent authentication vulnerability across all environments. Understanding how automated attacks work informs both offensive testing and defensive password policy design.

**Key Details:**

**Types of password attacks:**
- **Password Guessing:** Uses target-specific knowledge (pet name, birth year, company name)
- **Dictionary Attack:** Tests words from a wordlist
- **Brute Force:** Tests all character combinations — exponentially slower with longer passwords
- **Credential Stuffing:** Uses leaked username/password pairs from breaches — exploits password reuse
- **Password Spraying:** Tests a small number of common passwords against many accounts — evades per-account lockout
- **Hybrid Attack:** Dictionary words combined with common patterns (seasons, years, common substitutions)

**Wordlists:**
- `/usr/share/wordlists/rockyou.txt`: Classic breach-derived wordlist
- SecLists (`/usr/share/seclists/`): Comprehensive collection for various attack scenarios
- Custom wordlists tailored to the target are often most effective

**Hydra syntax:**
```bash
hydra -l USERNAME -P WORDLIST.txt TARGET_IP SERVICE
```

**Hydra options:**

| Option | Description |
|--------|-------------|
| `-l username` | Single login name |
| `-L users.txt` | File of usernames |
| `-p password` | Single password to try |
| `-P wordlist.txt` | Password list file |
| `-s PORT` | Non-default service port |
| `-V` or `-vV` | Verbose — show each attempt |
| `-t n` | Number of parallel connections |
| `-w n` | Wait time between connections |
| `-f` | Stop after first valid credential found |
| `-d` | Debug mode |

**Hydra examples:**
```bash
# FTP
hydra -l mark -P /usr/share/wordlists/rockyou.txt TARGET_IP ftp

# SSH
hydra -l frank -P /usr/share/wordlists/rockyou.txt TARGET_IP ssh

# IMAP
hydra -l lazie -P /usr/share/wordlists/rockyou.txt TARGET_IP imap

# POP3
hydra -l frank -P /usr/share/wordlists/rockyou.txt TARGET_IP pop3

# Credential stuffing (list of usernames)
hydra -L users.txt -P passwords.txt TARGET_IP ssh
```

**Mitigating password attacks:**
- Password policies (NIST SP 800-63B recommends length over complexity, blocking known compromised passwords)
- Account lockout (effective against brute force; bypassed by spraying)
- Rate limiting and throttling (delays automated tools; exponential backoff is most effective)
- CAPTCHA (behavioural analysis-based; harder to bypass than image recognition)
- MFA (most effective single defence — compromised password alone is insufficient)
- Passwordless authentication (FIDO2/WebAuthn passkeys eliminate passwords entirely)
- Breached password detection (check credentials against HaveIBeenPwned API at registration/login)

**Practical Relevance:** Hydra is used in every penetration test where authentication services are in scope. Finding a single valid credential pair confirms password policy weakness and often yields lateral movement opportunities through credential reuse.

---

## Methodology

### Sniffing Attack Workflow

```
Gain access to network traffic:
  - Same network segment (internal pentest)
  - Switch port mirroring configured
  - ARP spoofing via Bettercap/Ettercap
        |
        v
Capture traffic with tcpdump or Wireshark
        |
        v
Filter for target protocol:
  sudo tcpdump port 110 -A    (POP3)
  sudo tcpdump port 21 -A     (FTP)
  sudo tcpdump port 25 -A     (SMTP)
        |
        v
Extract credentials from captured traffic:
  - USER/PASS commands in POP3
  - USER/PASS commands in FTP
  - LOGIN command in IMAP
  - Credentials in HTTP POST bodies
        |
        v
Document: protocol, source/dest IPs, captured credentials
```

### Password Attack Workflow (Hydra)

```
Identify target service and port (from Nmap scan)
        |
        v
Identify or enumerate valid usernames
        |
        v
Select appropriate wordlist for target context
        |
        v
Run Hydra:
  hydra -l USERNAME -P WORDLIST TARGET_IP SERVICE
        |
        v
Valid credential found → test on other services (credential reuse)
        |
        v
Document: service, port, username, cracked password
Report: severity based on service criticality and password weakness
```

---

## Practical Activities

### Activity 1 — Capturing POP3 Credentials with tcpdump

**Objective:** Intercept and extract cleartext POP3 credentials from network traffic.

**Commands:**
```bash
# Capture POP3 traffic in ASCII format
sudo tcpdump port 110 -A

# Capture traffic to/from a specific host
sudo tcpdump host TARGET_IP -A

# Write to file for later analysis
sudo tcpdump port 110 -w capture.pcap

# Read from capture file
tcpdump -r capture.pcap -A
```

**Command Explanation:**
- `sudo` is required because packet capture requires root privileges
- `port 110` filters to only POP3 traffic
- `-A` displays captured packet contents in ASCII, making cleartext credentials readable in the output
- Packets containing `USER frank` and `PASS [REDACTED]` appear directly in the output

**Findings:** Username and password transmitted in separate TCP packets, both visible in plain ASCII in the capture output.

**Why It Matters:** This is one of the most compelling findings in an internal pentest report. The credential capture requires only network position — no exploitation. The severity is high because the same credentials are often reused on other systems.

---

### Activity 2 — IMAP Password Attack with Hydra

**Objective:** Identify the password for a known IMAP account using Hydra.

**Context:** Username `lazie` is known from earlier enumeration. The password is unknown.

**Commands:**
```bash
hydra -l lazie -P /usr/share/wordlists/rockyou.txt TARGET_IP imap
```

**Command Explanation:**
- `-l lazie` specifies the single known username
- `-P /usr/share/wordlists/rockyou.txt` specifies the password wordlist
- `TARGET_IP imap` specifies the target and service (Hydra handles the protocol interaction)
- Hydra opens parallel connections, tests each password from the wordlist, and reports success when valid credentials are found

**Findings:** Valid IMAP credentials identified: `lazie : [REDACTED]`

**Why It Matters:** A successful password attack against IMAP provides full mailbox access. Given IMAP's persistent server-side storage model, this yields the complete email history and ongoing access to new email — a critical finding.

---

### Activity 3 — SSH Key-Based Authentication Setup

**Objective:** Set up SSH public key authentication as the secure alternative to password authentication.

**Commands:**
```bash
# Generate Ed25519 key pair
ssh-keygen -t ed25519 -C "email@example.com"

# Copy public key to target
ssh-copy-id mark@TARGET_IP

# Connect using key (no password prompt if configured correctly)
ssh mark@TARGET_IP

# Verify connection works, then disable password authentication
# In /etc/ssh/sshd_config on the server:
# PasswordAuthentication no
```

**Command Explanation:**
- `ssh-keygen -t ed25519` generates an Ed25519 key pair (recommended over RSA)
- Private key stored at `~/.ssh/id_ed25519` — must be protected with a passphrase
- Public key stored at `~/.ssh/id_ed25519.pub` — safe to share
- `ssh-copy-id` appends the public key to `~/.ssh/authorized_keys` on the remote server

**Findings:** Successful key-based SSH connection without password prompt confirms the key pair is correctly configured.

**Why It Matters:** Key-based authentication eliminates the brute force attack surface against SSH. Password authentication on internet-facing SSH is a reportable finding because automated credential stuffing attacks are constant.

---

### Activity 4 — SFTP File Transfer

**Objective:** Transfer files securely using SFTP over SSH.

**Commands:**
```bash
# Connect via SFTP
sftp mark@TARGET_IP

# SFTP operations
# ls — list remote files
# put localfile.txt — upload
# get remotefile.txt — download
# bye — exit

# SCP alternative (note: deprecated in favour of SFTP)
scp document.txt mark@TARGET_IP:/home/mark/
```

**Command Explanation:**
- SFTP operates over the SSH connection — all data including file contents is encrypted in transit
- `scp` is still functional but OpenSSH has deprecated the SCP protocol in favour of SFTP for security reasons

**Why It Matters:** SFTP replaces FTP as the secure alternative for file transfer over SSH (port 22), eliminating the cleartext credential and data exposure of plain FTP.

---

## Observations and Analysis

- **Internal network sniffing is consistently productive:** In enterprise assessments, internal network segments frequently carry unencrypted traffic between systems — database connections, service-to-service APIs, legacy mail servers. Sniffing remains a high-value technique inside the perimeter even when perimeter controls are strong.

- **HSTS bypasses the SSL stripping window:** Once a browser has seen an HSTS header for a domain, it will refuse HTTP connections for the specified period. However, first-time visitors have no HSTS state yet — the first connection can still be stripped. HSTS preloading addresses this by shipping browser databases with known HSTS sites.

- **Password spraying vs brute force:** The distinction matters for detection avoidance. Brute force against one account is caught by account lockout. Password spraying one or two passwords against thousands of accounts is often below lockout thresholds. Tools like Hydra can simulate spraying with `-L users.txt -p Password123` (one password, many usernames).

- **TLS 1.2 vs 1.3 performance:** TLS 1.3's 1-RTT handshake is measurably faster than TLS 1.2's 2-RTT — roughly halving handshake latency. This is why TLS 1.3 adoption is accelerating and why supporting TLS 1.2 only is increasingly a performance finding as well as a security one.

- **STARTTLS downgrade vulnerability:** If an attacker strips the STARTTLS command from an SMTP or IMAP session before the client sees it, the client falls back to cleartext. Implicit TLS (separate port, always encrypted from the first byte) is not susceptible to this attack. Organisations should prefer implicit TLS ports (993, 995, 465) over STARTTLS upgrades when possible.

---

## Tools and Technologies Used

### tcpdump
- **Purpose:** CLI-based network packet capture
- **Common Usage:** `sudo tcpdump port PORT -A`
- **In This Room:** Capturing cleartext POP3 credentials to demonstrate sniffing attack impact

### Wireshark
- **Purpose:** GUI-based network packet capture and analysis
- **Common Usage:** Launch Wireshark, set interface, apply display filter (e.g., `pop`)
- **In This Room:** Visual demonstration of captured POP3 credentials

### Hydra
- **Purpose:** Fast, flexible online password attack tool supporting many protocols
- **Common Usage:** `hydra -l user -P wordlist TARGET_IP SERVICE`
- **In This Room:** IMAP password attack to recover the password for user `lazie`

### ssh / ssh-keygen / ssh-copy-id
- **Purpose:** Secure remote shell, key generation, and key distribution
- **Common Usage:** `ssh user@TARGET_IP`, `ssh-keygen -t ed25519`
- **In This Room:** Demonstrated as the secure alternative to Telnet, with key-based auth setup

### sftp / scp
- **Purpose:** Secure file transfer over SSH
- **Common Usage:** `sftp user@TARGET_IP`
- **In This Room:** Demonstrated as the secure alternative to FTP

---

## Protocol and Port Reference

| Protocol | Port | Purpose | Encryption |
|----------|------|---------|-----------|
| FTP | 21 | File transfer | Cleartext |
| FTPS | 990 | File transfer | Implicit TLS |
| SFTP | 22 | File transfer | SSH |
| HTTP | 80 | Web | Cleartext |
| HTTPS | 443 | Web | Implicit TLS |
| IMAP | 143 | Email (MDA) | Cleartext |
| IMAPS | 993 | Email (MDA) | Implicit TLS |
| POP3 | 110 | Email (MDA) | Cleartext |
| POP3S | 995 | Email (MDA) | Implicit TLS |
| SMTP | 25 | Email (MTA) | Cleartext |
| SMTP Submission | 587 | Email (MTA, client) | STARTTLS |
| SMTPS | 465 | Email (MTA) | Implicit TLS |
| SSH | 22 | Remote access + file transfer | Always encrypted |
| Telnet | 23 | Remote access | Cleartext |

---

## Key Learnings

- Sniffing attacks expose credentials from any cleartext protocol — network position is all that is needed, no exploitation required
- MITM attacks violate integrity and can affect both cleartext and encrypted protocols when TLS is improperly implemented
- TLS 1.2 and 1.3 are the current secure standards; TLS 1.0/1.1 are deprecated and finding them enabled is a reportable finding
- TLS 1.3 provides 1-RTT handshake, forward secrecy by default, and an encrypted handshake
- SSH replaced Telnet — always use key-based authentication, disable password authentication for internet-facing services
- Hydra is the standard tool for network service password attacks; Medusa and Ncrack are functional alternatives
- MFA is the single most effective defence against password attacks — a compromised password alone is not enough
- Credential stuffing (using leaked breach data) is the most prevalent real-world password attack technique

---

## Real-World Relevance

**Penetration Testing:** Sniffing + credential capture is one of the highest-impact internal pentest techniques. Hydra is used in virtually every engagement where authentication services are in scope. Finding weak service passwords that are also reused on domain accounts is a critical escalation path.

**Red Team Operations:** ARP spoofing + Responder for NTLMv2 hash capture is a standard internal network technique in Windows Active Directory environments. Captured hashes can be cracked offline or used directly in Pass-the-Hash attacks.

**Bug Bounty:** Testing for cleartext protocol exposure in cloud environments is valid where in scope. Password attack testing requires explicit scope permission.

**Enterprise Security:** TLS configuration assessments (checking for deprecated protocols and weak cipher suites) are standard components of quarterly security assessments in PCI-DSS and SOC2 environments. SSH key management audits catch orphaned keys and unauthorised access paths.

---

## Things Worth Remembering

**Hydra quick reference:**
```bash
# FTP
hydra -l mark -P rockyou.txt TARGET_IP ftp

# SSH
hydra -l frank -P rockyou.txt TARGET_IP ssh

# IMAP
hydra -l lazie -P rockyou.txt TARGET_IP imap

# POP3
hydra -l frank -P rockyou.txt TARGET_IP pop3

# Non-standard port
hydra -l user -P wordlist TARGET_IP -s PORT service

# List of usernames
hydra -L users.txt -P wordlist TARGET_IP ssh
```

**tcpdump quick reference:**
```bash
# Capture POP3 traffic (cleartext credentials)
sudo tcpdump port 110 -A

# Capture FTP traffic
sudo tcpdump port 21 -A

# Capture HTTP POST data
sudo tcpdump port 80 -A

# Capture to file
sudo tcpdump -w capture.pcap

# Read from file
tcpdump -r capture.pcap -A
```

**Key concepts:**
- Sniffing = Confidentiality violation (Disclosure)
- MITM = Integrity violation (Alteration)
- STARTTLS vs Implicit TLS: STARTTLS upgrades cleartext; Implicit TLS starts encrypted
- TLS 1.3 improvements: 1-RTT, forward secrecy by default, encrypted handshake, simplified ciphers
- SSH: Ed25519 keys preferred over RSA; always disable password auth for internet-facing services
- Hydra: `-l` single user, `-L` user list, `-P` password list, `-f` stop on first success

---

## Conclusion

Protocols and Servers 2 closes the loop on the protocols introduced in Part 1 by demonstrating what goes wrong when they are left unencrypted and how modern security controls address these weaknesses. Sniffing, MITM, and password attacks are not theoretical — they are standard techniques in every internal penetration test. TLS protects confidentiality and integrity at the transport layer, SSH replaces Telnet for all remote access, and Hydra quantifies the risk from weak passwords against real services. Together, these techniques and their mitigations represent the core of network service security assessment — skills that apply in every professional penetration test, red team engagement, and security architecture review.
