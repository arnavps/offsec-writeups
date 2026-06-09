# Firewalls Fundamentals

## Overview

A firewall is the primary network security control that inspects and filters traffic entering and leaving a network or device. It enforces a defined ruleset — allowing legitimate traffic and blocking what is not permitted. This room covers the four main types of firewalls (stateless, stateful, proxy, NGFW), how firewall rules are structured, the three rule actions, traffic directionality, Windows Defender Firewall, and the Linux Netfilter framework with its associated utilities.

---

## Topics Covered

- What a firewall is and why it matters
- Firewall types: stateless, stateful, proxy, NGFW
- Firewall rule components and actions
- Inbound, outbound, and forward rule directionality
- Windows Defender Firewall and network profiles
- Linux Netfilter framework: iptables, nftables, firewalld, ufw

---

## Key Concepts

### What is a Firewall?

A firewall inspects all incoming and outgoing network traffic on a device or network boundary. Every packet that attempts to enter or leave faces the firewall first. Based on a configured ruleset, the firewall decides whether to allow, deny, or forward that traffic.

Modern firewalls extend well beyond basic rule filtering — they offer deep packet inspection, intrusion prevention, SSL/TLS decryption, and threat intelligence integration depending on the type.

---

### Firewall Types

Different firewall types operate at different layers of the OSI model and offer different levels of inspection.

#### Stateless Firewall

- Operates at **OSI Layers 3 and 4** (Network and Transport)
- Filters traffic based solely on predetermined rules — source IP, destination IP, port, protocol
- Does **not** track connection state — every packet is evaluated independently against the ruleset
- No memory of previous connections: a packet from a source that previously violated rules is still re-evaluated from scratch
- **Advantage:** Fast processing, suitable for high-speed networks
- **Limitation:** Cannot enforce policies based on connection history; vulnerable to attacks that use fragmented or out-of-sequence packets

#### Stateful Firewall

- Operates at **OSI Layers 3 and 4**
- Filters by rules **and** maintains a state table tracking active connections
- Packets belonging to an established, permitted connection are automatically allowed without re-inspection
- Packets from a source that previously violated rules are blocked for all subsequent packets from that source
- **Advantage:** More accurate filtering; can apply context-aware policies
- **Limitation:** More resource-intensive than stateless due to state table maintenance

#### Proxy Firewall (Application-Level Gateway)

- Operates at **OSI Layer 7** (Application)
- Acts as an intermediary between the internal network and the internet — traffic does not flow directly between client and server
- Inspects the **content** of packets, not just headers
- Masks internal IP addresses by substituting its own IP on outbound requests (provides anonymity for internal hosts)
- Supports content filtering: allow/deny traffic based on what is inside it
- **Advantage:** Deep inspection; supports content-based policy enforcement
- **Limitation:** Higher latency due to full content inspection; can be a bottleneck

#### Next-Generation Firewall (NGFW)

- Operates from **OSI Layer 3 through Layer 7**
- Combines all capabilities of stateful and proxy firewalls and adds:
  - **Deep packet inspection (DPI)** — full content analysis of packet payloads
  - **Intrusion Prevention System (IPS)** — actively blocks malicious traffic in real time
  - **Heuristic analysis** — detects attack patterns and blocks them proactively
  - **SSL/TLS decryption** — decrypts encrypted traffic for inspection, then re-encrypts it
  - **Threat intelligence integration** — correlates traffic against known bad IPs, domains, and hashes
- **Advantage:** Comprehensive protection against both known and novel threats
- **Limitation:** Most expensive and resource-intensive option

#### Comparison Summary

| Type | OSI Layer | State Tracking | Content Inspection | IPS | SSL Decrypt |
|------|-----------|---------------|-------------------|-----|------------|
| Stateless | 3-4 | No | No | No | No |
| Stateful | 3-4 | Yes | No | No | No |
| Proxy | 7 | Yes | Yes | No | Optional |
| NGFW | 3-7 | Yes | Yes | Yes | Yes |

---

### Firewall Rules

Firewall rules define what traffic is permitted or blocked. Each rule is composed of the following fields:

| Component | Description |
|-----------|-------------|
| **Source Address** | IP address (or range) that originates the traffic |
| **Destination Address** | IP address (or range) that receives the traffic |
| **Protocol** | Communication protocol (TCP, UDP, ICMP, etc.) |
| **Port** | Port number associated with the service |
| **Direction** | Whether the rule applies to inbound or outbound traffic |
| **Action** | What to do with matching traffic: Allow, Deny, or Forward |

#### Rule Actions

**Allow** — permits the matching traffic:

| Action | Source | Destination | Protocol | Port | Direction |
|--------|--------|-------------|----------|------|-----------|
| Allow | 192.168.1.0/24 | Any | TCP | 80 | Outbound |

Allows all HTTP traffic from the internal network to the internet.

**Deny** — blocks the matching traffic:

| Action | Source | Destination | Protocol | Port | Direction |
|--------|--------|-------------|----------|------|-----------|
| Deny | Any | 192.168.1.0/24 | TCP | 22 | Inbound |

Blocks all inbound SSH connections to the internal network.

**Forward** — redirects traffic to a different destination or network segment:

| Action | Source | Destination | Protocol | Port | Direction |
|--------|--------|-------------|----------|------|-----------|
| Forward | Any | 192.168.1.8 | TCP | 80 | Inbound |

Redirects all incoming HTTP traffic to the internal web server at `192.168.1.8`.

#### Rule Directionality

| Direction | Applies To | Example |
|-----------|-----------|---------|
| **Inbound** | Traffic coming into the network or device | Allow HTTP (port 80) to a web server |
| **Outbound** | Traffic leaving the network or device | Block all outbound SMTP (port 25) except from the mail server |
| **Forward** | Traffic being routed between internal network segments | Forward HTTP traffic to an internal web server |

---

### Windows Defender Firewall

Windows Defender Firewall is the built-in host-based firewall in Windows. It provides basic traffic filtering, application allow/deny control, and custom rule creation.

Access via Windows search: `Windows Defender Firewall`

#### Network Profiles

Windows uses **Network Location Awareness (NLA)** to detect the current network type and automatically apply the corresponding firewall profile.

| Profile | Use Case | Typical Configuration |
|---------|----------|-----------------------|
| **Private** | Home or trusted office networks | More permissive — allows inbound connections from trusted devices |
| **Public / Guest** | Untrusted networks (coffee shops, airports, hotels) | Restrictive — blocks most inbound connections, allows only essential outbound |

Different rules can be configured for each profile — rules applied in the Private profile do not automatically apply on Public networks.

---

### Linux Firewall: Netfilter

**Netfilter** is the packet filtering framework built into the Linux kernel. It provides the core capabilities that all Linux firewall utilities build on:
- Packet filtering
- Network Address Translation (NAT)
- Connection tracking

Firewall utilities interact with the Netfilter framework to configure rules. The main options are:

| Utility | Description |
|---------|-------------|
| **iptables** | The most widely used Linux firewall utility. Directly interfaces with Netfilter. Complex syntax, but highly flexible and powerful |
| **nftables** | The successor to iptables. Improved syntax, better performance, and enhanced NAT capabilities. Also Netfilter-based |
| **firewalld** | A firewall management daemon that uses Netfilter under the hood. Works with predefined network zones (trusted, public, dmz, etc.) — simpler to manage than raw iptables for most use cases |
| **ufw** | Uncomplicated Firewall — a simplified frontend for iptables. Designed to be beginner-friendly. `ufw` commands translate into iptables rules automatically |

#### Basic ufw Commands

Enable the firewall:
```bash
sudo ufw enable
```

Allow a port:
```bash
sudo ufw allow 22/tcp
```

Deny a port:
```bash
sudo ufw deny 23/tcp
```

Allow traffic from a specific IP:
```bash
sudo ufw allow from 192.168.1.100
```

Check current rules and status:
```bash
sudo ufw status verbose
```

Delete a rule:
```bash
sudo ufw delete allow 22/tcp
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Firewall | Network security device that filters traffic based on rules |
| Stateless | Filters each packet independently, no connection memory |
| Stateful | Tracks connection state and filters based on connection history |
| Proxy firewall | Layer 7 intermediary that inspects packet contents |
| NGFW | Next-Generation Firewall — full-stack inspection with IPS, DPI, and SSL decryption |
| DPI | Deep Packet Inspection — inspecting the payload content of packets, not just headers |
| IPS | Intrusion Prevention System — actively blocks detected malicious traffic in real time |
| State table | A firewall's record of established connections used for stateful filtering |
| Netfilter | Linux kernel framework providing core firewall and NAT capabilities |
| NLA | Network Location Awareness — Windows mechanism for detecting and applying network profiles |

---

## Workflow / Process

```
Network traffic arrives at the firewall
        |
        v
Firewall checks the packet against its ruleset
(source IP, destination IP, port, protocol, direction)
        |
        v
Stateful firewall: is this part of an established connection?
  Yes --> Allow automatically (no re-inspection needed)
  No  --> Evaluate against rules
        |
        v
Does the packet match a rule?
  Match found --> Apply rule action (Allow / Deny / Forward)
  No match    --> Apply default policy (typically Deny All)
        |
        v
Allowed packets proceed to their destination
Denied packets are dropped (silently or with notification)
Forwarded packets are redirected to the configured destination
```

---

## Real-World Relevance

- Firewalls are the first line of defence in every network — understanding rule logic is essential for both configuring defences and identifying bypass opportunities in penetration tests
- Stateless firewalls are still common on high-throughput network devices where performance is critical (e.g. core routers in ISP infrastructure)
- NGFWs are the standard for enterprise perimeter security — products like Palo Alto, Fortinet, and Check Point dominate this market
- SSL/TLS decryption on NGFWs is a significant privacy and operational consideration — it allows encrypted traffic to be inspected, which is both a security capability and a compliance concern
- ufw is the standard tool for host-based firewall configuration on Ubuntu/Debian systems — it's commonly used in server hardening
- iptables rules are frequently encountered in CTFs and OSCP labs when pivoting through Linux systems or bypassing network restrictions
- From an offensive perspective, firewall rules define what ports and protocols are reachable — enumeration of firewall rules is part of internal network reconnaissance

---

## Key Learnings

- Stateless firewalls filter by rule only; stateful firewalls add connection tracking; proxy firewalls inspect content; NGFWs combine all capabilities with IPS and DPI
- Firewall rules are composed of: source, destination, protocol, port, direction, and action
- Three rule actions: Allow (permit), Deny (block), Forward (redirect)
- Rules apply to inbound, outbound, or forwarded traffic independently
- Windows Defender Firewall uses network profiles (Private, Public) — different rules apply per profile
- Linux firewalls are built on Netfilter; ufw provides a simplified interface for iptables rule management

---

## Additional Notes

- Firewall rule order matters — rules are typically evaluated top-to-bottom, and the first match is applied. A broad Allow rule above a specific Deny rule can nullify the Deny
- The default policy (what happens when no rule matches) should be **Deny All** in a secure configuration — only explicitly permitted traffic should be allowed
- A DMZ (Demilitarised Zone) is a common network design pattern where internet-facing servers are placed in a separate segment with specific firewall rules — different from both the internal network and the internet
- Host-based firewalls (Windows Defender, ufw) and network-based firewalls complement each other — defence in depth means relying on both, not just one

---

## Conclusion

Firewalls are a fundamental security control in every network. Understanding the four types — stateless, stateful, proxy, and NGFW — and how each works at different OSI layers provides the knowledge needed to select the right tool for a given scenario. Firewall rule structure is straightforward once the components are understood, and the Linux Netfilter stack with ufw makes host-based firewall management accessible even without deep iptables knowledge.
