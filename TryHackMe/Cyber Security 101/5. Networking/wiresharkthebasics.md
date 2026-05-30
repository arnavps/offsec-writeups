# Wireshark: The Basics

## Overview

Wireshark is the industry-standard open-source network packet analyser. It captures and inspects live network traffic and saved packet capture (PCAP) files, allowing analysts to examine every byte of every packet. This room covers the Wireshark interface, packet dissection, filtering, and key analysis features.

---

## Topics Covered

- Wireshark interface and GUI sections
- Loading and merging PCAP files
- Packet colouring
- Traffic sniffing
- Packet dissection by OSI layer
- Finding, marking, and commenting packets
- Exporting packets and objects
- Display filters — basic and advanced
- Follow Stream
- Expert Info

---

## Key Concepts

### Wireshark Use Cases

- Detecting and troubleshooting network problems (congestion, failures)
- Detecting security anomalies (rogue hosts, suspicious traffic, abnormal port usage)
- Investigating protocol behaviour and learning how protocols work

Wireshark is a packet analyser — it reads packets, it does not modify them and it is not an IDS.

---

### GUI Sections

| Section | Description |
|---|---|
| Toolbar | Menus and shortcuts for sniffing, filtering, sorting, exporting |
| Display Filter Bar | Query input for filtering displayed packets |
| Recent Files | Previously opened capture files |
| Capture Filter & Interfaces | Available network interfaces and capture filter input |
| Status Bar | Tool status, active profile, packet count |

**Three packet panes:**

| Pane | Description |
|---|---|
| Packet List | Summary of each packet — source, destination, protocol, info |
| Packet Details | Protocol breakdown of the selected packet |
| Packet Bytes | Hex and ASCII representation of the selected packet |

---

### Packet Dissection (OSI Layers)

Clicking a packet in the list opens its details broken down by OSI layer:

| Layer | What It Shows |
|---|---|
| Frame (Layer 1) | Physical layer details — frame number, capture time |
| Source MAC (Layer 2) | Source and destination MAC addresses |
| Source IP (Layer 3) | Source and destination IP addresses |
| Protocol (Layer 4) | TCP/UDP details — ports, flags, sequence numbers |
| Protocol Errors | TCP reassembly issues |
| Application Protocol (Layer 5–7) | HTTP, FTP, DNS, SMB details |
| Application Data | Raw application payload |

---

### Packet Colouring

Wireshark colour-codes packets by protocol and condition for quick visual identification. Two types:

- **Temporary** — applies only for the current session (right-click → Conversation Filter)
- **Permanent** — saved to profile (View → Coloring Rules)

---

### Traffic Sniffing

- Blue shark button — start capture
- Red button — stop capture
- Green button — restart capture

---

### Finding Packets

Edit → Find Packet supports four input types:
- Display filter
- Hex
- String
- Regex

Search can be conducted in the Packet List, Packet Details, or Packet Bytes pane.

---

### Follow Stream

Reconstructs the full application-level conversation from individual packets.

Right-click → Follow → TCP/UDP/HTTP Stream

- Server traffic highlighted in blue
- Client traffic highlighted in red
- Reveals plaintext credentials, HTTP requests/responses, and other application data

---

### Export Objects

Wireshark can extract files transferred over the wire:

File → Export Objects → HTTP / SMB / FTP / TFTP / DICOM

---

### Expert Info

Wireshark flags protocol anomalies automatically:

| Severity | Colour | Meaning |
|---|---|---|
| Chat | Blue | Normal workflow information |
| Note | Cyan | Notable events (application error codes) |
| Warn | Yellow | Unusual error codes or problem statements |
| Error | Red | Malformed packets or serious problems |

Access via: Analyse → Expert Information

---

### Display Filters

Display filters narrow the visible packets without discarding captured data.

**Basic filters:**

```
http                          # All HTTP traffic
dns                           # All DNS traffic
arp                           # All ARP traffic
tcp                           # All TCP traffic
udp                           # All UDP traffic
```

**Filter by IP:**

```
ip.addr == 10.10.10.111       # Any packet involving this IP
ip.src == 10.10.10.111        # Packets from this IP
ip.dst == 10.10.10.111        # Packets to this IP
ip.addr == 10.10.10.0/24      # Entire subnet
```

**Filter by port:**

```
tcp.port == 80
tcp.srcport == 1234
tcp.dstport == 443
udp.port == 53
```

**Filter by HTTP:**

```
http.response.code == 200
http.request.method == "GET"
http.request.method == "POST"
```

**Filter by DNS:**

```
dns.flags.response == 0       # DNS queries
dns.flags.response == 1       # DNS responses
dns.qry.type == 1             # A record queries
```

**Comparison operators:**

| English | C-Like | Description |
|---|---|---|
| eq | == | Equal |
| ne | != | Not equal |
| gt | > | Greater than |
| lt | < | Less than |
| ge | >= | Greater than or equal |
| le | <= | Less than or equal |

**Logical operators:**

```
(ip.src == 10.10.10.1) and (tcp.port == 80)
(ip.src == 10.10.10.1) or (ip.src == 10.10.10.2)
!(ip.src == 10.10.10.222)
```

**Advanced filters:**

```
http.server contains "Apache"                    # Contains string
http.host matches "\.(php|html)"                 # Regex match
tcp.port in {80 443 8080}                        # Set membership
upper(http.server) contains "APACHE"             # Case conversion
string(frame.number) matches "[13579]$"          # Convert to string
```

---

## Real-World Relevance

- Wireshark is the primary tool for network forensics and incident response — analysing captured traffic to reconstruct attack timelines
- Follow Stream reveals plaintext credentials transmitted over unencrypted protocols (HTTP, FTP, TELNET, POP3)
- Export Objects extracts malware, documents, and other files transferred over the network
- Display filters are used to isolate specific hosts, protocols, or conversations in large captures
- Expert Info highlights anomalies that may indicate attacks, misconfigurations, or protocol errors

---

## Key Learnings

- Wireshark has three packet panes: List (summary), Details (protocol breakdown), Bytes (hex/ASCII)
- Packet dissection maps directly to OSI layers — each layer's header is visible and clickable
- Follow Stream reconstructs full application conversations — reveals plaintext data
- Display filters use Wireshark's filter syntax; capture filters use BPF syntax — they are different
- Expert Info automatically flags protocol anomalies by severity
- Export Objects extracts files transferred over HTTP, SMB, FTP, and other protocols

---

## Conclusion

Wireshark is an indispensable tool for anyone working in network security. The ability to capture, filter, and dissect packets at every layer of the OSI model — and to reconstruct application-level conversations — makes it essential for both learning how protocols work and investigating security incidents.
