# Introduction to DevSecOps

## Overview

DevSecOps is the practice of integrating security tools, processes, and culture directly into DevOps workflows. Where traditional security was applied at the end of the development cycle as a gate, DevSecOps distributes security responsibilities throughout the pipeline — from the developer's IDE to production monitoring. The result is faster releases with fewer vulnerabilities reaching production, and a security posture that scales with development velocity rather than blocking it.

This room introduces the DevOps model, explains why bolting security on at the end fails, and covers how security is practically embedded into CI/CD pipelines through automation, tooling, and cultural shifts.

---

## Topics Covered

- The DevOps model and CI/CD pipelines
- Why traditional security models fail in DevOps environments
- The DevSecOps philosophy and "security as code"
- Security gates at each pipeline stage
- Tools integrated at each stage (SAST, DAST, SCA, secrets scanning, container scanning)
- Threat modelling in agile sprints
- Security culture: shared responsibility model

---

## Key Concepts

### DevOps and CI/CD

**DevOps** is a set of practices that combines software development (Dev) and IT operations (Ops) to shorten the development lifecycle and deliver continuous, high-quality software. The foundation is the CI/CD pipeline:

**Continuous Integration (CI):**
- Developers commit code frequently (multiple times per day) to a shared repository
- Each commit triggers an automated build and test process
- Issues are discovered and fixed immediately — before they accumulate

**Continuous Delivery/Deployment (CD):**
- Every successful CI build is automatically deployable (CD)
- In full Continuous Deployment, passing builds are automatically released to production
- The entire process is automated — from commit to running in production

**A typical CI/CD pipeline:**
```
Code Commit → Build → Unit Tests → Package → 
Deploy to Staging → Integration Tests → Deploy to Production
```

**The problem:** In a traditional security model, a security review happens once — near the end, before a major release. In DevOps, releases happen daily or continuously. A single security gate cannot keep pace.

---

### Why Traditional Security Fails in DevOps

| Traditional Model | DevOps Reality |
|------------------|---------------|
| Security reviewed at end of cycle | Releases happen multiple times per day |
| Security team owns security | Dozens of developers commit simultaneously |
| Manual penetration test before release | Manual testing cannot scale to daily releases |
| Vulnerability found late = expensive fix | Need to catch issues as early as possible |
| Security is a gate | Security must be a continuous process |

The fundamental mismatch: traditional security is a human-paced gate; DevOps moves at machine pace. DevSecOps resolves this by automating security checks so they run at machine pace within the pipeline.

---

### The DevSecOps Philosophy

**Core Principle:** Security is everyone's responsibility — not just the security team's.

**"Security as Code":** Security policies, controls, and checks are expressed as code — versioned, tested, and deployed like any other code. This makes security auditable, repeatable, and automated.

**"Shift Left":** Move security checks as early in the pipeline as possible. A finding in the IDE costs minutes to fix. The same finding in production can cost days.

**"Security by Default":** Secure defaults in code templates, IaC modules, container base images, and library choices mean developers start with security built in rather than added later.

**Shared Responsibility:**
- Developers are responsible for writing secure code (training, SAST in IDE, peer review)
- Security engineers are responsible for defining the controls, building the pipeline checks, and triaging findings
- Operations/Platform teams are responsible for infrastructure security (hardening, access control, secrets management)

---

### Security at Each Pipeline Stage

**1. Pre-Commit (Developer Machine):**
- SAST plugin in IDE (SonarLint, Semgrep) — real-time vulnerability feedback
- Pre-commit hooks — run secrets scanner and SAST on staged files before commit
- Tools: `git-secrets`, `detect-secrets`, Semgrep, SonarLint

**2. Commit / Source Control:**
- Peer code review with security checklist
- Pull Request security scan
- Branch protection rules preventing direct commits to main
- Tools: GitHub Code Scanning, GitLab SAST, Snyk Code

**3. Build:**
- Full SAST scan of the codebase
- Software Composition Analysis (SCA) — scan all third-party dependencies for known CVEs
- Secrets detection in codebase and history
- Tools: Semgrep, Checkmarx, Snyk Open Source, Dependabot, Trivy

**4. Test:**
- DAST against deployed application in staging
- API security scan (OWASP ZAP API scan, 42Crunch)
- Infrastructure-as-Code (IaC) scanning — Terraform, CloudFormation, Kubernetes manifests
- Tools: OWASP ZAP, Burp Suite, Checkov, tfsec, kube-bench

**5. Staging / Pre-Production:**
- Full DAST scan
- Container image scanning for OS and package vulnerabilities
- Penetration testing (manual, periodic)
- Compliance checks (CIS benchmarks, SOC 2 controls)
- Tools: Trivy, Clair, Aqua Security, AWS Inspector

**6. Production:**
- Runtime Application Self-Protection (RASP) — application-layer threat detection
- Security monitoring and alerting (SIEM, anomaly detection)
- Continuous vulnerability assessment
- Tools: Datadog Security, AWS GuardDuty, Falco (container runtime), Splunk

---

### Software Composition Analysis (SCA)

Modern applications are largely composed of open-source libraries. SCA scans the dependency tree — all direct and transitive dependencies — for known CVEs.

**Why it matters:** The 2021 Log4Shell vulnerability (CVE-2021-44228) affected any application using the log4j library, regardless of how secure the application code itself was. SCA would have flagged the vulnerable log4j version immediately.

**Common SCA tools:**
- **Snyk:** Developer-friendly SCA with IDE integration and fix PRs
- **Dependabot:** GitHub-native dependency update automation
- **OWASP Dependency-Check:** Open-source SCA scanner
- **Trivy:** Also handles container image scanning alongside SCA

**SCA in the pipeline:**
```bash
# Example: Snyk test in CI
snyk test --severity-threshold=high

# Example: OWASP Dependency-Check
dependency-check.sh --project "MyApp" --scan ./src --format HTML
```

---

### Secrets Management

Hardcoded secrets (API keys, passwords, tokens) in source code are one of the most common and severe findings in security assessments. DevSecOps addresses this at multiple levels:

**Prevention (shift left):**
- Pre-commit hooks scan for secrets before they enter version control
- IDE plugins warn when credentials are typed into code files
- Tools: `detect-secrets`, `git-secrets`, TruffleHog

**Detection:**
- Repository scanning to find secrets already committed (including in history)
- Tools: GitLeaks, TruffleHog, GitHub Secret Scanning

**Proper Secrets Management:**
- Secrets should be stored in dedicated secrets vaults, not in code, config files, or environment variables in plaintext
- Tools: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager
- Applications retrieve secrets at runtime via API call to the vault — no secret ever touches the codebase

---

### Infrastructure as Code (IaC) Security

As infrastructure is defined in code (Terraform, CloudFormation, Kubernetes YAML), security misconfigurations in that code become as important as vulnerabilities in application code.

**Common IaC security issues:**
- S3 buckets with public access enabled
- Security groups with 0.0.0.0/0 inbound rules
- Unencrypted EBS volumes or RDS instances
- Over-privileged IAM roles
- Missing logging configuration

**IaC scanning tools:**
- **Checkov:** Open-source IaC scanner supporting Terraform, CloudFormation, Kubernetes, ARM
- **tfsec:** Terraform-specific static analysis
- **kube-bench:** CIS Kubernetes Benchmark compliance checker
- **KICS:** Checkmarx open-source IaC scanner

---

### Container Security

Containers introduce new attack surfaces. Container security covers:

**Image Security:**
- Use minimal base images (Alpine, distroless) to reduce attack surface
- Scan container images for OS package vulnerabilities and application dependency CVEs
- Never run containers as root user
- Tools: Trivy, Clair, Snyk Container, Docker Scout

**Runtime Security:**
- Monitor container behaviour at runtime for anomalous syscalls or network activity
- Tools: Falco, Aqua Security

```bash
# Scan a container image with Trivy
trivy image nginx:latest

# Scan a local Dockerfile/image
trivy image myapp:1.0.0
```

---

## Workflow / Process

```
SDLC Phase → Security Activity → Tool
─────────────────────────────────────
Design       → Threat Modelling       → STRIDE, PASTA
             → Security Requirements  → OWASP ASVS
Development  → SAST (IDE)             → SonarLint, Semgrep
             → Secrets Detection      → detect-secrets, git-secrets
Pre-Commit   → Pre-commit SAST        → Semgrep, Husky
             → Secrets Hook           → git-secrets, detect-secrets
CI Build     → SAST (full)            → Semgrep, Checkmarx
             → SCA (dependencies)     → Snyk, Dependabot
             → IaC Scan               → Checkov, tfsec
             → Container Scan         → Trivy, Clair
Test Stage   → DAST                   → OWASP ZAP
             → API Scan               → ZAP API, 42Crunch
Pre-Prod     → Penetration Test       → Manual
Production   → Runtime Monitoring     → Falco, GuardDuty, SIEM
             → Continuous Scanning    → Snyk Monitor, AWS Inspector
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| DevSecOps | Integrating security into DevOps workflows and CI/CD pipelines |
| CI/CD | Continuous Integration / Continuous Delivery — automated build, test, deploy pipeline |
| Shift Left | Moving security activities earlier in the development process |
| Security as Code | Expressing security policies and controls as versioned, tested code |
| SCA | Software Composition Analysis — scanning third-party dependencies for known CVEs |
| IaC | Infrastructure as Code — defining infrastructure in Terraform, CloudFormation, etc. |
| RASP | Runtime Application Self-Protection — application-layer threat detection at runtime |
| Secrets Management | Storing and retrieving credentials via a dedicated vault rather than in code |
| Container Scanning | Checking container images for OS-level and application-level vulnerabilities |
| Pipeline Gate | A check in a CI/CD pipeline that can fail the build if a threshold is exceeded |

---

## Real-World Relevance

- DevSecOps adoption has grown sharply as organisations move to cloud-native architectures and rapid release cycles
- The SolarWinds supply chain attack (2020) highlighted that CI/CD pipeline integrity is itself a security concern — attackers inserted malicious code into the build process
- Tools like GitHub Dependabot, GitHub Code Scanning, and GitLab SAST are now included in default platform capabilities — DevSecOps is becoming the standard, not the exception
- The 2021 Twitch breach involved leaked source code — secrets scanning in the repository would have detected hardcoded credentials before they were exposed

---

## Key Learnings

- DevSecOps is not a tool — it is a culture and practice of shared security responsibility throughout the pipeline
- Manual security gates cannot scale to continuous delivery — automation is required
- Every stage of the CI/CD pipeline has a corresponding security tool category
- SCA is as important as SAST — third-party dependencies are a primary vulnerability source
- Secrets in code are a critical finding — prevent them with pre-commit hooks and manage secrets via vaults
- Container and IaC scanning extend security coverage to the entire application stack, not just the code

---

## Conclusion

DevSecOps resolves the fundamental tension between development velocity and security rigour by automating security checks at every stage of the CI/CD pipeline. Rather than blocking delivery with a manual security gate, it enables security to run at machine pace — flagging vulnerabilities as they are introduced, before they accumulate and harden. The shift from "security team owns security" to "security is everyone's responsibility" is a cultural change as much as a technical one, but the toolchain — SAST, SCA, DAST, container scanning, IaC analysis, and secrets management — provides the practical means to make that responsibility actionable.
