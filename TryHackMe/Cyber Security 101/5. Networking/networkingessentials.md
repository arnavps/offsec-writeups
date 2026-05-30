# Networking Essentials

## Overview

This room covers the essential protocols that make networks function automatically and efficiently — DHCP for IP assignment, ARP for MAC address resolution, ICMP for diagnostics, routing protocols for path selection, and NAT for sharing public IP addresses. These protocols are foundational for understanding network behaviour and are directly relevant to several attack techniques.

---

## Topics Covered

- DHCP — automatic IP configuration
- ARP — MAC address resolution
- ICMP — ping and traceroute
- Routing protocols: OSPF, EIGRP, BGP, RIP
- NAT — Network Address Translation

---

## Key Concepts

### DHCP (Dynamic Host Configuration Protocol)

DHCP automatically assigns IP addresses, subnet masks, default gateways, and DNS servers to devices when they join a network. It operates at the application layer over UDP — server listens on port 67, client sends from port 68.

**DORA process:**

```
1. DHCP Discover   Client → Broadcast    "Is there a DHCP server?"
2. DHCP Offer      Server → Client       "Here's an available IP"
3. DHCP Request    Client → Server       "I'll take that IP"
4. DHCP ACK        Server → Client       "Confirmed — it's yours"
```

---

### ARP (Address Resolution Protocol)

ARP resolves IP addresses to MAC addresses. Before a device can send a frame to another device on the same network, it needs the destination's MAC address. ARP broadcasts a request to the entire network asking "Who has this IP?" and the owner replies with their MAC.

**ARP process:**

```
ARP Request  →  Broadcast (ff:ff:ff:ff:ff:ff)  "Who has 192.168.1.1?"
ARP Reply    ←  Target device                   "I do — my MAC is XX:XX:XX:XX:XX:XX"
```

ARP is not encapsulated in IP — it sits directly in an Ethernet frame. It operates at Layer 2 (MAC addresses) but supports Layer 3 (IP) operations.

**Viewing ARP traffic with tcpdump:**

```bash
tcpdump -r capture.pcap arp
```

---

### ICMP (Internet Control Message Protocol)

ICMP is used for network diagnostics and error reporting. It operates at Layer 3.

**ping** — sends ICMP Echo Requests (Type 8) and listens for Echo Replies (Type 0). Tests connectivity and measures round-trip time.

```bash
ping example.com
ping 8.8.8.8
```

**traceroute / tracert** — discovers the route to a target by exploiting the TTL field. Each router decrements TTL by 1; when TTL reaches 0, the router drops the packet and sends an ICMP Time Exceeded message (Type 11) back to the sender. By sending packets with incrementing TTL values (1, 2, 3...), each router along the path reveals itself.

```bash
traceroute example.com    # Linux
tracert example.com       # Windows
```

Some routers don't respond to TTL-exceeded conditions — they appear as `* * *` in traceroute output.

---

### Routing Protocols

Routing protocols allow routers to share network topology information and calculate optimal paths.

| Protocol | Description |
|---|---|
| OSPF | Open Shortest Path First — routers share link-state information to build a complete network map and calculate shortest paths |
| EIGRP | Enhanced Interior Gateway Routing Protocol — Cisco proprietary; combines multiple routing algorithms; considers bandwidth and delay |
| BGP | Border Gateway Protocol — the internet's core routing protocol; exchanges routing information between autonomous systems (ISPs) |
| RIP | Routing Information Protocol — simple, hop-count based; used in small networks; maximum 15 hops |

---

### NAT (Network Address Translation)

NAT allows multiple devices on a private network to share a single public IP address. The NAT-enabled router maintains a translation table mapping internal private IP:port combinations to the external public IP:port.

**Why NAT exists:** IPv4 address exhaustion — there aren't enough public IPs for every device. NAT allows an entire organisation to use one public IP while internally using private RFC 1918 addresses.

---

## Important Terminology

| Term | Definition |
|---|---|
| DHCP | Dynamic Host Configuration Protocol — automatically assigns IP configuration |
| DORA | Discover, Offer, Request, Acknowledge — the four DHCP steps |
| ARP | Address Resolution Protocol — maps IP addresses to MAC addresses |
| ARP Cache | Local table storing IP-to-MAC mappings |
| ICMP | Internet Control Message Protocol — used for diagnostics and error reporting |
| TTL | Time to Live — decremented by each router; triggers ICMP Time Exceeded when it reaches 0 |
| Traceroute | Tool that uses TTL manipulation to discover the route to a target |
| OSPF | Link-state routing protocol for internal networks |
| BGP | The internet's inter-domain routing protocol |
| NAT | Network Address Translation — maps private IPs to a shared public IP |

---

## Real-World Relevance

- **DHCP starvation** — an attacker floods the network with DHCP Discover messages, exhausting the IP pool and preventing legitimate devices from getting addresses
- **Rogue DHCP server** — an attacker sets up a fake DHCP server to assign themselves as the default gateway, enabling man-in-the-middle attacks
- **ARP spoofing/poisoning** — an attacker sends fake ARP replies to associate their MAC with another device's IP, redirecting traffic through their machine (MITM)
- **ICMP is often blocked** — firewalls frequently block ICMP, so a non-response to ping doesn't confirm a host is down
- **Traceroute reveals network topology** — useful during reconnaissance to map the path to a target and identify intermediate infrastructure
- **NAT complicates attribution** — multiple users behind NAT share one public IP, making it harder to identify individual devices from external logs

---

## Key Learnings

- DHCP automates IP configuration via the DORA process (Discover → Offer → Request → ACK)
- ARP resolves IP addresses to MAC addresses via broadcast — vulnerable to spoofing
- ICMP powers ping (Echo Request/Reply) and traceroute (TTL manipulation)
- Routing protocols (OSPF, BGP, RIP, EIGRP) allow routers to share topology and calculate paths
- NAT allows many private IPs to share one public IP — essential for IPv4 address conservation

---

## Conclusion

DHCP, ARP, ICMP, and NAT are the invisible infrastructure that makes modern networks work. Each protocol has well-known attack vectors — DHCP starvation, ARP poisoning, ICMP-based reconnaissance — making them essential knowledge for both offensive and defensive security work.
