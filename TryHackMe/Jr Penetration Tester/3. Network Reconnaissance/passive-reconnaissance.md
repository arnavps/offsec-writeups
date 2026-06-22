# Passive Reconnaissance

## Overview

Passive reconnaissance is the discipline of gathering intelligence about a target exclusively from publicly available sources, without sending a single packet to the target system. This room covers the foundational passive recon toolkit used in penetration testing, bug bounty hunting, and threat intelligence work. Even as privacy regulations tighten, a significant amount of operationally useful information remains publicly exposed through DNS, WHOIS/RDAP, certificate transparency logs, and internet-facing device search engines.

**Main objectives:**
- Understand the distinction between passive and active reconnaissance and why it matters
- Use WHOIS and RDAP to extract domain registration intelligence
- Query DNS records using `nslookup` and `dig` against public resolvers
- Discover subdomains via DNSDumpster and Certificate Transparency logs (crt.sh)
- Enumerate exposed services using Shodan.io

**Skills introduced:** WHOIS enumeration, DNS querying, subdomain discovery, certificate log mining, Shodan

---

## Concepts Covered

### Passive vs Active Reconnaissance

**Definition:** Reconnaissance (recon) is the preliminary phase of a penetration test or attack where the goal is to learn as much about the target as possible before taking any action against it.

**Why It Matters:** The intelligence gathered in this phase directly shapes every decision that follows — which services to attack, which credentials to attempt, which vulnerabilities to research. Skipping or rushing recon leads to missed opportunities and incomplete coverage.

**Passive Reconnaissance:**
- Uses only publicly available information
- No packets sent to the target
- Analogy: observing a target through binoculars from a safe distance
- Examples: WHOIS lookups, DNS queries to public resolvers, certificate log searches, Shodan queries, LinkedIn scraping, GitHub searches

**Active Reconnaissance:**
- Requires direct interaction with the target or its personnel
- Leaves traces: log entries, IDS alerts, firewall blocks
- Examples: Nmap port scans, web directory brute-forcing, phishing, tailgating, social engineering calls
- Any direct interaction with a person affiliated with the target counts as active — even attending an event and asking employees about their tech stack

**Practical Relevance:** Passive recon is the preferred starting point in all authorised engagements. It is low-risk (no detection risk to the tester), carries no legal risk when done ethically, and frequently uncovers more than testers expect. The defender equivalent is monitoring your own passive footprint via Shodan alerts, CT log watchers, and automated OSINT tools.

---

### WHOIS and RDAP

**Definition:** WHOIS is a query/response protocol (RFC 3912) that retrieves registration details for domain names. Servers listen on TCP port 43.

**Why It Matters:** WHOIS can reveal registrar, registration/expiry dates, name servers, and historically personal contact information. Even with GDPR-era privacy redactions, the data that remains is operationally useful: dates (for timing social engineering), registrar identity, name servers (potential secondary targets), and abuse contacts.

**Key Details:**
- **Registrar:** the company that registered the domain (Namecheap, GoDaddy, etc.)
- **Dates:** Creation, Updated, Expiration — useful for estimating company age and timing
- **Name servers:** authoritative DNS servers — potential next targets if in scope
- **Status codes:** e.g., `clientTransferProhibited` means domain is locked against unauthorised transfers
- Personal details are now typically replaced by privacy service placeholders under GDPR/CCPA
- Historical WHOIS (whoxy.com) can reveal previous owners and registrar changes

**RDAP (Registration Data Access Protocol):** ICANN officially replaced WHOIS with RDAP for gTLDs as of 28 January 2025. RDAP uses HTTPS, returns structured JSON, supports internationalisation, and provides better privacy controls. Many tools now redirect to RDAP automatically.

**Practical Relevance:** WHOIS/RDAP is always the first step after identifying a target domain. The name servers identified here feed directly into DNS enumeration.

---

### DNS Records and Querying

**Definition:** DNS (Domain Name System) translates domain names to IP addresses and other data. Each record type serves a different purpose.

**Why It Matters:** DNS records reveal the IP addresses behind a domain, mail server infrastructure, email authentication configuration, and domain aliases. This information is essential for scoping the attack surface.

**Key DNS Record Types:**

| Record | Returns |
|--------|---------|
| A | IPv4 address(es) |
| AAAA | IPv6 address(es) |
| CNAME | Alias — points one domain name to another |
| MX | Mail servers and their priority |
| SOA | Primary name server, admin email, zone serial |
| TXT | Arbitrary text: SPF, DKIM, DMARC, domain verification tokens |

**Tool choice — `dig` vs `nslookup`:**
- Both query DNS, but `dig` is preferred for modern work
- `dig` provides cleaner output, displays TTL values by default, and is more reliable for scripting and complex queries
- `nslookup` is covered for compatibility — still found on Windows and in older documentation

**Practical Relevance:** A, MX, and TXT records are the three most commonly queried in recon. A records identify IPs for further scanning. MX records reveal email infrastructure (often Google Workspace or Microsoft 365, but sometimes self-hosted). TXT records reveal SPF/DMARC configuration, which is directly relevant to phishing assessments.

---

### Subdomain Discovery

**Definition:** The process of identifying all subdomains under a target domain that may host additional services, applications, or sensitive functionality.

**Why It Matters:** Subdomains expose attack surface that is invisible from the main domain alone. They frequently host forgotten services, outdated CMS installations, development panels, shadow IT, or misconfigured applications. A target's main domain might be hardened while `dev.target.com` or `staging.target.com` runs years-old software.

**Key Details:**

**DNSDumpster:** Aggregates public DNS data from search engine caches, zone transfer databases, and certificate records. Fully passive. Returns subdomains, resolved IPs with geolocation, MX/TXT/CNAME records, and a visual relationship map.

**Certificate Transparency (CT) Logs:** The most effective passive subdomain discovery method available. CT is a mandatory public logging framework (required since ~2015) that records every SSL/TLS certificate issued. Each certificate contains a Subject Alternative Name (SAN) field listing all domains it covers. Searching these logs via crt.sh (`%.target.com`) reveals subdomains without sending any traffic to the target.

- crt.sh operates in real time, has no rate limits for basic use, and typically returns 10–100x more subdomains than DNSDumpster alone
- SecurityTrails provides similar data with free limited searches

**Practical Relevance:** CT log enumeration is now considered a mandatory step in any web application or external infrastructure assessment. Finding a forgotten subdomain pointing to an exposed admin panel or outdated CMS is a common high-impact finding.

---

### Shodan.io

**Definition:** Shodan is a search engine for internet-connected devices. It continuously scans the public internet, collects service banners from open ports, and indexes them for search.

**Why It Matters:** Shodan reveals what services a target exposes to the internet, the software versions running behind those services, and which infrastructure provider hosts them — all without sending any traffic to the target.

**Key Details:**
- IP address, ASN (Autonomous System Number), and hosting provider
- Geographic location of the server
- Open ports and service banners with version strings
- Tags: `cdn`, `vuln` (matched against known CVEs)
- Supports search filters: `hostname:`, `org:`, `port:`, `http.component:`

**Practical Relevance:** Shodan is frequently the bridge between DNS enumeration and vulnerability research. Finding an exposed service with a specific version banner allows immediate cross-referencing against CVE databases. Defenders use Shodan alerts to monitor their own exposure.

---

## Methodology

The passive recon workflow follows a logical progression from broad to narrow:

```
Target domain identified
        |
        v
WHOIS/RDAP → Registrar, dates, name servers, abuse contacts
        |
        v
DNS queries → IPs (A/AAAA), mail infrastructure (MX),
              email policy (TXT: SPF/DMARC), aliases (CNAME)
        |
        v
Subdomain discovery → DNSDumpster (DNS aggregation),
                      crt.sh (CT log mining)
        |
        v
Service enumeration → Shodan.io (banners, ports, versions per IP)
        |
        v
Intelligence consolidated → Attack surface mapped
                           → Priority targets identified
                           → Active recon planned
```

Each stage builds on the previous one. WHOIS provides name servers that confirm the DNS infrastructure. DNS queries reveal IPs and mail servers. Subdomain discovery expands the known attack surface. Shodan contextualises each discovered IP with service and version information.

---

## Practical Activities

### Activity 1 — WHOIS Lookup

**Objective:** Extract domain registration intelligence for a target domain.

**Commands:**
```bash
# Legacy WHOIS lookup
whois tryhackme.com

# RDAP lookup via curl + jq (modern, preferred)
curl -s https://rdap.verisign.com/com/v1/domain/tryhackme.com | jq .
```

**Command Explanation:**
- `whois` queries the traditional WHOIS protocol on TCP 43
- `curl -s` fetches the RDAP JSON response silently from Verisign's public endpoint
- `| jq .` pipes the output to jq for formatted, readable JSON display

**Findings:**
- Registrar name and contact details
- Domain creation, last update, and expiry dates
- Authoritative name servers (feeds into DNS enumeration)
- Domain status codes (locked/unlocked)
- Most personal contact details now show privacy redaction

**Why It Matters:** Name servers become the first DNS targets. Expiry dates can inform social engineering timing. Privacy redaction is noted but historical tools (whoxy.com) can sometimes recover earlier contact data.

---

### Activity 2 — DNS Record Enumeration

**Objective:** Query different DNS record types to map the target's network infrastructure.

**Commands:**
```bash
# A records (IPv4 addresses) via nslookup - legacy method
nslookup -type=A tryhackme.com 1.1.1.1

# MX records (mail servers) via nslookup
nslookup -type=MX tryhackme.com

# TXT records (SPF/DMARC/verification) via nslookup
nslookup -type=TXT tryhackme.com

# A records via dig - preferred method
dig tryhackme.com A

# MX records via dig with specific resolver
dig @1.1.1.1 tryhackme.com MX

# TXT records via dig
dig tryhackme.com TXT
```

**Command Explanation:**
- `nslookup -type=A domain 1.1.1.1` — queries specifically for A records using Cloudflare's public resolver (1.1.1.1) instead of your ISP's resolver
- `dig @1.1.1.1 domain MX` — same as above, dig syntax; `@` specifies the resolver; the record type follows the domain name
- Using public resolvers like 1.1.1.1 avoids ISP query logging

**Findings:**
- A records return multiple IPs — these are often anycast addresses (Cloudflare CDN in this case)
- MX records show Google Workspace handling email (priority numbers: lower = higher priority)
- TXT records reveal SPF and DMARC configuration (relevant to phishing simulation scoping)

**Why It Matters:**
- Multiple IPs from anycast CDN suggest services may be proxied — useful context for scope decisions
- Google Workspace for email means the mail infrastructure is external and typically well-hardened
- SPF/DMARC configuration determines whether email spoofing is possible

---

### Activity 3 — Subdomain Discovery

**Objective:** Discover subdomains not visible in the main navigation or DNS records.

**Method 1: DNSDumpster (browser-based)**
- Navigate to https://dnsdumpster.com
- Search for `tryhackme.com`
- Review the host table for subdomains (e.g., `blog.tryhackme.com`)
- Review the visual graph showing relationships between subdomains, IPs, and mail servers

**Method 2: Certificate Transparency via crt.sh (browser-based)**
- Navigate to https://crt.sh
- Search `%.tryhackme.com`
- The `%` wildcard matches any subdomain prefix
- Results list every certificate issued for subdomains, with issue dates and certificate details

**Findings:**
- DNSDumpster surfaces subdomains with resolved IPs and geolocation
- crt.sh typically returns significantly more subdomains, including short-lived or recent ones
- Each discovered subdomain is a potential new target requiring its own DNS and service enumeration

**Why It Matters:** Subdomains are frequently the weakest link in an organisation's security posture. Finding `dev.target.com` or `legacy.target.com` often yields far more productive attack surface than the main domain.

---

### Activity 4 — Shodan.io Service Enumeration

**Objective:** Gather service and infrastructure intelligence on discovered IPs without touching the target.

**Method:** (Browser-based)
- Navigate to https://www.shodan.io
- Search by domain: `tryhackme.com` or by IP address from earlier DNS lookups (e.g., `104.26.10.229`)
- Review host detail pages for:
  - Hosting provider and ASN
  - Geographic location
  - Open ports with service banners
  - Vulnerability tags

**Shodan Search Filters:**
```
hostname:tryhackme.com
org:"TryHackMe"
port:443 country:US
http.component:"wordpress"
```

**Findings:**
- IP address confirmed to be hosted on Cloudflare CDN infrastructure
- Service banners reveal HTTP server type, version, and technology stack
- Vulnerability tags (if present) indicate services matching known CVEs

**Why It Matters:** Service version information from Shodan feeds directly into vulnerability research. Before any active interaction, you can already know which services are exposed and which CVEs may be applicable.

---

## Observations and Analysis

- **Anycast IP addresses:** Multiple A records for `tryhackme.com` resolving to Cloudflare IPs is a significant reconnaissance observation. It means the origin server's real IP is hidden behind the CDN. Direct scanning of Cloudflare IPs will not reveal the actual application server — a CDN bypass technique would be required in active phases.

- **Google Workspace MX records:** Email is handled externally by Google. This is relevant for phishing simulation scope — attacking the mail infrastructure directly is attacking Google's systems, not the target.

- **Privacy-redacted WHOIS:** The shift to GDPR/CCPA-compliant WHOIS has reduced the personal information available. However, historical services like whoxy.com preserve pre-redaction snapshots and can recover useful registrant data.

- **CT logs as the gold standard:** The number of subdomains found via crt.sh consistently exceeds DNSDumpster because CT logs capture every certificate issued, including internal infrastructure exposed via SAN fields in certificates.

- **RDAP transition:** ICANN formally deprecated the legacy WHOIS protocol for gTLDs on 28 January 2025. Tooling should migrate to RDAP (`curl` + public RDAP endpoints), though legacy `whois` clients continue to function via failover mechanisms.

---

## Tools and Technologies Used

### whois
- **Purpose:** Query domain registration information from WHOIS/RDAP databases
- **Common Usage:** `whois domain.com`
- **In This Room:** First step — identifies registrar, dates, and name servers; increasingly redirects to RDAP

### curl + jq
- **Purpose:** Query RDAP endpoints and format structured JSON output
- **Common Usage:** `curl -s https://rdap.verisign.com/com/v1/domain/DOMAIN | jq .`
- **In This Room:** Modern alternative to whois for structured, machine-readable RDAP data

### nslookup
- **Purpose:** DNS record lookup (legacy tool)
- **Common Usage:** `nslookup -type=A domain 1.1.1.1`
- **In This Room:** Demonstrated for compatibility; `dig` is preferred

### dig
- **Purpose:** DNS record lookup (modern, preferred tool)
- **Common Usage:** `dig @resolver domain TYPE`
- **In This Room:** Primary DNS enumeration tool; cleaner output, TTL display, scripting-friendly

### DNSDumpster
- **Purpose:** Passive subdomain and DNS aggregation
- **Common Usage:** https://dnsdumpster.com — search by domain
- **In This Room:** Subdomain discovery with visual relationship mapping

### crt.sh
- **Purpose:** Certificate Transparency log search for subdomain discovery
- **Common Usage:** https://crt.sh — search `%.domain.com`
- **In This Room:** Primary subdomain discovery method; returns the most comprehensive subdomain list

### Shodan.io
- **Purpose:** Internet-connected device and service search engine
- **Common Usage:** https://shodan.io — search by IP, hostname, or filter
- **In This Room:** Service banner and version intelligence on discovered IPs without touching the target

---

## Key Learnings

- Passive recon is stealthy and legally low-risk but yields substantial intelligence — it should always precede active techniques
- WHOIS is being replaced by RDAP; the modern query method uses HTTPS and returns structured JSON
- `dig` is the preferred DNS tool over `nslookup` — it provides cleaner output, TTL display, and better scripting support
- Certificate Transparency logs (crt.sh) consistently outperform DNS aggregation tools for subdomain discovery because they capture every certificate issued, not just publicly advertised records
- Shodan reveals service banners and version information for target IPs without requiring any interaction with the target
- Anycast/CDN IPs hide origin servers — discovering this early prevents wasted effort scanning CDN infrastructure
- The intelligence from passive recon directly structures the active recon phase — what to scan, which services to target, which subdomains to probe

---

## Real-World Relevance

**Penetration Testing:** Passive recon is the mandatory first phase of every external assessment. Discovering forgotten subdomains, identifying mail infrastructure, and mapping exposed services before any active scanning reduces noise and focuses effort on the highest-value targets.

**Bug Bounty Hunting:** CT log enumeration is a core technique for expanding scope and finding assets that other hunters overlook. Many high-severity bug bounty findings come from forgotten subdomains (`dev.`, `staging.`, `old.`) that were never hardened.

**Red Team Operations:** Operational security requires minimising detectable activity. Passive recon allows building a target profile without triggering a single alert — essential in engagements where stealth is prioritised.

**Threat Intelligence:** Blue teams and threat intelligence analysts use these same techniques to monitor their organisation's attack surface. Shodan alerts, CT log monitoring, and regular WHOIS checks are standard defensive practices in mature security programmes.

---

## Things Worth Remembering

**Commands:**
```bash
# WHOIS
whois tryhackme.com

# RDAP (modern)
curl -s https://rdap.verisign.com/com/v1/domain/tryhackme.com | jq .

# DNS - dig (preferred)
dig tryhackme.com A
dig @1.1.1.1 tryhackme.com MX
dig tryhackme.com TXT

# DNS - nslookup (legacy, for compatibility)
nslookup -type=A tryhackme.com 1.1.1.1
nslookup -type=MX tryhackme.com
nslookup -type=TXT tryhackme.com

# Subdomain discovery (browser)
# https://crt.sh → search: %.tryhackme.com
# https://dnsdumpster.com → search: tryhackme.com
```

**Quick Reference Table:**

| Purpose | Command/Resource |
|---------|-----------------|
| WHOIS lookup | `whois tryhackme.com` |
| RDAP (modern) | `curl -s https://rdap.verisign.com/com/v1/domain/DOMAIN | jq .` |
| A records | `dig tryhackme.com A` |
| MX records (specific resolver) | `dig @1.1.1.1 tryhackme.com MX` |
| TXT records | `dig tryhackme.com TXT` |
| CT log subdomain discovery | https://crt.sh → `%.domain.com` |
| DNS aggregation + graph | https://dnsdumpster.com |
| Service/banner intelligence | https://shodan.io |

**Key Concepts:**
- Passive recon = no packets to the target; fully stealthy
- Active recon = direct interaction; leaves log entries
- CT logs are the best passive subdomain source — mandatory step in external assessments
- RDAP replaced WHOIS for gTLDs as of January 2025
- Anycast IPs (CDN) hide origin servers — note this early
- `dig` over `nslookup` for all new work

---

## Conclusion

Passive reconnaissance establishes the intelligence foundation for every phase that follows. By the end of this room, you can identify a target domain's registration history, map its DNS infrastructure, enumerate subdomains through CT logs, and profile its exposed services through Shodan — all without triggering a single alert. This combination of WHOIS/RDAP, DNS enumeration, CT log mining, and Shodan querying is the standard baseline toolkit for the reconnaissance phase of any professional external assessment. The discipline of thoroughly mapping a target before touching it is what separates methodical pentesters from those who miss critical attack surface.
