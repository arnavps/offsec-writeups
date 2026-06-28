# Secure Network Architecture

## Executive Summary

This room establishes the foundational principles of designing networks that are both functional and secure. It moves beyond the naive assumption that subnets alone provide security and introduces the actual mechanisms that create enforced security boundaries: VLANs, security zones, firewall zone-pairs, and layer-two security protocols.

**Major concepts covered:** VLANs and 802.1q tagging, Router on a Stick (ROAS), security zones (External, DMZ, Trusted, Restricted, Management, Audit), traffic filtering and ACLs, stateful vs stateless firewalls, zone-pair policies, SSL/TLS inspection, DHCP snooping, and Dynamic ARP Inspection.

**Why these concepts matter:** A poorly designed network is not merely inefficient — it is a force multiplier for attackers. A single compromised endpoint in a flat network can reach every server, database, and domain controller. Proper segmentation, enforced by firewalls and access controls, limits the blast radius of any incident. This is the practical implementation of defence-in-depth at the network layer.

**Where they appear in real organisations:** Every enterprise network uses some form of VLAN segmentation. Security zones define the architecture of every corporate data centre, DMZ, and cloud virtual network. Zone-pair firewalls are the standard configuration model in Cisco ASA, Palo Alto NGFW, and Juniper SRX deployments. DHCP snooping and DAI are enabled by default in most enterprise switch configurations.

---

## Big Picture Overview

A network without security design is a flat space where every device can reach every other device. This is the network equivalent of a building where every door is always unlocked. Adding subnets organises traffic into logical groups but does not create enforcement — routes between subnets mean any device can still reach any other device if routing exists.

The solution is layered: VLANs create the logical boundaries at layer two, security zones define the trust model, and firewalls with zone-pair policies enforce the rules at layer three and above. Layer-two security protocols (DHCP snooping, DAI) then close the gaps that firewalls cannot reach.

```
Without security design:
  Workstation → Database (direct route → no restriction)
  BYOD Device → Domain Controller (direct route → no restriction)

With security design:
  Workstation (VLAN 20 / Trusted) → Firewall → Database (VLAN 30 / Restricted)
  BYOD Device (VLAN 10 / DMZ) → Firewall → [BLOCKED] Domain Controller
```

---

## Core Concepts

---

### Concept 1 — VLANs and 802.1q Tagging

**What Is It?**
A VLAN (Virtual LAN) is a logical segmentation of a physical network at Layer 2. Devices in different VLANs cannot communicate directly — even if they share the same physical switch — without routing through a layer-three device.

**Why Does It Exist?**
Subnets divide IP address space but do not enforce layer-two isolation. A device on the same physical switch segment can still send frames to any other device on that segment. VLANs solve this by tagging frames at the switch level so that only devices in the same VLAN receive each other's broadcast traffic and can communicate at layer two.

**The BYOD Problem:**
A Bring Your Own Device (BYOD) infected with a Remote Access Trojan (RAT) joins the corporate network. Without VLANs, routing allows it to reach internal servers. With VLANs, the BYOD device is isolated in the DMZ VLAN, and the firewall controls what it can reach.

**How It Works — 802.1q Tagging:**
IEEE 802.1q (dot1q) is the standardised protocol for VLAN tagging. When a frame leaves a switch port assigned to a VLAN, the switch inserts a 4-byte 802.1q tag into the Ethernet frame header containing:
- Tag Protocol Identifier (TPID): 0x8100 — identifies the frame as tagged
- Priority Code Point (PCP): 3 bits for QoS
- Drop Eligible Indicator (DEI): 1 bit
- VLAN Identifier (VID): 12 bits — up to 4094 VLANs

Because 802.1q is a vendor-agnostic standard, Cisco switches and Juniper routers can exchange tagged frames interchangeably.

**Configuring VLANs in Open vSwitch:**
```bash
# View current switch configuration
ovs-vsctl show

# Assign VLAN tag 10 to interface eth1
ovs-vsctl set port eth1 tag=10

# Configure a native VLAN (for untagged traffic)
ovs-vsctl set port eth0 tag=10 vlan_mode=native-untagged
```

The native VLAN handles any traffic that arrives untagged — typically traffic from older devices that do not send 802.1q-tagged frames. Setting a native VLAN ensures all traffic is categorised, preventing untagged frames from bypassing VLAN controls.

**Trunk Ports:**
A trunk is a switch port configured to carry frames from multiple VLANs simultaneously. Trunks use 802.1q tagging to distinguish which VLAN each frame belongs to. Trunks connect switches to other switches, and switches to routers.

```bash
# Add a bridge and interface to create a trunk (Open vSwitch)
ovs-vsctl add-br br0
ovs-vsctl add-port br0 eth0 tag=10
```

**Security Importance:**
VLANs prevent layer-two attacks (ARP floods, broadcast storms) from crossing VLAN boundaries. They are the foundational building block of all network segmentation.

**Limitations:**
VLANs alone do not enforce security — they create the boundaries, but without firewall rules, routing still allows cross-VLAN communication. VLAN hopping attacks (double tagging, rogue trunk negotiation) can bypass VLAN controls if switches are misconfigured.

**Common Misconceptions:**
- "VLANs are a security control." — They are a segmentation control. Security comes from the firewall rules applied to traffic crossing VLAN boundaries.
- "Native VLAN is just a default." — The native VLAN is a VLAN hopping attack vector if not properly managed. VLAN 1 is the default native VLAN on Cisco — change it.

---

### Concept 2 — Router on a Stick (ROAS)

**What Is It?**
ROAS is a routing architecture where a single physical router interface handles inter-VLAN routing by dividing its interface into multiple logical sub-interfaces, one per VLAN. The "stick" refers to the single physical connection between the switch and the router.

**Why Does It Exist?**
Before ROAS, inter-VLAN routing required a separate physical router port per VLAN — expensive and difficult to scale. ROAS consolidates all inter-VLAN traffic onto one trunk connection.

**How It Works:**
The router creates virtual sub-interfaces (e.g., `eth0.10`, `eth0.20`) each assigned to a VLAN ID. Tagged frames arrive on the trunk, and the router uses the VLAN tag to direct them to the correct sub-interface for routing.

**Configuring ROAS in VyOS:**
```bash
# Create sub-interface for VLAN 10 and assign IP
set interfaces ethernet eth0 vif 10 description 'VLAN 10'
set interfaces ethernet eth0 vif 10 address '192.168.100.1/24'
```

**Security Implication:**
ROAS enables inter-VLAN routing — but routes alone do not restrict traffic. Without firewall zone-pair rules, all VLANs can communicate freely through the router. The router is the insertion point for the firewall.

---

### Concept 3 — Security Zones

**What Is It?**
Security zones define the trust level and expected traffic profile of a network segment. Rather than thinking about individual IP addresses and ports, zones provide a conceptual framework: "traffic from the DMZ zone to the Trusted zone must be explicitly permitted."

**Why Does It Exist?**
ACLs and firewall rules are complex and error-prone when defined per-host. Zones abstract the problem — define what each zone represents, then define policies between zones. This makes the security architecture understandable and auditable.

**Standard Zone Definitions:**

| Zone | Trust Level | Examples | Typical Controls |
|------|------------|---------|-----------------|
| **External** | Untrusted | Internet, third-party networks | All inbound denied by default |
| **DMZ** | Semi-trusted | Public web servers, BYOD, remote users | Outbound to External allowed; inbound to Trusted blocked |
| **Trusted** | Internal | Corporate workstations, B2B connections | Full internal access; inbound from DMZ limited |
| **Restricted** | High-security | Domain controllers, databases, financial systems | Inbound only from specific Trusted sources; no direct internet |
| **Management** | Admin-only | Virtualisation management, backup servers | Accessible only from dedicated admin workstations |
| **Audit** | Security-only | SIEM, logging infrastructure, telemetry | Write-only from other zones; no outbound internet |

**Real-World Application:**
In an enterprise data centre, the web tier sits in the DMZ, the application tier in Trusted, and the database tier in Restricted. A web server compromise cannot directly reach the database because the firewall zone-pair policy blocks DMZ → Restricted traffic on database ports.

**Why Zones Are Not Sufficient Alone:**
"Even if there are routes between zones, there is no security boundary unless enforced by firewall rules." A zone is a label — the firewall is the enforcement point. Without firewall zone-pair policies, zones are merely documentation.

---

### Concept 4 — Traffic Filtering and ACLs

**What Is It?**
An Access Control List (ACL) is a ruleset that defines whether network traffic should be permitted or denied based on matching criteria (source/destination address, port, protocol). An ACL contains individual Access Control Entries (ACEs).

**Why Does It Exist?**
ACLs provide the most basic form of traffic control — they tell routers which packets to forward and which to drop. They predate stateful firewalls and remain relevant for simple filtering at routers and layer-3 switches.

**How ACLs Work in VyOS:**
```bash
# Create ACL 100
set policy access-list 100 description "Block external SSH attempts"

# Create a permit rule (ACE) for internal SSH
set policy access-list 100 rule 10 action permit
set policy access-list 100 rule 10 source network 192.168.10.0/24

# Create a deny rule for all other sources
set policy access-list 100 rule 20 action deny
set policy access-list 100 rule 20 source any
```

**ACL Processing:**
Rules are evaluated top-down. The first matching rule wins. An implicit deny-all exists at the end of most ACL implementations — traffic not matching any permit rule is dropped.

**Limitations of ACLs:**
ACLs are stateless — they evaluate each packet independently without knowledge of connection state. A packet with a manipulated TCP flag can appear to be part of an established connection and bypass inbound-only ACL rules. This is why stateful firewalls replaced ACLs as the primary access control mechanism.

---

### Concept 5 — Stateful Firewalls and Zone-Pairs

**Stateless vs Stateful:**

| Characteristic | Stateless (ACL) | Stateful (Firewall) |
|---------------|----------------|-------------------|
| Connection tracking | No | Yes |
| TCP state awareness | No | Yes |
| Can be bypassed via TCP flags | Yes | No |
| Performance | Higher | Slightly lower |
| Typical use | Router filtering | Primary security control |

**Zone-Pairs:**
A zone-pair is a directional firewall policy that defines what traffic is permitted from Zone A to Zone B. Because direction matters (DMZ→LAN is different from LAN→DMZ), each combination requires its own ruleset.

For N zones, the maximum number of zone-pairs is N × (N-1). A 4-zone topology requires up to 12 zone-pair rulesets.

**Zone-Pair Configuration in VyOS:**

Step 1 — Define zones and assign interfaces:
```bash
# Set default action for DMZ zone
set zone-policy zone dmz default-action drop
set zone-policy zone dmz interface eth0.30

# Repeat for LAN, WAN, LOCAL zones
set zone-policy zone lan default-action drop
set zone-policy zone lan interface eth0.20

set zone-policy zone wan default-action drop
set zone-policy zone wan interface eth0.10
```

Step 2 — Create firewall ruleset for each zone-pair direction:
```bash
# LAN → WAN ruleset
name lan-wan {
  default-action drop
  enable-default-log
  rule 1 {
    action accept
    state { established enable; related enable }
  }
  rule 2 {
    action drop
    log enable
    state { invalid enable }
  }
  rule 100 {
    action drop
    log enable
    protocol ipv4-icmp
  }
}

# WAN → LAN ruleset (allow ICMP responses)
name wan-lan {
  default-action drop
  enable-default-log
  rule 1 {
    action accept
    state { established enable; related enable }
  }
  rule 100 {
    action accept
    log enable
    protocol ipv4-icmp
  }
}
```

Step 3 — Apply rulesets to zone-pairs:
```bash
set zone-policy zone LAN from WAN firewall name lan-wan
set zone-policy zone WAN from LAN firewall name wan-lan
```

**Rule 1 — Established/Related:**
This rule is critical and must appear at the top of every zone-pair ruleset. It allows return traffic for connections that were already permitted. Without this, every response packet would be dropped even though the original request was permitted.

**Zone-Pair Matrix for a Standard 4-Zone Topology:**

| Zone A | Zone B | Protocol | Action |
|--------|--------|---------|--------|
| LAN | WAN | ICMP | Drop |
| WAN | LAN | ICMP | Accept |
| DMZ | LAN | HTTP/S | Accept |
| LAN | DMZ | ICMP | Accept |
| DMZ | WAN | — | Drop |
| WAN | DMZ | — | Drop |

**Important:** IPv6 must be considered separately. Forgetting IPv6 zone-pair rules is a common misconfiguration that leaves IPv6 traffic completely unfiltered.

---

### Concept 6 — SSL/TLS Inspection

**What Is It?**
SSL/TLS inspection (also called SSL decryption or HTTPS inspection) is the process of intercepting encrypted HTTPS connections, decrypting them, inspecting the content, re-encrypting, and forwarding them. It is effectively a controlled, transparent Man-in-the-Middle.

**Why Does It Exist?**
HTTPS is the default for all web traffic. This means malware, C2 (command and control) beacons, and data exfiltration can hide inside TLS-encrypted sessions that pass through firewalls unexamined. A threat actor with an implant on a LAN machine can communicate with their C2 server over HTTPS, and the firewall sees only an encrypted stream.

**How It Works:**
```
Client → [SSL Proxy] → Firewall/UTM → Internet Server
         ↑                ↑
         Decrypts here    Inspects here: IPS, Web Filter, DLP
         Re-encrypts with org CA cert
```

The SSL proxy presents its own certificate (signed by an enterprise CA trusted by all internal clients) to the client, while establishing a separate TLS session to the actual server. The client sees the proxy's certificate; the server sees the proxy as the client.

**Security Benefits:**
- IPS can inspect HTTP payloads inside HTTPS
- Web filtering can see the actual URL, not just the destination IP
- DLP can detect sensitive data exfiltration inside HTTPS
- Malware C2 communication over HTTPS becomes visible

**Limitations and Risks:**
- Every HTTPS session passes through a man-in-the-middle — this includes banking, healthcare, and personal sessions
- Certificate pinning (used by some apps) will break when the proxy's certificate is presented instead of the expected one
- Privacy implications: the organisation can read all employee HTTPS traffic
- Advanced threat actors route C2 through trusted CDNs (Cloudflare, AWS) that organisations typically exclude from inspection to avoid service breaks

**Design Consideration:**
Organisations must define an exclusion list — categories of sites (banking, healthcare, government) where inspection is bypassed. The trade-off between visibility and privacy must be explicitly documented and approved.

---

### Concept 7 — DHCP Snooping

**What Is It?**
DHCP snooping is a layer-2 security feature on switches that prevents rogue DHCP servers from assigning IP addresses to clients. It acts as a firewall between untrusted hosts and the legitimate DHCP server.

**The Problem It Solves:**
An attacker on the network can run a rogue DHCP server that responds to client DHCP requests before the legitimate server. The attacker can assign themselves as the default gateway (enabling MITM) or assign malicious DNS servers (enabling DNS hijacking).

**How It Works:**
- Switch ports are classified as **trusted** (connected to legitimate DHCP server) or **untrusted** (connected to clients)
- DHCP server responses (OFFER, ACK) arriving on untrusted ports are dropped
- The switch maintains a **DHCP Binding Database** mapping: MAC address → IP address → VLAN → interface

**DHCP Packets Dropped by Snooping:**
- DHCP packets from outside the network
- Packets where source MAC ≠ DHCP client hardware address
- DHCPRELEASE/DHCPDECLINE on an untrusted interface where the source IP does not match the binding database
- DHCP packets with a relay agent address ≠ 0.0.0.0

**The DHCP Binding Database:**
```
Router# show ip dhcp snoop bind
MacAddress         IpAddress    Lease(sec) Type          VLAN Interface
02:c8:85:b5:5a:aa  10.10.0.1    23453      dhcp-snooping 10   GigabitEthernet1/1
01:02:03:04:05:06  2.2.2.2      69445      dhcp-snooping 20   GigabitEthernet2/1
```

This database is used by Dynamic ARP Inspection as its source of truth for valid MAC-IP pairs.

---

### Concept 8 — Dynamic ARP Inspection (DAI)

**What Is It?**
DAI validates ARP packets by comparing the sender's MAC and IP address against the DHCP Binding Database. If a mismatch is detected (indicating ARP spoofing), the packet is intercepted, logged, and discarded.

**The Problem It Solves — ARP Spoofing:**
ARP has no authentication. Any device can send a gratuitous ARP claiming to be any IP address. An attacker sends ARP replies claiming their MAC address is the gateway's IP address. All clients update their ARP cache. All traffic intended for the gateway now flows through the attacker — a perfect layer-2 MITM.

**How DAI Works:**
1. DHCP snooping populates the binding database with legitimate MAC-IP pairs
2. DAI intercepts all ARP packets on untrusted ports
3. For each ARP packet: compare sender MAC + sender IP against the binding database
4. If they match → forward the ARP packet
5. If they do not match → drop the packet and log the event

**Valid ARP (passes DAI):**
```
ARP Request:
  Sender MAC: 02:c8:85:b5:5a:aa
  Sender IP: 10.10.0.1

DHCP Binding DB:
  02:c8:85:b5:5a:aa → 10.10.0.1 ✓ MATCH → Forward
```

**Invalid ARP (blocked by DAI — ARP spoofing detected):**
```
ARP Request:
  Sender MAC: 02:c8:85:bb:bb:bb  ← SPOOFED MAC
  Sender IP: 10.10.0.1

DHCP Binding DB:
  02:c8:85:b5:5a:aa → 10.10.0.1  ← LEGITIMATE ENTRY

MAC mismatch detected → Drop + Log
```

**DAI Dependency Chain:**
```
DHCP Snooping (builds binding database)
         ↓
Dynamic ARP Inspection (uses binding database to validate ARPs)
         ↓
IP Source Guard (optional — uses binding database to validate IP source addresses)
```

---

## Architecture and Relationships

### Complete Network Security Architecture

```
Internet (External Zone)
         |
    [WAN Interface]
         |
    [Firewall / Zone-Pair Engine]
         |
    ┌────┴──────────┐
    |               |
[DMZ Zone]    [LAN / Trusted Zone]
eth0.10       eth0.20
    |               |
Public Servers  Workstations
Web, Mail       Internal Apps
                    |
             [Restricted Zone]
             eth0.30
                    |
             Domain Controllers
             Databases
```

### Layer-2 Security Stack

```
Physical Switch Port
         |
    VLAN Tagging (802.1q)
         |
    DHCP Snooping (validates IP assignment, builds binding DB)
         |
    Dynamic ARP Inspection (validates ARP using binding DB)
         |
    IP Source Guard (optional — validates source IP against binding DB)
         |
    Trunk to Router/Firewall
         |
    Zone-Pair Firewall Policy
         |
    SSL/TLS Inspection (optional — for HTTPS visibility)
```

---

## Security Engineer Perspective

### Configuration Checklist

| Control | What to Configure | Common Mistake |
|---------|-----------------|---------------|
| VLANs | All ports assigned to explicit VLANs; no access ports on VLAN 1 | Leaving management interfaces on default VLAN 1 |
| Native VLAN | Change native VLAN from VLAN 1 to an unused VLAN | VLAN hopping via double-tagging against native VLAN 1 |
| Zone-pairs | Explicit rules for all zone directions; default-action drop | Forgetting IPv6 zone-pair rules |
| Established/Related | Rule 1 in every zone-pair ruleset | Forgetting this rule breaks all return traffic |
| DHCP Snooping | Enable on all VLANs; mark only uplink to DHCP server as trusted | Enabling on VLAN but forgetting to mark trusted port |
| DAI | Enable on all untrusted VLANs after DHCP snooping is running | Enabling DAI before DHCP snooping — no binding DB = all ARPs dropped |

### Monitoring and Detection Opportunities

| Event | Detection Signal | Tool |
|-------|----------------|------|
| VLAN hopping | Unexpected VLAN tags from access ports | Switch logs, SIEM |
| Rogue DHCP server | DHCP snooping drops on trusted ports | Switch logs |
| ARP spoofing | DAI drops with MAC mismatch | Switch logs, SIEM |
| Zone-pair policy violations | Firewall drop logs | Firewall logs, SIEM |
| SSL/TLS inspection evasion | Traffic to known CDN IPs with unusual patterns | SIEM, UEBA |

### Offensive Perspective

| Attack | Target | Detection |
|--------|--------|---------|
| VLAN hopping (double tagging) | Reach restricted VLANs from access port | DAI, switch port security |
| Rogue DHCP | MITM via gateway IP takeover | DHCP snooping drops |
| ARP poisoning | MITM for credential theft, session hijacking | DAI drops + logs |
| Zone-pair bypass | Find misconfigured rule or missing IPv6 rule | Firewall review, pen test |
| SSL/TLS interception bypass | Route C2 through CDN in inspection exclusion list | Behavioural analysis, JA3 fingerprinting |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **802.1q** | IEEE standard for VLAN tagging in Ethernet frames |
| **ACE** | Access Control Entry — a single rule within an ACL |
| **ACL** | Access Control List — a set of permit/deny rules for network traffic |
| **BYOD** | Bring Your Own Device — employee/visitor personal devices on the corporate network |
| **DAI** | Dynamic ARP Inspection — validates ARP packets against the DHCP binding database |
| **DHCP Binding Database** | Switch-maintained table of MAC→IP→VLAN→Interface mappings from DHCP snooping |
| **DHCP Snooping** | Layer-2 feature that prevents rogue DHCP servers |
| **DMZ** | Demilitarized Zone — semi-trusted network segment for public-facing or BYOD resources |
| **Native VLAN** | The VLAN assigned to untagged traffic on a trunk port |
| **ROAS** | Router on a Stick — inter-VLAN routing via a single trunk connection and sub-interfaces |
| **Security Zone** | A logical grouping of network segments sharing a trust level and policy |
| **SSL/TLS Inspection** | Intercepting and decrypting HTTPS traffic for content inspection |
| **Stateful Firewall** | Firewall that tracks connection state to enforce policies |
| **Stateless Firewall** | Firewall (ACL) that evaluates each packet independently without connection context |
| **Trunk** | A switch port carrying tagged traffic from multiple VLANs |
| **UTM** | Unified Threat Management — platform combining firewall, IPS, web filtering, DLP |
| **VLAN** | Virtual LAN — logical layer-2 network segment isolating broadcast domains |
| **VLAN Hopping** | Attack technique using double-tagging or rogue trunk negotiation to access other VLANs |
| **Zone-Pair** | A directional firewall policy applied to traffic flowing from one zone to another |

---

## Exam and Interview Revision

### Must Remember

- VLANs isolate layer-2 traffic but **do not enforce security** — routes exist between VLANs
- 802.1q is the **IEEE standard** for VLAN tagging — 4-byte tag, 12-bit VLAN ID, up to 4094 VLANs
- Native VLAN handles untagged traffic — change from default VLAN 1 to prevent VLAN hopping
- ROAS uses sub-interfaces (`eth0.10`, `eth0.20`) on a single trunk — "one stick"
- Security zones define trust levels; zone-pairs define the directional firewall policy between zones
- **Always include Rule 1 (established/related)** in zone-pair rulesets — allows return traffic
- Default action for every zone must be **drop** — explicit permit for all permitted traffic
- ACLs are **stateless** — evaluated per packet; no connection tracking
- Stateful firewalls track TCP state — cannot be bypassed by flag manipulation
- DHCP snooping **builds the binding database** that DAI depends on — deploy snooping before DAI
- DAI compares ARP sender MAC+IP against the DHCP binding database — mismatch = ARP spoofing
- SSL/TLS inspection is a **controlled MITM** — requires enterprise CA trust; certificate pinning breaks it
- Zone-pair policies must cover **both IPv4 and IPv6** — forgetting IPv6 leaves it unfiltered

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is the difference between a VLAN and a subnet? | Subnet = layer-3 address range. VLAN = layer-2 broadcast domain isolation. A VLAN typically has one subnet, but subnets don't inherently isolate layer-2 traffic. |
| Why is ROAS called "on a stick"? | All inter-VLAN traffic flows through a single physical trunk connection — one "stick" between switch and router. |
| What is the native VLAN and why is it a security risk? | The native VLAN receives untagged traffic on a trunk. If left as VLAN 1, a double-tagging attack can send frames into VLAN 1 from an access port — VLAN hopping. |
| What does DAI protect against? | ARP spoofing / ARP poisoning. It validates ARP sender MAC+IP pairs against the DHCP binding database and drops mismatches. |
| What must come before DAI? | DHCP snooping — it builds the binding database that DAI uses. |
| What is a zone-pair? | A directional firewall policy applied to traffic going from one security zone to another. DMZ→LAN and LAN→DMZ are separate zone-pairs. |
| What does the established/related rule do? | Allows return traffic for already-permitted connections to pass through the firewall. Without it, responses to outbound requests are dropped. |
| What are the downsides of SSL/TLS inspection? | It is a MITM — can intercept sensitive personal data (banking). Certificate pinning breaks. Advanced attackers can route C2 through CDNs typically excluded from inspection. |
