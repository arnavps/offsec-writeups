# OSI Model

## Overview

The OSI (Open Systems Interconnection) model is a conceptual framework that standardises how networked devices send, receive, and interpret data. It breaks network communication into seven distinct layers, each with a specific role. Understanding the OSI model is fundamental to networking and cybersecurity — most protocols, attacks, and defences map directly to one or more of its layers.

---

## Topics Covered

- The seven layers of the OSI model
- The role and responsibilities of each layer
- How data moves through the model (encapsulation)

---

## Key Concepts

### The Seven Layers

Data travels from Layer 7 down to Layer 1 when sending, and from Layer 1 up to Layer 7 when receiving. At each layer, specific information is added to or stripped from the data.

| Layer | Name | Key Responsibility |
|---|---|---|
| 7 | Application | Interface between user-facing software and the network |
| 6 | Presentation | Data formatting, encryption, and standardisation |
| 5 | Session | Establishing, maintaining, and terminating connections |
| 4 | Transport | End-to-end data transmission (TCP/UDP) |
| 3 | Network | Logical addressing and routing (IP) |
| 2 | Data Link | Physical addressing (MAC), framing |
| 1 | Physical | Raw bit transmission over physical hardware |

---

### Layer Breakdown

**Layer 7 — Application**
The layer users interact with directly. Protocols and rules here determine how applications present and handle data. Examples: HTTP, FTP, DNS, SMTP.

**Layer 6 — Presentation**
Responsible for data translation, formatting, and encryption. Ensures data from the application layer is in a format the receiving system can understand. Handles tasks like SSL/TLS encryption and data compression.

**Layer 5 — Session**
Manages the lifecycle of a communication session between two devices — opening, maintaining, and closing connections. Also handles session recovery if a connection drops unexpectedly.

**Layer 4 — Transport**
Controls how data is transmitted across the network. Responsible for segmentation, flow control, and error handling. The two primary protocols at this layer are:
- **TCP** — reliable, connection-oriented
- **UDP** — fast, connectionless

**Layer 3 — Network**
Handles logical addressing (IP addresses) and routing — determining the best path for data to travel between networks. Also responsible for reassembling data packets at the destination.

**Layer 2 — Data Link**
Focuses on physical addressing. Takes packets from Layer 3 and adds the MAC addresses of the source and destination devices, forming a frame. Every network-enabled device has a NIC with a unique MAC address used at this layer.

**Layer 1 — Physical**
The lowest layer — deals with the actual physical transmission of raw binary data (bits) over hardware such as cables, switches, and network cards.

---

## Important Terminology

| Term | Definition |
|---|---|
| OSI Model | A 7-layer framework standardising how network communication works |
| Encapsulation | The process of adding layer-specific headers/trailers as data moves down the OSI stack |
| Protocol | A set of rules governing how data is transmitted and interpreted |
| TCP | Transmission Control Protocol — reliable, connection-oriented transport |
| UDP | User Datagram Protocol — fast, connectionless transport |
| NIC | Network Interface Card — hardware that connects a device to a network |
| Frame | A Layer 2 data unit containing MAC addresses and an encapsulated packet |
| Packet | A Layer 3 data unit containing IP addresses and payload |

---

## Workflow / Process

### Encapsulation (Sending Data)

```
Layer 7 - Application   → Data
Layer 6 - Presentation  → Formatted/Encrypted Data
Layer 5 - Session       → Session info added
Layer 4 - Transport     → Segment (TCP/UDP header added)
Layer 3 - Network       → Packet (IP header added)
Layer 2 - Data Link     → Frame (MAC addresses added)
Layer 1 - Physical      → Bits transmitted over the wire
```

On the receiving end, this process is reversed — each layer strips its header and passes the data up.

---

## Real-World Relevance

- Most network attacks target specific OSI layers:
  - Layer 3: IP spoofing, routing attacks
  - Layer 4: SYN flood (TCP), port scanning
  - Layer 7: SQL injection, XSS, application-layer DDoS
- Firewalls, IDS/IPS, and other security tools are often described by which OSI layer they operate at
- Understanding encapsulation helps when analysing packet captures (e.g., in Wireshark)
- Penetration testers use OSI layer knowledge to understand where a vulnerability exists and how to exploit it

---

## Key Learnings

- The OSI model has 7 layers, each with a distinct role in network communication
- Data is encapsulated as it moves down the stack and de-encapsulated as it moves up
- Layer 4 (Transport) is where TCP and UDP operate
- Layer 3 (Network) handles IP addressing and routing
- Layer 2 (Data Link) handles MAC addressing and framing
- Most security tools and attacks can be mapped to a specific OSI layer

---

## Additional Notes

- The OSI model is a conceptual model — real-world implementations often use the TCP/IP model, which condenses the 7 layers into 4
- Memorisation tip (top to bottom): **All People Seem To Need Data Processing** (Application, Presentation, Session, Transport, Network, Data Link, Physical)

---

## Conclusion

The OSI model is one of the most referenced frameworks in networking and cybersecurity. It provides a common language for describing where in the communication stack something happens — whether that's a protocol, an attack, or a security control. Knowing it well makes everything else in networking significantly easier to understand.
