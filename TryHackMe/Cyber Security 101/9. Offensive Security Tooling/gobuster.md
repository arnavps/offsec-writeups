# Gobuster: The Basics

## Overview

Gobuster is an open-source offensive enumeration tool written in Go. It brute-forces web directories and files, DNS subdomains, and virtual hosts using wordlists. It sits at the intersection of the reconnaissance and scanning phases of a penetration test and is widely used in CTFs, bug bounty hunting, and professional engagements. This room covers Gobuster's three primary modes: `dir`, `dns`, and `vhost`.

---

## Topics Covered

- What Gobuster is and where it fits in the attack lifecycle
- Enumeration vs brute force
- Global flags and their usage
- `dir` mode — directory and file enumeration
- `dns` mode — subdomain enumeration
- `vhost` mode — virtual host enumeration
- Key flags for each mode

---

## Key Concepts

### Enumeration vs Brute Force

- **Enumeration** — listing all available resources, whether accessible or not. Gobuster enumerates directories, subdomains, and vhosts.
- **Brute force** — trying every entry from a list until a match is found. Gobuster uses wordlists to construct requests and identifies valid targets by their HTTP responses or DNS resolution.

### How Gobuster Works

Gobuster takes a target (URL or domain) and a wordlist, then appends each wordlist entry to the target. It sends a request for each constructed value and analyses the response. Valid targets are identified based on the HTTP status code returned or whether a DNS record resolves.

It does **not** enumerate recursively by default — if a directory is discovered, you must run a separate scan targeting that directory.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| `dir` mode | Brute-forces directories and files on a web server |
| `dns` mode | Brute-forces DNS subdomains |
| `vhost` mode | Brute-forces virtual hostnames on a web server |
| Wordlist | A file of candidate names, one per line, used to construct scan targets |
| Virtual hosting | Multiple websites hosted on the same server/IP, differentiated by the `Host` header |
| Status code | HTTP response code indicating whether a resource exists and is accessible |
| FQDN | Fully Qualified Domain Name — e.g. `sub.example.thm` |

---

## Global Flags

These flags apply across all Gobuster modes:

| Short | Long | Description |
|-------|------|-------------|
| `-t` | `--threads` | Number of concurrent threads. Default is 10. Increase for speed, decrease for stealth |
| `-w` | `--wordlist` | Path to the wordlist file |
| `--delay` | | Time to wait between requests — useful to evade rate-limiting detection |
| `--debug` | | Outputs debug information for troubleshooting unexpected errors |
| `-o` | `--output` | Write results to a specified output file |

---

## Practical Examples / Demonstrations

### dir Mode — Directory and File Enumeration

`dir` mode brute-forces paths on a web server to discover directories and files. HTTP status codes in the response indicate whether each path exists and whether it's accessible.

**Basic syntax:**
```bash
gobuster dir -u "http://www.example.thm" -w /path/to/wordlist
```

**Example with threads:**
```bash
gobuster dir -u "http://www.example.thm/" \
  -w /usr/share/wordlists/dirb/small.txt \
  -t 64
```

**Example with redirect following:**
```bash
gobuster dir -u "http://www.example.thm" \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -r
```

**Example with file extension filtering:**
```bash
gobuster dir -u "http://www.example.thm" \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x .php,.js
```

The `-x` flag appends the specified extensions to each wordlist entry, so Gobuster checks for `admin`, `admin.php`, and `admin.js` in a single scan.

#### Key `dir` Mode Flags

| Short | Long | Description |
|-------|------|-------------|
| `-c` | `--cookies` | Pass a cookie (e.g. session ID) with each request |
| `-x` | `--extensions` | File extensions to scan for (e.g. `.php,.html,.js`) |
| `-H` | `--headers` | Add custom headers to each request |
| `-k` | `--no-tls-validation` | Skip TLS certificate validation — useful for self-signed certs in CTFs |
| `-n` | `--no-status` | Suppress status codes in output |
| `-P` | `--password` | Password for authenticated scanning (use with `-U`) |
| `-U` | `--username` | Username for authenticated scanning (use with `-P`) |
| `-s` | `--status-codes` | Only display responses with these status codes (e.g. `200,301`) |
| `-b` | `--status-codes-blacklist` | Hide responses with these status codes (overrides `-s`) |
| `-r` | `--followredirect` | Follow HTTP redirects (301, 302) automatically |

#### Notes on URL Format

- The URL must include the protocol: `http://` or `https://`
- Use the hostname instead of IP where possible — web servers can host multiple sites on one IP (virtual hosting); the wrong IP may return a different site's content
- To enumerate a subdirectory, set it as the base URL: `http://www.example.thm/resources`
- Gobuster does not recurse — discovered directories must be scanned separately

---

### dns Mode — Subdomain Enumeration

`dns` mode brute-forces subdomains by performing DNS lookups for each wordlist entry prepended to the target domain. If a subdomain resolves, it's listed as a result.

Subdomains are important targets because a patch applied to the main domain may not exist on a subdomain. Each subdomain is effectively a separate attack surface.

**Basic syntax:**
```bash
gobuster dns -d example.thm -w /path/to/wordlist
```

**Example:**
```bash
gobuster dns -d example.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

If the wordlist entry is `mail`, Gobuster performs a DNS lookup for `mail.example.thm`. If it resolves, the subdomain is reported.

#### Key `dns` Mode Flags

| Short | Long | Description |
|-------|------|-------------|
| `-d` | `--domain` | Target domain to enumerate subdomains of |
| `-c` | `--show-cname` | Show CNAME records in output (cannot be used with `-i`) |
| `-i` | `--show-ips` | Show IP addresses that each subdomain resolves to |
| `-r` | `--resolver` | Use a custom DNS server for resolution |

---

### vhost Mode — Virtual Host Enumeration

`vhost` mode discovers virtual hosts on a web server. Virtual hosts are multiple websites running on the same server and IP address, differentiated by the `Host` header in the HTTP request.

**The difference between `vhost` and `dns` mode:**

| Mode | Method | How it works |
|------|--------|-------------|
| `dns` | DNS lookup | Queries DNS for `wordlist_entry.domain` to see if it resolves |
| `vhost` | HTTP request | Sends a request to the target URL with `Host: wordlist_entry.domain` and checks the response |

Virtual hosts don't necessarily have DNS entries — they can exist purely as server-side configurations. The `vhost` mode catches these where `dns` mode would miss them.

**Basic syntax:**
```bash
gobuster vhost -u "http://example.thm" -w /path/to/wordlist
```

#### Key `vhost` Mode Flags

| Short | Long | Description |
|-------|------|-------------|
| `-u` | `--url` | Base URL of the target server |
| `--append-domain` | | Appends the base domain to each wordlist entry (e.g. `word` → `word.example.thm`) |
| `-m` | `--method` | HTTP method to use (default: GET) |
| `--domain` | | Domain to append to each wordlist entry |
| `--exclude-length` | | Filter out responses of a specific body length (useful to remove false positives) |
| `-r` | `--follow-redirect` | Follow HTTP redirects |

---

## Workflow / Process

```
Identify the target (URL, domain, or IP)
        |
        v
Choose the appropriate mode: dir / dns / vhost
        |
        v
Select a wordlist appropriate to the target
        |
        v
Run Gobuster with required flags (-u/-d and -w)
        |
        v
Review HTTP status codes (dir) or resolved names (dns/vhost)
        |
        v
Investigate interesting findings (200, 301, 302 in dir mode)
        |
        v
Re-run against discovered subdirectories as needed (dir is not recursive)
```

---

## Real-World Relevance

- **Directory enumeration** exposes hidden admin panels, backup files, config files, and development endpoints not linked from the main site
- **Subdomain enumeration** is a standard part of external reconnaissance — subdomains often run older, less-patched software
- **Virtual host discovery** identifies sites that share an IP but aren't publicly documented — often staging environments or internal tools
- Common wordlists used in practice:
  - `/usr/share/wordlists/dirb/common.txt` — small, fast
  - `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` — comprehensive
  - `/usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt` — subdomain enumeration
- Gobuster is a standard tool in OSCP and bug bounty workflows

---

## Key Learnings

- Gobuster has three primary modes: `dir` (directories/files), `dns` (subdomains), `vhost` (virtual hosts)
- `-u` and `-w` are required for `dir` and `vhost`; `-d` and `-w` are required for `dns`
- `-t` increases thread count for faster scans; `--delay` slows requests to avoid detection
- `-x` in `dir` mode appends file extensions to every wordlist entry
- `dns` mode uses DNS resolution; `vhost` mode uses HTTP `Host` header manipulation — they find different things
- Gobuster does not recurse — run additional scans against discovered paths manually

---

## Additional Notes

- Gobuster is pre-installed on Kali Linux. On other systems: `go install github.com/OJ/gobuster/v3@latest`
- Using the hostname (`-u http://target.thm`) is preferable to IP when virtual hosting is involved
- The `--exclude-length` flag in `vhost` mode is very useful — many servers return a 200 response for all hostnames but with a consistent body length for the "not found" case; excluding that length removes noise
- SecLists (`/usr/share/wordlists/SecLists/`) is a broader wordlist collection that complements the default Kali wordlists — install with `sudo apt install seclists`

---

## Conclusion

Gobuster is a fast, flexible enumeration tool that is relevant at multiple stages of a web application assessment. Directory enumeration reveals attack surface that isn't visible from the front end, subdomain enumeration expands the scope of a target, and virtual host discovery finds sites that wouldn't be found any other way. Mastering the three modes and their key flags gives you a solid foundation for web reconnaissance.
