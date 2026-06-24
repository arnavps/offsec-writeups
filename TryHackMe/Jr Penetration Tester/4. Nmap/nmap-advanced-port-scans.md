# Nmap Advanced Port Scans

## Overview

Beyond basic SYN and Connect scans, Nmap provides specialised scan types designed for specific purposes: evading firewalls, bypassing IDS detection, mapping firewall rule sets, and obscuring the true source of a scan. This room covers the advanced TCP scan types (Null, FIN, Xmas, Maimon, ACK, Window, Custom), evasion and spoofing techniques (IP spoofing, MAC spoofing, decoy scans, fragmented packets), and the idle/zombie scan for stealthy port discovery.

**Main objectives:**
- Understand how TCP flag combinations other than SYN produce different responses from targets
- Use ACK and Window scans to map firewall rules
- Apply spoofing, decoys, fragmentation, and idle scanning for evasion
- Understand the diagnostic output options: `--reason`, `-v`, `-vv`, `-d`

**Skills introduced:** Null/FIN/Xmas/Maimon/ACK/Window scan types, custom flag combinations, IP/MAC spoofing, decoy scans, packet fragmentation, idle (zombie) scans

---

## Concepts Covered

### Null Scan (`-sN`)

**Definition:** Sends a TCP packet with no flags set — all six flag bits are zero.

**Why It Matters:** RFC 793 specifies that a TCP segment with unexpected flags to an open port should be silently dropped. A closed port should return RST. By exploiting this distinction, Null scan can infer port state without the SYN flag that most firewalls specifically watch for.

**Key Details:**
- Open port: no response (packet is silently dropped)
- Closed port: RST response
- From Nmap's perspective: no response = `open|filtered`; RST = `closed`
- Cannot distinguish between open and filtered — both produce no response
- Works against RFC 793-compliant implementations; does not work against Windows systems (Windows returns RST for all unexpected flags)
- Requires root/sudo privileges

**Practical Relevance:** Null scan is useful against systems where SYN scan is specifically blocked but arbitrary TCP flags are not. Combined with FIN or Xmas, it can identify ports that respond differently to different flag combinations.

---

### FIN Scan (`-sF`)

**Definition:** Sends a TCP packet with only the FIN flag set — indicating "no more data to send."

**Why It Matters:** Like Null scan, FIN exploits the RFC 793 distinction between open (no response) and closed (RST response) port behaviour. FIN packets are less commonly filtered than SYN packets because they appear to be connection teardown traffic.

**Key Details:**
- Open port: no response → `open|filtered`
- Closed port: RST response → `closed`
- Some firewalls silently drop FIN packets without sending RST — these appear as `open|filtered`
- Requires root/sudo privileges
- Does not work on Windows (RST returned for all unexpected flags regardless of port state)

---

### Xmas Scan (`-sX`)

**Definition:** Sets the FIN, PSH, and URG flags simultaneously — a pattern that "lights up like a Christmas tree" in packet analysis tools.

**Why It Matters:** Xmas scan is another RFC 793-based probe. It is sometimes effective against stateless firewalls that inspect individual packets rather than connection state.

**Key Details:**
- Open port: no response → `open|filtered`
- Closed port: RST response → `closed`
- Sets FIN + PSH + URG flags together
- Same limitations as Null and FIN: does not work against Windows systems
- Unusual flag combination may trigger IDS signatures despite bypassing some firewalls

---

### Maimon Scan (`-sM`)

**Definition:** Sets the FIN and ACK flags together. Originally described by Uriel Maimon in 1996, this scan exploits a specific behaviour in BSD-derived systems where open ports drop the packet rather than responding.

**Why It Matters:** Historical and educational — demonstrates how OS-specific TCP implementation quirks create unique fingerprinting and scanning opportunities.

**Key Details:**
- BSD-derived systems (specific versions): open ports drop the packet → `open|filtered`
- Most modern systems: RST returned regardless of port state → scan does not differentiate
- Limited practical value on modern networks
- Valuable for understanding how protocol implementation differences create security implications
- Requires root/sudo privileges

---

### TCP ACK Scan (`-sA`)

**Definition:** Sends a TCP packet with only the ACK flag set. The target returns RST regardless of whether the port is open or closed — the goal is not to identify open ports but to map firewall rules.

**Why It Matters:** ACK scan changes the investigation goal from "which ports are open?" to "which ports are not blocked by the firewall?" This is a fundamentally different and valuable question in penetration testing.

**Key Details:**
- Both open AND closed ports return RST → port state cannot be determined
- Firewall-blocked ports return no response (or ICMP unreachable) → appear as `filtered`
- Unblocked ports (regardless of open/closed) return RST → appear as `unfiltered`
- By comparing filtered vs unfiltered results, the firewall rule set can be inferred
- Critical caveat: `unfiltered` does not mean a service is listening — it only means the firewall is not blocking that port
- Requires root/sudo privileges

**Practical Relevance:** ACK scanning is one of the most valuable advanced techniques for understanding the defensive architecture around a target. Knowing which ports the firewall allows through informs subsequent scan and exploitation strategy.

---

### Window Scan (`-sW`)

**Definition:** Nearly identical to ACK scan but examines the TCP Window field value in the RST responses. On certain systems, the TCP Window value in RST responses differs between open and closed ports, revealing port state that ACK scan cannot determine.

**Why It Matters:** Window scan extracts more information than ACK scan on systems where TCP Window values in RST responses differ by port state. Against a firewall, it can identify which ports are not blocked while simultaneously inferring whether a service is listening.

**Key Details:**
- All ports return RST (same as ACK scan)
- On specific systems: RST for open port has a non-zero TCP Window value; RST for closed port has zero
- On Linux without a firewall: Window scan typically returns the same result as ACK scan (no differentiation)
- Against a firewall: ports that respond may show different Window values, revealing open vs closed distinction
- Results vary significantly by target OS and configuration — treat results as indicative, not definitive
- Requires root/sudo privileges

---

### Custom Scan (`--scanflags`)

**Definition:** Allows specifying any arbitrary combination of TCP flags for a custom probe packet.

**Why It Matters:** Enables crafting unique TCP flag combinations not covered by Nmap's built-in scan types. Useful for testing specific firewall rules, researching edge cases, or reproducing documented vulnerabilities.

**Key Details:**
- Syntax: `--scanflags FLAGNAMES` where flag names are combined: `RSTSYNFIN`, `URGACKPSHRSTSYNFIN`
- Must understand the target's expected behaviour to interpret results correctly
- Combined with knowledge from ACK/Window scans to craft targeted probes

**Key limitation:** ACK and Window scans reveal firewall rules, not services. Even if a firewall is not blocking a port, a service must be listening for exploitation to be possible. Always follow up with service detection (`-sV`) after mapping firewall rules.

---

### IP Address Spoofing (`-S`)

**Definition:** Forges the source IP address in all scan packets to appear as though the scan originates from a different host.

**Why It Matters:** If successful, the target logs the spoofed IP rather than the attacker's real IP. However, this technique has significant practical limitations.

**Key Details:**
- Syntax: `nmap -S SPOOFED_IP TARGET`
- Full command: `nmap -e NET_INTERFACE -Pn -S SPOOFED_IP TARGET`
  - `-e` specifies which network interface to use
  - `-Pn` disables ping (can't receive ping replies with spoofed IP)
- The scan is only useful if the attacker can monitor network traffic for responses
- Responses from the target are sent to the SPOOFED_IP — the attacker never sees them without network visibility
- Impractical in most real-world scenarios without access to the network path

**Three steps of IP spoofing:**
1. Attacker sends packet with spoofed source IP to target
2. Target replies to the spoofed IP address
3. Attacker must capture these replies (requires network visibility or a positioned MITM)

---

### MAC Address Spoofing (`--spoof-mac`)

**Definition:** Forges the source MAC address in Ethernet frames.

**Why It Matters:** MAC-based access controls (802.1X, MAC address filtering) can be bypassed when an attacker spoofs an authorised MAC address.

**Key Details:**
- Syntax: `--spoof-mac SPOOFED_MAC`
- Only works when attacker and target are on the same Ethernet (802.3) or Wi-Fi (802.11) network
- MAC addresses do not cross routers — only relevant for local network segments

---

### Decoy Scan (`-D`)

**Definition:** Mixes the attacker's real IP among multiple fake (decoy) IP addresses in scan traffic, making it harder for the target to identify which source is the real attacker.

**Why It Matters:** Decoy scanning increases the difficulty of attributing the scan to the real attacker. The target receives scan traffic from multiple sources simultaneously — the attacker's IP is one among many.

**Key Details:**
- Syntax: `nmap -D DECOY_IP1,DECOY_IP2,ME TARGET`
- `ME` represents the attacker's real IP address — its position in the list determines its sequence
- `RND` generates a random IP: `nmap -D 10.10.0.1,RND,RND,ME TARGET`
- The target logs traffic from all specified IPs (decoys + real attacker)
- Defenders must determine which of the sources was the actual scanner
- Decoy IPs should ideally be IPs that are actually online to prevent ICMP unreachable responses revealing the decoy is not real

---

### Fragmented Packets (`-f`, `-ff`)

**Definition:** Splits TCP/IP packets into smaller IP fragments to potentially evade firewalls and IDS signatures that inspect full packets.

**Why It Matters:** Some older or misconfigured firewalls and IDS systems inspect individual fragments rather than reassembling them. Fragments may individually appear harmless, bypass signature detection, and be reassembled by the target after passing the security device.

**Key Details:**
- `-f` — split IP data into 8-byte fragments
- `-ff` (or `-f -f`) — split into 16-byte fragments
- `--mtu VALUE` — custom fragment size (must be a multiple of 8)
- The 24-byte TCP header with `-f` splits into 3 fragments of 8 bytes each (3 × 8 = 24)
- Fragment identification (IP ID field) and fragment offset allow reassembly at the destination
- Modern IDS and NGFWs reassemble fragments before inspection — fragmentation is less effective than it was historically
- `--data-length NUM` — appends random data to packets to increase size and blend with legitimate traffic

**Practical Relevance:** Fragmentation is included in Nmap for completeness and for testing specific legacy firewall configurations. Modern NGFWs handle fragmentation correctly, so this technique has diminishing practical value against modern targets.

---

### Idle (Zombie) Scan (`-sI`)

**Definition:** An extremely stealthy scan technique that uses a third-party "idle" host (zombie) to perform port discovery on the target. The attacker's real IP never appears in the target's logs.

**Why It Matters:** When implemented correctly, the idle scan is one of the most stealthy techniques available — the target only sees traffic from the zombie host. However, it requires specific conditions to work correctly.

**Key Details:**

**Requirements:**
- An idle (low-traffic) host on the network with a predictable, incrementing IP ID field
- The attacker must be able to communicate with the zombie host
- Syntax: `nmap -sI ZOMBIE_IP TARGET`

**How it works — three-step process:**

**Step 1:** Attacker probes zombie host to record its current IP ID value.

**Step 2:** Attacker sends a SYN packet to the target port with the zombie's IP as the source address.

**Three scenarios:**
- **Port is open:** Target responds with SYN/ACK to the zombie. Zombie receives unexpected SYN/ACK and responds with RST, incrementing its IP ID.
- **Port is closed:** Target responds with RST to the zombie. Zombie ignores RST — IP ID not incremented.
- **Port is filtered:** Target drops the packet. No response to zombie — IP ID not incremented.

**Step 3:** Attacker probes zombie again to check its new IP ID.
- IP ID increased by 1: port was closed or filtered (zombie only responded to attacker's initial probe)
- IP ID increased by 2: port was open (zombie responded to both attacker's probe AND the target's SYN/ACK)

**Critical requirement:** The zombie must be genuinely idle — if it has significant traffic, IP ID values will be unpredictable and the scan results will be meaningless.

**Practical Relevance:** The idle scan concept is technically elegant and theoretically powerful. In practice, modern networks make finding a suitable zombie host with predictable IP IDs increasingly difficult. Modern TCP implementations use randomised IP IDs rather than incrementing values. However, understanding the idle scan builds critical knowledge about IP ID fields, covert channel techniques, and how IP-based trust relationships can be exploited.

---

## Methodology

### Firewall Mapping Workflow

```
Initial SYN scan to identify obviously open and closed ports
        |
        v
Run ACK scan (-sA) to identify firewall-unfiltered ports
  Unfiltered = firewall is not blocking this port
  Filtered = firewall is blocking
        |
        v
Run Window scan (-sW) on same ports
  Compare with ACK results:
  Port was unfiltered in ACK but shows open in Window = likely service running
  Port was unfiltered in ACK and shows closed in Window = port accessible but no service
        |
        v
Identify gap between firewall-open ports and service-running ports
(firewall rules may not reflect current service state)
        |
        v
Run SYN scan (-sS) on firewall-open ports to confirm service presence
Run version detection (-sV) to identify specific services
```

### Evasion Decision Tree

```
Standard SYN scan detected or blocked?
        |
        ├── Try fragmented packets: -f or -ff
        |
        ├── Try decoy scan: -D DECOY1,DECOY2,ME
        |
        ├── Try source port manipulation: --source-port PORT
        |
        ├── Try slow timing: -T1 with --max-rate 5
        |
        └── Try idle scan: -sI ZOMBIE_IP
            (requires suitable idle host with predictable IP ID)
```

---

## Practical Activities

### Activity 1 — Null, FIN, Xmas Scans

**Objective:** Probe target ports using abnormal TCP flag combinations to identify open|filtered vs closed ports.

**Commands:**
```bash
# Null scan (no flags)
sudo nmap -sN 10.48.139.207

# FIN scan (FIN flag only)
sudo nmap -sF 10.48.139.207

# Xmas scan (FIN + PSH + URG)
sudo nmap -sX 10.48.139.207
```

**Findings:**
- Closed ports return RST → confirmed closed
- Open or filtered ports return nothing → reported as `open|filtered`
- Cannot distinguish between genuinely open and firewall-filtered ports

**Why It Matters:** These scans bypass simple SYN-specific filters and provide an alternative view of port state. Ports that show `open|filtered` here are candidates for SYN scan confirmation and service detection.

---

### Activity 2 — ACK and Window Scans

**Objective:** Map firewall rules by observing which ports return RST (unfiltered by firewall) vs no response (filtered).

**Commands:**
```bash
# ACK scan
sudo nmap -sA 10.48.139.207

# Window scan
sudo nmap -sW 10.48.139.207
```

**Findings:**
- ACK scan shows `unfiltered` for ports not blocked by firewall, `filtered` for blocked ports
- Window scan may differentiate between open and closed among the unfiltered ports on certain systems
- Against a Linux system without a firewall: both scans show no differentiation
- Against a system behind a firewall: five ports identified as `unfiltered` (firewall not blocking them)

**Why It Matters:** ACK scan reveals the firewall's perspective. Knowing which ports the firewall allows through guides subsequent exploitation strategy — focus effort on ports the firewall permits.

---

### Activity 3 — Fragmented Packet Scan

**Objective:** Fragment SYN scan packets to test firewall/IDS evasion.

**Commands:**
```bash
# 8-byte fragments (SYN scan on port 80)
sudo nmap -sS -p80 -f TARGET_IP

# 16-byte fragments
sudo nmap -sS -p80 -ff TARGET_IP
```

**Command Explanation:**
- `-f` fragments the TCP header into 8-byte chunks; a 24-byte TCP header becomes 3 fragments
- Each fragment has its own 20-byte IP header
- The IP ID and fragment offset fields allow the target to reassemble them
- Wireshark shows the fragments as separate packets before reassembly

**Why It Matters:** Fragments may individually appear benign. Some older security devices inspect fragments individually rather than reassembling — allowing the probe to pass.

---

### Activity 4 — Decoy Scan

**Objective:** Obscure the true scanner IP among multiple decoy sources.

**Commands:**
```bash
# Specific decoy IPs with ME indicating real position
nmap -D 10.10.0.1,10.10.0.2,ME 10.48.139.207

# Random decoys + ME at fifth position
nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME 10.48.139.207
```

**Command Explanation:**
- `-D` specifies a comma-separated list of decoy IPs
- `ME` marks the position of the attacker's real IP
- `RND` generates a random IP for that position
- The target receives scan traffic from all listed sources simultaneously

**Why It Matters:** In a forensic investigation, the target would need to determine which of the five IP addresses was the actual scanner — significantly increasing attribution difficulty.

---

### Activity 5 — Verbose and Reason Flags

**Objective:** Obtain more detailed output explaining Nmap's conclusions.

**Commands:**
```bash
# Reason output
sudo nmap -sS --reason TARGET_IP

# Verbose
sudo nmap -sS -v TARGET_IP

# Very verbose
sudo nmap -sS -vv TARGET_IP

# Debug
sudo nmap -sS -d TARGET_IP
```

**Findings:**
- `--reason`: shows "syn-ack" for open TCP ports, "arp-response" for hosts confirmed via ARP
- `-v`/`-vv`: shows ports as they are discovered in real time rather than waiting for full scan completion
- `-d`/`-dd`: extremely detailed packet-level debugging output

**Why It Matters:** `--reason` is invaluable for understanding ambiguous results and for documentation. `-vv` allows monitoring scan progress in real time, which is useful for long scans.

---

## Observations and Analysis

- **Null/FIN/Xmas don't work against Windows:** Windows TCP implementations return RST for all unexpected flag combinations regardless of port state — these scans cannot differentiate open from closed on Windows targets. Check the OS before choosing scan type.

- **ACK scan changes the question:** The critical shift in ACK scan is moving from "what is running?" to "what can I reach through the firewall?" These are different questions with different strategic implications.

- **Firewall rules may not match reality:** The observation that firewall-unfiltered ports don't necessarily have services is an important real-world nuance. Firewall rules are often stale — they permit traffic for services that have been decommissioned. ACK/Window scans can reveal this gap.

- **Idle scan practicality:** Most modern TCP implementations use randomised IP IDs (as a mitigation against the idle scan technique itself). Finding a suitable zombie with a predictably incrementing IP ID is increasingly difficult on modern networks. The technique remains valuable as a conceptual foundation for understanding covert channel attacks.

- **Fragmentation and modern IDS:** Modern NGFW and IDS solutions reassemble IP fragments before inspection. Fragmentation is far more effective against older stateless packet filters than against modern deep-packet inspection systems. Don't rely on it as a primary evasion technique.

- **`--reason` for scan validation:** During real assessments, `--reason` helps validate that open port conclusions are based on SYN/ACK responses rather than assumptions. This is particularly useful when dealing with filtered/ambiguous results.

---

## Tools and Technologies Used

### Nmap
- **Purpose:** Advanced port scanning with evasion capabilities
- **Common Usage:** `sudo nmap [ADVANCED_FLAGS] TARGET`
- **In This Room:** Null/FIN/Xmas/Maimon/ACK/Window scans, decoy scanning, fragmentation, IP spoofing, idle scan, verbose output

---

## Key Learnings

- Null (`-sN`), FIN (`-sF`), Xmas (`-sX`) scan open ports produce no response; closed ports return RST — exploits RFC 793
- These scans do not work against Windows (RST returned for all unexpected flags)
- Maimon (`-sM`) is historically significant but rarely effective on modern systems
- ACK scan (`-sA`) maps firewall rules, not open ports — `unfiltered` means firewall not blocking, not that a service is running
- Window scan (`-sW`) extends ACK scan by examining TCP Window values in RST responses
- IP spoofing (`-S`) requires network visibility to receive responses — impractical without it
- Decoy scan (`-D`) mixes attacker IP among decoys — increases attribution difficulty
- Fragmentation (`-f`/`-ff`) may evade legacy stateless packet filters but not modern NGFWs
- Idle scan (`-sI`) uses zombie host's IP ID increments to infer target port state — extremely stealthy but requires ideal conditions
- `--reason` shows evidence for each conclusion; `-vv` shows real-time scan progress

---

## Real-World Relevance

**Penetration Testing:** ACK and Window scans are standard when a target appears to have a firewall blocking most ports — they reveal what the firewall permits, guiding the subsequent SYN scan. Decoy scans are used when attribution avoidance is a requirement of the engagement.

**Red Team Operations:** Fragmented packets, custom source ports, and slow timing are combined to evade IDS signatures. The idle scan, when a suitable zombie exists, is the gold standard for attribution-free port discovery.

**Firewall Assessment:** ACK scan output directly answers "what does this firewall's rule set permit?" — a question that is central to network security architecture reviews.

**Security Research:** Custom flag combinations (`--scanflags`) are used to test how specific systems respond to unexpected TCP states — useful for fingerprinting and vulnerability research.

---

## Things Worth Remembering

**Advanced scan commands:**
```bash
sudo nmap -sN TARGET   # Null (no flags)
sudo nmap -sF TARGET   # FIN
sudo nmap -sX TARGET   # Xmas (FIN+PSH+URG)
sudo nmap -sM TARGET   # Maimon (FIN+ACK)
sudo nmap -sA TARGET   # ACK (firewall mapping)
sudo nmap -sW TARGET   # Window (ACK + TCP Window analysis)
sudo nmap --scanflags SYNFIN TARGET  # Custom flags
```

**Evasion commands:**
```bash
sudo nmap -sS -f TARGET                    # Fragmented (8-byte)
sudo nmap -sS -ff TARGET                   # Fragmented (16-byte)
nmap -D DECOY1,DECOY2,ME TARGET            # Decoy scan
sudo nmap -S SPOOFED_IP -e eth0 -Pn TARGET # IP spoofing
sudo nmap -sI ZOMBIE_IP TARGET             # Idle/zombie scan
```

**Output options:**
```bash
--reason   # Show evidence for each conclusion
-v         # Verbose
-vv        # Very verbose
-d         # Debug
-dd        # Full debug
```

**Key insight table:**

| Scan | What it reveals | Key behaviour |
|------|----------------|---------------|
| Null/FIN/Xmas | Open\|filtered vs closed | No response = open\|filtered; RST = closed |
| ACK | Firewall rules | RST = unfiltered; nothing = filtered |
| Window | Firewall + open/closed | Examines TCP Window value in RST |
| Idle | Port state without attribution | Uses zombie's IP ID increment |

---

## Conclusion

Nmap Advanced Port Scans transforms the toolkit from straightforward discovery into a nuanced instrument for firewall mapping, evasion, and stealthy reconnaissance. The shift from "which ports are open?" (SYN scan) to "which ports does the firewall permit?" (ACK scan) represents a fundamental change in investigative perspective that is directly applicable in real penetration tests. The evasion techniques — spoofing, decoys, fragmentation, and idle scanning — are progressively more complex but each addresses specific detection mechanisms. Understanding when and why to use each technique, rather than applying them indiscriminately, is the mark of a methodical penetration tester.
