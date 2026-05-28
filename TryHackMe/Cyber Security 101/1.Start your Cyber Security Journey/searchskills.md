# Search Skills

## Overview

Knowing how to find information efficiently is one of the most underrated skills in cybersecurity. This room covers the tools and resources used to research vulnerabilities, investigate suspicious files, enumerate internet-facing systems, and read technical documentation. These skills apply directly to both offensive reconnaissance and defensive threat intelligence.

---

## Topics Covered

- Shodan — internet-wide scanning and device discovery
- VirusTotal — multi-engine malware and URL analysis
- CVE programme — standardised vulnerability identification
- Technical documentation: man pages and official docs
- GitHub as a security research resource

---

## Key Concepts

### Shodan

Shodan continuously scans the internet and indexes everything with a public network connection — servers, routers, IoT devices, industrial control systems, traffic cameras, and more. It's a search engine for internet-exposed infrastructure.

**Useful query filters:**

| Filter | Description | Example |
|---|---|---|
| `country` | Restrict results to a specific country | `country:IE` |
| `port` | Filter by port number | `port:22` |
| `org` | Scope to a named organisation or ASN | `org:AS7224` |
| `hostname` | Match against a hostname or domain | `hostname:fakebank.thm` |

Shodan is used in reconnaissance to identify exposed services, outdated software versions, and misconfigured devices before (or instead of) active scanning.

---

### VirusTotal

VirusTotal aggregates results from over 70 antivirus engines and website scanners into a single interface. You can submit:

- A file
- A URL
- A domain
- A file hash (MD5, SHA-1, SHA-256)

It returns a verdict from each engine — how many flagged it as malicious and what they called it. Not foolproof (false positives and false negatives exist), but it's a fast, widely-used first check for suspicious files and links.

Used heavily in blue team work for triage and threat intelligence gathering.

---

### CVE Programme

The Common Vulnerabilities and Exposures (CVE) programme is the industry-standard dictionary of known vulnerabilities. Every confirmed vulnerability is assigned a unique identifier:

```
CVE-YEAR-NUMBER
Example: CVE-2025-55182
```

High-impact vulnerabilities sometimes receive a common name (e.g., Heartbleed, Log4Shell, EternalBlue) for easier reference.

Each CVE is scored using **CVSS (Common Vulnerability Scoring System)** based on:

- **Impact** — what damage can the vulnerability cause?
- **Complexity** — how difficult is it to exploit?
- **Availability** — how accessible is the exploit?

CVSS scores range from 0–10. Critical vulnerabilities score 9.0+.

---

### Technical Documentation

**Official documentation** is always the most reliable and up-to-date source for any tool or platform. When troubleshooting or learning how to use a tool in a specific way, check the official docs first — not a random tutorial.

**Linux Man Pages**

Every command on Linux has a manual page accessible directly in the terminal:

```bash
man <command>
```

Example:
```bash
man nmap
man ls
man grep
```

Man pages cover syntax, all available flags, and usage examples. Essential for understanding tools without leaving the terminal.

---

### GitHub as a Security Resource

GitHub is a primary source for:

- Proof-of-concept (PoC) exploit code for CVEs
- Security research tools and scripts
- Detailed technical vulnerability analyses
- Faster disclosure than official channels

Searching a CVE identifier directly on GitHub (e.g., `CVE-2026-1337`) often returns repositories with PoC code, scanner scripts, or write-ups.

**Important caveat:** Not all PoCs are reliable. Some are incomplete, some are intentionally broken, and occasionally a "PoC" repository is itself malicious. Always review code before executing it.

---

## Important Terminology

| Term | Definition |
|---|---|
| Shodan | Internet-wide scanner that indexes publicly exposed devices and services |
| VirusTotal | Multi-engine file, URL, and hash analysis platform |
| CVE | Common Vulnerabilities and Exposures — unique identifier for known vulnerabilities |
| CVSS | Common Vulnerability Scoring System — rates vulnerability severity 0–10 |
| Man Page | Built-in Linux documentation for commands and tools |
| PoC | Proof of Concept — code demonstrating that a vulnerability is exploitable |
| ASN | Autonomous System Number — identifies a network operator (e.g., AWS, Cloudflare) |

---

## Practical Examples / Demonstrations

### Shodan search examples

```
# Find SSH servers in Ireland
country:IE port:22

# Find services on a specific domain
hostname:example.com

# Find services run by AWS
org:AS7224
```

### View a man page

```bash
man nmap
man curl
man find
```

### Search GitHub for a CVE

```
CVE-2021-44228        # Log4Shell
CVE-2019-9053         # CMS Made Simple SQLi
```

---

## Real-World Relevance

- Shodan is used in external reconnaissance to map an organisation's internet-facing attack surface without touching their systems
- VirusTotal is a standard first step when a suspicious file or URL is encountered during incident response or phishing analysis
- CVE identifiers are used in vulnerability management programmes to track, prioritise, and remediate known weaknesses
- CVSS scores drive patch prioritisation — critical scores (9.0+) are typically patched immediately
- GitHub PoC repositories are used by both attackers (to weaponise vulnerabilities quickly) and defenders (to understand and detect exploitation)

---

## Key Learnings

- Shodan indexes internet-exposed infrastructure — useful for recon and attack surface mapping
- VirusTotal provides a fast multi-engine verdict on files, URLs, and hashes
- CVEs are the universal language for discussing known vulnerabilities; CVSS scores their severity
- Man pages are the most reliable documentation for Linux commands — always available offline
- GitHub is a fast source for PoC code and vulnerability research, but requires careful verification before use

---

## Additional Notes

- Shodan has a free tier with limited results; a paid account unlocks full data and API access
- VirusTotal results should be treated as a signal, not a verdict — a clean result doesn't guarantee safety
- NVD (National Vulnerability Database) at nvd.nist.gov provides detailed CVE information including CVSS scores and affected versions
- `man -k <keyword>` searches man pages by keyword when you don't know the exact command name

---

## Conclusion

Effective research is a force multiplier in cybersecurity. Knowing where to look — Shodan for exposed infrastructure, VirusTotal for file analysis, CVE databases for vulnerability context, man pages for tool documentation, and GitHub for PoC code — dramatically speeds up both offensive and defensive work. These aren't optional skills; they're the foundation of how security professionals operate day to day.
