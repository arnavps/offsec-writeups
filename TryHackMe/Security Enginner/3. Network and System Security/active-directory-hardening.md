# Active Directory Hardening

## Executive Summary

Active Directory (AD) is the backbone of identity and access management in the vast majority of enterprise Windows environments. It is also the primary target in advanced persistent threat (APT) operations — compromising AD means compromising the entire organisation. This room covers AD architecture, authentication security controls, least privilege implementation, the Tiered Access Model, known attack techniques (Kerberoasting, brute force RDP, pass-the-hash), and Microsoft Security Compliance Toolkit deployment.

**Major concepts covered:** AD structure (domains, trees, forests, trusts), LAN Manager Hash, SMB signing, LDAP signing, password rotation, password policies, least privilege and RBAC, Tiered Access Model, account auditing, Kerberoasting, weak credentials, and Security Compliance Toolkit.

**Why it matters:** Every major enterprise ransomware incident and APT campaign targets Active Directory. Domain Admin compromise = complete organisational compromise. A domain controller (DC) owns authentication for every system in the domain — hardening AD directly reduces the blast radius of every attack.

**Where these concepts apply:** Any Windows domain environment — enterprise networks, hybrid Azure AD environments, government, financial, healthcare, and education sectors.

---

## Big Picture Overview

Active Directory is a centralised identity provider for an entire organisation. It answers:
- Who are you? (Authentication via Kerberos or NTLM)
- What can you access? (Authorisation via group membership and ACLs)
- What can you do on this computer? (Group Policy enforcement)

Because everything flows through AD, compromising it gives an attacker:
- Ability to authenticate as any user
- Access to all network shares, databases, and applications
- Control over Group Policy — can push malware to all machines
- Ability to create persistent backdoor accounts

This is why AD hardening is the single highest-impact security activity in a Windows enterprise.

---

## Core Concepts

---

### Concept 1 — AD Structure

#### Domain

The domain is the core organisational unit of Active Directory. It is a logical boundary containing user accounts, computer accounts, groups, and policies. All objects within a domain share the same database, schema, and security policies. A domain is identified by its DNS name (e.g., `corp.example.com`).

**Domain Controller (DC):** The server running AD DS (Active Directory Domain Services). Acts as the gatekeeper for authentication and authorisation for the entire domain. It:
- Stores the AD database (`NTDS.dit`)
- Handles Kerberos ticket issuance
- Enforces Group Policy
- Replicates with other DCs

**Security Importance:** The DC is the highest-value target in any Windows environment. A compromised DC = domain compromise.

#### Trees and Forests

**Trees:** A collection of domains sharing a contiguous DNS namespace. Domains within a tree have two-way transitive trusts by default — `parent.corp.com` trusts `child.corp.com` and vice versa.

**Forests:** A collection of trees sharing a common schema, global catalogue, and configuration. The forest is the ultimate security boundary in Active Directory. Trusts between forests must be explicitly created.

**Trust Relationships:**

| Trust Type | Direction | Transitive? | Description |
|-----------|----------|------------|------------|
| Parent-child | Two-way | Yes | Automatically created when child domain added |
| Tree-root | Two-way | Yes | Between domain tree roots in a forest |
| Forest trust | Configurable | Optional | Between separate forests — must be explicitly created |
| External trust | One-way or two-way | No | To non-forest domain (e.g., legacy NTLM) |
| Shortcut trust | One-way or two-way | Yes | Shortcut between domains in the same forest |

**Transitive Trust Implication:** If Domain A trusts Domain B, and Domain B trusts Domain C transitively, then Domain A automatically trusts Domain C. This has significant lateral movement implications — compromise of any domain in a transitive trust chain can impact all trusted domains.

#### Containers and Leaves

In AD's hierarchical object model:
- **Container objects:** Hold other objects (Organizational Units, the root domain, built-in containers)
- **Leaf objects:** Cannot hold other objects (user accounts, computer accounts, group objects)

---

### Concept 2 — LAN Manager Hash (LM Hash)

**What Is It?**
LM Hash is a legacy Windows password hash format from the LAN Manager era (pre-Windows NT). It is cryptographically weak:
- Passwords are converted to uppercase before hashing (reduces keyspace)
- Passwords are split into two 7-character chunks, each hashed independently (reduces brute force difficulty dramatically)
- No salt — makes rainbow table attacks effective

**Why It's a Problem:**
When both LM hash and NT hash are stored, an attacker who obtains the hash database can crack LM hashes in seconds or minutes using modern GPUs, revealing passwords up to 14 characters.

**How to Disable LM Hash Storage:**
```
Group Policy Management Editor →
Computer Configuration → Policies → Windows Settings →
Security Settings → Local Policies → Security Options →
"Network security: Do not store LAN Manager hash value on next password change"
→ Enable
```

After enabling, the change takes effect at the next password change. Existing LM hashes remain until users change their passwords or are forcibly reset.

**Attack:** Pass-the-Hash using LM hash allows attackers to authenticate without knowing the actual plaintext password.

---

### Concept 3 — SMB Signing

**What Is It?**
SMB signing adds a digital signature to SMB packets, ensuring that each packet has not been tampered with in transit. Without SMB signing, SMB traffic can be intercepted and modified by an on-path attacker.

**The Attack It Prevents — NTLM Relay:**
Without SMB signing:
1. Attacker performs LLMNR/NBNS poisoning to capture an NTLM authentication challenge-response
2. Attacker relays the captured authentication to another machine that trusts the same credentials
3. Attacker authenticates to that machine as the victim — without cracking the password

SMB signing ensures that authentication is bound to the specific connection — a relayed authentication attempt will fail because the signature cannot be forged.

**Configuration:**
```
Group Policy Management Editor →
Computer Configuration → Policies → Windows Settings →
Security Settings → Local Policies → Security Options →
"Microsoft network server: Digitally sign communications (always)" → Enabled
```

**Note:** SMB signing should be required on all domain controllers and recommended on all servers. Client-side signing is optional but recommended. Performance impact is minimal with modern hardware.

---

### Concept 4 — LDAP Signing

**What Is It?**
LDAP (Lightweight Directory Access Protocol) is the protocol used to query and modify Active Directory. LDAP signing ensures that LDAP requests are digitally signed, preventing man-in-the-middle attacks that could inject or modify directory queries.

**Why It Matters:**
Without LDAP signing, an attacker on the same network can:
- Replay LDAP authentication requests (replay attack)
- Inject modified LDAP responses to return forged directory data
- Intercept and modify LDAP write operations (modifying AD objects)

**Configuration:**
```
Group Policy Management Editor →
Computer Configuration → Policies → Windows Settings →
Security Settings → Local Policies → Security Options →
"Domain controller: LDAP server signing requirements" → "Require signing"
```

**LDAP Signing vs LDAPS:** LDAP signing adds a signature to LDAP packets over port 389. LDAPS (port 636) encrypts the entire LDAP session over TLS. Both provide protection; LDAPS provides stronger confidentiality.

---

### Concept 5 — Password Rotation and Policies

**The Service Account Password Problem:**
Service accounts — accounts used by applications to authenticate to other services — are notoriously difficult to rotate. Many organisations leave service account passwords unchanged for years because:
- Multiple applications may depend on the account
- Changing the password requires updating all dependent services simultaneously
- There is often no documented list of all dependencies

**Three Rotation Approaches:**

| Approach | Method | Pros | Cons |
|----------|--------|------|------|
| **Script-based rotation** | PowerShell in Scheduled Task | No additional infrastructure | Requires writing and maintaining scripts |
| **MFA on accounts** | Add MFA to reduce rotation frequency | Reduces need for rotation | MFA on service accounts is complex |
| **Group Managed Service Accounts (gMSA)** | Microsoft-native automatic rotation every 30 days | Fully automated; Windows-native | Requires Windows Server 2012 R2+; not supported by all applications |

**gMSA** is the recommended solution for Windows service accounts. The DC manages and automatically rotates the password; the service retrieves the current password from the DC at startup.

**Password Policy Settings:**
```
Group Policy Management Editor →
Computer Configuration → Policies → Windows Settings →
Security Settings → Account Policies → Password Policy
```

| Setting | Recommended Value |
|---------|-----------------|
| Enforce password history | 15 previous passwords |
| Minimum password length | 12–14 characters |
| Password complexity | Enabled |
| Store passwords using reversible encryption | Disabled |
| Maximum password age | 90 days (or 0 for never expire + MFA) |

---

### Concept 6 — Least Privilege and RBAC

**Account Types:**

| Account Type | Use | Privilege Level |
|-------------|-----|----------------|
| **Standard user account** | Daily work (email, applications) | Minimum required |
| **First-tier privileged account** | Server administration | Elevated — server admin only |
| **Second-tier privileged account** | Domain/forest administration | Domain Admin — very restricted use |
| **Shared account** | Guest/visitor access | Minimum — time-limited |

**Principle of Least Privilege:**
> "A subject should be given only those privileges needed to complete its task."

Every admin account beyond standard user access increases the blast radius of credential theft. A compromised standard user account grants access to the user's files and emails. A compromised Domain Admin grants complete domain control.

**Role-Based Access Control (RBAC) in AD:**
RBAC is implemented through security groups:
- Create groups aligned with job functions (e.g., `IT-Helpdesk`, `Finance-ReadOnly`, `Server-Admins-Tier1`)
- Assign permissions to groups, not individual users
- Add users to groups based on their role
- Review group memberships periodically — users change roles and roles accumulate over time ("privilege creep")

---

### Concept 7 — Active Directory Tiered Access Model (TAM)

**What Is It?**
The Tiered Access Model is a security architecture that separates AD assets into distinct tiers with strict boundaries. The goal is to prevent credential compromise in a lower tier from enabling access to higher tiers.

**The Three Tiers:**

| Tier | Contains | Accounts That Can Access |
|------|---------|------------------------|
| **Tier 0** | Domain Controllers, AD admin tools, forest root, privileged access workstations (PAWs) | Tier 0 accounts only |
| **Tier 1** | Member servers, applications, cloud services | Tier 1 accounts (and Tier 0 if needed) |
| **Tier 2** | User workstations, end-user devices, IoT | Tier 2 accounts (helpdesk + standard users) |

**The Core Rule:**
> "Privileged credentials must never cross boundaries, either accidentally or intentionally."

- A Domain Admin (Tier 0) account should **never** be used to log into a workstation (Tier 2)
- A server admin (Tier 1) account should **never** log into a DC (Tier 0)
- If credentials are cached on a compromised Tier 2 workstation, they should only be Tier 2 credentials

**Why This Matters:**
If a Domain Admin logs into a Tier 2 workstation and that workstation is compromised, the DA credentials are cached in LSASS (or can be extracted via Mimikatz). The attacker now has Domain Admin credentials from a workstation compromise.

**Implementation via Group Policy Objects (GPOs):**
GPOs enforce the tier boundaries using `Deny logon locally`, `Deny logon through Remote Desktop Services`, and `Deny access to this computer from the network` policies applied per tier.

```
Tier 0 DCs → GPO: Deny logon for Tier 1 and Tier 2 accounts
Tier 1 Servers → GPO: Deny logon for Tier 2 accounts
Tier 2 Workstations → GPO: Deny logon for Tier 0 and Tier 1 accounts
```

---

### Concept 8 — Account Auditing

**Three Audit Types:**

| Audit Type | What It Reviews | Detects |
|-----------|----------------|---------|
| **Usage audit** | What each account is accessing and doing | Unusual access patterns, dormant accounts still in use |
| **Privilege audit** | Whether each account has the minimum required access | Overprivileged accounts, privilege creep |
| **Change audit** | Changes to account permissions, passwords, settings | Unauthorised privilege changes, backdoor accounts |

**Audit Policy Configuration:**
```
Group Policy Management Editor →
Computer Configuration → Policies → Windows Settings →
Security Settings → Advanced Audit Policy Configuration
```

Enable auditing for:
- Account logon events
- Account management
- Directory service access
- Logon/logoff
- Privilege use
- Policy change

---

### Concept 9 — Known AD Attack Techniques

#### Kerberoasting

**What It Is:**
Kerberoasting exploits the Kerberos Ticket Granting Service (TGS) to extract service ticket hashes for offline cracking. Any authenticated domain user can request a service ticket for any Service Principal Name (SPN) registered in AD.

**Attack Flow:**
```
1. Attacker authenticates to the domain (any valid account)
2. Attacker requests a TGS for a target service (e.g., MSSQL service)
3. KDC issues TGS encrypted with the service account's NTLM hash
4. Attacker captures TGS — it is encrypted with the service account's password hash
5. Attacker runs offline brute force / dictionary attack against the TGS
6. If service account has a weak password → cracked → service account credentials obtained
```

**Why It's Hard to Detect:**
The initial TGS request is a normal, legitimate AD operation. No unusual network traffic. No failed authentication. Only detectable by correlating TGS requests for service accounts that are not regularly used.

**Mitigations:**
- Service accounts should have **long, complex, randomly generated passwords** (25+ characters) — makes offline cracking infeasible
- Use **gMSA** for service accounts — auto-rotated, complex passwords
- **Implement MFA** on privileged accounts
- Periodically reset **KRBTGT** account password — this invalidates all existing Kerberos tickets
- Monitor for **Event ID 4769** (Kerberos service ticket requested) with unusual encryption types (RC4 = likely Kerberoasting)

#### Brute Forcing RDP

**Attack:** Automated scanners find systems with port 3389 exposed to the internet and brute force weak credentials. Success grants remote desktop access.

**Mitigations:**
- Never expose RDP directly to the internet
- Place RDP behind VPN with MFA
- Enable Account Lockout Policy (5 failed attempts → 30-minute lockout)
- Monitor Event ID 4625 for multiple failures targeting the same account

#### Publicly Accessible Shares

AD configurations sometimes leave network shares accessible without authentication, providing an initial foothold for lateral movement.

**Detection:**
```powershell
# Enumerate open network shares
Get-SmbOpenFile
Get-SmbShare
```

Review all shares — any share accessible to `Everyone` or `Domain Users` without a business justification should be restricted.

---

### Concept 10 — Microsoft Security Compliance Toolkit (MSCT)

**What Is It?**
MSCT is a Microsoft-provided toolkit with pre-built security baselines for Windows Server, Windows client, and Microsoft 365. Security baselines are sets of recommended Group Policy settings based on Microsoft's security research and industry best practices.

**Components:**
- **Security baselines:** GPO backup files containing recommended security settings
- **Policy Analyser:** Tool for comparing GPOs — identifies conflicts, redundancies, and deviations from baselines

**Installing a Security Baseline:**
```
1. Download from Microsoft Security Compliance Website
2. Extract the zip: Windows Servers Security Baseline.zip
3. Open extracted folder → Scripts
4. Execute the desired baseline with PowerShell as Administrator
   e.g.: .\Baseline-LocalInstall.ps1
```

**Policy Analyser:**
The Policy Analyser compares multiple GPOs to identify:
- Conflicting settings between GPOs at different levels
- Redundant settings that appear in multiple GPOs
- Deviations from the security baseline

This is essential in complex AD environments where hundreds of GPOs may be applied at different OU levels, creating unpredictable effective configurations.

---

## Architecture and Relationships

### AD Trust and Attack Path

```
Forest A                          Forest B
  |                                 |
Domain A1 ←── Forest Trust ────→ Domain B1
  |                                 |
Domain A2 ──────────────────────────┘
(Child domain — transitive trust)

Attack Path:
Compromise Domain A2 (lower-security child domain)
         ↓
Kerberoast service account in A2 → crack password
         ↓
Lateral movement to Domain A1 (parent domain — bidirectional trust)
         ↓
Escalate to Enterprise Admin → full forest compromise
```

### Credential Tier Contamination (What TAM Prevents)

```
WITHOUT TAM:
Domain Admin logs into Tier 2 workstation
         ↓
Attacker compromises workstation
         ↓
Mimikatz extracts LSASS → DA credentials in memory
         ↓
Full domain compromise from workstation breach

WITH TAM:
GPO blocks DA from logging into Tier 2 workstations
         ↓
Compromised workstation only yields Tier 2 credentials
         ↓
Impact contained to Tier 2
```

---

## Security Engineer Perspective

### High-Priority AD Hardening Actions

| Priority | Action | Why |
|---------|--------|-----|
| Critical | Disable LM Hash storage | LM hash cracked in minutes |
| Critical | Enable SMB signing (required) | Prevents NTLM relay attacks |
| Critical | Enable LDAP signing | Prevents LDAP replay/MITM |
| Critical | Implement Tiered Access Model | Prevents credential tier contamination |
| Critical | Protect KRBTGT account | Golden Ticket attacks require KRBTGT hash |
| High | Use gMSA for service accounts | Prevents Kerberoasting |
| High | Deploy MSCT security baselines | Ensures comprehensive baseline coverage |
| High | Periodic privilege audit | Detects privilege creep |
| High | Account lockout policy | Mitigates brute force |
| Medium | Change auditing | Detects backdoor account creation |

### Monitoring and Detection

| Attack | Event ID | What to Alert On |
|--------|---------|-----------------|
| Kerberoasting | 4769 | TGS requests with RC4 encryption from unexpected accounts |
| Golden Ticket | 4769, 4624 | TGS with unusual ticket lifetime |
| Pass-the-Hash | 4624 | Logon type 3 (network) from unexpected source |
| Brute force | 4625 | Multiple failures against same account |
| New admin account | 4720, 4728 | User created and added to privileged group |
| DC replication (DCSync) | 4662 | DS-Replication-Get-Changes on DC from non-DC source |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **AD DS** | Active Directory Domain Services — the server role implementing AD |
| **DC** | Domain Controller — server hosting and managing the AD database |
| **DCSync** | Attack technique requesting DC replication data to extract password hashes |
| **Forest** | Collection of AD trees sharing schema, global catalogue, and configuration — ultimate security boundary |
| **gMSA** | Group Managed Service Account — service account with automatically rotated password |
| **Golden Ticket** | Forged Kerberos TGT created using the KRBTGT hash — grants full domain access |
| **Kerberoasting** | Requesting TGS for SPN accounts and cracking the ticket offline |
| **KRBTGT** | The Kerberos ticket-granting ticket account — its hash is used to sign all Kerberos tickets |
| **LM Hash** | Legacy weak Windows password hash — should be disabled |
| **LDAP** | Lightweight Directory Access Protocol — used to query and modify AD |
| **LLMNR/NBNS** | Legacy name resolution protocols commonly abused for credential capture |
| **MSCT** | Microsoft Security Compliance Toolkit — pre-built AD security baselines |
| **NTLM Relay** | Intercepting NTLM authentication and relaying it to another service |
| **SPN** | Service Principal Name — identifier used by Kerberos for service accounts |
| **TAM** | Tiered Access Model — architecture separating AD assets by privilege tier |
| **TGS** | Ticket Granting Service ticket — Kerberos ticket for a specific service |
| **TGT** | Ticket Granting Ticket — initial Kerberos ticket used to request service tickets |
| **Trust** | Relationship allowing resource sharing between AD domains |

---

## Exam and Interview Revision

### Must Remember

- Domain Controller = highest-value target in Windows environment; owns all authentication
- LM Hash is cryptographically weak — disable storage via Group Policy
- SMB signing prevents NTLM relay; required on all DCs, recommended on all servers
- LDAP signing prevents replay and MITM attacks on directory queries
- TAM has 3 tiers: **Tier 0** (DCs, AD tools), **Tier 1** (servers), **Tier 2** (workstations)
- Core TAM rule: privileged credentials must never cross tier boundaries
- Kerberoasting: request TGS for SPN → crack offline → detected by Event ID 4769 with RC4
- gMSA automatically rotates service account passwords every 30 days
- Golden Ticket = forged TGT using KRBTGT hash; requires periodic KRBTGT password reset
- DCSync attack = attacker impersonates DC to replicate all password hashes; detect via Event 4662
- Never expose RDP to internet; enforce Account Lockout Policy against brute force
- MSCT provides pre-built security baselines deployable via GPO

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is Kerberoasting? | Requesting Kerberos TGS tickets for SPN-registered service accounts, then cracking the encrypted ticket offline. Mitigated by strong service account passwords (25+ chars) or gMSA. |
| What is the Tiered Access Model? | Separates AD assets into Tier 0 (DCs), Tier 1 (servers), Tier 2 (workstations). Credentials must never cross tier boundaries to prevent lateral movement from compromised lower tiers reaching domain admin. |
| What is the difference between SMB signing and LDAP signing? | SMB signing protects file sharing traffic against NTLM relay. LDAP signing protects directory queries against replay and MITM. Both add digital signatures to their respective protocols. |
| What is a Golden Ticket? | A forged Kerberos TGT created using the KRBTGT account's hash. Grants any privilege to any resource in the domain. Remains valid until KRBTGT password is rotated (twice). |
| Why should LM Hash storage be disabled? | LM hash splits the password into two 7-char chunks, converts to uppercase, and hashes independently — extremely weak. Can be cracked in seconds. No legitimate use in modern environments. |
| What is DCSync? | Attack where an attacker impersonates a DC and requests password hash replication from a legitimate DC. Gives all domain password hashes. Requires domain replication privilege. Detected by Event 4662 from non-DC source. |
