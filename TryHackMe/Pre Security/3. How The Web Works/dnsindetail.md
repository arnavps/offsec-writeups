# DNS in Detail

## Overview

DNS (Domain Name System) is the internet's address book. It translates human-readable domain names into IP addresses that computers use to communicate. Without DNS, navigating the web would require memorising numerical IP addresses for every site. Understanding DNS is foundational for web security, reconnaissance, and understanding how traffic flows on the internet.

---

## Topics Covered

- What DNS is and why it exists
- Domain hierarchy (TLD, Second-Level Domain, Subdomain)
- DNS record types

---

## Key Concepts

### What is DNS?

DNS resolves domain names (e.g., `tryhackme.com`) to IP addresses (e.g., `104.26.10.229`). This allows users to use memorable names instead of raw IP addresses when accessing resources on the internet.

---

### Domain Hierarchy

A domain name is structured in a hierarchy, read from right to left:

```
admin.tryhackme.com
  |        |      |
Subdomain  SLD   TLD
```

**TLD (Top-Level Domain)**
The rightmost part of a domain name.

| Type | Description | Examples |
|---|---|---|
| gTLD (Generic) | Indicates the domain's purpose | `.com`, `.org`, `.net`, `.edu` |
| ccTLD (Country Code) | Indicates geographic location | `.uk`, `.ca`, `.de`, `.au` |

**Second-Level Domain (SLD)**
The part directly to the left of the TLD. In `tryhackme.com`, `tryhackme` is the SLD. Limited to 63 characters and can only use `a-z`, `0-9`, and hyphens.

**Subdomain**
Sits to the left of the SLD, separated by a period. In `admin.tryhackme.com`, `admin` is the subdomain. Multiple subdomains can be chained (e.g., `dev.api.tryhackme.com`), as long as the total length doesn't exceed 253 characters.

---

### DNS Record Types

| Record Type | Resolves To | Example |
|---|---|---|
| A | IPv4 address | `tryhackme.com` → `104.26.10.229` |
| AAAA | IPv6 address | `tryhackme.com` → `2606:4700:20::681a:be5` |
| CNAME | Another domain name | `store.tryhackme.com` → `shops.shopify.com` |
| MX | Mail server address for the domain | `tryhackme.com` → `alt1.aspmx.l.google.com` |
| TXT | Free-form text data | Used for SPF, DKIM, domain verification, etc. |

- **A / AAAA** — standard address resolution for IPv4 and IPv6 respectively
- **CNAME** — aliases one domain to another; useful for pointing subdomains to external services
- **MX** — directs email traffic to the correct mail server; includes a priority value
- **TXT** — stores arbitrary text; commonly used for email authentication (SPF, DKIM) and domain ownership verification

---

## Important Terminology

| Term | Definition |
|---|---|
| DNS | Domain Name System — resolves domain names to IP addresses |
| TLD | Top-Level Domain — the rightmost segment of a domain (`.com`, `.org`) |
| gTLD | Generic TLD — indicates domain purpose |
| ccTLD | Country Code TLD — indicates geographic region |
| SLD | Second-Level Domain — the core name of a domain (e.g., `tryhackme`) |
| Subdomain | A prefix to the SLD used to identify a specific service or section |
| A Record | Maps a domain to an IPv4 address |
| AAAA Record | Maps a domain to an IPv6 address |
| CNAME Record | Aliases one domain name to another |
| MX Record | Specifies the mail server for a domain |
| TXT Record | Stores arbitrary text data associated with a domain |

---

## Practical Examples / Demonstrations

### Query DNS records using `nslookup`

```bash
nslookup tryhackme.com
```

```bash
nslookup -type=MX tryhackme.com
```

```bash
nslookup -type=TXT tryhackme.com
```

### Query DNS records using `dig`

```bash
dig tryhackme.com A
```

```bash
dig tryhackme.com MX
```

```bash
dig store.tryhackme.com CNAME
```

---

## Workflow / Process

### DNS Resolution Flow

```
User types tryhackme.com in browser
        |
Browser checks local DNS cache
        |
If not cached → query Recursive DNS Resolver (usually ISP)
        |
Resolver checks its cache → if miss, queries Root DNS Server
        |
Root Server directs to TLD nameserver (.com)
        |
TLD nameserver directs to Authoritative Nameserver for tryhackme.com
        |
Authoritative Nameserver returns the IP address
        |
Resolver caches the result and returns IP to browser
        |
Browser connects to the IP address
```

---

## Real-World Relevance

- DNS enumeration is a core recon technique — tools like `dnsenum`, `sublist3r`, and `amass` discover subdomains by querying DNS records
- CNAME records reveal third-party services in use (e.g., Shopify, Cloudflare), which can be targets for subdomain takeover attacks
- MX records reveal email providers, useful for phishing campaign planning and email security assessment
- TXT records often expose internal information — SPF records list authorised mail servers, and misconfigured records can aid spoofing
- DNS cache poisoning attacks inject malicious records into resolvers to redirect users to attacker-controlled servers

---

## Key Learnings

- DNS translates domain names to IP addresses — it's fundamental to how the internet works
- Domain names have a hierarchy: TLD → SLD → Subdomain
- Different DNS record types serve different purposes — A/AAAA for addresses, CNAME for aliases, MX for mail, TXT for metadata
- DNS is a rich source of information during reconnaissance

---

## Additional Notes

- DNS queries are typically sent over UDP port 53; TCP port 53 is used for larger responses (e.g., zone transfers)
- DNS zone transfers (`AXFR`) can expose the entire DNS record set for a domain if misconfigured — always a high-value recon target
- TTL (Time to Live) on DNS records controls how long resolvers cache the result before querying again

---

## Conclusion

DNS is one of the most critical and most exploited protocols on the internet. A solid understanding of how domain hierarchies work and what each record type does is essential for web reconnaissance, identifying attack surface, and understanding how traffic is routed across the internet.
