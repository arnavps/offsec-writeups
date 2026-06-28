# Network Security Protocols

## Executive Summary

Network security protocols are the standardised methods that protect data confidentiality, integrity, and authenticity as it traverses networks. This room provides a deep technical understanding of the secure variants of common protocols — HTTPS, FTPS, SMTPS, POP3S, DNSSEC, SSH, SSL/TLS, SOCKS5, IPsec, and OpenPGP — explaining how they work internally, what attacks they prevent, and why they exist alongside their insecure counterparts.

**Why it matters:** Understanding the mechanics of secure protocols is essential for security engineers who must evaluate protocol choices, identify misconfigurations, investigate traffic anomalies, and design systems that protect data in transit. Every TLS handshake, every SSH connection, and every signed email relies on the concepts in this room.

**Where these concepts apply:** TLS underpins HTTPS, FTPS, SMTPS, POP3S. SSH secures remote administration. IPsec provides VPN tunnelling. DNSSEC prevents DNS spoofing. These are the protocols you will encounter in every enterprise environment.

---

## Big Picture Overview

Without secure protocols, all network communication is plaintext — readable by anyone with packet capture capability on the same network path. Secure protocols address this through three mechanisms:

```
Encryption    → Prevents reading (Confidentiality)
Authentication → Prevents impersonation (Authenticity)
Integrity     → Prevents modification (Integrity)

Insecure: HTTP, FTP, SMTP, Telnet, DNS
Secure:  HTTPS, FTPS, SMTPS, SSH, DNSSEC
```

Each secure protocol wraps or replaces its insecure counterpart with SSL/TLS or a custom cryptographic layer.

---

## Core Concepts

---

### Concept 1 — HTTPS

**What Is It?**
HTTPS (HyperText Transfer Protocol Secure) is HTTP transmitted over SSL/TLS. It provides an encrypted channel between web browser and web server on TCP port 443.

**Why It Exists:**
HTTP transmits everything in cleartext — credentials, session cookies, form data, and page content. Any on-path attacker (ISP, coffee shop Wi-Fi, corporate proxy) can read and modify HTTP traffic. HTTPS prevents this.

**How It Works:**
```
Client                              Server
  |------ ClientHello (TLS version, cipher suites, random) ------>|
  |<----- ServerHello (chosen cipher, certificate, random) -------|
  |       [Client verifies certificate against trusted CAs]        |
  |------ Pre-master secret (encrypted with server's public key) ->|
  |       [Both sides derive session keys from:                    |
  |        client random + server random + pre-master secret]      |
  |------ Finished (encrypted with session key) ------------------>|
  |<----- Finished (encrypted with session key) ------------------|
  |====== Encrypted HTTP traffic using AES session key ===========|
```

**What HTTPS Protects:**
- Confidentiality: session tokens, passwords, sensitive form data
- Integrity: page content cannot be modified in transit (no ad injection, no script injection)
- Authenticity: certificate chain verifies the server is who it claims to be

**What HTTPS Does Not Protect:**
- Metadata: destination IP, SNI (Server Name Indication) field, and packet sizes are visible
- Certificate validity checking requires OCSP or CRL — not always enforced
- The server itself — HTTPS protects the transport, not the server-side application

**Ports:**
- HTTP: TCP 80
- HTTPS: TCP 443

**Security Engineer Note:**
HSTS (HTTP Strict Transport Security) instructs browsers to only connect via HTTPS, preventing SSL stripping attacks. Configure via the `Strict-Transport-Security` response header.

---

### Concept 2 — FTPS

**What Is It?**
FTPS (File Transfer Protocol Secure) is FTP extended with TLS/SSL support. It adds encryption to both the command channel and data channel of FTP.

**FTP Architecture (Prerequisites):**
FTP uses two separate TCP connections:
- **Command/Control channel** (port 21): Authentication commands, directory listing commands
- **Data channel** (port 20 active / negotiated passive): Actual file data transfer

**FTP Modes:**

| Mode | Who Initiates Data Connection | Works Behind Firewall? |
|------|------------------------------|----------------------|
| **Active** | Server connects to client on port 20 | No — inbound connection to client blocked |
| **Passive** | Client connects to server on random high port | Yes — client initiates outbound |

In active mode, the server initiates the data connection back to the client. If the client is behind a firewall, this inbound connection is typically blocked. Passive mode solves this — the client initiates both connections.

**FTPS Connection Methods:**

| Method | Description | Ports |
|--------|------------|-------|
| **Explicit (STARTTLS)** | Client connects on port 21, then explicitly requests TLS upgrade via AUTH TLS command | 21 (command), negotiated (data) |
| **Implicit** | TLS is mandatory from the first connection; no unencrypted phase | 990 (command), 989 (data) |

**Security Benefit:**
FTPS prevents sniffing attacks against login credentials and file content. Without FTPS, FTP credentials and files are transmitted in cleartext.

**FTP Data Types:**
- **ASCII (Type A):** Text files — line ending conversion between OS formats
- **Binary (Type I):** Byte-for-byte transfer — required for non-text files (images, executables)
- **EBCDIC (Type E):** Mainframe character set
- **Local (Type L n):** For systems not supporting 8-bit bytes

**Common Mistake:** Transferring binary files in ASCII mode corrupts them due to line-ending conversion. Always verify transfer mode when transferring non-text files.

---

### Concept 3 — SMTPS

**What Is It?**
SMTPS (Simple Mail Transfer Protocol Secure) is SMTP extended with TLS encryption. It secures the email transmission channel between mail client and server.

**SMTP vs POP3 vs IMAP:**

| Protocol | Direction | Purpose |
|---------|----------|---------|
| **SMTP** | Client → Server (push) | Send email from client to server; relay between servers |
| **POP3** | Server → Client (pull) | Download email from server; typically deletes from server |
| **IMAP** | Server ↔ Client (sync) | Synchronise email across multiple devices; email stays on server |

**SMTP Models:**
- **End-to-end:** Sender's SMTP client connects directly to recipient's SMTP server
- **Store-and-forward:** SMTP server receives and queues messages; forwards when recipient server is available

**SMTP Components:**
- **User Agent (UA):** Email client (Outlook, Thunderbird) — creates and submits email
- **Mail Transfer Agent (MTA):** Relays email between servers (Postfix, Sendmail, Exchange)

**SMTPS and STARTTLS:**
SMTPS wraps SMTP inside TLS using the STARTTLS command. The exchange:
```
Client: EHLO mail.example.com
Server: 250-STARTTLS
Client: STARTTLS
Server: 220 Ready to start TLS
[TLS handshake begins]
[Encrypted SMTP session from here]
```

**Ports:**
- SMTP (cleartext): TCP 25
- SMTPS (submission with TLS): TCP 587 (STARTTLS) or 465 (implicit TLS)

**Why SMTPS Matters:**
- Prevents credential sniffing during SMTP authentication
- Prevents email content interception between client and server
- Limits spam from compromised/vulnerable domains
- Protects against phishing payload injection in transit

---

### Concept 4 — POP3S

**What Is It?**
POP3S (Post Office Protocol Secure) is POP3 wrapped in TLS. It secures the retrieval of email from a mail server to a client.

**POP3 Workflow:**
```
1. Client connects to mail server (port 110 / 995 for POP3S)
2. Client authenticates (username + password)
3. Client downloads all queued emails
4. By default: server deletes emails after download
5. Emails are saved only on the downloading device
```

**POP3 Limitations:**
- No synchronisation across multiple devices (emails downloaded to one device, gone from server)
- Credentials transmitted in cleartext (POP3 without S)

**POP3S — STARTTLS Exchange:**
```
Client: EHLO
Server: [capabilities including STARTTLS]
Server: EHLO (Extended HELO)
Client: STARTTLS
Server: [TLS handshake]
[Encrypted POP3 session from here]
```

**Ports:**
- POP3: TCP 110
- POP3S (TLS): TCP 995

**Why POP3S vs IMAP:**
POP3S is simpler and appropriate when a single-device email workflow is acceptable. IMAPS (IMAP over TLS, port 993) is preferred for multi-device access since it synchronises the mailbox state with the server.

---

### Concept 5 — DNSSEC

**What Is It?**
DNSSEC (Domain Name System Security Extensions) adds cryptographic signatures to DNS records, allowing resolvers to verify that DNS responses are authentic and unmodified.

**The Problem DNSSEC Solves — DNS Spoofing:**
Standard DNS has no authentication. A resolver sends a query; any machine that responds first can provide a forged answer. An on-path attacker can respond with a spoofed DNS reply pointing to a malicious server before the legitimate DNS server responds. The resolver accepts it — DNS cache poisoning.

```
Without DNSSEC:
  Client → DNS query for bank.com
  Attacker intercepts → responds with fake IP
  Client connects to attacker's server instead of bank.com

With DNSSEC:
  Client → DNS query for bank.com
  Attacker intercepts → provides fake response
  But: the fake response lacks a valid RRSIG (signature)
  Resolver verifies signature → invalid → discards fake response
  Client waits for authentic signed response
```

**How DNSSEC Works:**
1. The DNS zone owner generates a key pair: **Zone Signing Key (ZSK)** and **Key Signing Key (KSK)**
2. Each DNS record set (RRset) is digitally signed with the ZSK → creates an **RRSIG** (Resource Record Signature) record
3. The zone publishes its **DNSKEY** records (public keys)
4. A **DS (Delegation Signer)** record in the parent zone validates the child zone's KSK
5. Resolvers follow the **chain of trust** from the root zone down to validate signatures

**Chain of Trust:**
```
. (root) → signed with root KSK (trusted by resolvers)
  ↓
com. → DS record validated by root
  ↓
example.com → DS record validated by com.
  ↓
www.example.com A record → RRSIG validated by example.com ZSK
```

**What DNSSEC Provides:**
- **Authenticity:** Response is from the zone owner (signed with their private key)
- **Integrity:** Response has not been modified in transit (signature covers the record content)

**What DNSSEC Does Not Provide:**
- **Confidentiality:** DNS queries and responses are still in cleartext — visible to on-path observers
- **DoS protection:** A zone with DNSSEC is not harder to flood than one without

**DNS over HTTPS (DoH) and DNS over TLS (DoT)** provide confidentiality — they encrypt the DNS query itself. DNSSEC and DoH/DoT are complementary, not competing.

---

### Concept 6 — OpenPGP and Email Encryption

**The Email Trust Problem:**
Even with SMTPS/IMAPS securing the client-to-server hop, email content passes through multiple mail servers in transit, often in cleartext between them. The receiving mail server can read message content. With OpenPGP, the message itself is encrypted — end-to-end — so only the intended recipient can read it.

**PGP (Pretty Good Privacy):**
Created by Phil Zimmermann. OpenPGP is the open standard (RFC 4880). GnuPG (GPG) is the free, open-source implementation.

**OpenPGP Key Pair Usage:**

| Operation | Key Used | Security Property |
|-----------|---------|------------------|
| Encrypt for recipient | Recipient's **public key** | Confidentiality — only recipient's private key decrypts |
| Sign message | Sender's **private key** | Authenticity + Non-repudiation — verified with sender's public key |
| Verify signature | Sender's **public key** | Confirms message came from sender, unmodified |
| Decrypt message | Recipient's **private key** | Recovers plaintext |

**OpenPGP Commands:**
```bash
# Generate key pair
gpg --gen-key

# Encrypt and sign a message
gpg --encrypt --sign --armor -r recipient@example.com message.txt
# --encrypt -r: encrypt with recipient's public key
# --sign: sign with sender's private key
# --armor: ASCII output (instead of binary)

# Decrypt a message
gpg --decrypt message.txt.asc
```

**Encrypted Email Format:**
```
-----BEGIN PGP MESSAGE-----
[base64-encoded encrypted content]
-----END PGP MESSAGE-----
```

**Limitation:**
OpenPGP encrypts the message body but not the email headers (From, To, Subject, timestamps). Metadata about who is communicating is still visible. Subject lines often contain sensitive information that remains unprotected.

---

### Concept 7 — SSH (Secure Shell)

**The Problem SSH Solves:**
Telnet and rlogin transmitted all data — including login credentials — in cleartext. Anyone on the network path could capture credentials with a packet sniffer.

```
Telnet credential capture example (Wireshark "Follow TCP Stream"):
  Red text (client sends): admin [Enter]
  Blue text (server sends): Password:
  Red text (client sends): MyPassword123 [Enter]
  → Complete credential captured
```

SSH encrypted the same exchange so captured packets reveal nothing meaningful.

**What SSH Provides:**
- **Confidentiality:** All data encrypted (AES, ChaCha20)
- **Integrity:** HMAC ensures packets are not modified in transit
- **Authentication:** Server identity verified via host key; client authenticated via password or public key

**SSH Uses:**
- Remote shell access (primary use)
- SCP/SFTP — secure file transfer
- SSH tunnelling / port forwarding — tunnel other protocols through encrypted SSH channel
- Git over SSH

**SSH Host Key Verification:**
On first connection to a server, SSH displays the server's fingerprint and asks you to verify it. Accepting an unverified fingerprint is a TOFU (Trust On First Use) model. An attacker performing MITM would present a different fingerprint — this is why verifying the fingerprint against an out-of-band source is important in high-security environments.

```bash
# Generate SSH key pair
ssh-keygen -t ed25519  # Ed25519 preferred (smaller, faster, equally secure)
ssh-keygen -t rsa -b 4096  # RSA with 4096-bit key

# Copy public key to server
ssh-copy-id user@server

# SSH tunnelling (forward local port 8080 to remote port 80)
ssh -L 8080:localhost:80 user@server
```

**SSH vs Telnet:**

| Characteristic | Telnet | SSH |
|---------------|--------|-----|
| Encryption | None | AES/ChaCha20 |
| Authentication | Password (cleartext) | Password (encrypted) + Public key |
| Integrity | None | HMAC |
| Port | 23 | 22 |
| Status | Obsolete — disable everywhere | Current standard |

---

### Concept 8 — SSL/TLS Deep Dive

**What Is It?**
TLS (Transport Layer Security) is the cryptographic protocol providing confidentiality, integrity, and authentication for networked communications. SSL (Secure Sockets Layer) is its deprecated predecessor. Modern systems use TLS 1.2 or TLS 1.3.

**TLS Handshake — Detailed:**

```
Client                                        Server
  |                                              |
  |--- ClientHello --------------------------->  |
  |    TLS version: 1.3                          |
  |    Cipher suites: [AES-256-GCM-SHA384, ...]  |
  |    Client random: [32 bytes]                 |
  |    Supported groups: x25519, P-256           |
  |                                              |
  |<-- ServerHello ---------------------------   |
  |    Chosen cipher: AES-256-GCM-SHA384         |
  |    Server random: [32 bytes]                 |
  |    Key share: [server's ECDH public value]   |
  |                                              |
  |<-- Certificate --------------------------    |
  |    Server's X.509 certificate chain          |
  |                                              |
  |<-- CertificateVerify --------------------    |
  |    Signature over handshake transcript       |
  |                                              |
  |<-- Finished ----------------------------     |
  |    Encrypted with handshake keys            |
  |                                              |
  |    [Client verifies certificate chain        |
  |     against trusted root CAs]               |
  |                                              |
  |--- Finished --------------------------->     |
  |    Encrypted with handshake keys            |
  |                                              |
  |========= Encrypted application data ======= |
  |    Using AES-256-GCM session keys           |
```

**TLS Versions:**

| Version | Status | Notes |
|---------|--------|-------|
| SSL 2.0/3.0 | Broken — disable | POODLE, DROWN vulnerabilities |
| TLS 1.0/1.1 | Deprecated — disable | Known weaknesses; removed from most browsers 2020 |
| TLS 1.2 | Acceptable — current minimum | Still widely used; ensure strong cipher suites |
| TLS 1.3 | Preferred | Faster handshake; removed weak cipher suites; mandatory PFS |

**TLS 1.3 Improvements Over TLS 1.2:**
- All cipher suites provide PFS (ephemeral key exchange)
- Removed: RSA key exchange, RC4, DES, 3DES, MD5, SHA-1
- 1-RTT handshake (vs 2-RTT in TLS 1.2)
- 0-RTT session resumption (with security trade-offs)

**Protocol-Port Mappings:**

| Protocol | TLS Port |
|---------|---------|
| HTTPS | 443 |
| FTPS (implicit) | 990 |
| SMTPS | 465/587 |
| POP3S | 995 |
| IMAPS | 993 |
| LDAPS | 636 |

**Common TLS Misconfiguration:**
Cipher suite ordering matters — the server should prefer its own cipher suite order, not the client's. Use tools like `testssl.sh` or `ssllabs.com` to audit TLS configuration.

---

### Concept 9 — SOCKS5

**What Is It?**
SOCKS5 (Socket Secure version 5) is a proxy protocol that routes application-layer traffic through a proxy server without the application needing to know the proxy's implementation. Unlike HTTP proxies that are protocol-specific, SOCKS5 works at a lower level — it can proxy any TCP/UDP traffic.

**Why It Exists:**
Firewalls sometimes block direct connections between networks. SOCKS5 allows a client behind a firewall to establish connections through a proxy server on the other side, bypassing IP-based restrictions.

**SOCKS5 Handshake:**
```
1. Client → Proxy: version byte (0x05), number of auth methods, auth methods
2. Proxy → Client: version (0x05), chosen auth method
3. Client → Proxy: connection request (destination host, destination port)
4. Proxy establishes connection to destination
5. Data flows: Client ↔ Proxy ↔ Destination
```

**Security Uses:**
- **Penetration testing:** SSH SOCKS5 proxy tunnels traffic through an SSH connection into a target network (`ssh -D 1080 user@server` creates a SOCKS5 proxy)
- **Anonymisation:** Route traffic through a SOCKS5 proxy to hide source IP (Tor uses SOCKS5)
- **Firewall bypass:** Applications that support SOCKS5 can reach blocked destinations via an authorised proxy

**Security Concerns:**
- SOCKS5 itself does not encrypt traffic — it is a routing mechanism, not an encryption mechanism
- Attackers use SOCKS5 proxies to tunnel C2 traffic through corporate networks
- Monitor for unexpected SOCKS proxy usage via SIEM (unusual port 1080 connections, Tor traffic)

---

### Concept 10 — IPsec

**What Is It?**
IPsec (Internet Protocol Security) is a suite of protocols that authenticate and encrypt IP packets at the network layer. Unlike TLS which operates at the transport/application layer, IPsec operates at layer 3 — transparent to applications.

**IPsec Protocols:**

| Protocol | Provides | Mode |
|---------|---------|------|
| **AH (Authentication Header)** | Authentication + Integrity (no confidentiality) | Transport or Tunnel |
| **ESP (Encapsulating Security Payload)** | Authentication + Integrity + Confidentiality | Transport or Tunnel |
| **SA (Security Association) / IKE** | Key exchange and algorithm negotiation | — |

**AH is mandatory in IPsec v2; optional in IPsec v3.** Most deployments use ESP alone since it provides all three security properties.

**IPsec Modes:**

| Mode | What Is Protected | Use Case |
|------|-----------------|---------|
| **Transport** | TCP/UDP header and payload only (original IP header unprotected) | Host-to-host within a LAN |
| **Tunnel** | Entire original IP packet (new IP header added) | VPN — source and destination IPs hidden |

**IPsec in VPNs:**
```
Site A (192.168.1.0/24) ─── IPsec Tunnel ─── Site B (192.168.2.0/24)
              ↑                                        ↑
         VPN Concentrator                        VPN Concentrator

Traffic between Site A and Site B travels through encrypted IPsec tunnel
Source/destination LAN IPs are hidden inside the tunnel (tunnel mode)
```

**VPN Protocol Comparison:**

| Protocol | Encryption Layer | Common Use |
|---------|----------------|-----------|
| **IPsec (ESP tunnel)** | Network layer (L3) | Site-to-site VPN, remote access (Cisco) |
| **SSL/TLS (OpenVPN)** | Application layer | Remote access VPN; works through firewalls |
| **PPTP** | Weak MPPE encryption | Obsolete — do not use |
| **WireGuard** | ChaCha20/Poly1305 | Modern, fast, lightweight VPN |

---

## Architecture and Relationships

### Protocol Security Stack

```
Application Layer:
  HTTPS = HTTP + TLS
  FTPS  = FTP + TLS
  SMTPS = SMTP + TLS (STARTTLS)
  POP3S = POP3 + TLS
  IMAPS = IMAP + TLS
  LDAPS = LDAP + TLS

Session/Transport Layer:
  TLS (TLS 1.2 / TLS 1.3)
  SSH (own protocol — handles both transport and application)

Network Layer:
  IPsec (operates on IP packets directly)

DNS Security:
  DNSSEC (signs DNS record sets — not protocol encryption)
  DoH/DoT (encrypts DNS queries — not record signing)

Email End-to-End:
  OpenPGP/GnuPG (application-level encryption — independent of transport)
```

### TLS Certificate Validation Chain

```
Root CA (self-signed, in OS/browser trust store)
     ↓ signs
Intermediate CA
     ↓ signs
End-Entity Certificate (e.g., bank.com)
     |
     ├── Subject: bank.com
     ├── Public Key: [server's public key]
     ├── Valid: 2024-01-01 to 2025-01-01
     └── Issuer: Intermediate CA

Validation:
1. Is the certificate valid (not expired, not revoked)?
2. Is the issuer trusted (in chain to trusted root)?
3. Does the certificate name match the domain (CN or SAN)?
4. All three → connection proceeds; any fail → warning
```

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **AH** | Authentication Header — IPsec protocol providing authentication and integrity |
| **DNSSEC** | DNS Security Extensions — adds cryptographic signatures to DNS records |
| **DoH** | DNS over HTTPS — encrypts DNS queries using HTTPS |
| **DoT** | DNS over TLS — encrypts DNS queries using TLS |
| **ESP** | Encapsulating Security Payload — IPsec protocol providing auth, integrity, and confidentiality |
| **FTPS** | FTP Secure — FTP extended with TLS/SSL |
| **GPG / GnuPG** | GNU Privacy Guard — open-source OpenPGP implementation |
| **HSTS** | HTTP Strict Transport Security — forces HTTPS-only connections |
| **IKE** | Internet Key Exchange — IPsec key negotiation protocol |
| **IPsec** | Internet Protocol Security — network-layer authentication and encryption suite |
| **OpenPGP** | Open standard for end-to-end email encryption and signing |
| **PFS** | Perfect Forward Secrecy — session keys not derived from long-term keys |
| **POP3S** | POP3 Secure — POP3 wrapped in TLS (port 995) |
| **RRSIG** | Resource Record Signature — DNSSEC signature for a DNS record set |
| **SA** | Security Association — IPsec policy defining algorithms and keys for a connection |
| **SMTPS** | SMTP Secure — SMTP with TLS (ports 465/587) |
| **SNI** | Server Name Indication — TLS extension indicating requested hostname (visible in plaintext) |
| **SOCKS5** | Socket Secure version 5 — application-agnostic proxy protocol |
| **SPN** | Service Principal Name — identifier for Kerberos service (related to DNSSEC zone signing) |
| **STARTTLS** | Command to upgrade plaintext connection to TLS |
| **TLS** | Transport Layer Security — cryptographic protocol for secure communications |
| **ZSK / KSK** | Zone Signing Key / Key Signing Key — DNSSEC key pairs for signing DNS records |

---

## Exam and Interview Revision

### Must Remember

- HTTPS = HTTP over TLS; port 443; uses asymmetric for key exchange, symmetric for session
- FTPS: explicit (port 21, STARTTLS command) vs implicit (port 990, TLS from first byte)
- FTP active mode: server connects back to client (fails behind firewall); passive mode: client connects to server
- SMTPS: STARTTLS on port 587; implicit TLS on port 465
- POP3S: TLS on port 995; IMAPS: TLS on port 993
- DNSSEC provides authenticity and integrity for DNS; does NOT encrypt DNS queries
- DNS cache poisoning: mitigated by DNSSEC; query confidentiality requires DoH or DoT
- OpenPGP: encrypt with recipient's public key; sign with sender's private key
- SSH: replaces Telnet; encrypts all traffic including credentials; uses public key auth
- TLS 1.3 advantages: all cipher suites provide PFS; removed weak algorithms; faster handshake
- IPsec AH: auth + integrity (no confidentiality); ESP: auth + integrity + confidentiality
- IPsec transport mode: protects payload only; tunnel mode: wraps entire IP packet (VPN use)
- SOCKS5: layer-3 proxy protocol — routes any TCP/UDP traffic; does not encrypt by itself
- TLS 1.0/1.1 and SSL are deprecated and must be disabled

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is the TLS handshake? | ClientHello → ServerHello + Certificate → Key exchange → Both derive session keys → Finished messages → Encrypted session begins |
| What is the difference between FTPS explicit and implicit? | Explicit: connect to port 21, then issue AUTH TLS to upgrade. Implicit: TLS mandatory from first byte, port 990. |
| What does DNSSEC protect against? | DNS cache poisoning / DNS spoofing. Signs DNS records so resolvers can verify authenticity. Does not encrypt queries. |
| What is the difference between OpenPGP and S/MIME? | Both provide email encryption. OpenPGP uses a web of trust model (user-signed keys). S/MIME uses X.509 certificates from CAs (same as TLS). S/MIME is common in enterprise; OpenPGP in technical/open-source communities. |
| What is the difference between IPsec AH and ESP? | AH: authentication and integrity only — no confidentiality. ESP: authentication, integrity, AND confidentiality. In practice, ESP is used because it provides all three properties. |
| Why is TLS 1.3 preferred over 1.2? | All cipher suites provide PFS; removed weak algorithms (RSA key exchange, RC4, DES, 3DES); 1-RTT handshake is faster; cleaner security model. |
