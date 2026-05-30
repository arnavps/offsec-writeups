# Nmap: The Basics

## Overview

Nmap (Network Mapper) is the industry-standard open-source network scanner. It discovers live hosts, identifies open ports, detects running services and their versions, and fingerprints operating systems. This room covers the essential Nmap scan types, service detection, timing control, and output formats.

---

## Topics Covered

- Target specification
- Host discovery (`-sn`)
- TCP connect scan (`-sT`)
- SYN scan (`-sS`)
- UDP scan (`-sU`)
- Port range options
- OS and service detection
- Timing templates
- Verbosity and debugging
- Saving scan output

---

## Key Concepts

### Target Specification

```bash
nmap 192.168.0.1              # Single IP
nmap 192.168.0.1-10           # IP range
nmap 192.168.0.0/24           # Subnet
nmap example.thm              # Hostname
nmap -sL 192.168.0.0/24       # List targets without scanning
```

---

### Host Discovery

**`-sn`** — Ping scan. Discovers live hosts without port scanning. Uses ICMP echo, ICMP timestamp, TCP SYN to port 443, and TCP ACK to port 80.

```bash
nmap -sn 192.168.0.0/24
```

---

### Port Scanning

**TCP Connect Scan (`-sT`)**
Completes the full three-way handshake. Works without root privileges. More detectable — connections are logged.

```bash
nmap -sT 10.10.10.10
```

**SYN Scan / Stealth Scan (`-sS`)**
Sends only the SYN packet — never completes the handshake. Fewer logs, considered stealthier. Requires root/sudo.

```bash
sudo nmap -sS 10.10.10.10
```

**UDP Scan (`-sU`)**
Scans UDP ports. Slower than TCP scans. Closed UDP ports return ICMP port unreachable.

```bash
sudo nmap -sU 10.10.10.10
```

---

### Port Range Options

| Option | Description |
|---|---|
| `-F` | Fast mode — scan 100 most common ports |
| `-p 80` | Scan specific port |
| `-p 80,443,8080` | Scan multiple specific ports |
| `-p 1-1024` | Scan port range |
| `-p-` | Scan all 65535 ports |
| `-p1-1023` | Scan well-known ports |

---

### Service and OS Detection

| Option | Description |
|---|---|
| `-sV` | Service and version detection |
| `-O` | OS detection |
| `-A` | OS detection + version detection + traceroute + scripts |
| `-Pn` | Skip host discovery — treat all hosts as online |

```bash
sudo nmap -sV 10.10.10.10
sudo nmap -O 10.10.10.10
sudo nmap -A 10.10.10.10
sudo nmap -Pn 10.10.10.10
```

---

### Timing Templates

| Template | Name | Approximate Speed |
|---|---|---|
| `-T0` | Paranoid | ~9.8 hours for 100 ports |
| `-T1` | Sneaky | ~27 minutes |
| `-T2` | Polite | ~40 seconds |
| `-T3` | Normal (default) | ~0.15 seconds |
| `-T4` | Aggressive | ~0.13 seconds |
| `-T5` | Insane | Fastest, may miss results |

Slower timing reduces the chance of triggering IDS alerts.

**Additional timing options:**

```bash
--min-parallelism <n>     # Minimum parallel probes
--max-parallelism <n>     # Maximum parallel probes
--min-rate <n>            # Minimum packets per second
--max-rate <n>            # Maximum packets per second
--host-timeout <time>     # Maximum time to wait for a host
```

---

### Verbosity and Debugging

```bash
nmap -v 10.10.10.10        # Verbose
nmap -vv 10.10.10.10       # More verbose
nmap -v4 10.10.10.10       # Specify verbosity level
nmap -d 10.10.10.10        # Debug output
nmap -d9 10.10.10.10       # Maximum debug output
```

---

### Saving Output

| Option | Format |
|---|---|
| `-oN filename` | Normal (human-readable) |
| `-oX filename` | XML |
| `-oG filename` | Grepable |
| `-oA basename` | All three formats simultaneously |

```bash
nmap -sS -oA scan_results 10.10.10.10
```

---

## Practical Examples / Demonstrations

### Lab walkthrough

**How many TCP ports are open on 10.10.100.85?**

```bash
nmap -sT 10.10.100.85
# Answer: 6
```

**Find the web server and access it:**

```bash
nmap -sT 10.10.100.85
# Port 8008 open (HTTP)
# Browse to http://10.10.100.85:8008/
# Answer: THM{SECRET_PAGE_38B9P6}
```

---

## Complete Option Reference

| Option | Description |
|---|---|
| `-sL` | List scan — list targets without scanning |
| `-sn` | Ping scan — host discovery only |
| `-sT` | TCP connect scan |
| `-sS` | TCP SYN scan (stealth) |
| `-sU` | UDP scan |
| `-F` | Fast mode — 100 most common ports |
| `-p[range]` | Specify port range |
| `-Pn` | Treat all hosts as online |
| `-O` | OS detection |
| `-sV` | Service version detection |
| `-A` | OS + version + traceroute + scripts |
| `-T<0-5>` | Timing template |
| `-v` | Verbose output |
| `-d` | Debug output |
| `-oN` | Normal output |
| `-oX` | XML output |
| `-oG` | Grepable output |
| `-oA` | All output formats |

---

## Real-World Relevance

- Nmap is the first tool used in almost every penetration test — host discovery and port scanning define the attack surface
- SYN scan (`-sS`) is preferred over connect scan (`-sT`) for stealth — fewer logs on the target
- `-sV` and `-A` reveal service versions — used to identify vulnerable software
- `-O` fingerprints the OS — informs which exploits are applicable
- Timing templates balance speed against detectability — `-T2` or `-T1` for stealthy scans, `-T4` for speed in lab environments
- `-oA` saves results in all formats — XML output is used by vulnerability management tools and frameworks like Metasploit

---

## Key Learnings

- Nmap discovers hosts, open ports, services, and OS in a single tool
- SYN scan is stealthier than connect scan — requires root privileges
- `-A` combines OS detection, version detection, and traceroute
- Timing templates control scan speed vs detectability
- Always save scan results with `-oA` for documentation and later reference
- Run Nmap with sudo to access all scan types — non-root users are limited to connect scans

---

## Conclusion

Nmap is the essential network reconnaissance tool. From discovering live hosts to identifying service versions and OS fingerprints, it provides the information needed to understand a target network's attack surface. Mastering its scan types, timing options, and output formats is a foundational skill for any security professional.
