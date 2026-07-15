# OWASP Top 10 2025: Application Design Flaws

## Overview

This room covers four OWASP Top 10:2025 categories that relate to failures in architecture, system design, and the environment in which applications are deployed. Unlike code-level bugs, many of these vulnerabilities exist because of how systems are configured, what components they depend on, how cryptography is applied, or what assumptions were baked in during the design phase. These categories are:

- **A02: Security Misconfigurations**
- **A03: Software Supply Chain Failures**
- **A04: Cryptographic Failures**
- **A06: Insecure Design**

---

## Topics Covered

- Security Misconfigurations: what they are, common patterns, prevention, and lab walkthrough
- Software Supply Chain Failures: dependency risks, SolarWinds context, prevention
- Cryptographic Failures: weak algorithms, hardcoded keys, ECB mode, lab walkthrough
- Insecure Design: logic flaws, AI-era risks, prevention, lab walkthrough

---

## Key Concepts

### A02: Security Misconfigurations

#### What It Is

Security misconfigurations occur when systems, servers, or applications are deployed with unsafe defaults, incomplete settings, or unnecessarily exposed services. These are not bugs in the code itself — they are mistakes in how the environment and deployment are configured. They create easy, low-effort entry points for attackers.

#### Why It Matters

Modern applications rely on complex stacks — cloud services, third-party APIs, containerised deployments. A single exposed admin panel, publicly accessible storage bucket, or overly permissive IAM role can compromise an entire system without requiring any code exploitation.

Real-world example: In 2017, Uber exposed an AWS S3 bucket containing sensitive driver and rider data. The bucket was publicly accessible — no credentials required. Attackers could download the data directly. This was purely a configuration failure.

#### Common Patterns

- Default credentials or weak passwords left unchanged
- Unnecessary services or endpoints exposed to the internet
- Cloud storage with public access (S3 buckets, Azure Blob, GCP buckets)
- Missing authentication or authorisation on API endpoints
- Verbose error messages that expose stack traces, framework versions, or internal paths
- Outdated software, frameworks, or containers with known CVEs
- Exposed AI/ML inference endpoints without access controls

#### Prevention

- Harden default configurations and remove all unused features, services, and endpoints
- Enforce least privilege across all systems and cloud permissions
- Limit network exposure — segment sensitive resources
- Keep all software and dependencies patched
- Strip stack traces and system information from production error responses
- Audit cloud permissions and storage configurations regularly
- Integrate automated configuration checks into the CI/CD deployment pipeline

#### Lab Walkthrough

**Endpoint:** `GET /api/user/<user_id>`

The endpoint was intended to accept numeric user IDs. However, when a non-numeric value (`admin`) was provided:

```bash
curl http://TARGET:5002/api/user/admin
```

The application returned a verbose internal error response — exposing implementation details instead of returning a clean, safe error message.

**Vulnerabilities identified:**
- No input type validation
- Verbose error output in production
- Poor API configuration

**Flag:** `THM{V3RB0S3_3RR0R_L34K}`

---

### A03: Software Supply Chain Failures

#### What It Is

Software supply chain failures occur when applications depend on components, libraries, packages, or AI models that are compromised, outdated, or not properly verified. The vulnerability is not in your code — it is in what your code depends on.

#### Why It Matters

Modern applications are built from dozens to hundreds of third-party dependencies. A single compromised package gives an attacker the same level of access as if they had compromised your own codebase — without ever needing to attack you directly. Supply chain attacks are increasingly automated, making them hard to detect and very high impact.

Real-world example: The **SolarWinds Orion** compromise (2021). Attackers inserted malicious code into a trusted software update package. Thousands of organisations installed the update automatically, giving attackers a persistent foothold in their networks. The flaw was in the build and distribution process — not in SolarWinds' core application logic.

In AI contexts, unverified third-party models or fine-tuned datasets can embed backdoors, hidden behaviours, or biased outputs — compromising systems that rely on them.

#### Common Patterns

- Using unverified, unmaintained, or outdated libraries
- Automatically installing updates without integrity verification
- Depending on third-party AI models without monitoring or auditing
- Insecure CI/CD pipelines that allow tampering at the build stage
- No cryptographic verification (checksums, signatures) for packages or updates
- No monitoring for newly disclosed vulnerabilities in deployed dependencies

#### Prevention

- Verify all third-party components, libraries, and AI models before use
- Monitor and patch dependencies continuously — not just at initial deployment
- Sign, verify, and audit all software updates and packages using cryptographic checksums
- Lock down CI/CD pipelines — restrict who and what can modify the build process
- Track provenance and licensing for all dependencies
- Implement runtime monitoring for anomalous behaviour from dependencies

#### Lab Walkthrough

The application imported a local library (`lib/vulnerable_utils.py`) without verification. Inside the library, a `debug_info()` function exposed sensitive internal data. The application's `/api/process` endpoint passed user input directly to the library without restriction.

The debug trigger was found by reviewing the source code:

```python
if data == 'debug':
    return jsonify(debug_info())
```

**Exploitation:**
```bash
curl -X POST http://TARGET:5003/api/process \
  -H "Content-Type: application/json" \
  -d '{"data":"debug"}'
```

The response returned sensitive debug information from the unverified dependency.

**Vulnerabilities identified:**
- Unverified third-party library imported and trusted implicitly
- Debug functionality exposed in production via the library
- No input restriction preventing access to the debug path

**Flag:** `THM{SUPPLY_CH41N_VULN3R4B1L1TY}`

---

### A04: Cryptographic Failures

#### What It Is

Cryptographic failures occur when sensitive data is not properly protected — either because encryption is absent, a weak/deprecated algorithm is used, keys are poorly managed, or cryptographic material is exposed. This allows attackers to access data that should be private.

This category covers:
- Storing passwords without hashing (or using weak hashes like MD5)
- Using deprecated algorithms (MD5, SHA-1, DES, AES-ECB)
- Hardcoded secrets in source code, configuration files, or client-side scripts
- Transmitting sensitive data without encryption (HTTP instead of HTTPS)
- Poor key management and rotation practices

#### ECB Mode — Why It Is Dangerous

AES-ECB (Electronic Codebook) mode encrypts each block independently with the same key. This means identical plaintext blocks produce identical ciphertext blocks — patterns in the plaintext are preserved in the ciphertext. ECB should never be used for encrypting real data. Preferred modes: AES-GCM, ChaCha20-Poly1305.

#### Hardcoded Keys — Why They Are Dangerous

Embedding a secret key in client-side JavaScript means every person who visits the page can retrieve it using browser DevTools or a simple curl request. There is no security boundary.

#### Prevention

- Use strong, modern algorithms: AES-GCM, ChaCha20-Poly1305, TLS 1.3
- Use secure key management services: AWS KMS, Azure Key Vault, HashiCorp Vault
- Never hardcode secrets in source code, configuration files, or client-side scripts
- Hash passwords with slow, adaptive functions: bcrypt, scrypt, Argon2
- Rotate keys regularly with a documented key lifecycle policy
- Maintain an inventory of all certificates and keys with expiry tracking

#### Lab Walkthrough

Reviewing the page source revealed a link to `/static/js/decrypt.js`. Fetching this file exposed:

```javascript
const SECRET_KEY = "my-secret-key-16";
const ENCRYPTION_MODE = "ECB";
```

The key was hardcoded in client-side JavaScript. The encrypted document was Base64-encoded AES-ECB ciphertext visible on the page.

**Decryption script:**

```python
from Crypto.Cipher import AES
import base64

key = b"my-secret-key-16"
enc = "Nzd42HZGgUIUlpILZRv0jeIXp1WtCErwR+j/w/lnKbmug31opX0BWy+pwK92rkhjwdf94mgHfLtF26X6B3pe2fhHXzIGnnvVruH7683KwvzZ6+QKybFWaedAEtknYkhe"

ciphertext = base64.b64decode(enc)
cipher = AES.new(key, AES.MODE_ECB)
plaintext = cipher.decrypt(ciphertext).rstrip(b"\x00").decode('utf-8')
print(plaintext)
```

**Vulnerabilities identified:**
- Secret key hardcoded in client-side JavaScript
- Weak encryption mode (AES-ECB)
- Cryptographic material fully exposed on the public client side

**Flag:** `THM{CRYPTO_FAILURE_H4RDCOD3D_K3Y}`

---

### A06: Insecure Design

#### What It Is

Insecure design occurs when flawed logic or unsafe assumptions are baked into a system's architecture from the start. Unlike misconfigurations (deployment errors) or implementation bugs (code errors), insecure design cannot be patched — it requires rethinking how the system works.

Sources of insecure design:
- Skipped or inadequate threat modelling during development
- No defined security requirements for features
- Incorrect assumptions about how users (or attackers) will interact with the system
- Test or debug bypasses left accessible in production

In the AI era, new forms of insecure design include:
- **Prompt injection** — user input blended with system prompts, allowing attackers to hijack context or extract hidden data
- **Blind trust in model output** — acting on AI decisions without validation or human review
- **Poisoned models** — models fine-tuned on unsafe data or pulled from unverified sources, embedding hidden behaviours

Real-world example: **Clubhouse** — the early design assumed users would only access the platform through the mobile app. The backend API, however, had no proper authentication. Researchers could query user data, room information, and private conversations directly against the API without any credentials.

#### Common Insecure Design Patterns in 2025

- Weak business logic for recovery, approval, or account management flows
- Assumptions that only official clients will access APIs
- AI components with unchecked authority — LLMs that can take actions without validation
- Debug, test, or admin bypasses shipped to production
- No abuse-case review or adversarial design thinking

#### Prevention

- Build threat modelling into every stage of development, not just initial design
- Define security requirements per feature before implementation
- Apply least privilege across users, APIs, AI agents, and services
- Treat every AI model as untrusted until proven otherwise
- Validate and filter all model inputs and outputs
- Separate system prompts from user-controlled content
- Require human review for high-risk automated actions
- Continuously test for logic flaws and abuse paths as features are added

#### Lab Walkthrough

The frontend restricted access to mobile users only — desktop users were shown a restricted UI. However, the backend APIs had no authentication at all.

**Step 1 — Access the users endpoint directly:**
```bash
curl http://TARGET:5005/api/users
```
Response included a list of users with an admin account visible.

**Step 2 — Access admin messages directly:**
```bash
curl http://TARGET:5005/api/messages/admin
```
Returned private messages — including the flag — with no authentication required.

**Vulnerabilities identified:**
- Frontend-only access control — the backend enforced nothing
- API endpoints fully accessible without authentication
- Design assumed only official mobile clients would interact with the backend

**Flag:** `THM{1NS3CUR3_D35IGN_4SSUMPT10N}`

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Security Misconfiguration | Deployment or configuration error creating an exploitable weakness |
| Supply Chain Attack | Compromising a target by attacking a dependency, library, or tool they trust |
| CI/CD | Continuous Integration / Continuous Deployment — automated build and release pipeline |
| Cryptographic Failure | Sensitive data inadequately protected due to absent, weak, or misused cryptography |
| AES-ECB | Weak AES mode where identical plaintext blocks produce identical ciphertext blocks |
| AES-GCM | Modern, authenticated AES mode — preferred over ECB |
| Hardcoded Secret | A credential or key embedded directly in source code or a client-side file |
| Insecure Design | Architectural or logic flaws baked in from the design phase — cannot be patched |
| Threat Modelling | Systematic process of identifying threats and mitigations during design |
| Prompt Injection | Attacker-controlled input manipulating an AI system's prompt context |
| Least Privilege | Granting only the minimum permissions required for a task |

---

## Real-World Relevance

- **Security Misconfigurations** are the single most common finding across cloud security assessments — misconfigured S3 buckets, open database ports, and default credentials are discovered constantly
- **Supply chain attacks** have become one of the highest-impact attack vectors — the SolarWinds, 3CX, and XZ Utils incidents all demonstrate how a trusted dependency becomes a trusted backdoor
- **Cryptographic failures** underpin a large proportion of data breaches — weak password hashing (unsalted MD5) means a single database dump can expose millions of credentials via rainbow tables
- **Insecure design** is increasingly relevant as AI is integrated into applications — LLM-based systems with insufficient guardrails are a growing attack surface
- AES-ECB's weakness is visually demonstrable with the "ECB penguin" — encrypting an image with ECB reveals the original pattern in the ciphertext, making the flaw immediately tangible
- CI/CD pipeline security is a major focus area in modern DevSecOps — a compromised build pipeline is equivalent to a compromised codebase

---

## Key Learnings

- Security misconfigurations are deployment errors, not code bugs — verbose errors, exposed endpoints, and public buckets are all misconfigurations
- Supply chain failures exploit trust in dependencies — one compromised library can own the entire application
- Cryptographic failures include both absent encryption and incorrectly applied encryption — hardcoded keys and weak modes (ECB) are as dangerous as no encryption
- Insecure design cannot be fixed with a patch — it requires rethinking architecture and assumptions
- Frontend-only access control is not access control — all enforcement must happen server-side
- Threat modelling must happen throughout the development lifecycle, including for AI components

---

## Additional Notes

- The difference between A02 (Misconfiguration) and A06 (Insecure Design) is the origin of the flaw: misconfiguration is a deployment/setup error; insecure design is a fundamental architectural or logic error present before deployment
- The SolarWinds attack is the canonical supply chain example — the malicious `SUNBURST` backdoor was distributed in a signed, legitimate update package
- ECB mode is trivially distinguishable from secure modes — do not use it for any real data
- Prompt injection is a novel but rapidly growing attack class — OWASP has published a separate "OWASP Top 10 for LLM Applications" addressing AI-specific threats

---

## Conclusion

Application design flaws represent failures at a deeper level than individual bugs — they affect how the entire system is configured, what it trusts, how it protects data, and what assumptions it makes. Security misconfigurations, supply chain failures, cryptographic failures, and insecure design each require different mitigations: hardening deployments, verifying dependencies, using modern cryptography correctly, and embedding security thinking into the design process from the start. Together, they represent some of the most consistently exploited weaknesses across the modern application landscape.
