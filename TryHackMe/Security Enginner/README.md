# TryHackMe: Security Engineer

A structured path through the engineering, architectural, and operational disciplines of cybersecurity — covering governance and risk, cryptography, identity management, network and system hardening, software security, incident management, and the OWASP Top 10. This path approaches security from the practitioner's perspective: not just understanding threats, but designing and operating systems that resist them.

---

## Path Overview

| # | Section | Topics |
|---|---------|--------|
| 1 | Introduction to Security Engineering | Security engineer role, cryptography, IAM, security principles |
| 2 | Threats and Risks | Governance, regulation, threat modelling, risk management, vulnerability management |
| 3 | *(not covered)* | — |
| 4 | Network and System Security | Secure network architecture, system hardening (Linux, Windows, AD), network devices, protocols, cloud, virtualisation, auditing |
| 5 | Software Security | SDLC, SAST, DAST, DevSecOps, OWASP API Top 10, CVE weaponization, CTF labs |
| 6 | Managing Incidents | IR and IM fundamentals, logging for accountability, first responder duties, cyber crisis management |
| 7 | OWASP Top 10 (2025) | Access control, authentication failures, data handling, cryptographic failures, injection |

---

## Section Writeups

### 1. Introduction to Security Engineering

**Files:** `security-engineer.md` · `introduction-to-cryptography.md` · `identity-and-access-management.md` · `security-principles.md`

Established the foundation of the security engineering role — what organisations expect from a security engineer, and how the role spans asset management, policy creation, secure design, risk management, vulnerability management, compliance, and incident response support. The security engineer is not just a technical practitioner but a trusted advisor to business leadership on the risk implications of technical decisions.

Cryptography covered the full arc from classical ciphers through to modern cryptographic systems: symmetric encryption (AES, block and stream ciphers, cipher modes), asymmetric encryption (RSA, key pair mechanics, digital signatures), Diffie-Hellman key exchange and its MITM vulnerability, cryptographic hash functions (SHA-256, HMAC), PKI and certificate chains, and password security through salted hashing and key derivation functions (PBKDF2, bcrypt, Argon2).

Identity and Access Management covered the IAAA model (Identification, Authentication, Authorisation, Accountability) with each pillar explained in depth. Authentication factors (something you know/have/are), MFA, DAC/RBAC/MAC access control models, replay attacks and their nonce-based mitigations, SSO and its protocols (SAML, OAuth, OIDC, Kerberos), and the distinction between IdM and IAM.

Security principles covered the CIA Triad and Parkerian Hexad, defence-in-depth, the principle of least privilege, Zero Trust vs Trust-But-Verify, formal security models (Bell-LaPadula, Biba, Clark-Wilson), and the ISO/IEC 19249 architectural principles.

---

### 2. Threats and Risks

**Files:** `governance-and-regulation.md` · `threat-modelling.md` · `risk-management.md` · `vulnerability-management.md`

Covered the strategic and operational frameworks that translate security principles into organisational practice.

Governance and regulation established the relationships between governance (direction-setting), regulation (external minimums), and compliance (demonstrating adherence). Key regulations and standards examined in depth: GDPR (including 72-hour breach notification and two penalty tiers), PCI DSS (12 requirements across 6 objectives), HIPAA, GLBA, NIST 800-53 (20 control families), NIST 800-63B (digital identity guidelines), ISO/IEC 27001 (ISMS with PDCA cycle and SoA), and SOC 2 (Type I vs Type II). The GRC (Governance, Risk, Compliance) framework was covered as the integrating discipline.

Threat modelling introduced four major frameworks: MITRE ATT&CK (real-world adversary TTPs mapped to tactics and techniques), DREAD (Damage, Reproducibility, Exploitability, Affected Users, Discoverability — scored risk prioritisation), STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, DoS, Elevation of Privilege — threat categorisation against the CIA triad), and PASTA (seven-step, risk-centric, business-aligned methodology). Attack trees and attack paths were covered as visual modelling tools. The ATT&CK Navigator was introduced for annotating and communicating threat coverage.

Risk management followed the NIST SP 800-30 four-step process: Frame Risk (assumptions, constraints, tolerance, priorities), Assess Risk (qualitative vs quantitative analysis — SLE, ARO, ALE calculations), Respond to Risk (mitigate, transfer, accept, avoid), and Monitor Risk (effectiveness, change, compliance). Supply chain risk and the SolarWinds attack as a canonical real-world example.

Vulnerability management established the six-phase lifecycle (Discover → Prioritise → Assess → Report → Remediate → Verify) mapped to the NIST CSF. SCAP components (CVE, CVSS, NVD, CPE) and the distinction between CVSS score and actual risk in context. Covered Greenbone/OpenVAS as the practical scanning platform.

---

### 4. Network and System Security

**Files:** `secure-network-architecture.md` · `linux-system-hardening.md` · `windows-system-hardening.md` · `active-directory-hardening.md` · `network-device-hardening.md` · `network-security-protocols.md` · `virtualization-and-containers.md` · `intro-to-cloud-security.md` · `auditing-and-monitoring.md`

The largest section in the path, covering the full technical breadth of hardening an enterprise environment at every layer.

Secure network architecture moved beyond subnetting to real enforcement: 802.1q VLAN tagging, Router on a Stick (ROAS) with VyOS sub-interfaces, security zones (External, DMZ, Trusted, Restricted, Management, Audit), ACLs and ACEs, stateful vs stateless firewalls, zone-pair policies with the critical established/related rule, SSL/TLS inspection as controlled MITM, DHCP snooping, and Dynamic ARP Inspection (DAI) — with the entire dependency chain from DHCP binding database through ARP validation.

Linux system hardening covered the full defence-in-depth stack: physical security and GRUB password, LUKS disk encryption (PBKDF2 key derivation, key slots), host firewall (iptables, nftables, UFW), SSH hardening (`PermitRootLogin no`, `PasswordAuthentication no`, public key authentication), user account controls (sudo, nologin shells, libpwquality password policy), service minimisation, update management (LTS cycles, Dirty COW as a real-world kernel CVE example), and log file locations.

Windows system hardening covered services and registry (Run key persistence), Event Viewer and key security event IDs (4624, 4625, 4720, 4728, 7045), UAC (levels, bypass techniques), Group Policy and password/lockout policies, Windows Defender Firewall, SMBv1 disabling (EternalBlue/WannaCry context), DNS and ARP hardening, RDP security (BlueKeep/CVE-2019-0708), Office hardening with ASR rules, BitLocker (TPM requirements), Windows Sandbox, and Secure Boot.

Active Directory hardening covered the AD structural model (domains, trees, forests, trust types and transitive trust implications), LM Hash disabling, SMB signing against NTLM relay, LDAP signing, password rotation approaches (gMSA), password policies, least privilege and RBAC, the Tiered Access Model (Tier 0/1/2 with GPO enforcement), account auditing, Kerberoasting (SPN-based TGS cracking — Event ID 4769 detection), DCSync attacks (Event ID 4662), and Microsoft Security Compliance Toolkit deployment.

Network device hardening covered the endpoint vs network device distinction, common attack vectors, firmware updates, secure protocol migration (Telnet → SSH, HTTP → HTTPS, SNMPv1/v2 → v3), VPN hardening (OpenVPN: AES-256-CBC cipher, SHA256 auth, tls-crypt for PFS, tls-version-min 1.2), router/switch hardening with OpenWrt, port security, ARP spoofing prevention, rogue DHCP prevention, and network monitoring tools (Nagios, Zabbix, PRTG, SolarWinds).

Network security protocols covered HTTPS (TLS handshake in full), FTPS (active vs passive mode, explicit vs implicit TLS), SMTPS (STARTTLS on 587), POP3S (port 995), DNSSEC (RRSIG, zone signing, chain of trust, what it protects vs what it does not), OpenPGP (GPG email encryption with public/private key roles), SSH (vs Telnet), SSL/TLS deep dive (version history, TLS 1.3 improvements, cipher suite ordering), SOCKS5, and IPsec (AH vs ESP, transport vs tunnel mode, VPN applications).

Virtualisation and containers covered Type 1 hypervisors (ESXi, Proxmox, KVM — bare metal), Type 2 hypervisors (VirtualBox, VMware Workstation — hosted), containers (Linux namespaces and cgroups, Docker images/Dockerfiles/commands, Docker security risks including socket exposure and privileged containers), and Kubernetes (Pod, Deployment, Service, RBAC, etcd encryption, Pod Security Admission, self-healing and horizontal scaling).

Cloud security covered service models (IaaS/PaaS/SaaS with shared responsibility model), deployment models (public/private/hybrid/community cloud with associated risks), the cloud data lifecycle (Create → Store → Use → Share → Archive → Destroy/crypto shredding), AWS IAM (identities, entities, principals, policies with default deny), Security Groups vs NACLs, S3 encryption, disaster recovery tiers (Cold/Warm/Hot DR with RTO/RPO), and AWS CloudTrail/CloudWatch/GuardDuty.

Auditing and monitoring covered the distinction between logging (passive/historical), monitoring (active/real-time), and auditing (systematic/periodic), ISAE 3402, COBIT, ISO 27001, PCI DSS, SOX audit frameworks, the six-stage audit process, Linux log files and aureport/ausearch commands for security investigation, Windows event log types and key security event IDs, and the relationship between all three disciplines.

---

### 5. Software Security

**Files:** `sdlc.md` · `sast.md` · `dast.md` · `introduction-to-devsecops.md` · `owasp-api-security-top-10-1.md` · `owasp-api-security-top-10-2.md` · `weaponizing-vulnerabilities.md` · `mothers-secret.md` · `traverse.md`

Covered the full software security lifecycle — from secure development process through to automated tooling and applied exploitation.

SDLC covered the Secure Software Development Lifecycle: SDLC phases (Planning → Requirements → Design → Development → Testing → Deployment → Maintenance) with security activities at each phase, Waterfall vs Agile vs DevSecOps, Shift Left philosophy, threat modelling (STRIDE, DREAD, PASTA) during design, ASVS security requirements, and secure coding standards.

SAST (Static Application Security Testing) covered how static analysis works (pattern matching, AST analysis, taint analysis, control flow analysis), what it detects (SQLi, command injection, hardcoded secrets, cryptographic weaknesses), where it fits in the pipeline (IDE → pre-commit → PR → CI), its limitations (false positives, runtime context unavailable), and Semgrep as a practical tool with custom rule writing in YAML.

DAST (Dynamic Application Security Testing) covered how DAST tools work (crawling, active scanning, fuzzing, passive scanning), what it detects vs what SAST detects, DAST in the SDLC pipeline, OWASP ZAP modes and interface (automated scan, authenticated scanning, passive scanning), and ZAP Docker commands for CI/CD integration.

DevSecOps covered the CI/CD pipeline model, why traditional security gates fail at DevOps pace, security at each pipeline stage (pre-commit hooks, PR scanning, build SAST/SCA, staging DAST, production monitoring), SCA (dependency scanning — the Log4Shell context), secrets management (detect-secrets, HashiCorp Vault), IaC scanning (Checkov, tfsec), and container scanning (Trivy).

OWASP API Security Top 10 covered all ten categories across two rooms: BOLA/IDOR, Broken User Authentication, Excessive Data Exposure, Rate Limiting failures, BFLA, Mass Assignment, Security Misconfiguration (CORS, debug mode, verbose errors), Injection, Improper Assets Management (forgotten API versions), and Insufficient Logging and Monitoring.

Weaponizing Vulnerabilities covered the CVE/CVSS system (score ranges, base metric groups, CVSS limitations), Exploit-DB and searchsploit usage, NVD, PoC vs weaponized exploit, Metasploit Framework (modules, payloads, Meterpreter), manual exploitation methodology, and responsible disclosure with the 90-day standard.

The two CTF labs applied these concepts in a chained attack format. Mother's Secret involved source code analysis of an Express.js application to identify route-chaining authentication bypass, extract a base64-encoded flag from obfuscated JavaScript, and exploit an unsanitised file path parameter to read `/opt/m0th3r` via LFI. Traverse demonstrated a realistic multi-stage web chain: Gobuster directory enumeration revealing exposed email logs, LFI via `file://` protocol bypass to read credentials, authenticated access, and SQLMap-based privilege escalation to administrator.

---

### 6. Managing Incidents

**Files:** `intro-to-ir-and-im.md` · `logging-for-accountability.md` · `becoming-a-first-responder.md` · `cyber-crisis-management.md`

Covered the full operational lifecycle of a cyber incident — from the moment an alert fires to the post-crisis documentation review.

Incident Response and Management fundamentals established the SOC as the detection filter (events → alerts → triage → incidents), the clear distinction between Incident Response (what happened? — technical investigation) and Incident Management (how do we respond? — process and decisions), the four-level escalation model (Level 1 SOC → Level 2 CERT → Level 3 CSIRT → Level 4 CMT), all team roles and their specific functions, the NIST Incident Management framework in full (Preparation, Detection & Analysis, Containment/Eradication/Recovery in correct order, Post-Incident), and the six common pitfalls (insufficient hardening, logging, alerting, scope determination, accountability, and backups).

Logging for Accountability covered accountability as the fourth IAAA pillar, non-repudiation and its relationship to the STRIDE Repudiation threat, SIEM three-component architecture (Forwarder → Indexer → Search Head), four data ingestion methods (agent, port-forwarding, syslog, upload), log storage tiers (hot/warm/cold with the PCI DSS 12-month/90-day availability requirement), what constitutes a good log, the distinction between manual and automated log sources, and log correlation and enrichment — building coherent investigative narratives from multiple independent sources.

Becoming a First Responder covered the critical importance of not powering off a compromised host, the IETF evidence volatility order (RFC 3227: registers → ARP/RAM → temp files → disk → remote logs → physical config → backups), the three major DON'Ts (don't shut down, don't trust system programs, don't modify file access times), chain of custody requirements for legal proceedings, incident playbooks and call trees, all four containment methods (network segmentation, physical isolation, virtual/EDR isolation, and rate limiting as the "dial-up technique"), BCP vs DRP, BCP creation steps (BIA → recovery actions → team structure → testing), and all six BCP metrics (RPO, RTO, WRT, MTD, MTBF, MTTR) with the critical RTO + WRT ≤ MTD relationship. Documentation templates and the two-timestamp requirement for action accountability.

Cyber Crisis Management covered the CMT invocation threshold (Level 4), the autocratic governance model with the Golden Hour process (Assembly → CSIRT Briefing → Crisis Triage → Holding Statements), CMT roles (Chair, Executives, Communications, Legal, Operations, SMEs, Scribe), the static CMT principle, the cyclic operational model (Information Updates → Triage → Action Discussions → Approvals), nuclear actions and their business impact trade-offs, holding statements for narrative control, internal and external communication strategy, GDPR 72-hour breach notification, regulatory engagement, law enforcement contact, ransom payment legality considerations, and the SME's specific responsibility to translate technical scope into business impact language for non-technical executives.

---

### 7. OWASP Top 10 (2025)

**Files:** `owasp-iaaa-failures.md` · `owasp-application-design-flaws.md` · `owasp-insecure-data-handling.md`

Applied the OWASP Top 10:2025 framework across three thematic rooms, connecting each vulnerability class to its root cause, impact, and remediation.

**IAAA Failures** — A01 Broken Access Control, A07 Authentication Failures, A09 Security Logging and Monitoring Failures. The IAAA dependency chain and what breaks at each stage. IDOR and function-level access control bypass. Authentication enumeration, weak passwords, session management failures. Consequences of absent or insufficient logging for accountability.

**Application Design Flaws** — A02 Security Misconfiguration, A03 Software and Data Integrity Failures (supply chain), A04 Cryptographic Failures (design angle), A06 Vulnerable and Outdated Components / Insecure Design. Verbose error message exploitation, SolarWinds-context supply chain attacks, hardcoded credential discovery and AES-ECB mode decryption, and insecure design assumptions about backend API behaviour.

**Insecure Data Handling** — A04 Cryptographic Failures (implementation angle), A05 Injection, A08 Software or Data Integrity Failures. Rolling your own crypto and its predictable failures. SQL injection mechanics and parameterised query prevention. Command injection, SSTI, and prompt injection as emerging AI-specific risk. Software integrity failures in update pipelines and third-party data trust.

---

## Skills Developed

Completing this path developed a connected set of security engineering skills that span governance, architecture, operations, and application security:

**Security Engineering Role and Governance**
- Understanding the security engineer's scope: policy, risk, architecture, assessment, compliance, incident support
- Governance, regulation, and compliance frameworks (GDPR, PCI DSS, ISO 27001, SOC 2, NIST 800-53)
- GRC as an integrated discipline — governance sets direction, risk quantifies impact, compliance verifies adherence

**Cryptography**
- Symmetric encryption (AES), asymmetric encryption (RSA, key pairs), Diffie-Hellman key exchange
- Cryptographic hash functions, HMAC, digital signatures
- PKI and certificate chain validation
- Password security: salted hashing, KDFs (PBKDF2, bcrypt, Argon2)

**Identity and Access Management**
- IAAA model, non-repudiation, authentication factors and MFA
- Access control models: DAC, RBAC, MAC
- SSO protocols: SAML, OAuth 2.0, OIDC, Kerberos
- Replay attacks, nonce-based mitigations

**Threat Modelling and Risk**
- MITRE ATT&CK Navigator for threat mapping and detection gap analysis
- STRIDE threat categorisation with CIA triad mapping
- DREAD risk scoring methodology
- PASTA seven-step risk-centric framework
- NIST SP 800-30 risk management: SLE, ARO, ALE calculations
- Supply chain risk — SolarWinds as the canonical example

**Vulnerability Management**
- CVE/CVSS/NVD/SCAP framework
- Six-phase lifecycle: Discover → Prioritise → Assess → Report → Remediate → Verify
- Greenbone/OpenVAS for practical vulnerability scanning
- CISA KEV as the highest-priority remediation signal

**Network and System Hardening**
- VLAN segmentation, 802.1q tagging, ROAS, security zones, zone-pair firewall policies
- DHCP snooping → DAI dependency chain for layer-2 security
- Linux: LUKS encryption, iptables/nftables/UFW, SSH hardening, libpwquality
- Windows: UAC, Group Policy, BitLocker, SMB signing, Event IDs
- Active Directory: Tiered Access Model, Kerberoasting detection, LM Hash, SMB/LDAP signing, gMSA
- Network devices: OpenVPN hardening (AES-256-CBC, SHA256, tls-crypt for PFS), OpenWrt

**Secure Protocols**
- TLS 1.3 handshake mechanics and improvements over 1.2
- FTPS active vs passive mode, SMTPS, DNSSEC chain of trust
- IPsec AH vs ESP, transport vs tunnel mode
- SSH vs Telnet, SOCKS5 proxy mechanics

**Cloud and Virtualisation Security**
- AWS Shared Responsibility Model and IAM default-deny
- Security Groups vs NACLs, CloudTrail, GuardDuty
- Cloud data lifecycle and crypto shredding
- Docker security risks (socket exposure, privileged containers)
- Kubernetes RBAC, etcd encryption, Pod Security Admission

**Software Security**
- Secure SDLC with Shift Left philosophy
- SAST (taint analysis, Semgrep rule writing), DAST (OWASP ZAP), SCA
- DevSecOps pipeline security at every stage
- OWASP API Security Top 10 — all ten vulnerability categories
- CVE research, Exploit-DB, Metasploit, responsible disclosure

**Incident Response and Management**
- IR (what happened) vs IM (how to respond) — both required, neither sufficient alone
- NIST framework: Preparation → Detection/Analysis → Contain/Eradicate/Recover → Post-Incident
- Evidence volatility order (RFC 3227) and chain of custody
- BCP metrics: RPO, RTO, WRT, MTD, MTBF, MTTR
- CMT Golden Hour, holding statements, static CMT principle
- Non-repudiation through authenticated, tamper-proof logs

**OWASP Top 10**
- All ten vulnerability classes with root cause, impact, and remediation
- Practical labs: verbose error exploitation, AES-ECB decryption, API backend assumption bypass

---

## Notes

- Section 3 (Identity and Access Management as a standalone module) was not covered in these writeups — IAM content is included within Section 1
- The path is structured to build from foundational principles upward: governance informs architecture, architecture informs hardening, hardening supports incident response
- Notes marked as study guides rather than walkthroughs — the goal is understanding mechanisms, not completing tasks
- All technical details are drawn from room content and verified external sources
