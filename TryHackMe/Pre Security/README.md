# TryHackMe — Pre Security

The Pre Security learning path is where it all starts. Before you can break into systems or defend them, you need to understand how they actually work. This path builds that foundation — covering everything from how the internet routes a packet to how a CPU boots, from what HTTP looks like under the hood to why weak passwords still bring down entire systems.

These aren't surface-level overviews. Each room was worked through properly, and the notes reflect that — written to be useful references, not just completion trophies.

---

## What This Path Covers

### 1. Introduction to Cyber Security
The starting point. Two perspectives: offensive (breaking in to find weaknesses) and defensive (monitoring, responding, protecting). Introduced `dirb` for directory brute-forcing on the offensive side, and covered the SOC, threat intelligence, DFIR, and malware analysis on the defensive side. Sets the tone for everything that follows.

### 2. Network Fundamentals
The backbone of everything. How devices identify themselves with IP and MAC addresses, how LANs are physically structured (star, bus, ring topologies), how the OSI model layers communication from physical bits to application data, how TCP and UDP differ and why it matters, and how networks are extended with firewalls, VPNs, and VLANs. Also covered ARP, DHCP, subnetting, and port forwarding.

### 3. How The Web Works
DNS resolution, HTTP request/response cycles, URL structure, status codes, headers, cookies — the full picture of what happens between typing a URL and seeing a webpage. Also covered how websites are built (HTML/CSS/JS), what sensitive data exposure looks like in source code, HTML injection, and the infrastructure layer: load balancers, CDNs, WAFs, databases, and virtual hosting.

### 4. Computer Fundamentals
What's inside a machine and why it matters for security. CPU, RAM, storage, motherboard, NIC, PSU — each component's role and how they connect. The full boot sequence from power button to OS handoff (UEFI, POST, bootloader). Different computer types and their security profiles. The client-server model, protocols, and ports. Virtualization — hypervisors, VMs, containers, Docker. And cloud computing — IaaS, PaaS, SaaS, and the major providers.

### 5. Operating Systems Basics
How an OS manages hardware, processes, memory, files, users, and devices. The kernel/user space separation and why it's a security boundary. Windows and Linux from a security perspective — account types, Windows Defender, firewall profiles, and the CLI tools used in both environments. Finished with a practical demonstration of how weak passwords and poor security hygiene lead to full root compromise via SSH and command history.

### 6. Software Basics
How data is represented and encoded at the lowest level — binary, hex, octal, ASCII, Unicode, UTF-8. Why encoding mismatches cause garbled text and how encoding is abused in injection attacks. Core programming concepts in Python and JavaScript: variables, conditionals, loops. And SQL fundamentals — tables, rows, columns, SELECT/WHERE/ORDER BY — the direct foundation for understanding SQL injection.

### 7. Attacks and Defenses
The path closes by tying everything together. The CIA Triad as a security mindset, not just a definition. Cryptography — plaintext vs ciphertext, symmetric vs asymmetric encryption, the Caesar cipher as a teaching tool, and how HTTPS combines both in a hybrid approach. The offensive mindset: thinking like an attacker, chaining weaknesses, understanding scope and enumeration. The defensive mindset: prevention, detection, mitigation, analysis, response — and why continuous adaptation matters.

---

## What This Path Built

Working through Pre Security built a mental model of how everything connects:

- A web request starts with DNS, travels over TCP, hits a load balancer, passes through a WAF, reaches a web server running on an OS with a kernel managing memory and processes, returns data encoded in UTF-8, displayed by a browser parsing HTML and executing JavaScript
- An attacker targeting that same request has entry points at every layer — DNS poisoning, TCP hijacking, WAF bypass, SQL injection, XSS, weak credentials, misconfigured file permissions
- A defender protecting it needs visibility at every layer — network logs, application logs, OS events, and the ability to detect, contain, and recover

That's the value of this path. Not individual facts, but the ability to see the whole picture.

---

## Modules

| Module | Rooms | Status |
|---|---|---|
| 1. Introduction to Cyber Security | Offensive Security Intro, Defensive Security Intro | Complete |
| 2. Network Fundamentals | Networking Basics, LAN Topologies, OSI Model, Packets and Frames, Extending Your Network | Complete |
| 3. How The Web Works | DNS in Detail, HTTP in Detail, How Websites Work, Putting It All Together | Complete |
| 4. Computer Fundamentals | Inside a Computer, Computer Types, Client Server Basics, Virtualization Basics, Cloud Computing Fundamentals | Complete |
| 5. Operating Systems Basics | OS Introduction, Windows Basics, Linux CLI Basics, Windows CLI Basics, OS Security | Complete |
| 6. Software Basics | Data Representation, Data Encoding, Python Simple Demo, JavaScript Simple Demo, Database SQL Basics | Complete |
| 7. Attacks and Defenses | The CIA Triad, Cryptography Concepts, Become a Hacker, Become a Defender | Complete |

---

## Up Next

**TryHackMe — Cyber Security 101**
