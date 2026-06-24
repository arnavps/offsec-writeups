# Nmap Live Host Discovery

## Overview

Before scanning ports or enumerating services, a penetration tester must first determine which hosts on a network are actually online. Scanning non-existent or offline hosts wastes time and generates unnecessary noise. This room covers Nmap's host discovery phase — the first phase in any Nmap scan — and explains how different protocols (ARP, ICMP, TCP, UDP) can be used at different network layers to identify live hosts, each with its own characteristics, limitations, and appropriate use cases.

**Main objectives:**
- Understand why host discovery is a necessary first step before port scanning
- Learn how ARP, ICMP, TCP, and UDP scans work at different network layers
- Understand which scan type is appropriate for different network positions (same subnet vs remote)
- Learn the difference between privileged and unprivileged scan behaviour
- Understand reverse DNS lookup and when to enable or disable it

**Skills introduced:** ARP scan, ICMP echo/timestamp/address mask scans, TCP SYN/ACK ping scans, UDP ping scans, Masscan comparison

---

## Concepts Covered

### Nmap Scan Phases

**Definition:** Nmap executes scans in a structured sequence of phases, though not all phases run by default — they are controlled by command-line arguments.

**Why It Matters:** Understanding the phases helps explain why specific flags enable specific behaviours, and why skipping host discovery when you know a target is alive (e.g., `-Pn`) can save time.

**Key Details — The Nmap scan pipeline:**
1. Enumerate Targets
2. Discover live hosts (host discovery)
3. Reverse DNS lookup
4. Scan ports
5. Detect versions
6. Detect OS
7. Traceroute
8. Scripts
9. Write output

**Practical Relevance:** This room focuses exclusively on phase 2 (host discovery). Port scanning, version detection, OS detection, tracerouting, and scripts are covered in subsequent Nmap rooms. Knowing the pipeline prevents confusion about what each `-s` flag controls.

---

### Why Host Discovery Matters

**Definition:** Host discovery is the process of identifying which IP addresses in a target range are assigned to live (online) hosts before investing time in port-level scanning.

**Why It Matters:** Port scanning every IP in a /16 subnet (65,536 addresses) blindly wastes enormous time and generates significant noise. Identifying live hosts first and scanning only those reduces scan duration, minimises detection risk, and focuses effort productively.

**Key Details:**
- Nmap by default runs a ping scan (host discovery) then proceeds to scan only live hosts
- `-sn` flag runs host discovery only, without proceeding to port scanning
- Without any host discovery options, Nmap's default behaviour varies by privilege level and network position

**Default Nmap Host Discovery Behaviour:**

| Scenario | Discovery Method |
|---------|-----------------|
| Privileged user, local network (Ethernet) | ARP requests |
| Privileged user, remote network | ICMP echo + TCP ACK 80 + TCP SYN 443 + ICMP timestamp |
| Unprivileged user, remote network | TCP 3-way handshake on ports 80 and 443 |

**Practical Relevance:** During internal penetration tests, ARP is used automatically on the local subnet. For external assessments, Nmap uses multiple methods simultaneously because any single method might be blocked.

---

### ARP Scan

**Definition:** ARP (Address Resolution Protocol) is a link-layer protocol that resolves IP addresses to MAC addresses. ARP queries are broadcast within the local subnet — they cannot be routed across subnet boundaries.

**Why It Matters:** ARP is the most reliable host discovery method on a local network because ARP broadcasts are not filtered by host-based firewalls (they operate at Layer 2, below the IP stack). A host that responds to an ARP query is definitively online.

**Key Details:**
- ARP sends a broadcast frame to `FF:FF:FF:FF:FF:FF` asking "Who has this IP address?"
- Online hosts with that IP respond with their MAC address
- ARP works only within the same subnet — ARP packets cannot cross routers
- ARP scan is the default when scanning local network as a privileged user
- Useful for internal network enumeration and post-exploitation lateral movement mapping
- Because ARP is not filtered by firewalls, it reliably identifies hosts that block ICMP

**Practical Relevance:** After gaining access to an internal system, ARP scanning quickly maps the local network segment. ARP results are highly reliable compared to ICMP-based scans, which are frequently blocked by host firewalls.

---

### ICMP Echo Scan

**Definition:** ICMP (Internet Control Message Protocol) Echo Request (Type 8) / Echo Reply (Type 0) is the classic "ping" mechanism for testing host reachability.

**Why It Matters:** ICMP echo is the most intuitive host discovery method, but it is also the most commonly blocked. Windows hosts have ICMP Echo blocked by default in their host firewall. Many network firewalls also block ICMP. Understanding its limitations prevents false negatives.

**Key Details:**
- Sends ICMP Type 8 (Echo Request), expects ICMP Type 0 (Echo Reply) in response
- On the local subnet, ARP precedes the ICMP request (Nmap handles this automatically)
- Enabled with `-PE` flag
- Combine with `-sn` to suppress subsequent port scanning: `nmap -PE -sn TARGET`
- Many firewalls block ICMP — a non-response does not confirm the host is offline

---

### ICMP Timestamp Scan

**Definition:** ICMP Timestamp Request (Type 13) / Timestamp Reply (Type 14) is an alternative ICMP type for probing host liveness.

**Why It Matters:** When ICMP Echo is blocked, Timestamp requests may still pass through because they are a different ICMP type with separate firewall rules. Using multiple ICMP types increases the probability of getting a response.

**Key Details:**
- Enabled with `-PP` flag
- Less commonly blocked than ICMP Echo
- If ICMP Echo packets are blocked at the target or in transit, timestamp requests may still get through

---

### ICMP Address Mask Scan

**Definition:** ICMP Address Mask Request (Type 17) / Address Mask Reply (Type 18) is a third ICMP type used for host probing.

**Why It Matters:** Provides yet another alternative when other ICMP types are blocked. However, this type is more likely to be blocked than Echo or Timestamp.

**Key Details:**
- Enabled with `-PM` flag
- Often blocked by modern firewalls and operating systems
- In practical testing, this scan may return no results even when hosts are confirmed online — demonstrating that different packet types are handled differently at the network and host level

**Important observation:** Running `-PM` scan on a subnet that `-PE` confirmed had live hosts may return no results at all. This demonstrates why security professionals must know multiple discovery techniques — if one is blocked, another may succeed.

---

### TCP SYN Ping Scan

**Definition:** A host discovery technique that sends TCP SYN packets to specified ports. An open port responds with SYN/ACK; a closed port responds with RST. Either response confirms the host is alive.

**Why It Matters:** When ICMP is entirely blocked, TCP-based host discovery works against hosts that have services running. Most internet-accessible hosts have at least one TCP port open (80, 443, 22). TCP SYN ping takes advantage of this.

**Key Details:**
- Enabled with `-PS` followed by port(s): `-PS22`, `-PS21-25`, `-PS80,443,8080`
- Default port is 80 if no port is specified
- Privileged users: Nmap sends SYN, receives SYN/ACK or RST, tears down connection without completing the handshake
- Unprivileged users: Must complete the full TCP 3-way handshake (more detectable)
- The goal is only to determine if the host is alive — the port state itself is not the focus here

---

### TCP ACK Ping Scan

**Definition:** Sends TCP packets with the ACK flag set to probe host liveness. Any live host should respond with a TCP RST packet (because no connection was established to acknowledge).

**Why It Matters:** Some firewalls allow ACK packets through that would block SYN packets (because ACK packets appear to be part of an established connection). This makes ACK ping scans useful for bypassing certain firewall configurations.

**Key Details:**
- Enabled with `-PA` followed by port(s): `-PA21`, `-PA80,443,8080`
- Default port is 80
- Requires privileged user — unprivileged users will attempt a 3-way handshake instead
- A live host returns RST because an ACK with no prior connection is unexpected

---

### UDP Ping Scan

**Definition:** Sends UDP packets to specified ports. Unlike TCP, UDP is connectionless. The host discovery mechanism relies on closed UDP ports returning ICMP Port Unreachable (Type 3, Code 3) responses.

**Why It Matters:** Many systems have specific UDP services running (DNS on 53, SNMP on 161/162). UDP ping exploits the ICMP error response from closed UDP ports to confirm host liveness when TCP methods are blocked.

**Key Details:**
- Enabled with `-PU` followed by port(s): `-PU53,161,162`
- Sending to a closed UDP port triggers ICMP Port Unreachable → confirms host is online
- Sending to an open UDP port may receive no response (depends on the service)
- Can be combined with TCP scan types in the same command

---

### Target Specification

**Definition:** The methods Nmap accepts for specifying which hosts to scan.

**Why It Matters:** Flexible target specification allows efficient scanning of ranges, lists, and files without manually specifying individual IPs.

**Key Details:**

| Input format | Example | What it scans |
|-------------|---------|--------------|
| Single IP | `10.10.10.5` | One host |
| List of IPs | `10.10.10.5 10.10.10.6 example.com` | Three targets |
| Range | `10.10.10.15-20` | 6 IPs: .15, .16, .17, .18, .19, .20 |
| Subnet | `10.10.10.0/24` | 256 addresses |
| File | `nmap -iL hosts.txt` | All IPs/hostnames in file |

**Dry run (list without scanning):** `nmap -sL TARGETS` shows the hosts Nmap would scan without sending any packets. Nmap performs reverse DNS on listed targets — useful for previewing scope and discovering hostnames.

---

### Reverse DNS Lookup

**Definition:** rDNS resolves IP addresses to hostnames. Normal DNS resolves hostnames to IPs. Reverse DNS is the opposite direction.

**Why It Matters:** Hostnames often reveal system roles and network structure (e.g., `mail.company.local`, `dc01.domain.com`, `dev-server.internal`). This information enriches enumeration results and helps prioritise targets.

**Key Details:**
- `-R` forces reverse DNS lookup for all discovered hosts (including offline ones)
- `-n` disables DNS lookup entirely — faster, avoids DNS query logging
- Default: Nmap only performs reverse DNS on online hosts
- rDNS records are not always configured; results may be missing or misleading
- Use `--dns-servers DNS_SERVER` to specify a custom DNS server for lookups

---

## Methodology

The host discovery workflow follows a logical progression:

```
Define target scope (IPs, ranges, subnets, file)
        |
        v
Dry run to preview targets (optional):
  nmap -sL TARGET_RANGE
        |
        v
Choose discovery method based on network position:
  Same subnet → ARP scan (-PR) — most reliable
  Remote network → ICMP Echo (-PE), Timestamp (-PP), TCP SYN/ACK (-PS/-PA), UDP (-PU)
        |
        v
Run host discovery (no port scanning):
  nmap -sn [SCAN_FLAGS] TARGET_RANGE
        |
        v
Analyse results:
  - Which hosts responded?
  - Which probe types returned responses?
  - Which hosts might be alive but filtered?
        |
        v
Compile confirmed live host list
        |
        v
Proceed to port scanning against live hosts only
```

Each phase exists for a specific reason:
- ARP is used on local subnets because it is not filtered and operates at Layer 2
- Multiple ICMP types are tried because individual types may be selectively blocked
- TCP/UDP probes are used when ICMP is entirely blocked by firewalls
- `-sn` prevents premature port scanning before the scope is confirmed

---

## Practical Activities

### Activity 1 — ARP Scan (Local Subnet)

**Objective:** Discover all live hosts on the local subnet using ARP broadcast requests.

**Commands:**
```bash
sudo nmap -PR -sn 10.200.6.0/24
```

**Command Explanation:**
- `-PR` — tells Nmap to use ARP requests for host discovery
- `-sn` — host discovery only; suppress port scanning
- `10.200.6.0/24` — scan the entire /24 subnet (256 addresses)
- `sudo` required — ARP scanning requires root privileges

**Findings:**
- Nmap sends ARP broadcast requests sequentially to each IP in the subnet
- Online hosts respond with ARP replies containing their MAC addresses
- Each ARP reply confirms that host is alive at Layer 2

**Why It Matters:** ARP scanning is the most reliable local network discovery method because host-based firewalls do not block ARP (it operates below the IP stack). This makes it the preferred first-pass discovery method during internal penetration tests.

---

### Activity 2 — ICMP Echo Scan

**Objective:** Discover live hosts using ICMP ping requests.

**Commands:**
```bash
sudo nmap -PE -sn 10.200.6.0/24
```

**Command Explanation:**
- `-PE` — ICMP Echo Request (Type 8) for host discovery
- `-sn` — host discovery only

**Findings:**
- Hosts that respond to ICMP Echo confirm they are online
- Hosts with ICMP blocked (Windows default firewall, edge firewalls) will not appear

**Why It Matters:** ICMP Echo is intuitive but unreliable in hardened environments. Comparing results between `-PE` and `-PR` on the same subnet reveals which hosts have ICMP blocked — itself a useful security finding.

---

### Activity 3 — ICMP Timestamp and Address Mask Scans

**Objective:** Probe hosts with alternative ICMP types to discover hosts that block ICMP Echo.

**Commands:**
```bash
# ICMP Timestamp scan
sudo nmap -PP -sn 10.200.6.0/24

# ICMP Address Mask scan
sudo nmap -PM -sn 10.200.6.0/24
```

**Findings:**
- Timestamp (`-PP`) may return hosts that blocked Echo (`-PE`)
- Address Mask (`-PM`) returned no hosts in testing, even though hosts confirmed alive by ARP scan
- The `-PM` null result directly demonstrates firewall-level packet filtering

**Why It Matters:** Observing that `-PM` returns zero results while `-PE` or `-PP` found hosts confirms that specific ICMP types are being dropped by a firewall or the target system. This teaches a critical lesson: always try multiple techniques when reconnaissance yields no results.

---

### Activity 4 — TCP SYN Ping Scan

**Objective:** Use TCP SYN packets to confirm host liveness when ICMP is blocked.

**Commands:**
```bash
# TCP SYN ping on default port 80
sudo nmap -PS -sn 10.200.6.0/24

# TCP SYN ping on specific ports
sudo nmap -PS22,80,443 -sn 10.200.6.0/30
```

**Command Explanation:**
- `-PS` — TCP SYN ping
- Port list after `-PS` specifies which ports to probe
- Without port specification, port 80 is used by default

**Findings:**
- Two hosts discovered (matches ARP scan results)
- Open port responded with SYN/ACK; closed port responded with RST
- Either response confirms host is alive

**Why It Matters:** TCP-based discovery is essential in environments where ICMP is entirely blocked. Using well-known service ports (22, 80, 443) maximises the probability of getting a response from hosts that have services running.

---

### Activity 5 — TCP ACK Ping Scan

**Objective:** Use TCP ACK packets for host discovery, particularly useful against firewalls that allow ACK traffic.

**Commands:**
```bash
sudo nmap -PA -sn 10.200.6.0/24
sudo nmap -PA22,80,443 -sn 10.200.6.0/30
```

**Findings:**
- Two hosts confirmed as alive
- ACK scan produced consistent results with SYN scan

**Why It Matters:** Some stateless firewalls allow ACK packets through while blocking SYN packets (a misconfiguration — they should both be blocked for uninitiated connections). ACK ping exploits this, providing discovery capability where SYN ping fails.

---

### Activity 6 — UDP Ping Scan

**Objective:** Use UDP packets to trigger ICMP Port Unreachable responses from closed UDP ports, confirming host liveness.

**Commands:**
```bash
sudo nmap -PU53,161,162 -sn 10.200.6.0/30
```

**Findings:**
- Two hosts discovered
- Closed UDP ports on live hosts returned ICMP Port Unreachable responses

**Why It Matters:** UDP discovery is the last resort when all TCP and ICMP methods are blocked. DNS (53) and SNMP (161/162) are chosen because they are common UDP services — probing them maximises response likelihood.

---

## Observations and Analysis

- **ARP vs ICMP discrepancy:** When an ARP scan finds hosts that an ICMP scan misses, those hosts are specifically blocking ICMP Echo. This is itself security-relevant information — the target system or an intermediate device is filtering ICMP.

- **ICMP Address Mask scan returns nothing:** The `-PM` scan found no hosts despite ARP confirming hosts are alive. This directly demonstrates that modern operating systems and firewalls do not respond to ICMP Type 17 Address Mask requests. This packet type is considered obsolete and is actively dropped.

- **Privileged vs unprivileged behaviour:** Without `sudo`, Nmap falls back to TCP 3-way handshake completion for host discovery. This is more detectable and slower, but it is the only option available to non-privileged users. Always run Nmap as root or with `sudo` for production scans.

- **`-sn` is critical:** Without `-sn`, host discovery is immediately followed by port scanning of discovered hosts. During the scoping phase of an engagement, this can trigger unexpected scan noise. Always use `-sn` when the objective is only to identify live hosts.

- **Masscan comparison:** Masscan can perform similar host/port discovery but is significantly more aggressive with packet rates. The syntax is similar (e.g., `masscan 10.200.6.0/24 -p443`), but its speed makes it more detectable and more likely to cause disruption. Nmap is preferred for controlled assessments.

- **Dry run with `-sL`:** Running `nmap -sL TARGET_RANGE` before any active scan is a valuable practice. It previews the target list, triggers reverse DNS lookups, and helps confirm scope before sending any discovery packets to the target.

---

## Tools and Technologies Used

### Nmap (Network Mapper)
- **Purpose:** Industry-standard network mapping, host discovery, port scanning, service enumeration, and scripting
- **Common Usage:** `nmap [OPTIONS] TARGET`
- **In This Room:** Host discovery phase — ARP, ICMP, TCP, and UDP probe types to identify live hosts

### Masscan
- **Purpose:** Ultra-fast internet-scale port/host scanner
- **Common Usage:** `masscan TARGET -p PORT`
- **In This Room:** Referenced as a high-speed alternative to Nmap; syntax is similar but packet rate is far more aggressive

---

## Key Learnings

- ARP scanning is the most reliable discovery method on a local subnet — it cannot be blocked by host-based firewalls because it operates at Layer 2
- ARP cannot cross routers — it is bounded to the local subnet
- ICMP Echo (`-PE`) is frequently blocked by host firewalls (Windows defaults) and edge firewalls
- Alternative ICMP types (Timestamp `-PP`, Address Mask `-PM`) may succeed when Echo fails — but `-PM` is increasingly blocked
- TCP SYN (`-PS`) and ACK (`-PA`) ping scans work when ICMP is entirely blocked
- UDP ping (`-PU`) is the last resort — relies on ICMP Port Unreachable from closed UDP ports
- Always use `-sn` for host-discovery-only scans to prevent automatic port scanning
- `-n` disables DNS lookups for faster, quieter scans; `-R` forces reverse DNS for all hosts
- Privileged users (root/sudo) have access to all scan types; unprivileged users fall back to TCP 3-way handshake

---

## Real-World Relevance

**Penetration Testing:** Host discovery is the foundation of every engagement scope validation. Running multiple discovery techniques (ARP on local, ICMP+TCP on remote) ensures the live host list is complete before investing time in port scanning.

**Internal Assessments:** Post-exploitation ARP scans map the compromised network segment quickly and reliably without the noise of full port scans. ARP results feed lateral movement planning.

**Red Team Operations:** `-sn` scans with timing control (`-T1`) and targeted probe types keep discovery quiet. A single slow ARP sweep generates far less noise than a full TCP connect scan.

**Enterprise Security:** Security teams monitor for ARP sweep patterns on internal networks as an indicator of post-exploitation activity. Understanding what legitimate and malicious discovery looks like from both sides improves both detection and evasion capability.

---

## Things Worth Remembering

**Host Discovery Quick Reference:**

| Scan Type | Command | Protocol/Layer |
|-----------|---------|---------------|
| ARP Scan | `sudo nmap -PR -sn 10.0.0.0/24` | Layer 2 — local subnet only |
| ICMP Echo | `sudo nmap -PE -sn 10.0.0.0/24` | Network layer |
| ICMP Timestamp | `sudo nmap -PP -sn 10.0.0.0/24` | Network layer |
| ICMP Address Mask | `sudo nmap -PM -sn 10.0.0.0/24` | Network layer |
| TCP SYN Ping | `sudo nmap -PS22,80,443 -sn 10.0.0.0/30` | Transport layer |
| TCP ACK Ping | `sudo nmap -PA22,80,443 -sn 10.0.0.0/30` | Transport layer |
| UDP Ping | `sudo nmap -PU53,161,162 -sn 10.0.0.0/30` | Transport layer |

**Key Options:**

| Option | Purpose |
|--------|---------|
| `-sn` | Host discovery only — suppress port scanning |
| `-n` | No DNS lookup — faster, quieter |
| `-R` | Reverse DNS for all hosts (including offline) |
| `-sL` | List targets only — dry run, no packets sent |
| `-iL file.txt` | Read targets from file |
| `--dns-servers DNS` | Use specific DNS server |

**Key concepts:**
- ARP = most reliable on LAN; cannot cross routers
- Multiple ICMP types exist because individual types may be selectively blocked
- TCP/UDP probes work when ICMP is completely blocked
- Always `sudo` for best scan capability
- Use `-sn` during scope definition to avoid premature port scanning

---

## Conclusion

Nmap Live Host Discovery establishes the foundational skill of determining which hosts on a network are actually online before any deeper investigation begins. By understanding how ARP, ICMP, TCP, and UDP operate at different network layers — and why each may succeed or fail depending on firewall rules, network position, and host configuration — a penetration tester can reliably map the live attack surface even in environments with partial ICMP or TCP filtering. The ability to choose and interpret the right discovery technique for the right situation is what separates methodical network enumeration from brute-force scanning that wastes time and generates unnecessary detection risk.
