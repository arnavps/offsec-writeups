# Nmap Basic Port Scans

## Overview

With live hosts identified, the next step in any penetration test is determining which services are running on those hosts. Port scanning maps open, closed, and filtered ports — the open ones indicate running services that can be enumerated, fingerprinted, and tested for vulnerabilities. This room introduces the three fundamental Nmap port scan types: TCP Connect, TCP SYN, and UDP, along with port selection, timing controls, and parallelisation options.

**Main objectives:**
- Understand the six possible port states Nmap identifies
- Learn how TCP Connect, TCP SYN, and UDP scans work at the protocol level
- Understand the TCP header and its flags as the foundation of TCP scan types
- Control scan scope with port selection options
- Control scan speed and stealth with timing and parallelisation options

**Skills introduced:** TCP Connect scan (`-sT`), TCP SYN scan (`-sS`), UDP scan (`-sU`), port selection, timing templates, rate limiting, probe parallelisation

---

## Concepts Covered

### The Six Port States

**Definition:** Nmap classifies every probed port into one of six states based on how the target responds (or fails to respond) to the probe.

**Why It Matters:** Open ports are the primary finding of port scanning — they indicate services to investigate further. Filtered ports indicate firewall intervention. Understanding all six states prevents misinterpreting scan results.

**Key Details:**

| State | Meaning |
|-------|---------|
| **open** | A service is actively listening on this port |
| **closed** | No service is listening; the port is accessible (not blocked by a firewall) |
| **filtered** | Nmap cannot determine open/closed — packets are being dropped by a firewall |
| **unfiltered** | Accessible but Nmap cannot determine open/closed (seen with ACK scan `-sA`) |
| **open\|filtered** | Nmap cannot distinguish between open and filtered |
| **closed\|filtered** | Nmap cannot distinguish between closed and filtered |

**Practical Relevance:** The most common states encountered are `open` (find it, investigate), `closed` (no service, skip), and `filtered` (firewall present — try different scan types or evasion). `filtered` ports are particularly significant because they reveal firewall rules worth mapping.

---

### TCP Header and Flags

**Definition:** The TCP header is the first 24 bytes of a TCP segment. It contains source and destination ports, sequence/acknowledgement numbers, and six flag bits that control connection state.

**Why It Matters:** All TCP scan types work by crafting specific flag combinations. Understanding the flags explains why different scan types produce different responses.

**Key Details — TCP flags (left to right):**

| Flag | Meaning |
|------|---------|
| **URG** | Urgent pointer field is significant; process immediately |
| **ACK** | Acknowledges received data |
| **PSH** | Push data to application promptly |
| **RST** | Reset the connection; sent for unexpected packets or when no service is listening |
| **SYN** | Synchronise sequence numbers; initiates TCP 3-way handshake |
| **FIN** | No more data to send; graceful connection close |

**Critical flags for port scanning:**
- **SYN** starts connections — sent by client to open a port
- **SYN/ACK** is returned by an open port to the SYN
- **RST** is returned by a closed port to a SYN, or by any system to an unexpected packet
- **ACK** acknowledges data receipt

**Practical Relevance:** Knowing how RST, SYN/ACK, and RST/ACK behave under different scenarios is the foundation for understanding why FIN, Xmas, and Null scans work differently (Advanced Port Scans room).

---

### TCP Connect Scan (`-sT`)

**Definition:** TCP Connect scan completes the full TCP 3-way handshake (SYN → SYN/ACK → ACK) to determine whether a port is open. After connection confirmation, it immediately tears down the connection by sending RST/ACK.

**Why It Matters:** TCP Connect is the only scan type available to unprivileged (non-root) users. It is the most compatible scan — any system capable of making TCP connections can run it.

**Key Details:**
- Completes the full 3-way handshake: SYN → SYN/ACK → ACK → RST/ACK
- Open port: responds with SYN/ACK; connection established then torn down
- Closed port: responds with RST; no connection established
- Requires no special privileges
- More detectable than SYN scan — full connection appears in server logs
- Slower than SYN scan due to connection completion overhead

**When to use:** When running as an unprivileged user, or when the target environment requires full connection establishment to trigger accurate responses.

---

### TCP SYN Scan (`-sS`)

**Definition:** TCP SYN scan (also called "half-open" or "stealth" scan) sends a SYN packet and reads the response without completing the 3-way handshake. After receiving SYN/ACK, Nmap sends RST to tear down the half-open connection.

**Why It Matters:** SYN scan is faster, less likely to be logged (no full connection established), and is the default scan for privileged users. It is the most widely used Nmap scan type in practice.

**Key Details:**
- Sends SYN; receives SYN/ACK (open) or RST (closed) or no response (filtered)
- Does NOT complete the TCP handshake — sends RST immediately after SYN/ACK
- Requires root/sudo privileges
- Default scan mode when running as privileged user
- "Stealthy" because most logging occurs at full connection establishment — a half-open connection may not be logged by some services
- Faster than `-sT` because it does not wait for connection completion

**SYN vs Connect scan comparison:**
- TCP Connect (`-sT`): SYN → SYN/ACK → ACK → RST/ACK (4 packets, full connection)
- TCP SYN (`-sS`): SYN → SYN/ACK → RST (3 packets, half-open)

**Practical Relevance:** SYN scan is the standard choice for all penetration testing contexts where root access is available. It is reliable, fast, and produces cleaner results with less log pollution on the target.

---

### UDP Scan (`-sU`)

**Definition:** UDP is connectionless — there is no handshake, and a service may not respond to a UDP packet at all. Nmap infers port state by observing whether closed ports return ICMP Port Unreachable errors.

**Why It Matters:** Many important services run on UDP: DNS (53), DHCP (67/68), SNMP (161/162), TFTP (69), NTP (123). These services are invisible to TCP-only scans. Missing UDP services can mean missing critical attack surface.

**Key Details:**
- Sending to an **open UDP port**: service may respond (if it does, the port is clearly open), or may not respond at all (open|filtered)
- Sending to a **closed UDP port**: target returns ICMP Type 3 Code 3 (Port Unreachable) → port confirmed closed
- Open UDP ports are identified by NOT receiving an ICMP Port Unreachable response
- UDP scans are significantly slower than TCP scans — rate limiting is common
- Can be combined with TCP scan types in the same command: `nmap -sU -sS TARGET`
- `--top-ports 10` scans only the 10 most common ports — useful for quick initial UDP assessment

**Practical Relevance:** SNMP (161) in particular is a high-value UDP target — community strings "public" and "private" often provide device configuration data. DNS (53) may allow zone transfers. Always include UDP in comprehensive assessments.

---

### Port Selection

**Definition:** Controls which ports Nmap scans rather than scanning all 65,535 ports by default.

**Why It Matters:** Scanning all ports is thorough but slow. Selecting specific ports or the top N most common ports allows efficient targeted scanning when time is limited.

**Key Details:**

| Option | Effect |
|--------|--------|
| `-p22,80,443` | Scan ports 22, 80, and 443 only |
| `-p1-1023` | Scan ports 1 through 1023 |
| `-p20-25` | Scan ports 20, 21, 22, 23, 24, 25 |
| `-p-` | Scan all 65,535 ports |
| `-F` | Scan 100 most common ports |
| `--top-ports 10` | Scan 10 most common ports |
| `-r` | Scan ports in sequential order (default is random) |

---

### Timing Templates

**Definition:** Nmap provides six timing templates that control the speed and aggressiveness of scans.

**Why It Matters:** Faster scans are more detectable and may produce less accurate results due to packet loss. Slower scans are quieter but impractical for large engagements. The right timing depends on the engagement context.

**Key Details:**

| Template | Speed | Notes |
|---------|-------|-------|
| `-T0` (paranoid) | Slowest | 1 port at a time, 5 minutes between probes. Used for extreme stealth |
| `-T1` (sneaky) | Very slow | Designed for IDS evasion in real engagements |
| `-T2` (polite) | Slow | Reduces load on network/target |
| `-T3` (normal) | Default | Balanced speed and accuracy |
| `-T4` (aggressive) | Fast | Commonly used in CTFs and practice environments |
| `-T5` (insane) | Fastest | May lose accuracy due to packet loss |

**Practical guidance:**
- Real engagements where stealth matters: `-T1`
- CTFs and practice targets: `-T4`
- Default (no flag): `-T3`
- Avoid `-T5` in real engagements — packet loss reduces scan accuracy

---

### Rate Limiting and Parallelisation

**Definition:** Fine-grained controls for packet send rate and probe parallelisation, complementing the broad timing templates.

**Why It Matters:** Rate controls allow precise tuning beyond what timing templates provide — useful when bandwidth is limited, when trying to stay below IDS thresholds, or when scanning through slow network connections.

**Key Details:**

| Option | Effect |
|--------|--------|
| `--min-rate 15` | Send at least 15 packets per second |
| `--max-rate 50` | Send at most 50 packets per second |
| `--min-parallelism 100` | Maintain at least 100 probes in parallel |
| `--max-parallelism 100` | Use at most 100 probes in parallel |

Parallelisation controls how many simultaneous probes Nmap maintains — including both host discovery and port probing probes.

---

## Methodology

```
Live host list obtained from host discovery phase
        |
        v
Determine required privilege level:
  Root available → TCP SYN scan (-sS) preferred
  Non-root → TCP Connect scan (-sT) only option
        |
        v
Select port scope:
  Quick scan: -F (100 ports) or --top-ports N
  Comprehensive: -p- (all 65,535 ports)
  Targeted: -p PORT_LIST
        |
        v
Run TCP scan (SYN or Connect)
  sudo nmap -sS TARGET          # preferred
  nmap -sT TARGET               # non-privileged
        |
        v
Run UDP scan for common UDP services
  sudo nmap -sU --top-ports 20 TARGET
        |
        v
Adjust timing for context:
  Engagement stealth: -T1 or --max-rate 10
  Practice/CTF: -T4
  Default: omit (uses T3)
        |
        v
Review open ports → feed into version detection (-sV)
and service enumeration (next rooms)
```

---

## Practical Activities

### Activity 1 — TCP Connect Scan

**Objective:** Perform a full TCP Connect scan to identify open ports and observe the scan mechanics.

**Commands:**
```bash
nmap -sT 10.49.159.30
```

**Command Explanation:**
- `-sT` — TCP Connect scan; completes the full 3-way handshake for each probed port
- Can be run without `sudo` — unprivileged scan
- Nmap completes SYN → SYN/ACK → ACK → RST/ACK for each open port

**Findings:**
- List of open TCP ports with service names
- Full 3-way handshake completion means entries appear in server connection logs

**Why It Matters:** Even though TCP Connect is more detectable, it works without privileges and is compatible with all environments. Understanding it establishes the baseline for understanding why SYN scan is preferred.

---

### Activity 2 — TCP SYN Scan

**Objective:** Perform a TCP SYN (half-open) scan as the primary port discovery method.

**Commands:**
```bash
sudo nmap -sS 10.10.105.229
```

**Command Explanation:**
- `-sS` — TCP SYN scan; sends SYN, reads response, sends RST without completing handshake
- `sudo` required — raw socket access needed for SYN scan
- Default scan type for privileged users

**Findings:**
- Identical open ports as TCP Connect scan
- Scan completes faster
- Connection does not appear in server-side logs in the same way as `-sT`

**Why It Matters:** SYN scan is the standard for penetration testing — faster, less logged, and reliable. The results are equivalent to Connect scan for determining open/closed state, but the mechanism is fundamentally different.

---

### Activity 3 — UDP Scan

**Objective:** Scan the most common UDP ports to identify UDP services running on the target.

**Commands:**
```bash
sudo nmap -sU --top-ports 10 10.49.159.30
```

**Command Explanation:**
- `-sU` — UDP scan
- `--top-ports 10` — scan only the 10 most common UDP ports
- Closed UDP ports return ICMP Port Unreachable; open ports may or may not respond

**Findings:**
- UDP services that are present (open or open|filtered state)
- ICMP Port Unreachable responses confirm closed ports

**Why It Matters:** UDP services are invisible to TCP scans. Finding SNMP (161) or DNS (53) open provides additional attack surface. `open|filtered` state is normal for UDP — the service may simply not respond to the probe packet type used.

---

## Observations and Analysis

- **SYN vs Connect — logging difference:** TCP Connect scan completes the handshake, so modern services (Apache, nginx, SSH) log an incoming connection from the scanner's IP. SYN scan does not complete the connection, so at the application level, no connection event is recorded. However, network-level firewall and IDS logs will still capture the SYN packets.

- **UDP scan slowness:** UDP scans are significantly slower than TCP scans because many systems rate-limit ICMP Port Unreachable responses. This prevents Nmap from flooding the target with excessive UDP probes. Factor this into time planning for comprehensive assessments.

- **`-p-` vs `-F` trade-off:** Full port scans (`-p-`) are essential for finding services on non-standard ports (common in real environments). However, scanning 65,535 ports takes substantially longer than `-F` (100 ports). A common methodology is to run a quick `-F` scan first, then run `-p-` in the background as a comprehensive follow-up.

- **Timing template impact:** `-T5` may miss open ports due to packet loss when scanning across congested networks. In CTF environments with reliable local network connections, `-T4` or `-T5` are acceptable. In real engagements, stick to `-T3` or lower and use `--max-rate` for fine-grained control.

- **Combining scan types:** `-sU` and `-sS` can be run simultaneously: `sudo nmap -sU -sS TARGET`. This is efficient but slower than running each separately. UDP results will take longer to complete.

---

## Tools and Technologies Used

### Nmap
- **Purpose:** Network mapping, port scanning, service detection
- **Common Usage:** `nmap [OPTIONS] TARGET`
- **In This Room:** TCP Connect (`-sT`), TCP SYN (`-sS`), UDP (`-sU`) scans; port selection; timing controls

---

## Key Learnings

- Six port states: `open`, `closed`, `filtered`, `unfiltered`, `open|filtered`, `closed|filtered`
- TCP SYN scan (`-sS`) is the default and preferred scan for privileged users — faster, less logged
- TCP Connect scan (`-sT`) is the only option for unprivileged users — completes full handshake
- UDP scan (`-sU`) is essential for finding DNS, SNMP, DHCP, NTP and other UDP services
- UDP open ports are identified by the ABSENCE of ICMP Port Unreachable responses
- Port selection: `-p-` (all), `-F` (100 common), `--top-ports N`, `-pPORT_LIST`
- Timing: `-T0` (paranoid) to `-T5` (insane); `-T4` for CTFs, `-T1` for stealth engagements
- Rate controls: `--max-rate`, `--min-rate`, `--min-parallelism`, `--max-parallelism`

---

## Real-World Relevance

**Penetration Testing:** TCP SYN scan is run on every engagement. The combination of a quick `-F` scan for initial results and a full `-p-` scan running in the background is standard practice. UDP scan of top ports (DNS, SNMP, NTP) is mandatory for comprehensive coverage.

**Bug Bounty Hunting:** Port scanning in-scope IP ranges to find services on non-standard ports frequently yields high-severity findings — exposed admin panels, development services, or backup servers on high ports.

**Enterprise Security:** Security teams run periodic port scans against their own infrastructure to identify unauthorised services. Comparing current scan results against a baseline detects new listening services that may indicate compromised systems or policy violations.

**Red Team Operations:** SYN scan with `-T1` and `--max-rate 5` is slow but quiet. Combined with source port manipulation (covered in Advanced scans), this minimises IDS detection during initial reconnaissance.

---

## Things Worth Remembering

**Core scan type commands:**
```bash
# TCP Connect (no root needed)
nmap -sT TARGET

# TCP SYN (root required, default for privileged)
sudo nmap -sS TARGET

# UDP (root required)
sudo nmap -sU TARGET

# UDP top 10 ports
sudo nmap -sU --top-ports 10 TARGET

# Combined TCP SYN + UDP
sudo nmap -sS -sU TARGET
```

**Port selection:**
```bash
-p-                    # all 65,535 ports
-p22,80,443            # specific ports
-p1-1023               # range
-F                     # 100 most common
--top-ports 10         # 10 most common
```

**Timing:**
```bash
-T0  # paranoid (5 min between probes)
-T1  # sneaky (real-world stealth)
-T3  # normal (default)
-T4  # aggressive (CTF/practice)
-T5  # insane (may lose accuracy)
--max-rate 50          # cap at 50 packets/sec
--min-parallelism 100  # at least 100 parallel probes
```

**Key concept table:**

| Option | Purpose |
|--------|---------|
| `-sT` | TCP Connect scan |
| `-sS` | TCP SYN scan (default for root) |
| `-sU` | UDP scan |
| `-p-` | All ports |
| `-F` | 100 most common ports |
| `-T<0-5>` | Timing template |
| `--max-rate N` | Limit packets per second |
| `--min-parallelism N` | Minimum parallel probes |

---

## Conclusion

Nmap Basic Port Scans establishes the core scanning capability that every penetration tester uses daily. Understanding the mechanical difference between TCP Connect and TCP SYN scans — and why SYN is preferred — builds the intuition for interpreting scan results accurately. Adding UDP scanning ensures that no service layer is overlooked. Port selection and timing controls transform Nmap from a blunt instrument into a precisely tuned reconnaissance tool suited to the specific constraints of any engagement, whether speed is critical or stealth is paramount.
