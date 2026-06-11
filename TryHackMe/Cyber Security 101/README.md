# TryHackMe: Cyber Security 101

A complete path through the fundamentals of cybersecurity — covering both offensive and defensive disciplines, from operating systems and networking up through exploitation, web hacking, malware analysis, and secure design.

---

## Path Overview

| # | Section | Topics |
|---|---------|--------|
| 1 | Start Your Cyber Security Journey | Offensive vs defensive security, search skills |
| 2 | Linux Fundamentals | Linux CLI, file system, permissions, processes |
| 3 | Windows and AD Fundamentals | Windows OS, registry, AD, users, GPO |
| 4 | Command Line | Windows CMD, Linux shells, PowerShell (6 parts) |
| 5 | Networking | Protocols, OSI/TCP-IP, Nmap, Wireshark, tcpdump |
| 6 | Cryptography | Symmetric/asymmetric crypto, hashing, PKI, John the Ripper |
| 7 | Exploitation Basics | Metasploit, Meterpreter, EternalBlue, CVE-2024-21413 |
| 8 | Web Hacking | HTTP, JavaScript, SQL, Burp Suite |
| 9 | Offensive Security Tooling | Hydra, Gobuster, SQLMap, Shells |
| 10 | Defensive Security | SOC, DFIR, Incident Response, Logs |
| 11 | Security Solutions | SIEM, Firewalls, IDS, Vulnerability Scanning |
| 12 | Defensive Security Tooling | CyberChef, CAPA, REMnux, FlareVM |
| 13 | Build Your Cybersecurity Career | Security principles, CIA, trust models |
| 14 | OWASP Top 10 (2025) | IAAA failures, design flaws, data handling |

---

## Section Writeups

### 1. Start Your Cyber Security Journey

**Files:** `offensivesecurityintro.md` · `defensivesecurityintro.md` · `searchskills.md`

Introduced the two sides of cybersecurity: offensive security (finding and exploiting weaknesses) and defensive security (protecting systems, detecting attacks, and responding to incidents). Covered how to find reliable information using search engines, technical databases, and OSINT techniques — a skill that underpins every other topic in the path.

---

### 2. Linux Fundamentals

**Files:** `linuxfundamentalspart1.md` · `linuxfundamentalspart2.md` · `linuxfundamentalspart3.md`

Learned to navigate and operate a Linux system from the command line — the primary environment for most security tools and servers. Covered the file system hierarchy, file operations, permissions (read/write/execute, `chmod`, `chown`), process management, package management, and essential shell utilities. Linux literacy is a prerequisite for nearly every subsequent section in this path.

---

### 3. Windows and Active Directory Fundamentals

**Files:** `windowsfundamentals1.md` · `windowsfundamentals2.md` · `windowsfundamentals3.md` · `activedirectorybasics.md`

Covered the Windows operating system in depth — the filesystem, user account structure, the registry, system configuration tools (Task Manager, MSConfig, Event Viewer), and built-in security utilities. Active Directory introduced the architecture of enterprise Windows networks: domains, domain controllers, users, groups, Group Policy Objects (GPOs), and how authentication works in a domain environment. Understanding Windows and AD is essential for both attacking and defending corporate environments.

---

### 4. Command Line

**Files:** `windowscommandline.md` · `linuxshells.md` · PowerShell parts 1–6

The command line is the primary interface for security work on both platforms. Covered Windows CMD for file navigation, process management, and network diagnostics. Linux shells covered Bash, environment variables, piping, and scripting fundamentals. PowerShell was covered in six parts: introduction and basics, piping/filtering/sorting, system and network information gathering, real-time analysis, and scripting — including the security implications of execution policy bypass and script-based attacks.

---

### 5. Networking

**Files:** `networkingconcepts.md` · `networkingessentials.md` · `networkingcoreprotocols.md` · `networkingsecureprotocols.md` · `nmapthebasics.md` · `tcpdumpthebasics.md` · `wiresharkthebasics.md` · `wiresharkpacketoperations.md`

Built a thorough understanding of how networks work — from the OSI and TCP/IP models through IP addressing, subnetting, routing, and switching. Covered the core protocols every security professional needs to know: DNS, HTTP/S, FTP, SMTP, SSH, and TLS. Learned how to perform network reconnaissance with Nmap (host discovery, port scanning, service/version detection, NSE scripts), capture and analyse traffic with tcpdump, and inspect packets in detail with Wireshark. Secure protocols covered the cryptographic mechanisms that protect network communication.

---

### 6. Cryptography

**Files:** `cryptographybasics.md` · `hashingbasics.md` · `publickeycryptographybasics.md` · `johntheripper.md`

Learned how cryptography is used to protect data. Covered symmetric encryption (AES, cipher modes), asymmetric encryption (RSA, key pairs, digital signatures), and the Public Key Infrastructure (PKI) that makes TLS and certificate-based authentication work. Hashing covered the properties of cryptographic hash functions, common algorithms (MD5, SHA-1, SHA-256), and why certain algorithms are no longer safe for password storage. John the Ripper covered offline hash cracking techniques — wordlist attacks, rule-based attacks, and how weak hashing enables credential recovery.

---

### 7. Exploitation Basics

**Files:** `metasploit-introduction.md` · `metasploit-exploitation.md` · `metasploit-meterpreter.md` · `blue.md` · `monikerlink-cve-2024-21413.md`

Introduced the structured exploitation workflow using the Metasploit Framework — modules, payloads, options, and the full exploitation lifecycle. Meterpreter was covered as a post-exploitation platform: process migration, file system access, privilege escalation, and credential dumping with `hashdump`. Practical exploitation was applied in two real CVE contexts:

- **Blue (MS17-010 / EternalBlue)** — exploiting an unauthenticated SMBv1 buffer overflow on Windows 7, achieving SYSTEM-level access, dumping NTLM hashes, and cracking them with John the Ripper
- **CVE-2024-21413 (Moniker Link)** — a Windows Outlook vulnerability allowing NTLM credential theft via maliciously crafted hyperlinks

---

### 8. Web Hacking

**Files:** `web-application-basics.md` · `javascript-essentials.md` · `sql-fundamentals.md` · `burp-suite-basics.md`

Covered the full foundation of web application security. Web application basics introduced the front end / back end architecture, URL anatomy, the complete HTTP request and response structure, all HTTP methods and their security implications, status codes, request/response headers, and security headers (CSP, HSTS, X-Content-Type-Options, Referrer-Policy). JavaScript essentials covered variables, functions, control flow, and the security implications of client-side code — client-side validation bypass, hardcoded secrets, and XSS via browser interaction functions. SQL fundamentals covered relational databases, CRUD operations, operators, and SQL functions as the prerequisite to SQL injection. Burp Suite introduced the intercepting proxy workflow — the Proxy, Repeater, Intruder, Decoder, and Target modules — that underpins all manual web application testing.

---

### 9. Offensive Security Tooling

**Files:** `hydra.md` · `gobuster.md` · `sqlmap-basics.md` · `shells-overview.md`

Covered the core offensive tools used in web application and network testing:

- **Hydra** — online password brute-forcing across SSH, FTP, and HTTP POST login forms; understanding the form string syntax and failure indicator
- **Gobuster** — directory/file enumeration (`dir` mode), subdomain enumeration (`dns` mode), and virtual host discovery (`vhost` mode)
- **SQLMap** — automated SQL injection detection and exploitation; GET-based testing, authenticated cookie-based testing, database enumeration with `--dbs`, table enumeration, and data extraction with `--dump`
- **Shells** — the conceptual and practical understanding of reverse shells, bind shells, and web shells; how named pipe payloads work; listener tools (Netcat, Rlwrap, Ncat, Socat); and PHP/Bash/Python payload categories

---

### 10. Defensive Security

**Files:** `soc-fundamentals.md` · `digital-forensics-fundamentals.md` · `incident-response-fundamentals.md` · `logs-fundamentals.md`

Switched to the defensive side of the discipline. SOC fundamentals covered the structure, roles (L1/L2/L3, Detection Engineer, SOC Manager), and workflows of a Security Operations Center — including alert triage with the 5 Ws and the People/Process/Technology model. Digital forensics covered the NIST four-phase process (Collection, Examination, Analysis, Reporting), evidence acquisition best practices (chain of custody, write blockers, proper authorisation), and Windows forensics tools (FTK Imager, Autopsy, DumpIt, Volatility). Incident response introduced false vs true positives, the five incident types, and the SANS PICERL framework in detail. Logs covered the six log categories, Windows Event IDs, Apache access log structure, and Linux command-line log analysis with `cat`, `grep`, and `less`.

---

### 11. Security Solutions

**Files:** `introduction-to-siem.md` · `firewalls-fundamentals.md` · `ids-fundamentals.md` · `vulnerability-scanner-overview.md`

Covered the core security products that protect and monitor enterprise environments. SIEM introduced centralised log collection, log parsing and normalisation, correlation, real-time alerting, and log ingestion methods. Firewalls covered all four types (stateless, stateful, proxy, NGFW) with their OSI layer context, complete firewall rule structure, the three actions (Allow/Deny/Forward), Windows Defender Firewall profiles, and the Linux Netfilter stack with ufw. IDS fundamentals covered HIDS vs NIDS deployment modes, signature-based vs anomaly-based vs hybrid detection, and Snort's three operating modes. Vulnerability scanning covered authenticated vs unauthenticated scans, internal vs external scans, the major scanners (Nessus, Qualys, Nexpose, OpenVAS), and the CVE/CVSS framework for vulnerability identification and prioritisation.

---

### 12. Defensive Security Tooling

**Files:** `cyberchef-basics.md` · `capa-basics.md` · `remnux-getting-started.md` · `flarevm-arsenal-of-tools.md`

Covered the practical tools used in defensive and forensic analysis. CyberChef introduced the recipe-based data transformation tool — encoding/decoding operations, extractors, Base64 mechanics, and URL encoding. CAPA covered automated static binary analysis — interpreting MITRE ATT&CK mappings, MAEC values (Launcher/Downloader), MBC objectives/behaviors/methods, and the namespace/capability taxonomy in output. REMnux covered the malware analysis Linux environment — VBA macro dropper chain analysis, Volatility 3 Windows memory plugins, batch preprocessing with shell loops, and string extraction. FlareVM introduced the Windows-based analysis toolkit — covering Procmon, Process Explorer, HxD, CFF Explorer, Wireshark, PEStudio, and FLOSS in the context of initial malware investigation.

---

### 13. Build Your Cybersecurity Career

**Files:** `security-principles.md`

Covered the foundational security principles that underpin every decision in cybersecurity. The CIA Triad (Confidentiality, Integrity, Availability) and its adversarial counterpart DAD (Disclosure, Alteration, Destruction). Extended properties: Authenticity and Nonrepudiation. The Parkerian Hexad. Formal security models: Bell-LaPadula (confidentiality — "no read up, no write down"), Biba (integrity — "no read down, no write up"), and Clark-Wilson (integrity through controlled transformation procedures). Defence-in-Depth. ISO/IEC 19249 architectural principles (domain separation, layering, encapsulation, redundancy, virtualisation) and design principles (least privilege, attack surface minimisation, centralised validation, fail-safe error handling). Trust models: Trust but Verify vs Zero Trust with microsegmentation. The precise distinction between Vulnerability, Threat, and Risk.

---

### 14. OWASP Top 10 (2025)

**Files:** `owasp-iaaa-failures.md` · `owasp-application-design-flaws.md` · `owasp-insecure-data-handling.md`

Completed the path with a focused study of the OWASP Top 10:2025, organised into three themed rooms:

**IAAA Failures** — A01 Broken Access Control, A07 Authentication Failures, A09 Logging and Alerting Failures. The IAAA dependency chain: Identity → Authentication → Authorisation → Accountability. Common authentication weaknesses (enumeration, weak passwords, session issues). IDOR and function-level access control failures. The consequences of absent logging.

**Application Design Flaws** — A02 Security Misconfigurations, A03 Software Supply Chain Failures, A04 Cryptographic Failures, A06 Insecure Design. Misconfigurations with verbose error exploitation lab. Supply chain attacks with SolarWinds context. Hardcoded keys and AES-ECB decryption lab. Insecure design with backend API assumption bypass lab.

**Insecure Data Handling** — A04 Cryptographic Failures (implementation angle), A05 Injection, A08 Software or Data Integrity Failures. Rolling your own crypto. SQL injection mechanics and parameterised query prevention. Command injection, SSTI, and prompt injection. Integrity failures in update pipelines and third-party data.

---

## Skills Developed

By completing this path, the following practical and conceptual skills were developed:

**Operating Systems**
- Linux and Windows administration from the command line
- Active Directory structure and enterprise authentication
- PowerShell scripting and security-relevant cmdlets

**Networking**
- Protocol analysis at every OSI layer
- Network reconnaissance with Nmap and traffic analysis with Wireshark
- Understanding of secure vs insecure protocol design

**Cryptography**
- Symmetric and asymmetric encryption
- Hashing and its role in password security
- PKI, digital signatures, and TLS mechanics
- Offline hash cracking with John the Ripper

**Offensive Security**
- Metasploit exploitation framework and Meterpreter post-exploitation
- Real CVE exploitation (MS17-010, CVE-2024-21413)
- Web application testing with Burp Suite
- Credential brute-forcing with Hydra
- Directory and subdomain enumeration with Gobuster
- SQL injection detection and exploitation with SQLMap
- Reverse shells, bind shells, and web shells

**Defensive Security**
- SOC operations, alert triage, and incident response (PICERL framework)
- Digital forensics (NIST process, chain of custody, disk and memory imaging)
- Log analysis — Windows Event IDs, Apache access logs, Linux CLI tools
- SIEM architecture, log normalisation, and correlation

**Security Solutions**
- Firewall types (stateless, stateful, proxy, NGFW) and rule configuration
- IDS deployment and detection methods (signature, anomaly, hybrid)
- Vulnerability scanning, CVE/CVSS, and remediation prioritisation

**Malware Analysis and Forensic Tooling**
- Static analysis with CAPA and PEStudio
- Memory forensics with Volatility 3
- Data transformation and IOC extraction with CyberChef
- String extraction and malware dropper chain analysis in REMnux

**Security Principles and Application Security**
- CIA Triad, DAD Triad, and Parkerian Hexad
- Formal security models (Bell-LaPadula, Biba, Clark-Wilson)
- Zero Trust and Defence-in-Depth
- OWASP Top 10:2025 — the ten most critical web application vulnerability classes

---

## Notes

- Sections 1–9 are primarily offensive or foundational in focus
- Sections 10–14 cover the defensive, analytical, and principled aspects of the discipline
- The path is structured to build upward — each section draws on knowledge from the sections before it
- All writeups are personal study notes — technical details are drawn from the rooms and verified sources, not generated
