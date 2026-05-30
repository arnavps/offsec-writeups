# Networking Secure Protocols

## Overview

Plaintext protocols transmit data in the clear — anyone with access to the network can read or modify it. TLS, SSH, and VPNs are the three primary approaches to securing network communication. This room covers how TLS secures existing protocols, how SSH replaced TELNET, and how VPNs create private tunnels over public infrastructure.

---

## Topics Covered

- TLS (Transport Layer Security)
- HTTPS, SMTPS, POP3S, IMAPS
- SSH — secure remote access, SFTP, tunnelling
- FTPS vs SFTP
- VPNs

---

## Key Concepts

### TLS (Transport Layer Security)

TLS is a cryptographic protocol that operates at the transport layer. It provides:

- **Confidentiality** — encrypts data so it cannot be read in transit
- **Integrity** — ensures data has not been modified in transit
- **Authentication** — verifies the identity of the server (and optionally the client)

TLS is added to existing plaintext protocols by wrapping them in an encrypted session. The result is typically indicated by appending "S" (Secure) to the protocol name.

**Protocols secured with TLS:**

| Plaintext | Secure | Default Port |
|---|---|---|
| HTTP | HTTPS | 443 |
| SMTP | SMTPS | 465 / 587 |
| POP3 | POP3S | 995 |
| IMAP | IMAPS | 993 |

**HTTPS connection flow:**

```
1. TCP three-way handshake with server
2. TLS handshake — negotiate cipher, exchange certificates, establish session key
3. HTTP communication over the encrypted TLS session
```

---

### SSH (Secure Shell)

SSH replaced TELNET as the standard for remote terminal access. It provides end-to-end encryption, integrity protection, and strong authentication.

**Key SSH features:**

- **Secure authentication** — supports password, public key, and two-factor authentication
- **Confidentiality** — all traffic is encrypted; protects against eavesdropping
- **Integrity** — cryptographic protection against data modification
- **Tunnelling** — can route other protocols through an SSH tunnel (VPN-like)
- **X11 Forwarding** — run graphical applications over SSH

```bash
ssh username@hostname
ssh -i privatekey.pem username@hostname    # Key-based authentication
```

**SFTP (SSH File Transfer Protocol)**
Secure file transfer built into SSH. Uses port 22.

```bash
sftp username@hostname
get filename      # Download
put filename      # Upload
```

**FTPS (FTP Secure)**
FTP secured with TLS. Uses port 990. Requires certificate setup and can be difficult with strict firewalls due to separate control and data connections. Not to be confused with SFTP.

---

### VPN (Virtual Private Network)

A VPN creates an encrypted tunnel between two endpoints over an untrusted network (the internet), allowing devices to communicate as if they were on the same private network.

**Why VPNs exist:**
The TCP/IP protocol suite was designed for packet delivery, not privacy. VPNs add the "private" layer that the original design lacked.

**Common use cases:**
- Connecting remote offices to a central network
- Securing remote worker access to corporate resources
- Anonymising internet traffic

**Requirements:** Internet connectivity + VPN server + VPN client

---

### Three Approaches to Securing Network Traffic

| Approach | How It Works | Best For |
|---|---|---|
| TLS | Wraps existing protocols in an encrypted session | Securing HTTP, SMTP, POP3, IMAP, and many others |
| SSH | Encrypted remote access with tunnelling capability | Remote administration, secure file transfer, tunnelling plaintext protocols |
| VPN | Encrypted tunnel connecting two networks or endpoints | Connecting branch offices, securing remote access |

---

## Important Terminology

| Term | Definition |
|---|---|
| TLS | Transport Layer Security — cryptographic protocol for securing network communication |
| SSL | Secure Sockets Layer — TLS's predecessor; deprecated |
| HTTPS | HTTP over TLS |
| SMTPS | SMTP over TLS |
| POP3S | POP3 over TLS |
| IMAPS | IMAP over TLS |
| SSH | Secure Shell — encrypted remote terminal protocol (port 22) |
| SFTP | SSH File Transfer Protocol — secure file transfer over SSH |
| FTPS | FTP Secure — FTP over TLS (port 990) |
| VPN | Virtual Private Network — encrypted tunnel over public infrastructure |
| Certificate | A digital document binding a public key to an identity, signed by a CA |
| CA | Certificate Authority — trusted entity that signs certificates |

---

## Real-World Relevance

- HTTPS is the baseline security requirement for any web application — HTTP transmits credentials and session tokens in plaintext
- TLS misconfiguration (expired certificates, weak cipher suites, outdated TLS versions) is a common finding in security assessments
- SSH key-based authentication is significantly more secure than password authentication — brute-force attacks against SSH passwords are extremely common
- SSH tunnelling is used by both administrators (to secure legacy protocols) and attackers (to exfiltrate data or bypass firewalls)
- VPN split tunnelling misconfigurations can expose corporate traffic
- FTPS vs SFTP confusion is common — SFTP is generally preferred as it uses a single port and is simpler to firewall

---

## Key Learnings

- TLS secures existing protocols by wrapping them in an encrypted session — HTTP becomes HTTPS, SMTP becomes SMTPS, etc.
- Secure protocol port numbers differ from their plaintext counterparts
- SSH replaced TELNET — it provides encryption, integrity, and strong authentication on port 22
- SFTP is SSH-based file transfer; FTPS is FTP over TLS — they are different protocols
- VPNs create encrypted tunnels over public infrastructure for private communication

---

## Conclusion

TLS, SSH, and VPNs are the three pillars of network security. Together they address the fundamental weakness of the original TCP/IP design — the absence of built-in privacy and integrity. Understanding how each works, where it applies, and where it can fail is essential for both securing systems and identifying weaknesses during security assessments.
