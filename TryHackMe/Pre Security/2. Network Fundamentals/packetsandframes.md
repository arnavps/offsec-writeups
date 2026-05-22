# Packets and Frames

## Overview

Packets and frames are the fundamental units of data transmission in networking. They are distinct concepts tied to different layers of the OSI model. This room covers how data is structured for transmission, the TCP and UDP protocols, the TCP three-way handshake, and how ports work.

---

## Topics Covered

- Packets vs Frames
- IP packet headers
- TCP — headers, handshake, and connection teardown
- UDP — headers and characteristics
- TCP vs UDP comparison
- Ports and common port numbers
- Using `netcat` to connect to a host

---

## Key Concepts

### Packets vs Frames

Both are units of data, but they exist at different OSI layers:

| Unit | OSI Layer | Contains |
|---|---|---|
| Frame | Layer 2 (Data Link) | MAC addresses + encapsulated packet |
| Packet | Layer 3 (Network) | IP addresses + payload |

A frame encapsulates a packet. When data moves between networks, the frame is stripped and rebuilt at each hop, but the packet remains intact.

### IP Packet Headers

Key fields in an IP packet:

| Header Field | Description |
|---|---|
| Time to Live (TTL) | Limits how long a packet can exist on the network; decremented at each hop — prevents infinite routing loops |
| Checksum | Integrity check; if data is altered in transit, the checksum won't match and the packet is discarded |
| Source Address | IP address of the sending device |
| Destination Address | IP address of the intended recipient |

---

### TCP (Transmission Control Protocol)

TCP is a connection-oriented protocol that guarantees reliable, ordered data delivery. It operates at Layer 4 of the OSI model.

**TCP vs UDP:**

| | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (handshake required) | Connectionless |
| Reliability | Guarantees delivery and order | No guarantee of delivery |
| Speed | Slower — more overhead | Faster — minimal overhead |
| Use cases | Web browsing, file transfer, email | Streaming, VoIP, DNS, gaming |

**TCP Packet Headers:**

| Header Field | Description |
|---|---|
| Source Port | Randomly chosen port on the sender's side (0–65535) |
| Destination Port | Port of the target service (e.g., 80 for HTTP) |
| Source IP | Sender's IP address |
| Destination IP | Recipient's IP address |
| Sequence Number | Random initial number assigned to the first data segment |
| Acknowledgement Number | Sequence number + 1, confirming receipt of the previous segment |
| Checksum | Mathematical integrity check for the TCP segment |
| Data | The actual payload being transmitted |
| Flag | Controls handshake behaviour (SYN, ACK, FIN, RST, etc.) |

---

### TCP Three-Way Handshake

Before data is exchanged, TCP establishes a connection through a three-step process:

| Step | Message | Direction | Description |
|---|---|---|---|
| 1 | SYN | Client → Server | Client initiates connection, sends its Initial Sequence Number (ISN) |
| 2 | SYN/ACK | Server → Client | Server acknowledges client's ISN and sends its own ISN |
| 3 | ACK | Client → Server | Client acknowledges server's ISN; connection is established |
| 4 | DATA | Both directions | Actual data transmission begins |
| 5 | FIN | Either side | Cleanly closes the connection |
| — | RST | Either side | Abruptly terminates the connection due to an error |

**Sequence number example:**

```
Client ISN: 0
Server ISN: 5000

SYN     → Client sends ISN = 0
SYN/ACK → Server sends ISN = 5000, ACKs client's 0
ACK     → Client ACKs server's 5000, sends data with sequence 1 (0 + 1)
```

Each subsequent segment increments the sequence number by 1, allowing both sides to detect missing or out-of-order data.

**TCP Connection Teardown:**
TCP closes connections cleanly using FIN packets. Since TCP reserves system resources for open connections, closing them promptly is best practice.

---

### UDP (User Datagram Protocol)

UDP is a stateless, connectionless protocol. There is no handshake, no acknowledgement, and no guarantee of delivery. It trades reliability for speed.

**UDP Packet Headers:**

| Header Field | Description |
|---|---|
| Time to Live (TTL) | Expiry timer to prevent packets clogging the network |
| Source Address | Sender's IP address |
| Destination Address | Recipient's IP address |
| Source Port | Randomly chosen port on the sender's side |
| Destination Port | Port of the target service |
| Data | The payload being transmitted |

UDP packets have fewer headers than TCP, contributing to lower overhead and faster transmission.

---

### Ports

Ports are numerical identifiers (0–65535) that direct incoming data to the correct application or service on a device. Think of them as doors — each service listens on a specific door.

**Common ports:**

| Protocol | Port | Description |
|---|---|---|
| FTP | 21 | File Transfer Protocol — file sharing |
| SSH | 22 | Secure Shell — encrypted remote terminal access |
| HTTP | 80 | Web traffic (unencrypted) |
| HTTPS | 443 | Web traffic (encrypted with TLS) |
| SMB | 445 | File and printer sharing on Windows networks |
| RDP | 3389 | Remote Desktop Protocol — graphical remote access |

---

## Practical Examples / Demonstrations

### Connect to a host using netcat

```bash
nc 8.8.8.8 1234
```

This opens a TCP connection to IP `8.8.8.8` on port `1234`. `netcat` is a versatile tool used for port scanning, banner grabbing, file transfer, and creating reverse shells.

---

## Workflow / Process

### TCP Three-Way Handshake Flow

```
Client  --[SYN, ISN=0]-----------> Server
Client  <--[SYN/ACK, ISN=5000]---- Server
Client  --[ACK, SEQ=1]-----------> Server
         (Connection established)
Client  <--[DATA]----------------> Server
Client  --[FIN]-------------------> Server
         (Connection closed)
```

### UDP Transmission Flow

```
Client  --[Data]-->  Server
(No handshake, no acknowledgement, no teardown)
```

---

## Real-World Relevance

- TCP's three-way handshake is the basis for the **SYN flood** DoS attack — an attacker sends many SYN packets without completing the handshake, exhausting server resources
- Port scanning (e.g., with `nmap`) works by probing ports to identify open services and their versions
- UDP's lack of verification makes it useful for **DNS amplification** DDoS attacks
- Understanding sequence numbers is important for **TCP session hijacking**
- Common ports are the first thing checked during reconnaissance — knowing them by memory is essential

---

## Key Learnings

- Packets (Layer 3) and frames (Layer 2) are distinct — frames encapsulate packets
- TCP is reliable and ordered; UDP is fast and stateless
- The TCP three-way handshake (SYN → SYN/ACK → ACK) establishes a connection before data flows
- Sequence numbers ensure data is received in order and nothing is missed
- Ports direct traffic to the correct service on a device
- `netcat` is a fundamental tool for manual TCP/UDP connections

---

## Additional Notes

- Ports 0–1023 are well-known ports reserved for standard services
- Ports 1024–49151 are registered ports for applications
- Ports 49152–65535 are dynamic/ephemeral ports used for source ports in outbound connections
- RST packets during a scan can indicate a closed port; no response often indicates a filtered port (firewall)

---

## Conclusion

Packets and frames are the building blocks of all network communication. Understanding how TCP and UDP structure and transmit data — and how ports direct that data to the right service — is critical for both network analysis and security testing. The TCP handshake in particular is referenced constantly across offensive and defensive security topics.
