# Introduction to Cloud Security

## Executive Summary

Cloud computing has fundamentally changed how organisations build and operate technology infrastructure. This room provides a foundation in cloud security — covering service models (IaaS, PaaS, SaaS), deployment models (public, private, hybrid, community), the cloud data lifecycle, access management (IAM), security policies, network security controls, storage security, disaster recovery, and monitoring. AWS is used as the primary reference platform throughout.

**Why it matters:** Cloud misconfigurations are the leading cause of cloud data breaches. Public S3 buckets, over-permissive IAM roles, unencrypted storage, and missing logging are routine findings that expose sensitive data at scale. Security engineers operating in cloud environments must understand the shared responsibility model and the specific controls available in each service model.

**Where these concepts apply:** Every organisation using AWS, GCP, or Azure. Cloud-native applications, hybrid deployments, cloud-hosted databases, containerised workloads, and serverless architectures all depend on the cloud security foundations covered here.

---

## Big Picture Overview

Cloud computing shifts the responsibility boundary between the organisation and the provider:

```
On-Premises: Organisation responsible for everything
  Hardware → Network → Hypervisor → OS → Runtime → Data → Application

IaaS: Provider responsible for hardware/network/hypervisor
  Organisation responsible for OS → Runtime → Data → Application

PaaS: Provider responsible through OS and Runtime
  Organisation responsible for Data → Application

SaaS: Provider responsible for everything
  Organisation responsible for Data access controls only
```

This is the **Shared Responsibility Model** — the most important concept in cloud security. Misunderstanding where provider responsibility ends and customer responsibility begins is the root cause of many cloud breaches.

---

## Core Concepts

---

### Concept 1 — Cloud Characteristics and Service Models

**Key Cloud Characteristics:**

| Characteristic | Description |
|---------------|-------------|
| **Scalability** | Resources scaled up or down on demand — pay for what you use |
| **Simplicity** | Minimal configuration; fast provisioning via console or API |
| **Cost effective** | No upfront hardware investment; operational expense model |
| **Automation** | Limited human administration; programmable infrastructure |

**Service Models:**

#### IaaS — Infrastructure as a Service

**Provider responsibility:** Data centres, hardware, networking, hypervisors
**Customer responsibility:** Operating systems, middleware, runtime, applications, data

**Examples:** AWS EC2, Google Compute Engine, Azure Virtual Machines

**Security implication:** Customer must harden the OS, configure firewalls, manage patches, and secure applications. The provider secures the physical infrastructure.

#### PaaS — Platform as a Service

**Provider responsibility:** Everything in IaaS + OS and runtime environment
**Customer responsibility:** Application code and data

**Examples:** AWS Elastic Beanstalk, Google App Engine, Azure App Service, Heroku

**Security implication:** Customer focuses on application security and data security. OS patching and infrastructure hardening are provider responsibilities.

#### SaaS — Software as a Service

**Provider responsibility:** Everything — infrastructure, OS, runtime, application
**Customer responsibility:** Data governance, access control, user management, configuration

**Examples:** Microsoft 365, Salesforce, Google Workspace, Slack

**Security implication:** Customer controls who has access and what data is processed, but cannot audit or modify the application itself. Vendor security assessments and data processing agreements are critical.

---

### Concept 2 — Cloud Deployment Models

#### Public Cloud

Resources shared among multiple customers via virtualisation. Multiple organisations' workloads run on the same physical hardware, isolated by hypervisors.

**Risks:**
- **Vendor lock-in:** Difficult to migrate data and workloads to another provider
- **Competitor co-location:** Your data and your competitor's data may be on the same physical hardware (though isolated)
- **Privilege escalation:** Misconfigured multi-tenant environments could allow one tenant to access another's data

**Examples:** AWS, Google Cloud Platform (GCP), Microsoft Azure

#### Private Cloud

Resources dedicated to a single customer — no shared tenancy.

**Risks:**
- **Personnel threats:** Insider threats from provider's data centre staff
- **Natural disasters:** Physical concentration of data creates geographic risk
- **External attacks:** Same attack vectors as public cloud; no physical sharing benefits

#### Hybrid Cloud

Combination of public and private cloud. Sensitive production data in private; non-sensitive testing/development in public.

**Use case:** Organisation uses private cloud for production databases containing PII (compliance requirement) and public cloud for application testing (cost-effective).

#### Community Cloud

Infrastructure shared among organisations with common interests (same industry, regulatory requirements).

**Risks:**
- **Vulnerability propagation:** Vulnerable node in the community could expose other members
- **Policy enforcement:** Difficult to enforce consistent security baselines across all community members

---

### Concept 3 — Cloud Data Lifecycle

Every piece of data goes through six phases, each with distinct security requirements:

| Phase | Description | Security Requirements |
|-------|-------------|----------------------|
| **Create/Update** | Data is created or imported | SSL/TLS for transit; encryption; define ownership and classification |
| **Store** | Data persists in a database or storage | Encryption at rest; access controls; backup |
| **Use** | Data is accessed and processed | Secure connections; authentication; access restriction; secure virtualisation |
| **Share** | Data shared within or outside organisation | DLP controls; jurisdictional compliance; data classification enforcement |
| **Archive** | Long-term storage | Encryption; physical security; geographic location; backup procedures |
| **Destroy** | Data is permanently deleted | Crypto shredding (destroy encryption keys, not data); secure deletion verification |

**Data Classification:**

| Class | Sensitivity | Examples | If Exposed |
|-------|------------|---------|-----------|
| **Confidential** | Highest | PII, financial data, health records, IP | Regulatory penalties, reputational damage |
| **Internal** | Moderate | Business processes, internal communications | Moderate harm |
| **Public** | None | Marketing materials, public documentation | No consequence |

**Crypto Shredding:**
Instead of overwriting data (difficult at cloud scale), destroy the encryption keys that protect the data. Without keys, encrypted data is computationally irreversible — equivalent to secure deletion.

---

### Concept 4 — Identity and Access Management (IAM)

IAM is the "heart of access management" in cloud environments. It governs who can do what to which resources.

**IAM Core Components:**

| Component | Description |
|-----------|-------------|
| **Identities** | Users, groups, roles, service accounts — who or what is acting |
| **Resources** | AWS services and objects — EC2 instances, S3 buckets, RDS databases |
| **Entities** | Subset of resources used for authentication (users and roles) |
| **Principals** | Person or application requesting access to resources after sign-in |
| **Policies** | JSON documents defining allowed/denied actions on resources |

**AWS IAM Key Features:**
- Grant access to AWS resources to other accounts without sharing credentials
- Role-based access — least privilege principle enforcement
- MFA support for enhanced account security
- Cross-account access via IAM roles
- Comprehensive audit trail via CloudTrail

**Creating an IAM User with Administrative Access (AWS):**
```
1. IAM → Users → Add users
2. Enter username; enable console access
3. Create/select group with AdministratorAccess policy
4. Review and create
5. Save credentials (password, access key) securely
```

**Best Practices:**
- Root account: use only for initial setup; enable MFA; do not create access keys
- Use IAM roles for EC2 instances (not hardcoded access keys)
- Enforce MFA on all privileged accounts
- Rotate access keys regularly
- Review IAM permissions quarterly (access creep)

---

### Concept 5 — Cloud Security Policies

**Policy Types:**

| Type | Description | Example |
|------|-------------|---------|
| **Identity-based** | Attached to users/groups/roles — grants permissions | Allow IAM user to read S3 objects |
| **Resource-based** | Attached to resources — controls who accesses the resource | S3 bucket policy allowing specific accounts |
| **Session-based** | Temporary, time-limited permissions for specific operations | Temporary credentials valid for 1 hour |

**Policy Evaluation Logic:**
```
Request comes in
         ↓
Explicit Deny? → Yes → DENY
         ↓ No
Explicit Allow? → Yes → ALLOW
         ↓ No
DENY (implicit deny — default)
```

All access is denied by default in AWS. Every permission must be explicitly granted.

**Example Policy (deny all RDS access):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "rds:*",
      "Resource": "*"
    }
  ]
}
```

**Example Policy (time-based access):**
Add a condition restricting access to specific dates/times — useful for contractors, temporary access, and maintenance windows.

---

### Concept 6 — Cloud Network Security

**Three-Layer Network Security Model:**

#### Layer 1 — Security Groups

Security groups are instance-level virtual firewalls. They operate on a **default deny, explicit allow** model:
- No deny rules — absence of an allow rule = implicit deny
- Stateful — return traffic for allowed connections is automatically permitted
- Applied to individual EC2 instances, RDS instances, Lambda functions

```
Security Group "WebServer":
  Inbound: Allow TCP 443 from 0.0.0.0/0 (any internet)
  Inbound: Allow TCP 22 from 10.0.0.0/8 (internal only)
  Outbound: Allow all (default)
```

#### Layer 2 — Network Access Control Lists (NACLs)

NACLs operate at the VPC subnet level, protecting all instances in a subnet:
- **Can have explicit deny rules** (unlike Security Groups)
- Stateless — must define rules for both inbound and outbound, including return traffic
- Rules evaluated in numerical order; first match wins

**Security Group vs NACL:**

| Characteristic | Security Group | NACL |
|---------------|---------------|------|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Deny rules | No | Yes |
| Rule evaluation | All rules | Numbered order, first match |
| Use case | Per-instance control | Subnet-wide control, blocking specific IPs |

**Creating a NACL Rule to Block SSH (Port 22):**
```
VPC → Network ACLs → Create ACL
→ Edit Inbound Rules → Add rule:
  Rule #: 100
  Type: SSH (22)
  Source: 0.0.0.0/0
  Allow/Deny: Deny
→ Save
```

#### Layer 3 — Vendor-Specific Security Solutions

- **AWS DNS Firewall:** Filters DNS queries from VPC resources — blocks malware C2 via DNS
- **AWS Network Firewall:** Managed stateful firewall for VPC-level inspection
- **AWS Shield:** DDoS protection (Standard free; Advanced paid)
- **AWS WAF:** Web Application Firewall for CloudFront and ALB

---

### Concept 7 — Cloud Storage Security

**AWS Storage Services:**

| Service | Type | Use Case |
|---------|------|---------|
| **S3** | Object storage | Files, backups, static websites, data lakes |
| **RDS** | Managed relational database | MySQL, PostgreSQL, Aurora |
| **DynamoDB** | Managed NoSQL | Key-value, document storage |
| **EFS** | Managed file system | Shared file storage for EC2 |
| **ElastiCache** | In-memory cache | Redis, Memcached |

**Storage Security Best Practices:**

| Control | Implementation |
|---------|---------------|
| Encryption at rest | Enable server-side encryption (SSE) — AWS managed keys (SSE-S3) or customer managed (SSE-KMS) |
| Encryption in transit | TLS for all storage API calls (enforced via bucket policy requiring `aws:SecureTransport`) |
| Access policies | Bucket policies restricting access to specific IAM principals; no public access |
| Versioning | Enable S3 versioning to recover from accidental deletion or ransomware |
| Access logging | Enable S3 access logs and CloudTrail data events |
| MFA Delete | Require MFA for bucket deletion and version deletion |

**Creating an S3 Bucket with Encryption:**
```
1. S3 → Create bucket
2. Bucket name (globally unique)
3. AWS region
4. Enable server-side encryption → SSE-S3 (Amazon S3 managed keys)
   OR SSE-KMS (customer managed via AWS KMS)
5. Create bucket
```

**Public S3 Bucket Risk:**
Public S3 buckets are among the most common cloud security failures. Any object in a public bucket is accessible to anyone on the internet by URL. AWS now defaults to blocking public access — verify this setting is enabled for all buckets.

---

### Concept 8 — Disaster Recovery in the Cloud

**Cloud Disaster Recovery (CDR) Approaches:**

| Approach | Description | RTO | Cost |
|----------|-------------|-----|------|
| **Cold DR** | Store snapshots/backups; restore when needed | Hours to days | Lowest |
| **Warm DR** | Near-real-time replication; standby environment not actively running | Minutes to hours | Medium |
| **Hot DR** | Active-active configuration with load balancing; instant failover | Near-zero | Highest |

**Key Terms:**
- **RTO (Recovery Time Objective):** Maximum acceptable time to restore service after a disaster
- **RPO (Recovery Point Objective):** Maximum acceptable data loss (how far back the restore point is)

**Hot DR Architecture:**
```
Production Site (Region 1)          DR Site (Region 2)
  ├── EC2 instances                    ├── EC2 instances (active)
  ├── RDS (primary)    ←sync→          ├── RDS (read replica)
  └── S3 (primary)    ←replication→   └── S3 (replicated)
           ↑                                   ↑
           └──── Load Balancer ────────────────┘
                  (routes traffic to both sites)
```

---

### Concept 9 — Cloud Monitoring and Logging

**AWS Monitoring and Logging Services:**

| Service | Purpose |
|---------|---------|
| **IAM Credential Report** | Lists all IAM users and credential status (MFA, key age, last used) |
| **AWS CloudTrail** | Logs all API calls to AWS services — who did what, when, from where |
| **Amazon CloudWatch** | Monitors resource utilisation, application performance, custom metrics |
| **Amazon GuardDuty** | Continuous threat detection — analyses CloudTrail, VPC Flow Logs, DNS logs for malicious activity |
| **AWS Security Hub** | Aggregates security findings from multiple AWS services and third-party tools |

**CloudTrail — Essential for Security:**
Every API call to any AWS service is logged:
- Who made the call (IAM user, role, or account)
- What API was called (e.g., `DeleteBucket`, `CreateUser`)
- When it was called (timestamp)
- Where it came from (source IP)
- What the outcome was (success or error)

Without CloudTrail, it is impossible to investigate a security incident in AWS.

**Generating an IAM Credential Report:**
```
IAM → Credential Report → Download Report (CSV)

Report contains:
  - user_name
  - password_last_used
  - password_last_changed
  - mfa_active (true/false)
  - access_key_1_last_used_date
  - user_creation_time
```

Reviewing this report regularly identifies: accounts without MFA, dormant accounts with active keys, keys that have not been rotated.

---

### Concept 10 — Cloud Updates and Patch Management

**AWS Systems Manager Patch Manager:**
Automates OS and application patching across AWS infrastructure:
- Patch policies define which patches to install (security updates, critical updates)
- Scan option: identify missing patches across all managed instances
- Install option: automatically apply patches on schedule
- Supports EC2 instances running Windows and Linux

**Patch Management Workflow:**
```
Define Patch Baseline (what to patch and when)
         ↓
Configure Maintenance Window (when to apply patches)
         ↓
Patch Manager scans instances for missing patches
         ↓
Applies patches during maintenance window
         ↓
Reports patching status and compliance
```

---

## Architecture and Relationships

### AWS Security Architecture

```
Internet
    ↓
Route 53 (DNS Firewall)
    ↓
CloudFront / WAF (DDoS protection, web filtering)
    ↓
VPC (Virtual Private Cloud)
  ├── Public Subnet
  │   └── Internet Gateway → ALB → NACL → Security Group → EC2 (web tier)
  ├── Private Subnet
  │   └── NACL → Security Group → EC2 (app tier)
  └── Private Subnet
      └── NACL → Security Group → RDS (data tier)

Monitoring:
  CloudTrail → logs all API calls
  CloudWatch → monitors metrics and alerts
  GuardDuty → threat detection
  Security Hub → centralised findings
```

### Shared Responsibility Model

```
Customer Responsibility:
  Data, Classification, IAM policies
  OS patches (IaaS), Application security
  Network controls (Security Groups, NACLs)
  Encryption configuration
  Monitoring and logging setup

AWS Responsibility:
  Physical data centres
  Hardware (servers, networking, storage)
  Hypervisor
  Managed service infrastructure
  Physical security
```

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **CDR** | Cloud Disaster Recovery — strategies for maintaining business continuity using cloud |
| **CloudTrail** | AWS service logging all API calls to AWS services |
| **CloudWatch** | AWS monitoring service for metrics, logs, and alerts |
| **Crypto shredding** | Destroying encryption keys to render encrypted data permanently inaccessible |
| **GuardDuty** | AWS threat detection service analysing logs for malicious activity |
| **IaaS** | Infrastructure as a Service — provider manages hardware; customer manages OS and above |
| **IAM** | Identity and Access Management — controls who can access AWS resources |
| **NACL** | Network Access Control List — subnet-level stateless firewall in AWS VPC |
| **PaaS** | Platform as a Service — provider manages infrastructure and OS |
| **RPO** | Recovery Point Objective — maximum acceptable data loss after a disaster |
| **RTO** | Recovery Time Objective — maximum acceptable time to restore service |
| **S3** | Simple Storage Service — AWS object storage |
| **SaaS** | Software as a Service — provider manages everything; customer manages data access |
| **Security Group** | Instance-level virtual firewall in AWS — stateful, default deny |
| **Shared Responsibility Model** | Division of security responsibilities between cloud provider and customer |
| **SSE** | Server-Side Encryption — encryption applied by the cloud provider to stored data |
| **VPC** | Virtual Private Cloud — isolated virtual network in AWS |

---

## Exam and Interview Revision

### Must Remember

- Shared Responsibility Model: provider secures physical infrastructure; customer secures data and access configuration
- IaaS: customer manages OS and above. PaaS: customer manages app and data. SaaS: customer manages data access only
- Cloud data lifecycle: Create → Store → Use → Share → Archive → **Destroy** (crypto shredding)
- IAM default: **implicit deny** — all access denied unless explicitly allowed
- Security Groups: stateful, no deny rules, instance-level
- NACLs: stateless, can have deny rules, subnet-level
- S3 public access = critical misconfiguration — verify Block Public Access is enabled
- CloudTrail logs ALL API calls — essential for incident investigation and forensics
- GuardDuty provides continuous threat detection — analyses CloudTrail, VPC Flow Logs, DNS logs
- IAM Credential Report: identify accounts without MFA, stale access keys, dormant accounts
- Hot DR = near-zero RTO; Cold DR = lowest cost, highest RTO
- Crypto shredding: destroy the encryption key; encrypted data becomes permanently inaccessible

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is the Shared Responsibility Model? | Division of security duties between cloud provider and customer. Provider secures physical infrastructure; customer secures what they deploy on it. Scope of customer responsibility decreases from IaaS to PaaS to SaaS. |
| What is the difference between Security Groups and NACLs? | Security Groups: stateful (return traffic auto-allowed), no deny rules, instance-level. NACLs: stateless (must allow return traffic explicitly), can deny, subnet-level. |
| What is crypto shredding? | Destroying the encryption keys that protect encrypted data, rendering the data permanently inaccessible. Used in cloud to "delete" data without physically overwriting storage. |
| Why is CloudTrail important? | Logs every API call to every AWS service — provides a complete audit trail. Essential for incident investigation (who deleted what, when, from where), compliance, and forensics. |
| What is the IAM default-deny principle? | All access to AWS resources is denied by default. Permissions must be explicitly granted via IAM policies. Even an IAM user with no policies cannot do anything. |
| What does GuardDuty detect? | Malicious activity based on analysis of CloudTrail logs, VPC Flow Logs, and DNS logs. Detects: unusual API calls, suspicious IAM activity, port scanning, cryptocurrency mining, compromised credentials, C2 communications. |
