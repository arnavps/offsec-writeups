# Wireshark: Packet Operations

## Overview

Building on the basics, this room covers Wireshark's statistics features, advanced display filtering, and packet-level operations. These capabilities allow analysts to quickly understand the scope of a capture, identify key endpoints and conversations, and apply precise filters to isolate events of interest.

---

## Topics Covered

- Statistics menu: Resolved Addresses, Protocol Hierarchy, Conversations, IPv4/IPv6, DNS, HTTP
- Capture filters vs display filters
- Display filter syntax: comparison operators, logical expressions
- Protocol filters: IP, TCP/UDP, HTTP, DNS
- Advanced filters: `contains`, `matches`, `in`, `upper`, `lower`, `string`
- Bookmarks, filter buttons, and profiles

---

## Key Concepts

### Statistics Menu

**Resolved Addresses**
Lists all IP addresses and their resolved hostnames from DNS answers in the capture. Quickly identifies accessed resources.
`Statistics → Resolved Addresses`

**Protocol Hierarchy**
Breaks down all protocols in the capture as a tree with packet counts and percentages. Shows overall traffic composition.
`Statistics → Protocol Hierarchy`

**Conversations**
Lists all conversations between endpoint pairs in five formats: Ethernet, IPv4, IPv6, TCP, UDP.
`Statistics → Conversations`

**IPv4 / IPv6 Statistics**
Narrows statistics to packets containing a specific IP version.
`Statistics → IPvX Statistics`

**DNS Statistics**
Breaks down DNS traffic — rcode, opcode, query types, response stats.
`Statistics → DNS`

**HTTP Statistics**
Breaks down HTTP traffic — request methods, response codes, original requests.
`Statistics → HTTP`

---

### Capture Filters vs Display Filters

| Type | Purpose | When Set | Syntax |
|---|---|---|---|
| Capture Filter | Saves only matching packets — discards the rest | Before capture starts; cannot change during capture | BPF (Berkeley Packet Filter) |
| Display Filter | Shows/hides packets from an existing capture | Anytime during or after capture | Wireshark display filter syntax |

Capture filters use BPF syntax. Display filters use Wireshark's protocol-aware syntax. They are not interchangeable.

**Capture filter examples:**

```
tcp port 80                    # Capture only HTTP traffic
host 192.168.1.1               # Capture traffic to/from this host
tcp port 22                    # Capture SSH traffic
```

---

### Display Filter Syntax

**Comparison operators:**

| English | C-Like | Example |
|---|---|---|
| eq | == | `ip.src == 10.10.10.100` |
| ne | != | `ip.src != 10.10.10.100` |
| gt | > | `ip.ttl > 250` |
| lt | < | `ip.ttl < 10` |
| ge | >= | `ip.ttl >= 0xFA` |
| le | <= | `ip.ttl <= 0xA` |

**Logical operators:**

```
(ip.src == 10.10.10.100) && (tcp.port == 80)     # AND
(ip.src == 10.10.10.100) || (ip.src == 10.10.10.111)  # OR
!(ip.src == 10.10.10.222)                         # NOT
```

Note: Use `!(value)` style rather than `!=value` for consistent results.

---

### Protocol Filters

**IP filters:**

```
ip                             # All IP packets
ip.addr == 10.10.10.111        # Any packet involving this IP
ip.addr == 10.10.10.0/24       # Entire subnet
ip.src == 10.10.10.111         # Packets from this IP
ip.dst == 10.10.10.111         # Packets to this IP
```

**TCP/UDP filters:**

```
tcp.port == 80
tcp.srcport == 1234
tcp.dstport == 80
udp.port == 53
udp.srcport == 1234
udp.dstport == 5353
```

**HTTP filters:**

```
http
http.response.code == 200
http.request.method == "GET"
http.request.method == "POST"
```

**DNS filters:**

```
dns
dns.flags.response == 0        # DNS queries only
dns.flags.response == 1        # DNS responses only
dns.qry.type == 1              # A record queries
```

---

### Advanced Filters

**`contains`** — case-sensitive string search within a field:

```
http.server contains "Apache"
```

**`matches`** — regex pattern match (case-insensitive):

```
http.host matches "\.(php|html)"
```

**`in`** — set membership:

```
tcp.port in {80 443 8080}
```

**`upper`** — convert field to uppercase before comparison:

```
upper(http.server) contains "APACHE"
```

**`lower`** — convert field to lowercase before comparison:

```
lower(http.server) contains "apache"
```

**`string`** — convert non-string field to string:

```
string(frame.number) matches "[13579]$"    # Odd-numbered frames
```

---

### Bookmarks, Filter Buttons, and Profiles

- **Bookmarks** — save frequently used display filters for quick reuse
- **Filter buttons** — one-click application of saved filters
- **Profiles** — save complete Wireshark configurations (colouring rules, filter buttons, column layout) for different investigation types

`Edit → Configuration Profiles` or click the profile name in the status bar.

---

## Practical Examples / Demonstrations

### Task answers from the room

**Q: What is the number of IP packets?**
`Statistics → Protocol Hierarchy` → count IP packets

**Q: What is the number of Ethernet II frames?**
`Statistics → Protocol Hierarchy` → Ethernet II count

**Q: What is the number of HTTP GET requests?**
Filter: `http.request.method == "GET"` → check packet count in status bar

**Q: What is the number of "200 OK" HTTP responses?**
Filter: `http.response.code == 200`

**Q: What is the total number of DNS packets?**
Filter: `dns`

**Q: What is the IP address of the host that sent the most packets?**
`Statistics → Conversations → IPv4` → sort by packets

---

## Real-World Relevance

- Protocol Hierarchy quickly reveals unexpected protocols in a capture — a sign of tunnelling or unusual activity
- Conversations identifies the most active endpoints — useful for spotting C2 communication or data exfiltration
- DNS statistics reveal DNS tunnelling (unusually high query volume or long domain names)
- HTTP statistics expose unusual response codes (large numbers of 404s may indicate scanning; 200s on unexpected paths may indicate successful exploitation)
- Advanced filters like `contains` and `matches` are used to hunt for specific strings (credentials, malware signatures, suspicious user agents) across large captures

---

## Key Learnings

- Statistics menu provides a high-level overview before diving into individual packets
- Capture filters and display filters use different syntax and serve different purposes
- Display filter colour coding: green = valid, red = invalid, yellow = unreliable
- Advanced operators (`contains`, `matches`, `in`) enable precise pattern-based filtering
- Profiles allow switching between pre-configured investigation setups instantly

---

## Conclusion

Wireshark's statistics and advanced filtering capabilities transform raw packet data into actionable intelligence. The ability to quickly understand traffic composition, identify key conversations, and apply precise filters to isolate specific events is what makes Wireshark an essential tool for network forensics, incident response, and protocol analysis.
