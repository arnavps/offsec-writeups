# Content Discovery

## Overview

Content discovery is the process of finding web application content that wasn't meant to be publicly accessible or isn't linked anywhere obvious. This room covers all three approaches: manual checks against known disclosure files, OSINT-based intelligence gathering, and automated brute-force enumeration using Gobuster. Together these techniques map the full attack surface of a web application before any exploitation begins.

**Main objectives:**
- Perform manual content discovery using robots.txt, sitemap.xml, HTTP headers, and framework documentation
- Apply OSINT techniques: Google Dorking, Wappalyzer, Wayback Machine, GitHub, and S3 bucket discovery
- Use Gobuster's `dir`, `dns`, and `vhost` modes for automated content enumeration
- Understand when and how to combine all three approaches

**Skills introduced:** robots.txt/sitemap.xml analysis, HTTP header reading, Google Dorking operators, Gobuster dir/dns/vhost modes, DNS configuration for lab environments

---

## Concepts Covered

### Manual Content Discovery

**Definition:** Checking specific files and locations that web servers expose by convention, which may contain information about sensitive paths.

**Why It Matters:** These files are often overlooked precisely because they are not linked from the application itself. They provide a ready-made map of locations worth investigating.

**robots.txt:**
- Purpose: tells search engine crawlers which pages not to index
- Security implication: paths listed under `Disallow:` are often sensitive — `/staff-portal`, `/admin`, `/internal` etc.
- robots.txt is a hint for bots, not a security control — the paths it lists are still accessible

**sitemap.xml:**
- Purpose: tells search engines which pages the owner wants indexed
- Security implication: may include endpoints not reachable via normal browsing — `/s3cr3t-area`, old content, staging pages, parameter-based URLs like `/news/article?id=1`
- The `id=` style parameters in sitemaps are injection test candidates

**HTTP Headers:**
- `Server:` — reveals web server software and version (e.g., `Apache/2.4.58`)
- `X-Powered-By:` — reveals backend language/framework (e.g., `PHP/8.1`, `Express`)
- Custom headers — applications sometimes add non-standard headers that reveal internal identifiers or feature flags

**Framework Stack:**
- Once a framework is identified (from favicon hash, header, page source, or copyright notice), visit the framework's own website
- Framework documentation often reveals default admin paths, configuration file locations, and default credentials
- Knowing the framework version enables CVE research

---

### OSINT Techniques

**Definition:** Open-Source Intelligence — using freely available public tools and data to gather information about a target without directly interacting with it.

**Why It Matters:** Information a target has already shared publicly is the easiest and safest intelligence to collect. OSINT finds things that are technically public but not obviously visible.

**Google Dorking:**

| Operator | Example | Effect |
|----------|---------|--------|
| `site:` | `site:example.com` | Restrict to results from one domain |
| `inurl:` | `inurl:admin` | Find URLs containing a word |
| `filetype:` | `filetype:pdf` | Find specific file types |
| `intitle:` | `intitle:admin` | Find pages with specific title |
| `intext:` | `intext:password` | Find pages with specific body text |
| `cache:` | `cache:example.com` | Google's cached copy |

Combine operators: `site:example.com filetype:pdf` returns all PDFs indexed from that domain.

**Wappalyzer:** Browser extension that fingerprints technology stacks in real time. Detects CMS, web server, JS frameworks, CDNs, analytics — often including version numbers. Immediately useful for CVE research.

**Wayback Machine (web.archive.org):** Archives the internet since the late 1990s. Find content removed from the live site — old login forms, deleted API endpoints, pages published briefly before takedown. Archived content sometimes remains accessible even after removal.

**GitHub:** Version control for source code. Developers accidentally commit API keys, credentials, `.env` files, and configuration files to public repositories. Search by company name or domain. Critically: check the **commit history** — sensitive data removed in a later commit still exists in the history.

**S3 Buckets:** Amazon S3 is widely used for file hosting. URL format: `https://{name}.s3.amazonaws.com`. Misconfigurations make buckets publicly accessible. Common naming patterns: `{company}-assets`, `{company}-backup`, `{company}-www`, `{company}-dev`. Check URLs found in page source or GitHub repos for exposed bucket content.

---

### Automated Content Discovery with Gobuster

**Definition:** Gobuster is an open-source Go-based enumeration tool that rapidly sends requests against a web server to discover existing directories, files, subdomains, and virtual hosts.

**Why It Matters:** Manual checking and OSINT can only discover content that is known or referenced somewhere. Automated enumeration uses wordlists to discover paths that have never been linked or referenced anywhere.

**Global Gobuster flags:**

| Flag | Purpose |
|------|---------|
| `-t` / `--threads` | Concurrent threads (default 10; increase for speed) |
| `-w` / `--wordlist` | Path to the wordlist |
| `-o` / `--output` | Write results to file |
| `--delay` | Wait between requests (useful against rate-limited servers) |

**Wordlist location:** `/usr/share/wordlists/SecLists/` on AttackBox

---

### Gobuster `dir` Mode

**Definition:** Brute-forces directory and file paths against a web server.

**Key flags:**

| Flag | Purpose |
|------|---------|
| `-u` | Target URL (required) |
| `-w` | Wordlist (required) |
| `-x` | File extensions to append (e.g., `-x .php,.txt,.js`) |
| `-r` | Follow redirects |
| `-k` | Skip TLS verification (lab environments) |
| `-s` | Show only specific status codes |

---

### Gobuster `dns` Mode

**Definition:** Brute-forces DNS subdomains by querying each wordlist entry as a subdomain of the target domain.

**Key flags:**

| Flag | Purpose |
|------|---------|
| `-d` | Target domain |
| `-i` | Show resolved IP addresses |
| `-r` | Use a custom DNS resolver |
| `--wildcard` | Continue even when wildcard DNS is detected |

---

### Gobuster `vhost` Mode

**Definition:** Discovers virtual hosts by sending HTTP requests with varying `Host:` header values, without relying on DNS.

**Why vhost differs from dns:**
- `dns` mode: performs actual DNS lookups — finds subdomains registered in DNS
- `vhost` mode: sends HTTP requests with each wordlist entry as the `Host:` header — finds virtual hosts on the same IP not registered in DNS

**Key flags:**

| Flag | Purpose |
|------|---------|
| `-u` | Target IP/URL |
| `--domain` | Base domain to append to each wordlist entry |
| `--append-domain` | Combine wordlist entries with the base domain |
| `--exclude-length` | Filter out false positives by response size |

---

## Methodology

```
Manual checks (quick wins):
  Visit /robots.txt → note Disallow paths
  Visit /sitemap.xml → map endpoints, note parameters
  curl -v TARGET → read Server, X-Powered-By headers
  Identify framework → check framework docs for admin paths + default creds
        |
        v
OSINT (passive intelligence):
  Google Dork: site:TARGET filetype:pdf, inurl:admin, intitle:login
  Wappalyzer → technology fingerprint
  Wayback Machine → removed pages, old endpoints
  GitHub → source code, commit history, accidentally committed secrets
  S3 → try naming patterns, check page source for bucket URLs
        |
        v
Automated enumeration:
  Gobuster dir → directories and files
    gobuster dir -u URL -w common.txt -x .php,.txt
  Gobuster dns → subdomains
    gobuster dns -d domain.thm -w subdomains-top1million.txt --wildcard
  Gobuster vhost → virtual hosts
    gobuster vhost -u URL --domain domain.thm -w subdomains.txt --append-domain --exclude-length N
        |
        v
Feed all findings into the next phase:
  Interesting paths → manual testing
  Admin panels → credential testing
  API endpoints → parameter testing
  Sensitive files → review for credentials / configuration
```

---

## Practical Activities

### Activity 1 — robots.txt and sitemap.xml

**Objective:** Discover sensitive paths and endpoint inventory from convention files.

**Commands:**
```bash
# Visit robots.txt
curl http://TARGET_IP/robots.txt

# Visit sitemap.xml
curl http://TARGET_IP/sitemap.xml
```

**Findings:**
- `robots.txt` contained `Disallow: /staff-portal` — a path intentionally hidden from search engines but still accessible
- `sitemap.xml` listed `/customers/login`, `/s3cr3t-area`, and parameter URLs like `/news/article?id=1,2,3`

**Why It Matters:** Disallowed paths are prime targets. The id= parameters in the sitemap are candidates for IDOR and injection testing.

---

### Activity 2 — HTTP Header Analysis

**Objective:** Extract server and framework version information from HTTP response headers.

**Commands:**
```bash
curl http://TARGET_IP -v
```

**Command Explanation:**
- `-v` (verbose) outputs both request and response headers
- The response headers section reveals `Server:`, `X-Powered-By:`, and any custom headers

**Findings:**
- `Server: Apache/2.4.58` — exact web server version
- `PHPSESSID` cookie confirming PHP session management
- Custom header containing a flag — a non-standard header left in by the developer

**Why It Matters:** Exact version numbers from headers enable targeted CVE research. Custom headers indicate information the developer didn't realise they were exposing.

---

### Activity 3 — Gobuster Directory Enumeration

**Objective:** Discover directories and files not linked from the application.

**Commands:**
```bash
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
```

**Command Explanation:**
- `dir` — directory/file enumeration mode
- `-u` — target URL
- `-w` — wordlist of common directory and file names

**Findings:**
- `/admin` → 302 redirect to login page (admin panel exists)
- `/customers` → 302 redirect to login
- `/development.log` → 200 OK (log file accessible)
- `/private` → 200 OK
- `/assets` → browsable directory
- `robots.txt`, `sitemap.xml` confirmed

**Why It Matters:** The `/development.log` file is an immediate investigation target — development logs frequently contain stack traces, database queries, error messages, and sometimes credentials. The `/private` directory requires further investigation.

---

### Activity 4 — DNS Subdomain Enumeration

**Objective:** Discover subdomains of the target domain.

**Lab setup required:**
```bash
# Add target IP to DNS resolver
sudo nano /etc/resolv-dnsmasq
# Add: nameserver TARGET_IP as first line

# Restart dnsmasq
/etc/init.d/dnsmasq restart

# Add hostname to /etc/hosts
echo "TARGET_IP example.thm" | sudo tee -a /etc/hosts
```

**Command:**
```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
```

**Command Explanation:**
- `dns` — DNS subdomain enumeration mode
- `-d` — target domain
- `-w` — subdomain wordlist
- `--wildcard` — continue even when wildcard DNS is detected (prevents false all-match results)

---

### Activity 5 — Virtual Host Enumeration

**Objective:** Discover virtual hosts running on the target IP not registered in public DNS.

**Command:**
```bash
gobuster vhost -u "http://TARGET_IP" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```

**Command Explanation:**
- `vhost` — virtual host enumeration via `Host:` header manipulation
- `--domain example.thm` — base domain to use
- `--append-domain` — combine each wordlist entry with the domain (e.g., `admin` → `admin.example.thm`)
- `--exclude-length 250-320` — filter responses of this size to remove false positives (size range common to default/error responses)

---

## Observations and Analysis

- **robots.txt is reconnaissance, not restriction:** Listing paths in `Disallow:` actively helps attackers by providing a curated list of sensitive locations. Removing sensitive paths from robots.txt entirely is better than listing them.

- **Sitemap reveals parameter-based endpoints:** The `id=` parameters in the sitemap are high-priority injection test candidates. They indicate data retrieval based on user-supplied IDs — classic IDOR and SQLi territory.

- **HTTP headers as fingerprinting source:** The combination of `Server:` and `X-Powered-By:` headers reveals the full stack without any active probing. Security-conscious deployments suppress these headers — finding them exposed is a reportable misconfiguration.

- **Development logs on production web server:** A `development.log` file accessible via HTTP on a production server is a significant finding. These files commonly contain database connection errors (revealing credentials), stack traces (revealing internal paths), and SQL queries (revealing schema structure).

- **Virtual hosts vs DNS subdomains:** The distinction matters for tooling choice. `gobuster dns` finds subdomains with DNS records; `gobuster vhost` finds virtual hosts that exist at the server level but have no public DNS record — these are often staging, development, or internal environments.

---

## Tools and Technologies Used

### Gobuster
- **Purpose:** Automated web content discovery — directories, files, subdomains, virtual hosts
- **Common Usage:** `gobuster dir -u URL -w wordlist`, `gobuster dns -d domain -w wordlist`
- **In This Room:** All three modes: `dir` for path enumeration, `dns` for subdomains, `vhost` for virtual hosts

### Wappalyzer
- **Purpose:** Browser extension for automatic technology fingerprinting
- **Common Usage:** Install extension; visit any site; view detected technologies
- **In This Room:** Technology stack identification to guide CVE research

### Wayback Machine (web.archive.org)
- **Purpose:** Historical web content archive
- **Common Usage:** Search domain; browse historical snapshots
- **In This Room:** Finding removed pages and endpoints not accessible on the live site

---

## Key Learnings

- Manual checks (robots.txt, sitemap.xml, HTTP headers) should always precede automated scanning
- robots.txt `Disallow:` paths are target lists, not security controls
- `curl -v` is the fastest way to read HTTP headers for technology fingerprinting
- Gobuster `dir` requires `-u` and `-w`; use `-x` to check for specific file extensions
- Gobuster `dns` performs DNS lookups; `vhost` manipulates the `Host:` header — different mechanisms finding different things
- `--exclude-length` is essential for vhost scans to filter false positives
- OSINT (GitHub, Wayback Machine, Google Dorks) finds content the target has already exposed publicly — often overlooked
- All three approaches (manual, OSINT, automated) work together — each finds different things

---

## Content Discovery Method Reference

| Method | Techniques | What It Finds |
|--------|-----------|--------------|
| Manual | robots.txt, sitemap.xml, headers, framework docs | Convention-exposed paths, framework version, server info |
| OSINT | Google Dorks, Wappalyzer, Wayback, GitHub, S3 | Leaked credentials, removed pages, tech stack, exposed storage |
| Automated | Gobuster dir, dns, vhost | Unlinked directories/files, subdomains, virtual hosts |

---

## Conclusion

Content discovery transforms a surface-level understanding of a web application into a comprehensive map of its attack surface. The combination of manual convention-file checks, OSINT intelligence gathering, and automated brute-force enumeration ensures that no accessible resource is overlooked. The directories and files discovered here — admin panels, log files, private directories, hidden subdomains — feed directly into every subsequent phase of a web application penetration test. Content discovery is not a preliminary step to skip; it is where the real attack surface is established.
