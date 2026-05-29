# Active Directory Basics

## Overview

Active Directory (AD) is the backbone of identity and access management in Windows enterprise environments. It centralises user accounts, computer accounts, security policies, and authentication for an entire organisation. Understanding AD is fundamental to both offensive security (AD is the primary target in most enterprise attacks) and defensive security (protecting AD means protecting the entire domain).

---

## Topics Covered

- What Active Directory is and why it exists
- AD objects: users, machines, security groups, OUs
- Default security groups
- Delegation
- Device types in an AD domain
- Group Policy Objects (GPO) and SYSVOL
- Kerberos authentication
- NetNTLM authentication
- Trees, forests, and trust relationships

---

## Key Concepts

### What is Active Directory?

A Windows domain is a group of users and computers managed by a single organisation. Active Directory (AD) is the centralised repository that stores and manages all objects in that domain. The server running AD services is called a **Domain Controller (DC)**.

**Key benefits:**
- Centralised identity management — manage all users from one place
- Centralised policy management — deploy security settings across all machines simultaneously

The core service is **AD DS (Active Directory Domain Services)** — a catalogue of every object on the network.

---

### AD Objects

**Users**
The most common AD object. Users are security principals — they can be authenticated and assigned privileges.

Two types:
- People — employees and individuals who need network access
- Service accounts — accounts used by services (IIS, MSSQL, etc.) with only the privileges needed to run that service

**Machines**
Every computer that joins the domain gets a machine object. Machine accounts are security principals with limited domain rights. They are local administrators on their own machine.

Machine account naming: computer name + `$`
Example: a computer named `DC01` has a machine account `DC01$`

Machine account passwords are 120 random characters, automatically rotated.

**Security Groups**
Groups allow assigning permissions to multiple users at once. Members inherit the group's privileges. Groups can contain users, machines, and other groups.

**Default domain security groups:**

| Group | Description |
|---|---|
| Domain Admins | Full administrative control over the entire domain, including all DCs |
| Server Operators | Can administer DCs but cannot change administrative group memberships |
| Backup Operators | Can access any file regardless of permissions — used for backups |
| Account Operators | Can create and modify user accounts in the domain |
| Domain Users | All user accounts in the domain |
| Domain Computers | All computer accounts in the domain |
| Domain Controllers | All DCs in the domain |

**Organisational Units (OUs)**
OUs are containers used to organise objects within AD. They are the primary mechanism for applying Group Policy. A user can only belong to one OU at a time.

**Security Groups vs OUs:**

| | OUs | Security Groups |
|---|---|---|
| Purpose | Apply policies and configurations | Grant permissions to resources |
| User membership | One OU per user | Multiple groups per user |
| Use case | Role-based policy application | Access control to files, printers, shares |

---

### Delegation

Delegation allows granting specific users control over certain OUs without making them Domain Admins. For example, a helpdesk team can be delegated the ability to reset passwords in a specific OU without having broader administrative rights.

---

### Device Types in an AD Domain

| Device | Description |
|---|---|
| Workstations | End-user computers — where employees do their work. Should never have privileged accounts signed in. |
| Servers | Provide services to users and other servers |
| Domain Controllers | Manage the AD domain. Most sensitive devices in the network — contain hashed passwords for all domain accounts. |

---

### Group Policy Objects (GPO)

GPOs are collections of settings applied to OUs. They can target users, computers, or both, and are used to enforce security baselines, software deployment, and configuration standards across the domain.

**GPO distribution via SYSVOL:**
GPOs are stored on the DC and distributed via a network share called SYSVOL, located at:

```
C:\Windows\SYSVOL\sysvol\
```

All domain-joined computers sync their GPOs from SYSVOL. Changes can take up to 2 hours to propagate. To force an immediate sync:

```cmd
gpupdate /force
```

---

### Kerberos Authentication

Kerberos is the default authentication protocol for modern Windows domains. It uses a ticket-based system — users authenticate once and receive tickets that prove their identity to services.

**Key components:**
- **KDC (Key Distribution Center)** — runs on the DC; issues tickets
- **TGT (Ticket Granting Ticket)** — proof of initial authentication; used to request service tickets
- **TGS (Ticket Granting Service ticket)** — grants access to a specific service
- **SPN (Service Principal Name)** — identifies the service being accessed
- **Session Key** — used to encrypt subsequent requests

**Authentication flow:**

```
1. Client → KDC
   Sends: username + timestamp encrypted with password-derived key
   Receives: TGT (encrypted with krbtgt hash) + Session Key

2. Client → KDC (when accessing a service)
   Sends: TGT + SPN + username/timestamp encrypted with Session Key
   Receives: TGS (encrypted with Service Owner's hash) + Service Session Key

3. Client → Service
   Sends: TGS
   Service decrypts TGS using its account's password hash
   Connection established
```

The user's password is never transmitted over the network. The TGT is encrypted with the `krbtgt` account hash — users cannot read its contents.

---

### NetNTLM Authentication

NetNTLM is a legacy challenge-response authentication protocol kept for compatibility. It should be considered obsolete but remains enabled in most environments.

**Authentication flow:**

```
1. Client → Server: Authentication request
2. Server → Client: Random challenge (nonce)
3. Client → Server: Response = NTLM hash + challenge + other data
4. Server → DC: Forwards challenge + response for verification
5. DC → Server: Verifies by recalculating the response; returns result
6. Server → Client: Authentication result
```

The user's password hash is never sent directly. For local accounts, the server verifies the response itself using the local SAM database — no DC involvement needed.

---

### Trees, Forests, and Trust Relationships

**Trees**
When a company expands and needs separate AD structures that share the same namespace, domains can be joined into a tree. A tree has a root domain and child subdomains.

Example:
```
thm.local (root)
├── uk.thm.local
└── us.thm.local
```

Each subdomain has its own DC and can be managed independently. The **Enterprise Admins** group has administrative privileges across all domains in the tree.

**Forests**
When multiple trees with different namespaces are joined together, the result is a forest. This typically occurs after company mergers or acquisitions.

```
thm.local (tree 1)
mht.local (tree 2)
→ Together they form a forest
```

**Trust Relationships**
Trust relationships allow users in one domain to access resources in another.

| Trust Type | Description |
|---|---|
| One-way trust | Domain AAA trusts Domain BBB — users in BBB can access resources in AAA |
| Two-way trust | Both domains mutually trust each other — default when joining a tree or forest |

A trust relationship does not automatically grant access to all resources — it only enables the possibility of authorising cross-domain access. Actual permissions must still be explicitly configured.

---

## Important Terminology

| Term | Definition |
|---|---|
| Active Directory (AD) | Centralised directory service for managing users, computers, and policies in a Windows domain |
| Domain Controller (DC) | Server running AD DS — the authoritative source for domain authentication and policy |
| AD DS | Active Directory Domain Services — the core AD service |
| Security Principal | An object (user, machine, group) that can be authenticated and assigned privileges |
| OU | Organisational Unit — a container for organising AD objects and applying GPOs |
| GPO | Group Policy Object — a collection of settings applied to OUs |
| SYSVOL | Network share on DCs used to distribute GPOs to domain-joined computers |
| KDC | Key Distribution Center — the Kerberos service on the DC |
| TGT | Ticket Granting Ticket — proof of initial Kerberos authentication |
| TGS | Ticket Granting Service ticket — grants access to a specific service |
| SPN | Service Principal Name — identifies a service in Kerberos |
| krbtgt | The service account whose hash encrypts all TGTs |
| NetNTLM | Legacy challenge-response authentication protocol |
| SAM | Security Account Manager — local database storing password hashes on Windows |
| Tree | Multiple AD domains sharing the same namespace |
| Forest | Multiple AD trees with different namespaces joined together |
| Trust relationship | A configured link allowing cross-domain resource access |
| Enterprise Admins | Group with administrative privileges across all domains in a forest |
| Delegation | Granting specific AD management rights to non-admin users |

---

## Practical Examples / Demonstrations

### Force GPO sync

```cmd
gpupdate /force
```

### Connect to a remote Windows machine via RDP

```cmd
mstsc
```

Then enter the target IP, username, and password.

### View domain information (PowerShell)

```powershell
Get-ADDomain
Get-ADUser -Filter *
Get-ADGroup -Filter *
Get-ADComputer -Filter *
```

---

## Real-World Relevance

- Domain Controllers are the highest-value targets in enterprise attacks — compromising a DC means compromising the entire domain
- Kerberoasting attacks request TGS tickets for service accounts and crack them offline — possible because TGS tickets are encrypted with the service account's password hash
- Pass-the-Ticket attacks steal Kerberos tickets from memory and use them to authenticate as the victim
- Golden Ticket attacks forge TGTs using the `krbtgt` hash — granting persistent, undetectable domain admin access
- NetNTLM is vulnerable to relay attacks (NTLM relay) — an attacker intercepts an authentication attempt and relays it to another service
- GPO abuse is a common lateral movement and persistence technique — attackers with write access to a GPO can push malicious scripts to all machines in the linked OU
- SYSVOL historically contained Group Policy Preferences (GPP) files with encrypted passwords — a well-known credential exposure issue
- Trust relationships can be abused for cross-domain privilege escalation in multi-domain environments

---

## Key Learnings

- Active Directory centralises identity and policy management for Windows domains
- AD objects include users, machines, security groups, and OUs — each with distinct purposes
- Domain Controllers are the most sensitive devices in a domain — they hold all credential hashes
- GPOs enforce configuration and security settings across the domain via SYSVOL
- Kerberos is the default authentication protocol — ticket-based, credentials never transmitted
- NetNTLM is legacy but still present — vulnerable to relay attacks
- Trees and forests allow multi-domain structures; trust relationships enable cross-domain access

---

## Additional Notes

- BloodHound is the standard tool for AD attack path analysis — it maps relationships between AD objects to find privilege escalation paths
- `net user /domain` and `net group /domain` are basic CMD commands for AD enumeration
- The `krbtgt` account password should be rotated after any domain compromise — all existing TGTs become invalid
- AD recycle bin (if enabled) allows recovery of deleted AD objects — useful for forensics

---

## Conclusion

Active Directory is the central nervous system of Windows enterprise environments. Every user, computer, and policy flows through it. Understanding its structure — objects, OUs, GPOs, authentication protocols, and trust relationships — is the foundation for understanding how enterprise attacks work, from initial access through to full domain compromise. AD security is one of the most important and most complex areas in cybersecurity.
