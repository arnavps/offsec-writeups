# Cyber Kill Chain

## Overview

The Cyber Kill Chain is a security framework introduced by Lockheed Martin in 2011, inspired by military kill chain doctrine. It models how a cyber attack progresses through seven stages from initial reconnaissance to the attacker's final objectives. Understanding each stage — and the countermeasures available at each — helps security teams identify where in an attack they can intervene to break the chain and prevent or limit damage.

**Main objectives:**
- Understand each of the seven stages of the Cyber Kill Chain
- Recognise attacker techniques at each stage
- Identify countermeasures that defenders can apply at each stage
- Understand why breaking the chain at any stage can prevent a full compromise

---

## Concepts Covered

### The Cyber Kill Chain — Framework Overview

**Definition:** A sequential model of a targeted cyber attack, broken into seven stages. Defenders can break the attack at any stage to prevent the attacker from reaching their objective.

**Why it matters:** By mapping attack activity to stages, security teams can identify gaps in detection and defence coverage. The model also provides a common language for describing attack progression.

**The seven stages:**

| Stage | Attacker's Goal |
|-------|----------------|
| 1. Reconnaissance | Gather information about the target |
| 2. Weaponisation | Create a deliverable exploit payload |
| 3. Delivery | Transmit the payload to the target |
| 4. Exploitation | Execute the payload to exploit a vulnerability |
| 5. Installation | Establish persistent access on the target |
| 6. Command & Control (C2) | Establish covert communication with the target |
| 7. Actions on Objectives | Execute the final goal (exfiltration, destruction, etc.) |

---

### Stage 1: Reconnaissance

**Definition:** The information-gathering phase where the attacker learns about the target's infrastructure, personnel, and vulnerabilities before taking any active steps.

**Why it matters:** The quality of reconnaissance determines the quality of every subsequent stage. Poorly researched targets lead to failed attacks. Well-researched targets enable precise, effective exploitation.

**Passive reconnaissance (no interaction with target):**
- WHOIS database lookups — domain registration details, contact information
- DNS queries — DNS servers, IP addresses of public hosts
- Website crawling and scraping — technology stack, employee names, structure
- Social media reconnaissance — personnel, job titles, technology references
- Google Dorking — search engine queries revealing sensitive files or exposed admin panels

**Active reconnaissance (direct interaction with target):**
- Network port scanning — live hosts and running services
- Vulnerability scanning — known weaknesses in public services
- Physical reconnaissance — visiting premises, observing entry points, security measures, and personnel behaviour

**Countermeasures:**
- Minimise public information exposure (website content, social media, DNS records)
- Use WHOIS privacy services
- Monitor and analyse network logs for scanning traffic
- Check service logs for reconnaissance attempts

---

### Stage 2: Weaponisation

**Definition:** The creation of a deliverable payload tailored to exploit the vulnerabilities discovered in reconnaissance. The output is a file or mechanism ready to be sent to the target.

**Why it matters:** Without a working payload, an attacker cannot translate reconnaissance into access. Weaponisation is where the attacker's technical preparation happens — outside the defender's visibility.

**Key details:**
- Payloads can be exploit code, macro-enabled Office documents, executable files, or scripts
- **Exploit kits** are automated platforms containing multiple exploits — they package the exploit within a payload like an executable or document
- **Microsoft Office macros** are one of the most common weaponisation methods — macros execute a predefined set of instructions when the document is opened
- Payloads are often obfuscated or encrypted to evade detection by AV and security tools

**Delivery vehicles for payloads:**
- Phishing email attachment
- Malicious web page hosting the exploit kit
- USB memory drive left in an accessible location

**Countermeasures:**
- User security awareness training — teach users to inspect email sources before opening attachments
- Disable macros in Office documents (or restrict to signed/trusted sources via Group Policy)
- Disable unnecessary browser plugins and software features
- Reduce the attack surface by removing unneeded software

---

### Stage 3: Delivery

**Definition:** The mechanism by which the weaponised payload is transmitted to the target.

**Why it matters:** A perfect payload is useless if it cannot reach the target. Delivery is where attacker creativity is most visible — new delivery methods constantly emerge as defenders block known ones.

**Delivery methods:**

| Method | Description |
|--------|-------------|
| Phishing emails | Malicious attachments or links to malicious downloads |
| Spear phishing | Targeted phishing crafted to impersonate a trusted source |
| Malicious web links | Exploit kits hosted on compromised or attacker-controlled sites |
| File-sharing platforms | Malicious files uploaded to legitimate file-sharing services |
| Malvertising | Malicious advertisements on legitimate websites redirecting to exploit sites |
| Smishing (SMS phishing) | Text messages with malicious links or malware download instructions |
| Social engineering | Convincing a target to download and run a malicious program |
| Physical delivery | USB drives left in accessible locations; malicious DVDs mailed to targets |

**Key detail on file names:** `invoice.pdf.exe` attempts to disguise an executable as a PDF by using a misleading filename — a classic social engineering technique.

**Countermeasures:**
- Security awareness training (phishing, safe browsing, social engineering recognition)
- Email and web filtering
- Web Application Firewalls (WAFs) blocking malicious file delivery
- Network monitoring and patch management
- DNS filtering to block access to known malicious domains

---

### Stage 4: Exploitation

**Definition:** The execution of the payload to exploit a vulnerability on the target system, achieving unauthorised access or code execution.

**Why it matters:** Exploitation converts a payload into actual access. Without successful exploitation, the attacker's preparation yields nothing.

**Exploitation vectors:**
- Default or weak passwords on services
- Phishing or credential theft providing valid authentication
- Software vulnerabilities (unpatched CVEs, SQL injection, buffer overflow)
- Zero-day exploits — vulnerabilities unknown to the vendor at the time of use, with no patch available

**Countermeasures:**
- Enforce password complexity requirements; mandate MFA
- Patch management — apply software updates promptly
- Vulnerability scanning — identify unpatched issues before attackers do
- Intrusion Prevention Systems (IPS) — detect and block known exploit patterns
- WAFs — block web application attacks (SQLi, XSS, CSRF)

---

### Stage 5: Installation

**Definition:** Establishing persistent access on the compromised system so the attacker can return without repeating the exploitation phase.

**Why it matters:** Without persistence, the attacker loses access as soon as the session ends or the system reboots. Installation ensures the attacker can continue their work over time.

**Persistence mechanisms:**

| Mechanism | Description |
|-----------|-------------|
| Scheduled tasks (Windows) | Tasks set to execute at boot or on schedule |
| Cron jobs (Linux) | Commands running at scheduled intervals |
| Startup script modification | Adding commands to scripts executed at system start |
| Service/daemon installation | A persistent process that starts automatically |
| Malware/rootkits | Malicious software with persistence built in |
| Web shells | Script placed on a web server allowing browser-based command execution |
| LOLBins (Living-off-the-Land) | Using legitimate Windows tools and binaries for malicious purposes to blend in |

**Web shells** deserve specific attention: after exploiting a web application, the attacker deploys a script that accepts commands via HTTP(S). The shell runs over standard web protocols, camouflaging malicious activity within normal web traffic.

**Countermeasures:**
- Monitor new processes and services with Endpoint Detection and Response (EDR)
- Regular system audits comparing current state against a secure baseline
- Application allowlisting — only approved software can execute
- Configuration management tools to detect and revert unauthorised changes

---

### Stage 6: Command and Control (C2)

**Definition:** Establishing a covert communication channel between the compromised system and the attacker's infrastructure, enabling the attacker to issue commands and receive data.

**Why it matters:** Without C2, the attacker has a foothold but no way to interact with it at will. C2 infrastructure transforms a one-time exploit into an ongoing, controllable presence.

**C2 communication techniques:**

| Technique | Description |
|-----------|-------------|
| Common protocol blend | HTTP, HTTPS, DNS, SMTP traffic — blends with legitimate traffic |
| Encrypted channels | HTTPS C2 hides command content from network monitoring |
| DNS tunnelling | Commands encoded within DNS queries to bypass firewalls |
| Social media C2 | Commands sent via platform direct messages (X/Twitter, etc.) |
| Cloud service abuse | Dropbox, Google Docs used as C2 channels — trusted domains bypass filters |

**C2 resilience mechanisms:**
- **DGA (Domain Generation Algorithms)** — generate thousands of domain names algorithmically; malware iterates until it finds an active one; blocking one domain is insufficient
- **Fast Flux** — a single domain resolves to hundreds of rotating IP addresses every few minutes; compromised devices act as proxies, hiding the real C2 server

**Countermeasures:**
- Network monitoring (firewall, IDS, IPS) for unusual traffic patterns
- DNS traffic analysis — long queries, high volume to suspicious domains
- Honeypots — detect and analyse C2 communication attempts
- Encryption inspection (TLS decryption) to inspect HTTPS C2
- Behavioural analytics — unexpected DNS activity after hours, unusual data volumes

---

### Stage 7: Actions on Objectives

**Definition:** The final stage where the attacker executes their original goals — the reason the entire attack was conducted.

**Why it matters:** This is where the attack causes its intended impact. All previous stages exist to reach this point.

**Common attacker objectives:**

| Objective | Description |
|-----------|-------------|
| Data exfiltration | Stealing sensitive files, credentials, or intellectual property |
| Ransomware | Encrypting data and demanding payment for decryption keys |
| Destructive attack | Deleting or corrupting data to disrupt operations |
| Financial theft | Unauthorised wire transfers or fraudulent transactions |
| Lateral movement | Moving to other systems to expand access or reach a higher-value target |
| ICS manipulation | Disrupting Industrial Control Systems (factories, utilities, infrastructure) |
| Persistent presence | Establishing long-term access for future operations |

**Countermeasures:**
- Data Loss Prevention (DLP) solutions — detect and block unauthorised data exfiltration
- Backup and recovery plans — mitigate ransomware and destructive attacks
- Network segmentation — isolate critical systems; limit lateral movement paths
- Access controls and least privilege — limit who can access sensitive systems
- User activity monitoring — detect unusual data access, transfer volumes, or off-hours activity
- EDR solutions — detect and alert on suspicious endpoint behaviour (file encryption, bulk data access)

---

## Methodology

### Using the Kill Chain as a Defender's Framework

The kill chain is most powerful when used in reverse — as a detection and prevention checklist.

For each stage, a security team should ask:

| Stage | Defender's Question |
|-------|-------------------|
| Reconnaissance | What information about us is publicly accessible? Are we monitoring for scanning? |
| Weaponisation | Do our users know how to recognise suspicious attachments? Are macros disabled? |
| Delivery | Are email and web filters in place? Is security awareness training running? |
| Exploitation | Are all systems patched? Is MFA enforced? Are IPS/WAF deployed? |
| Installation | Is EDR monitoring process creation and new services? Are we auditing against baselines? |
| C2 | Are we monitoring DNS, HTTP, and network behaviour for anomalies? |
| Actions | Do we have DLP? Segmentation? Backup and recovery? |

Breaking the chain at any stage prevents the attacker from reaching the next. The more stages defended, the lower the probability of a complete compromise.

---

## Key Learnings

- The kill chain is a sequential model — each stage enables the next; breaking any stage interrupts the attack
- Reconnaissance sets the quality of all subsequent stages — defenders should minimise public exposure and monitor for scanning
- Weaponisation often involves Office macros — disabling or restricting macros significantly reduces this attack surface
- Delivery is the most creative stage — attackers constantly adapt; user training is the most broadly effective defence
- Zero-day exploits are only exploitable at stage 4 — patches, IPS, and WAFs are the primary controls
- Persistence (stage 5) is what separates a one-time exploit from an ongoing compromise — EDR and baseline auditing are key controls
- C2 resilience mechanisms (DGA, Fast Flux) make simple IP blocking insufficient — behavioural detection is required
- The final stage is the attacker's goal — DLP, segmentation, backups, and access controls limit impact if all previous stages fail to stop the attack

---

## Real-World Relevance

**Penetration testing:** Pentesters follow the kill chain progression even when not explicitly naming it — recon, payload creation, delivery, exploitation, persistence, and post-exploitation map directly to stages 1–7.

**Security operations:** SOC analysts use kill chain mapping to understand where in an attack an alert fires — an alert at stage 6 (C2 beaconing) means stages 1–5 have already succeeded. That context drives incident response priorities.

**Threat intelligence:** Kill chain stage mapping is used alongside ATT&CK to understand what a threat actor has done and what they are likely to do next.

**Red team operations:** Red teams plan engagements using the kill chain to simulate real threat actors — completing all seven stages demonstrates the full compromise path to leadership.

---

## Things Worth Remembering

- Kill chain stages: **Recon → Weaponisation → Delivery → Exploitation → Installation → C2 → Actions on Objectives**
- Passive recon: WHOIS, DNS, OSINT, Google Dorking (no direct target interaction)
- Active recon: port scanning, vuln scanning, physical observation (direct interaction)
- Weaponisation most common vectors: macro-enabled Office docs, exploit kits
- Delivery most dangerous methods: spear phishing (targeted), malvertising, physical (USB drops)
- Zero-day: unknown vulnerability with no patch — exploitable only at stage 4
- Persistence mechanisms: scheduled tasks, cron jobs, services, web shells, startup modification, LOLBins
- C2 resilience: DGA (thousands of rotating domains), Fast Flux (thousands of rotating IPs)
- Breaking the chain at any stage stops progression — defence does not need to be perfect everywhere, but must cover all stages
- DLP, segmentation, and backups are the last lines of defence at stage 7

---

## Conclusion

The Cyber Kill Chain provides a structured model for understanding how attacks progress from initial information gathering to final impact. For defenders, it is a diagnostic tool — mapping which stages they can detect and disrupt, and identifying gaps in coverage. For penetration testers and red teamers, it provides a mental model for planning engagements and ensuring full coverage. For incident responders, it provides context for what has already happened and what is likely to come next. Mastering the kill chain as both an offensive and defensive concept is foundational to operating effectively in any security role.
