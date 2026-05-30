# Networking Concepts

## Overview

This room covers the foundational networking models and protocols that underpin all network communication. Understanding the OSI and TCP/IP models, how TCP and UDP work, and how data is encapsulated as it travels through the stack is essential for understanding every network-based attack and defence technique.

---

## Topics Covered

- OSI model layers and their functions
- TCP/IP model and its mapping to OSI
- IP addressing: public vs private, network configuration commands
- Routing
- UDP and TCP — differences, use cases, three-way handshake
- Encapsulation
- The life of a packet
- TELNET for manual protocol interaction

---

## Key Concepts

### OSI Model

| Layer | Name | Function | Example Protocols |
|---|---|---|---|
| 7 | Application | Services and interfaces for applications | HTTP, FTP, DNS, SMTP, IMAP |
| 6 | Presentation | Data encoding, encryption, compression | Unicode, MIME, JPEG, MPEG |
| 5 | Session | Establish, maintain, synchronise sessions | NFS, RPC |
| 4 | Transport | End-to-end communication, segmentation | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP, IPSec |
| 2 | Data Link | Reliable transfer between adjacent nodes | Ethernet (802.3), WiFi (802.11) |
| 1 | Physical | Physical transmission media | Electrical, optical, wireless signals |

### TCP/IP Model Mapping

| OSI Layer | TCP/IP Layer | Protocols |
|---|---|---|
| 7, 6, 5 | Application | HTTP, HTTPS, FTP, SSH, DNS, SMTP, IMAP |
| 4 | Transport | TCP, UDP |
| 3 | Internet | IP, ICMP, IPSec |
| 2, 1 | Link | Ethernet 802.3, WiFi 802.11 |

---

### IP Addressing

**Checking network configuration:**

```bash
# Linux
ifconfig
ip address show    # or: ip a s

# Windows
ipconfig
```

**Private IP ranges (RFC 1918):**

| Range | CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | 10/8 |
| 172.16.0.0 – 172.31.255.255 | 172.16/12 |
| 192.168.0.0 – 192.168.255.255 | 192.168/16 |

Private IPs are used within local networks. Public IPs are used on the internet.

---

### Routing

A router forwards packets toward their destination by inspecting the IP address and choosing the best next hop. Routers operate at Layer 3. A packet typically passes through multiple routers before reaching its destination.

---

### UDP (User Datagram Protocol)

- Connectionless — no handshake required
- No delivery confirmation
- Fast — minimal overhead
- Layer 4
- Uses port numbers (1–65535) to identify processes
- Suitable for: DNS, DHCP, VoIP, live streaming

---

### TCP (Transmission Control Protocol)

- Connection-oriented — requires a three-way handshake before data transfer
- Guarantees delivery and ordering via sequence and acknowledgement numbers
- Layer 4
- Suitable for: HTTP, HTTPS, SSH, FTP, email

**Three-Way Handshake:**

```
1. SYN      Client → Server   Client sends initial sequence number (ISN)
2. SYN-ACK  Server → Client   Server acknowledges and sends its own ISN
3. ACK      Client → Server   Client acknowledges server's ISN
             (Connection established — data transfer begins)
```

---

### Encapsulation

As data moves down the OSI stack, each layer adds a header (and sometimes a trailer):

```
Application data
    ↓ Transport layer adds TCP/UDP header → Segment / Datagram
    ↓ Network layer adds IP header → Packet
    ↓ Data Link layer adds Ethernet/WiFi header + trailer → Frame
    ↓ Physical layer transmits as bits
```

On the receiving end, each layer strips its header and passes the data up.

---

### The Life of a Packet

When you search on TryHackMe:

1. Browser prepares an HTTPS/HTTP request
2. TCP establishes a connection via three-way handshake
3. HTTP request is sent as TCP segments
4. IP layer adds source and destination IP addresses
5. Link layer adds MAC addresses and sends to the router
6. Each router strips the frame, inspects the IP destination, and forwards
7. Process reverses at the destination — data travels back up the stack

---

### TELNET

TELNET is a plaintext remote terminal protocol (TCP port 23). While insecure for remote administration (replaced by SSH), it is useful for manually interacting with any TCP service to understand protocol behaviour.

```bash
telnet <IP> <port>
```

Example — manually send an HTTP request:

```bash
telnet 10.10.10.10 80
GET /index.html HTTP/1.1
Host: 10.10.10.10
```

---

## Important Terminology

| Term | Definition |
|---|---|
| OSI Model | 7-layer framework for network communication |
| TCP/IP Model | 4-layer practical implementation of networking |
| Encapsulation | Adding headers at each layer as data moves down the stack |
| Segment | TCP data unit (Layer 4) |
| Datagram | UDP data unit (Layer 4) |
| Packet | IP data unit (Layer 3) |
| Frame | Data Link data unit (Layer 2) |
| Port | Numerical identifier for a process on a host (0–65535) |
| Three-Way Handshake | TCP connection establishment: SYN → SYN-ACK → ACK |
| TTL | Time to Live — limits how many routers a packet can traverse |
| TELNET | Plaintext remote terminal protocol (TCP port 23) |

---

## Real-World Relevance

- The OSI model is the reference framework for describing where attacks and defences operate — Layer 3 for IP spoofing, Layer 4 for SYN floods, Layer 7 for application attacks
- TCP's three-way handshake is the basis for SYN flood DoS attacks — sending SYN packets without completing the handshake exhausts server resources
- Understanding encapsulation is essential for reading packet captures in Wireshark or tcpdump
- TELNET is used in CTFs and labs to manually interact with services and understand protocol behaviour
- Port numbers are the foundation of firewall rules, port scanning, and service enumeration

---

## Key Learnings

- The OSI model has 7 layers; TCP/IP condenses these into 4
- TCP is reliable and connection-oriented; UDP is fast and connectionless
- The three-way handshake (SYN → SYN-ACK → ACK) establishes every TCP connection
- Encapsulation adds headers at each layer going down; de-encapsulation strips them going up
- Private IP ranges (RFC 1918) are not routable on the internet
- TELNET allows manual interaction with any TCP service

---

## Conclusion

Networking concepts form the bedrock of cybersecurity. Every attack and defence technique maps to one or more layers of the OSI model. Understanding how TCP and UDP work, how packets are encapsulated, and how data flows from application to wire is essential context for everything that follows.
