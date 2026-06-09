# Google Dorking — Reference & Cheatsheet

**Source:** https://gist.github.com/sundowndev/283efaddbcf896ab405488330d1bbc06
**Author:** sundowndev
**Category:** OSINT · Reconnaissance · Google Hacking

> *"Using advanced Google search operators to find exposed data, misconfigured servers, and sensitive files."*

---

## What is Google Dorking?

Google Dorking (also called Google Hacking) is the practice of using advanced search operators to find information that is publicly indexed by Google but not easily discoverable through normal search queries. It is a core OSINT reconnaissance technique used by penetration testers, bug bounty hunters, and security researchers to surface exposed sensitive data, misconfigured servers, and unintended file disclosures.

The technique relies entirely on data Google has already indexed — it is passive reconnaissance that does not directly interact with the target.

---

## Search Filters Reference

| Filter | Description | Example |
|--------|-------------|---------|
| `allintext` | All keywords must appear in page body | `allintext:"admin password"` |
| `intext` | At least one keyword in page body | `intext:"index of /"` |
| `inurl` | Keyword in the URL | `inurl:"admin/login"` |
| `allinurl` | All keywords in the URL | `allinurl:"wp-admin login"` |
| `intitle` | Keyword in page title | `intitle:"Apache2 Ubuntu Default Page"` |
| `allintitle` | All keywords in page title | `allintitle:"index of" "parent directory"` |
| `site` | Restrict results to a specific domain | `site:target.com` |
| `filetype` | Search for a specific file extension | `filetype:pdf` |
| `link` | Find pages linking to a URL | `link:"target.com"` |
| `numrange` | Search for a number range | `numrange:8080-8090` |
| `before` / `after` | Restrict by date | `filetype:pdf before:2020-01-01` |
| `inanchor` | Keywords in anchor text linking to a page | `inanchor:"login"` |
| `related` | Find pages similar to a URL | `related:github.com` |
| `cache` | View Google's cached version (deprecated Sep 2024) | `cache:target.com` |

> **Note:** As of September 2024, Google has retired the `cache:` operator. See: https://developers.google.com/search/updates#cache-docs

---

## Operators

### Exact Phrase Match

Wrap a phrase in quotes to search for it literally:

```
"index of /backup"
```

### OR Operator

Returns results matching either term:

```
site:facebook.com | site:twitter.com
```

### AND Operator

Both conditions must match:

```
site:facebook.com & intext:"login"
```

### Combining Operators

```
(site:facebook.com | site:twitter.com) & intext:"login"
```

### Include / Exclude Results

Force inclusion with `+`, exclude with `-`:

```
site:facebook.* -site:facebook.com
```

### Wildcard `*`

The asterisk acts as a wildcard where you don't know the exact term:

```
"admin * password"
site:*.target.com
```

### Synonyms `~`

Prepend `~` to a word to also match synonyms:

```
~configure
```

---

## Practical Dork Examples

### Exposed Directories

```
intitle:"index of /"
intitle:"index.of" "parent directory" "size" "last modified"
intext:"index of /" +passwd
```

### Configuration and Credential Files

```
filetype:config inurl:web.config inurl:ftp
filetype:env intext:"DB_PASSWORD"
filetype:yml inurl:.env
filetype:xml inurl:"config"
filetype:log inurl:"password"
ext:(doc | pdf | xls | txt) intext:confidential inurl:confidential
```

### Login Portals

```
intitle:"login" site:target.com
inurl:/admin/login.php
inurl:wp-login.php site:target.com
intitle:"Plesk" inurl:":8443"
```

### Exposed Databases

```
intitle:"phpMyAdmin" inurl:"/phpmyadmin"
inurl:".php?id=" site:target.com
filetype:sql intext:"INSERT INTO"
```

### Camera and Device Exposure

```
intitle:"webcamXP 5" inurl:8080
inurl:"/view/view.shtml"
inurl:top.htm inurl:currenttime
intitle:"Live View / – AXIS"
```

### Sensitive Documents

```
filetype:pdf site:gov "confidential"
filetype:xls intext:"username password email"
ext:(doc | pdf | xls | txt | ps | rtf | odt | sxw | psw | ppt | pps | xml) (intext:"confidential salary" | intext:"budget approved")
```

### Git and Source Code Exposure

```
site:github.com "target.com" password
site:github.com "target.com" secret_key
site:pastebin.com "target.com" password
```

### Subdomain Enumeration via Dorking

```
site:*.target.com -site:www.target.com
```

---

## Bug Bounty Dork Patterns

```
site:target.com ext:php inurl:?
site:target.com inurl:redirect
site:target.com inurl:open
site:target.com inurl:url=http
site:target.com inurl:return
site:target.com intitle:"swagger"
site:target.com inurl:"/api/v"
site:target.com filetype:json inurl:api
```

---

## GHDB — Google Hacking Database

The **Google Hacking Database** (GHDB) at https://www.exploit-db.com/google-hacking-database maintains a categorised collection of dorks for:

- Footholds (login pages, admin portals)
- Files containing usernames or passwords
- Sensitive directories
- Vulnerable servers and services
- Error messages
- Online shopping info
- Network or vulnerability data

Always check GHDB before building custom dorks from scratch.

---

## Defensive Countermeasures

Prevent your own infrastructure from appearing in dangerous dorks:

- Add a `robots.txt` with `Disallow` for sensitive paths (note: this is advisory only)
- Use `<meta name="robots" content="noindex">` on admin or internal pages
- Avoid putting credentials, API keys, or sensitive data in web-accessible files
- Monitor GHDB and run dorks against your own domains periodically
- Remove configuration backup files (`web.config.bak`, `.env.bak`) from web roots
- Use `.htaccess` rules or server config to deny access to sensitive extensions

---

## Key Learnings

- **Google indexes what servers expose publicly** — if a sensitive file is accessible via HTTP, Google may index it. Dorking finds these exposures passively without touching the server.
- **`filetype:` and `inurl:` together are the most powerful combination** — they narrow results to specific file types in specific URL patterns, quickly surfacing backup files, config files, and credentials.
- **`site:*.target.com`** enumerates subdomains that Google has indexed — faster than DNS brute-forcing for common subdomains, though less exhaustive.
- **Combine operators for precision** — a dork like `site:target.com filetype:sql intext:"INSERT INTO"` is highly specific and almost always indicates a misconfiguration when it returns results.
- **`cache:` is retired** — as of September 2024 this operator no longer works. Use the Wayback Machine (https://web.archive.org) for cached content instead.
- **Dorking is purely passive** — no packets are sent to the target. It is legal to run dorks on any public site for security research purposes, as you are querying Google's index, not the target directly.

## Conclusion

Google Dorking remains one of the highest-value passive reconnaissance techniques available to penetration testers and bug bounty hunters. A well-crafted dork can surface login portals, backup database dumps, API keys in `.env` files, and exposed admin panels without sending a single packet to the target. The cheatsheet above covers the core operators and the most frequently useful patterns. For production engagements, pair dorking with automated tools like `googler`, `dorkscout`, or `pagodo` to iterate efficiently across GHDB categories. Always check your own infrastructure with the same dorks used by attackers.
