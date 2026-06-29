# DAST — Dynamic Application Security Testing

## Overview

Dynamic Application Security Testing (DAST) is the practice of testing a running application from the outside — sending requests, observing responses, and identifying vulnerabilities without access to the underlying source code. DAST is a black-box approach, making it closer to what a real attacker would do. It tests the application as it behaves at runtime, catching vulnerability classes that SAST cannot detect.

Where SAST finds vulnerabilities in code structure, DAST finds vulnerabilities in application behaviour. Both are necessary — they cover different ground, and a mature S-SDLC uses both.

---

## Topics Covered

- What DAST is and how it differs from SAST
- How DAST tools work (crawling, fuzzing, active scanning)
- What DAST can and cannot detect
- DAST in the CI/CD pipeline
- OWASP ZAP as a primary DAST tool
- Authenticated vs unauthenticated scanning

---

## Key Concepts

### SAST vs DAST

| Characteristic | SAST | DAST |
|---------------|------|------|
| Access to code | Yes (white box) | No (black box) |
| Tests | Source code at rest | Running application |
| When | Development, CI check | Late testing, staging, production |
| Finds | Code-level flaws, hardcoded secrets | Runtime behaviour, configuration issues |
| Runtime context | No | Yes |
| Authenticated testing | N/A | Possible with configuration |
| False positives | Higher (no runtime context) | Lower (tests actual behaviour) |

Neither replaces the other — SAST + DAST together provide significantly better coverage than either alone.

---

### How DAST Tools Work

**Crawling:**
The tool navigates the application like a browser — following links, submitting forms, identifying all accessible endpoints. The crawl phase builds a site map of the application's attack surface.

**Active Scanning / Fuzzing:**
For each discovered endpoint and input field, the tool sends a series of crafted payloads designed to trigger known vulnerability classes. If the response contains indicators of vulnerability (error messages, unexpected data, injected content), the tool flags it.

**Passive Scanning:**
The tool monitors requests and responses passing through a proxy without actively sending additional requests. Identifies issues like missing security headers, insecure cookies, and information disclosure without interacting with the application directly.

**Authentication Handling:**
DAST tools can be configured with credentials to perform authenticated scanning — testing areas of the application accessible only after login. Without this, only unauthenticated attack surface is tested.

---

### What DAST Detects

| Category | Examples |
|----------|---------|
| Injection | SQL injection, command injection, LDAP injection detected via response anomalies |
| XSS | Reflected and stored XSS via payload injection and response analysis |
| Security misconfigurations | Missing security headers, verbose error messages, directory listing |
| Broken access control | Parameter manipulation to access other users' data |
| Insecure cookies | Missing HttpOnly, Secure, SameSite flags |
| Sensitive data in responses | PII or tokens in API responses |
| Outdated software | Version disclosure via headers correlated with known CVEs |
| CSRF | Missing or bypassable CSRF tokens |
| Business logic flaws (some) | Numeric manipulation, skip-step order flows |

**What DAST cannot detect:**
- Hardcoded secrets (not exposed at runtime)
- Vulnerabilities in dead code paths never reached during the scan
- Logic flaws requiring specific knowledge of the application's expected business flow
- Memory corruption vulnerabilities (buffer overflows) in interpreted languages

---

### DAST in the SDLC

DAST runs later in the pipeline than SAST — it requires a deployed, running application:

| Environment | Typical DAST Usage |
|-------------|-------------------|
| Development | Developer runs ZAP against local instance (quick scan) |
| Staging | Full automated DAST scan in CI/CD pipeline |
| Pre-production | Authenticated scan with business-logic test cases |
| Production | Passive scan only — active scanning against production carries risk |

**Note on production scanning:** Active DAST scanning against production environments can trigger alerts, corrupt data, or cause unintended writes. It should be performed with explicit approval and typically during off-peak hours.

---

### OWASP ZAP (Zed Attack Proxy)

**Definition:** ZAP is the most widely used open-source DAST tool, maintained by OWASP. It functions as an intercepting HTTP proxy — sitting between the browser and the web server — and provides both passive and active scanning capabilities.

**Key Features:**
- Intercepting proxy for manual testing and request manipulation
- Spider (crawler) for automated application mapping
- Active scanner for automated vulnerability detection
- Ajax Spider for crawling JavaScript-heavy SPAs
- Passive scanner monitoring traffic through the proxy
- Fuzzer for targeted payload injection
- Scriptable via ZAP's script console (JavaScript, Python, Ruby)
- REST API for CI/CD integration

**ZAP Modes:**

| Mode | Purpose |
|------|---------|
| Standard | Default — all actions allowed |
| Safe | No potentially harmful requests sent — for production monitoring |
| Protected | Only scan targets explicitly in scope |
| ATTACK | Aggressive scanning — maximise vulnerability detection |

**ZAP Interface Sections:**

| Panel | Content |
|-------|---------|
| Sites tree | Discovered site structure and all requests |
| History | Every request and response passing through the proxy |
| Alerts | Vulnerability findings with severity and description |
| Active Scan | Configure and run active vulnerability scanning |
| Spider | Configure automated crawling |

---

### ZAP Automated Scan

The automated scan combines spidering and active scanning in a single workflow:

1. Navigate to Automated Scan
2. Enter the target URL
3. Choose: standard spider or Ajax spider (for JS-heavy apps)
4. Click Attack
5. ZAP spiders the application, then runs the active scanner against discovered endpoints
6. Review the Alerts tab for findings

**Note:** The automated scan is a starting point. Manual testing, authenticated scanning, and targeted fuzzing are required for comprehensive coverage.

---

### Authenticated Scanning

Testing only unauthenticated areas misses the majority of a modern web application's attack surface. ZAP supports several methods for authenticated scanning:

**Form-Based Authentication:**
Configure ZAP with the login form URL, username field, password field, and credentials. ZAP authenticates and maintains session state throughout the scan.

**Script-Based Authentication:**
For complex authentication flows (MFA, OAuth, CSRF tokens in login), custom scripts handle the authentication sequence.

**Session Management:**
Configure how ZAP detects when it has been logged out (e.g., presence of a login redirect) and re-authenticates automatically.

**Manual Session:**
Authenticate manually in ZAP's browser, then start scanning — ZAP inherits the established session cookies.

---

### Passive Scanning in ZAP

Passive scanning occurs automatically as traffic flows through the ZAP proxy — no additional configuration required.

**What passive scanning detects:**
- Missing security headers (`Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security`)
- Insecure cookies (missing HttpOnly, Secure, SameSite)
- Information disclosure in headers (Server version, X-Powered-By)
- Mixed content (HTTPS pages loading HTTP resources)
- Password fields submitted over HTTP
- Session token in URL (instead of header/cookie)

Passive scanning is safe for production — it only observes traffic without sending crafted payloads.

---

## Workflow / Process

```
Set up ZAP:
  Configure browser to use ZAP as HTTP proxy
  Import target domain into ZAP scope
        |
        v
Passive reconnaissance:
  Browse the application manually
  ZAP passively scans all traffic — no active probing yet
  Review passive alerts (missing headers, insecure cookies)
        |
        v
Spider / Crawl:
  Run standard spider for traditional web applications
  Run Ajax spider for JavaScript SPAs and React/Angular apps
  Review discovered URLs in the Sites tree
        |
        v
Active scanning:
  Configure scope to target application only
  Select scan policy (default, full, or custom)
  Run active scan against discovered endpoints
  ZAP sends payloads for injection, XSS, misconfiguration detection
        |
        v
Review alerts:
  Filter by risk level (High, Medium, Low, Informational)
  Manually verify each finding — confirm or mark as false positive
  Note: ZAP alerts are starting points for manual confirmation, not final reports
        |
        v
CI/CD integration:
  ZAP Docker container runs automated baseline scan in pipeline
  SARIF or XML report generated and uploaded to findings dashboard
  Pipeline fails if findings above threshold severity
```

---

## ZAP in CI/CD Pipelines

ZAP provides an official Docker image for headless CI/CD integration:

```bash
# Baseline scan (passive only — safe for production)
docker run -t owasp/zap2docker-stable zap-baseline.py -t https://target.example.com

# Full scan (active scanning — staging only)
docker run -t owasp/zap2docker-stable zap-full-scan.py -t https://target.example.com

# API scan (OpenAPI/Swagger/GraphQL definition)
docker run -t owasp/zap2docker-stable zap-api-scan.py -t https://target.example.com/api/openapi.json -f openapi
```

Output can be formatted as HTML, JSON, XML, or SARIF for integration with security dashboards and CI feedback.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| DAST | Dynamic Application Security Testing — testing running applications from the outside |
| Spider | Automated crawler that maps all accessible application endpoints |
| Ajax Spider | Crawler that executes JavaScript to map SPAs and dynamic content |
| Passive Scanning | Observing traffic without sending crafted payloads — safe for production |
| Active Scanning | Sending crafted payloads to actively probe for vulnerabilities |
| Fuzzing | Sending large numbers of varied inputs to find unexpected application behaviour |
| Intercepting Proxy | A proxy (like ZAP or Burp Suite) that sits between client and server to inspect/modify traffic |
| SARIF | Static Analysis Results Interchange Format — standard format for security finding reports |
| ZAP | OWASP Zed Attack Proxy — the primary open-source DAST tool |
| Authenticated Scan | DAST scan performed with valid session credentials to test authenticated functionality |

---

## Real-World Relevance

- DAST is a standard component of security assessments — penetration testers typically run ZAP or Burp Suite's scanner as part of initial automated coverage before manual testing begins
- Automated DAST in CI/CD pipelines is required by several security frameworks (PCI-DSS Requirement 6.3.2, NIST SP 800-53)
- ZAP is widely used in DevSecOps pipelines as a free alternative to commercial scanners (Burp Suite Pro, Checkmarx DAST)
- Bug bounty hunters use DAST tools to find the easy issues quickly before focusing on manual logic testing
- The combination of SAST (code level) + DAST (runtime level) + manual penetration testing gives the broadest vulnerability coverage

---

## Key Learnings

- DAST tests a running application — it sees runtime behaviour that SAST cannot
- DAST is lower false-positive than SAST because it tests actual responses, not code structure
- Unauthenticated scans miss most of the attack surface of a modern application — configure authentication
- ZAP's passive scan is safe for production; active scanning should only target staging or dev environments
- CI/CD integration via ZAP Docker containers brings DAST into the automated pipeline
- DAST findings still require manual verification — not all alerts are exploitable

---

## Conclusion

DAST fills the gap left by SAST — it tests how the application actually behaves when running, catching misconfigurations, injection vulnerabilities, and broken access controls that only manifest at runtime. OWASP ZAP provides both manual and automated DAST capabilities, making it accessible from developer local testing through to CI/CD pipeline integration. The most effective security programmes use SAST and DAST together: catching code-level flaws early and runtime behaviour flaws in staging, before either reach production.
