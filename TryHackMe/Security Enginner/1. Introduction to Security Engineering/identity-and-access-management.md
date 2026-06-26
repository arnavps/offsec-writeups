# Identity and Access Management

## Overview

Identity and Access Management (IAM) covers the four pillars of access security: Identification, Authentication, Authorisation, and Accountability — collectively the IAAA model. This room also covers access control models (DAC, RBAC, MAC), authentication mechanisms (something you know/have/are, MFA), identity management (IdM) vs IAM as distinct concepts, and Single Sign-On (SSO). These concepts underpin every login system, every access control policy, and every audit trail in every organisation.

**Why it matters:** Every security breach, privilege abuse, or insider threat intersects with IAM. Poor authentication, over-permissioned accounts, missing audit logs, and absent MFA are consistently among the top causes of real-world incidents.

**Where these concepts apply:** Active Directory and Entra ID, cloud IAM (AWS IAM, GCP IAM), zero trust architecture, PAM (Privileged Access Management), SIEM log correlation, compliance (HIPAA, PCI DSS, SOC 2), SSO and federation (SAML, OAuth, OIDC).

---

## Core Concepts

---

### Concept 1 — The IAAA Model

#### Definition

IAAA is the four-stage model describing how users interact with systems and how access is controlled and tracked.

```
IDENTIFICATION → AUTHENTICATION → AUTHORISATION → ACCOUNTABILITY
  (Who are you?)  (Prove it)        (What can you do?)  (What did you do?)
```

| Stage | Question | Example |
|-------|---------|---------|
| **Identification** | Who are you claiming to be? | Entering a username or email |
| **Authentication** | Can you prove that identity? | Providing the correct password + OTP |
| **Authorisation** | What are you allowed to access? | Read-only access to the sales folder |
| **Accountability** | What did you do? | Audit log showing files accessed at 14:32 |

#### Why IAAA Matters

Each stage is a distinct security control. Failures at any stage break the chain:
- No identification → cannot differentiate users
- Weak authentication → impersonation attacks succeed
- No authorisation checks → any authenticated user accesses any resource
- No accountability → incidents cannot be investigated; attackers can act without trace

---

### Concept 2 — Identification

#### Definition

Identification is the process of a user (or process or system) **claiming** a specific identity. It is a declaration — not yet proof. The claimed identity must be unique within the system.

#### Forms of Identification

| Type | Examples |
|------|---------|
| **Username** | `tanderson`, `thomas01`, `neo` — format depends on the organisation |
| **Email address** | Guaranteed unique; widely used to avoid username collision problems |
| **National ID / Passport number** | Government-issued unique identifiers |
| **Student / Employee ID** | Organisation-issued unique numbers |
| **Mobile phone number** | Unique per SIM — used by messaging apps for identification |

#### Key Point

Identification carries no proof of truth. When you tell someone your name at a party, they have no way to verify it — identification alone is not security. Authentication is what adds the proof requirement.

---

### Concept 3 — Authentication

#### Definition

Authentication is the process of **verifying** the claimed identity. It answers: is this person actually who they say they are?

#### Authentication Factors

There are five authentication factor types, three primary and two secondary:

**Primary Factors:**

| Factor | Definition | Examples |
|--------|-----------|---------|
| **Something you know** | Memorised secret | Password, passphrase, PIN, security question answer |
| **Something you have** | Physical possession | Hardware security key, OTP via SMS, authenticator app TOTP, smart card, bank card |
| **Something you are** | Biometric characteristic | Fingerprint, facial recognition, retina scan, voice recognition |

**Secondary Factors:**

| Factor | Definition | Examples |
|--------|-----------|---------|
| **Somewhere you are** | Geographic or network location | Corporate VPN, office IP range, geofencing |
| **Something you do** | Behavioural pattern | Typing rhythm (keystroke dynamics), mouse movement patterns |

#### Something You Know — Detail

- Passwords: `4SNoPawKkdFiCdnm` — high entropy, hard to remember
- Passphrases: `Judge Battle Advise Pain 9` — multiple words, easier to remember, equally secure
- PINs: typically 4–6 digits — lower entropy, compensated by lockout policies
- Patterns: a traced path on a grid — equivalent to a PIN in security terms

**Weakness:** Can be phished, guessed, shoulder-surfed, or leaked in data breaches.

#### Something You Have — Detail

- **OTP via SMS:** Code sent to registered phone number; possession of the SIM proves you have that number
- **TOTP authenticator app** (Google Authenticator, Authy): Time-based one-time password generated locally — more secure than SMS
- **Hardware security key** (YubiKey, Titan Security Key, Nitrokey, Thetis): Plugs into USB/USB-C or communicates via NFC; phishing-resistant; highest security for second factor
- **Smart card / bank card:** Physical card + PIN = classic 2FA

#### Something You Are — Detail

- Fingerprint readers — widely deployed on mobile devices and laptops
- Facial recognition — increasingly common on smartphones
- Retina/iris scanners — used in high-security physical access control
- Voice recognition — used in phone banking and voice assistants

**Weakness:** Biometrics cannot be changed if compromised. A stolen password can be reset; a stolen fingerprint cannot. Biometric data must be stored securely (usually as a mathematical representation, not a raw image).

#### Multi-Factor Authentication (MFA)

**Definition:** Using two or more distinct authentication factors. Two-factor authentication (2FA) is the most common form.

**Why MFA matters:**
- Defeats password-only attacks (phishing, credential stuffing, brute force)
- Even if the attacker has the password, they cannot authenticate without the second factor
- The ATM is the oldest 2FA example: card (have) + PIN (know)

**MFA combinations:**
- Password + SMS OTP (know + have)
- Password + authenticator app TOTP (know + have)
- Password + hardware key (know + have — phishing-resistant)
- Password + fingerprint (know + are)
- Smart card + PIN (have + know)

**Security note:** SMS-based OTP is the weakest MFA form due to SIM-swapping attacks. Authenticator apps (TOTP) are stronger. Hardware keys (FIDO2/WebAuthn) are the strongest — phishing is impossible because the key only responds to legitimate domains.

---

### Concept 4 — Authorisation and Access Control

#### Authorisation vs Access Control

| Concept | Role |
|---------|------|
| **Authorisation** | Defines what a user is *allowed* to do — the policy |
| **Access Control** | Enforces the authorisation policy — the mechanism |

**Example:** Linda books a hotel room. The hotel *authorises* her to access room 214 and public areas only (policy). The electronic key card only unlocks room 214 (enforcement). Authorisation decides; access control enforces.

#### Three Main Access Control Models

---

##### Discretionary Access Control (DAC)

**Definition:** The resource owner explicitly controls who can access their resource and what permissions they have. Access is at the owner's discretion.

**How it works:**
- Owner grants or revokes access to specific users
- Permissions set per-user or per-group on each resource
- Common in consumer platforms (Google Drive, Dropbox sharing)

**Example:** You share a graduation photo album on Google Photos with specific family member accounts and grant view permission.

**Advantages:** Simple; owner has full control; works well for small-scale sharing.

**Disadvantages:** Does not scale — managing permissions for hundreds of users across thousands of files becomes unmanageable. User role changes require manually updating permissions on every resource.

---

##### Role-Based Access Control (RBAC)

**Definition:** Users are assigned to roles; access permissions are assigned to roles, not directly to users. Access is determined by role membership.

**How it works:**
```
User → assigned to → Role → grants access to → Resources

Example:
  Edward → Sales Role → Access to: sales documents, CRM, customer database
  Linda  → HR Role    → Access to: HR records, payroll system
  Neither has access to the other's resources.
```

**Advantages:**
- Scales efficiently — adding a user to a role gives all role permissions in one step
- Role changes are clean — remove from old role, add to new role
- Easier to audit — see what each role can access, then see who is in each role
- Standard in enterprise Active Directory (security groups), AWS IAM, and cloud platforms

**Disadvantages:** Requires well-defined role taxonomy; role explosion (too many fine-grained roles) can become unmanageable.

---

##### Mandatory Access Control (MAC)

**Definition:** Access decisions are made by the system, not the resource owner. Users and resources are assigned security labels (classifications); the system enforces access based on policy rules — users cannot override them.

**How it works:**
- Resources have classification labels (e.g., Top Secret, Secret, Unclassified)
- Users have clearance levels
- The system enforces: a user can only access resources at or below their clearance level
- Users cannot grant access to others — they do not have discretion

**Use cases:** Government/military classified systems; environments requiring strict data separation.

**Linux MAC implementations:**
- **AppArmor** — profile-based; shipped with Debian and Ubuntu
- **SELinux** — flexible label-based MAC; standard on Red Hat and Fedora

**Advantages:** Highest security; users cannot accidentally or intentionally misconfigure access.

**Disadvantages:** Significantly restricts user abilities; complex to administer; unsuitable for most commercial environments.

---

#### DAC vs RBAC vs MAC Summary

| Characteristic | DAC | RBAC | MAC |
|---------------|-----|------|-----|
| Who sets access | Resource owner | Administrator (via roles) | System (via policy) |
| Scalability | Low | High | High |
| User control | High | Limited | None |
| Typical use | File sharing, consumer apps | Enterprise systems, cloud | Government, high-security |
| Flexibility | High | Medium | Low |

---

### Concept 5 — Accountability

#### Definition

Accountability ensures users can be held responsible for their actions by recording what they do in a system after authentication and authorisation. Accountability requires **logging**.

#### Why It Matters

Without logs, it is impossible to:
- Investigate security incidents (who accessed what, when, from where)
- Detect insider threats (unusual access patterns)
- Satisfy compliance requirements (HIPAA, PCI DSS, GDPR all require audit trails)
- Provide forensic evidence in legal proceedings

#### Logging Requirements

| Requirement | Why |
|-------------|-----|
| Log all authentication events (success and failure) | Detect brute force, account compromise, credential theft |
| Log all access to sensitive resources | Audit trail for compliance; detect unauthorised access |
| Log all privileged actions | Track what admins do — privileged abuse is a top insider threat vector |
| Logs must be tamper-resistant | Attackers delete logs to hide activity — write-only remote log storage prevents this |
| Centralise logs | Distributed logs are hard to correlate; centralisation enables SIEM analysis |

#### Log Forwarding

**Definition:** Sending log data from one system to another for centralised storage and analysis.

**Why centralise:**
- Correlate events across multiple systems (e.g., failed login on server + lateral movement on another)
- Single location for incident investigation
- Attacker would need to compromise the log server separately to delete evidence
- Feeds SIEM for real-time detection

#### SIEM (Security Information and Event Management)

**Definition:** Technology that aggregates log data from multiple sources and analyses it for signs of security threats.

**SIEM capabilities:**

| Capability | Description |
|-----------|-------------|
| Log aggregation | Collects logs from endpoints, servers, network devices, applications, cloud |
| Correlation | Links related events across systems — e.g., failed login + successful login + data access |
| Alerting | Notifies security teams when defined thresholds or patterns are triggered |
| Compliance reporting | Generates audit evidence reports automatically |
| Forensic investigation | Full event history allows timeline reconstruction after incidents |
| Threat detection | Pattern matching against known attack signatures and anomaly baselines |

**Authentication flow through SIEM:**
```
User logs in → Authentication event generated
     |
     v
Log forwarded to SIEM
     |
     v
SIEM correlates: 50 failed logins + 1 success from same IP
     |
     v
Alert: Potential brute force → credential stuffing attack
     |
     v
SOC analyst investigates
```

---

### Concept 6 — Identity Management (IdM) and IAM

#### Identity Management (IdM)

**Definition:** Policies and technologies focused specifically on managing digital identities — user accounts, their attributes, authentication mechanisms, and permissions.

**Core functions:**
- User provisioning (creating and managing accounts)
- Authentication management (passwords, MFA, SSO)
- Permission management (assigning access rights)
- Deprovisioning (revoking access when users leave)

**Uses a centralised identity store** (e.g., Active Directory, LDAP) as the single source of truth for all user identities.

#### Identity and Access Management (IAM)

**Definition:** A broader concept encompassing IdM plus all processes and technologies to manage and secure access rights across the organisation's systems.

**IAM extends IdM with:**
- Role-based access control governance
- Identity governance and compliance management
- Access certification (periodic reviews — does this user still need this access?)
- Privileged Access Management (PAM)
- Federation and SSO across multiple systems
- Integration with compliance frameworks (HIPAA, GDPR, PCI DSS)

#### IdM vs IAM

| Aspect | IdM | IAM |
|--------|-----|-----|
| Focus | User identity itself | Identity + access rights + governance |
| Scope | Identity attributes and authentication | Broader — includes access evaluation and enforcement |
| Complexity | Lower | Higher |
| Typical tool | Active Directory, LDAP | Okta, Azure AD / Entra ID, AWS IAM |

**Note:** The boundary between IdM and IAM is blurry in practice — many sources use them interchangeably. The key distinction is that IAM includes evaluating and governing access decisions, not just managing identity records.

---

### Concept 7 — Authentication Attacks and Protocols

#### Why Native Authentication Protocols Are Hard to Get Right

Building custom authentication protocols is notoriously difficult. Even seemingly reasonable designs have fundamental weaknesses. Understanding common attack patterns explains why established, tested protocols (TLS, Kerberos, OAuth) must be used rather than homegrown alternatives.

#### Replay Attack

**Definition:** An attacker captures a valid authentication message and replays it later to authenticate as the victim — without ever knowing the actual password.

**Scenario:**
1. Alice sends her encrypted password to the server: `enc(password)`
2. Attacker Mallory intercepts `enc(password)`
3. Mallory sends `enc(password)` to the server
4. Server sees valid encrypted credentials → authenticates Mallory as Alice

**The attack works even though Mallory never learned the password** — she only captured and replayed the ciphertext.

**Fix:** Make each authentication response unique and time-limited. Include the current timestamp in the authentication token. The server only accepts tokens from the last few seconds. A replayed token from 10 minutes ago is rejected.

```
Unique challenge-response:
  Server sends challenge (nonce)
  Client responds: encrypt(password + nonce + timestamp)
  Server verifies: is nonce fresh? is timestamp recent? does the decrypted value match?
```

This is why authentication protocols use **nonces** (numbers used once) and **timestamps**.

---

### Concept 8 — Single Sign-On (SSO)

#### Definition

SSO allows users to authenticate once and gain access to all authorised applications and systems without re-authenticating for each one.

#### The Problem SSO Solves

Without SSO, a user might need separate credentials for:
- Workstation login
- Email client
- CRM system
- HR system
- Development tools
- Cloud services

Managing multiple strong, unique passwords is cognitively infeasible for most users — they either reuse passwords (dangerous) or use weak ones (also dangerous).

#### How SSO Works

```
User authenticates once to the SSO Identity Provider (IdP)
     |
     v
IdP issues a token (e.g., SAML assertion, OAuth token, Kerberos ticket)
     |
     v
User accesses Application A → presents token → app validates with IdP → access granted
User accesses Application B → presents token → app validates with IdP → access granted
User accesses Application C → presents token → app validates with IdP → access granted
(No re-authentication required for any application)
```

#### Benefits of SSO

| Benefit | Detail |
|---------|--------|
| **One strong password** | Easier for users to remember and maintain one strong credential |
| **Easier MFA** | Configure MFA once on the IdP — all applications inherit it automatically |
| **Simpler support** | Password resets are centralised — one system, one process |
| **Efficiency** | No repeated logins when moving between applications |
| **Centralised access control** | Revoke access at the IdP level — immediately affects all applications |
| **Centralised audit trail** | All authentication events in one place for SIEM correlation |

#### SSO Protocols

| Protocol | Use Case |
|----------|---------|
| **SAML 2.0** | Enterprise SSO, web applications — XML-based assertions |
| **OAuth 2.0** | Delegated authorisation ("Login with Google") |
| **OIDC (OpenID Connect)** | Identity layer on top of OAuth 2.0 — provides authentication |
| **Kerberos** | Internal enterprise SSO — Active Directory's authentication protocol |

#### Security Considerations

- SSO is a high-value target — compromise of the IdP gives access to all connected systems
- MFA on the SSO IdP is critical — it is the single authentication gate for everything
- Session token security — tokens must be short-lived, securely stored, and revocable
- SSO logout must propagate to all connected applications — "single sign-out"

---

## Architecture and Relationships

### Full IAAA Flow

```
User claims identity (Identification)
  e.g., enters username: "alice"
         |
         v
User proves identity (Authentication)
  e.g., enters password + TOTP code
  System checks: is password correct? is TOTP valid?
         |
         v
System evaluates permissions (Authorisation)
  e.g., RBAC: alice has "Sales" role → access to sales resources
         |
         v
User accesses resource
         |
         v
Action is logged (Accountability)
  e.g., SIEM receives: alice accessed /sales/report.xlsx at 14:32 from 192.168.1.45
```

### Access Control Model Selection

```
Shared personal files (family, small team)
         |
         v
         DAC (owner controls sharing manually)

Enterprise application access (hundreds of users, defined job roles)
         |
         v
         RBAC (manage roles, not individual users)

Government classified systems (strict data compartmentalisation)
         |
         v
         MAC (system enforces classification labels; users cannot override)
```

---

## Security Engineer Perspective

- **MFA on privileged accounts is non-negotiable** — compromise of an admin account without MFA is a critical finding in any assessment
- **Principle of least privilege** — users and service accounts should have only the access they need for their current role; nothing more
- **Periodic access reviews** — RBAC roles accumulate over time; former employees or role-changers retain access they no longer need (access creep). Regular reviews clean this up
- **Privileged Access Management (PAM)** — admin credentials should be stored in a PAM vault, checked out per session, and recorded — never shared or reused
- **Log everything, retain appropriately** — PCI DSS requires 1 year log retention; HIPAA requires 6 years; GDPR requires evidence of lawful processing
- **SSO reduces attack surface** — fewer credential stores means fewer breach points; but secure the IdP with MFA and monitor it heavily
- **Service accounts** — non-human accounts (applications, scripts) are frequently over-privileged and lack MFA; they are common lateral movement targets

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **2FA** | Two-Factor Authentication — using exactly two authentication factors |
| **Access Control** | Technical mechanism that enforces authorisation policies |
| **Accountability** | Logging and audit capability enabling users to be held responsible for their actions |
| **AppArmor** | Linux MAC implementation; profile-based; ships with Debian/Ubuntu |
| **Authentication** | Verifying the claimed identity of a user or system |
| **Authorisation** | Defining what an authenticated user is permitted to access or do |
| **Biometrics** | Authentication using physical characteristics — fingerprint, face, retina, voice |
| **DAC** | Discretionary Access Control — resource owner controls access permissions |
| **FIDO2/WebAuthn** | Standards for hardware key-based phishing-resistant authentication |
| **Hardware Security Key** | Physical device (USB/NFC) used as a second authentication factor |
| **HMAC** | Hash-based MAC — hash with a secret key providing integrity and authenticity |
| **IAM** | Identity and Access Management — managing identities and access rights |
| **IdM** | Identity Management — managing digital identities and their attributes |
| **Identification** | A user claiming an identity — no proof required at this stage |
| **Log Forwarding** | Sending logs to a centralised server for storage and analysis |
| **MAC** | Mandatory Access Control — system-enforced access based on security labels |
| **MFA** | Multi-Factor Authentication — two or more distinct authentication factors |
| **Nonce** | A number used once — prevents replay attacks |
| **OIDC** | OpenID Connect — identity layer over OAuth 2.0 for SSO |
| **PAM** | Privileged Access Management — controls and audits access to admin/privileged accounts |
| **PIN** | Personal Identification Number — numeric something-you-know factor |
| **Principle of Least Privilege** | Users should have only the minimum access required for their role |
| **RBAC** | Role-Based Access Control — access granted by role membership |
| **Replay Attack** | Capturing and resubmitting a valid authentication message to authenticate without knowing the password |
| **SAML** | Security Assertion Markup Language — XML-based SSO protocol |
| **SELinux** | Security-Enhanced Linux — flexible MAC implementation; standard on RHEL/Fedora |
| **SIEM** | Security Information and Event Management — log aggregation and threat detection |
| **SSO** | Single Sign-On — authenticate once, access all authorised systems |
| **TOTP** | Time-Based One-Time Password — authenticator app generates time-limited codes |

---

## Exam and Interview Revision

### Must Remember

- IAAA = **Identification → Authentication → Authorisation → Accountability**
- Identification = claiming identity; Authentication = proving identity
- Three primary auth factors: **something you know, have, are**
- MFA = two or more factors from different categories
- **SMS OTP < TOTP < Hardware key** in terms of MFA security
- Authorisation defines the policy; Access Control enforces it
- DAC = owner controls; RBAC = role membership; MAC = system enforces
- RBAC scales better than DAC for enterprises
- MAC is used for classified/high-security environments (government, military)
- Accountability requires **tamper-resistant, centralised logs**
- SIEM = aggregates logs, correlates events, generates alerts, provides forensic history
- Replay attack = capturing and reusing a valid authentication token — mitigated by nonces and timestamps
- SSO = authenticate once, access everything — IdP is a critical high-value target requiring strong MFA
- IdM = managing identity records; IAM = broader (includes access governance, compliance)
- Principle of least privilege — always assign minimum required access

### Common Interview Questions

| Question | Key Points |
|----------|-----------|
| What is the difference between authentication and authorisation? | Authentication proves identity. Authorisation defines what that identity can access. You must be authenticated before you can be authorised. |
| What are the three primary authentication factors? | Something you know (password), something you have (hardware key), something you are (biometric). MFA combines two or more from different categories. |
| Why is SMS OTP weaker than a hardware key? | SMS is vulnerable to SIM-swapping — attacker ports your number to their SIM. Hardware FIDO2 keys are phishing-resistant — they only respond to the legitimate domain. |
| What is the difference between DAC, RBAC, and MAC? | DAC: owner controls access. RBAC: role membership determines access — scales well for enterprises. MAC: system enforces access via security labels — users have no discretion. |
| What is a replay attack and how is it prevented? | Attacker captures a valid auth token and resubmits it. Prevented by nonces (used-once values) and timestamps — the server rejects tokens it has seen before or that are too old. |
| What is SSO and what is the security risk? | Authenticate once, access all connected systems. Risk: the IdP becomes a single high-value target — compromise it and you have access to everything. Mitigated by strong MFA on the IdP. |
| What is the purpose of log forwarding? | Centralise logs for correlation and analysis; prevent attackers from covering their tracks by deleting local logs; feeds SIEM for real-time detection. |
| What is the difference between IdM and IAM? | IdM manages identity records and authentication. IAM is broader — includes access governance, role management, compliance, and access lifecycle management. |
| What is the principle of least privilege? | Users and service accounts should only have the minimum access required for their current role. Excess permissions increase blast radius of compromise. |
| What is PAM and why does it matter? | Privileged Access Management — controls, vaults, and audits admin credentials. Prevents shared admin passwords, ensures session recording, enables just-in-time access. Admin accounts are prime attacker targets. |
