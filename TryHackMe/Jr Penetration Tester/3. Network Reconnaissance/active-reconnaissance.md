# Active Reconnaissance

## Overview

Active reconnaissance is the process of directly interacting with a target system or network to gather information. Unlike passive recon, which collects data from public sources without touching the target, active recon transmits packets, makes connections, and probes services. This room introduces the fundamental tools for active reconnaissance — web browsers with Developer Tools, ping, traceroute, telnet, and netcat — and builds the skills for understanding what direct interaction with a target reveals before moving on to dedicated scanners.

**Main objectives:**
- Distinguish passive from active reconnaissance and understand the risk difference
- Use a web browser and its Developer Tools for reconnaissance against web applications
- Understand how ping and traceroute work at the IP level
- Use telnet for manual banner grabbing against TCP services
- Use netcat as a flexible client and server for port probing and banner extraction

**Skills introduced:** HTTP/HTTPS protocol basics, Developer Tools reconnaissance, TTL-based tracerouting, banner grabbing, TCP port probing

---

## Concepts Covered

### Passive vs Active Reconnaissance

**Definition:**
- **Passive recon:** Collecting intelligence from public sources without sending packets to the target
- **Active recon:** Directly engaging a target by transmitting packets, connecting to services, or interacting with personnel

**Why It Matters:** Active techniques leave traces — log entries in web servers, firewall logs, IDS/WAF alerts, and honeypot triggers. Every active probe creates evidence of your activity. Understanding this distinction is critical for managing detection risk during engagements.

**Key Details:**
- Active recon is riskier and more detectable than ever: CDNs (Cloudflare, Akamai), WAFs, and zero-trust models log or block unusual probes
- IPv6 adoption means many hosts respond to ping6 but filter ICMPv4 — always test both
- HTTPS dominates web traffic; plaintext protocols are largely obsolete for web interaction but remain relevant for legacy systems and for understanding protocol fundamentals
- Despite encrypted protocols being standard, tools like telnet remain worth understanding because they demonstrate banner grabbing and cleartext protocol mechanics that carry forward to modern alternatives

**Critical Rule:** Never perform active reconnaissance without explicit, signed legal authorisation — a penetration testing contract or clearly defined bug bounty scope. Unauthorised probing is illegal in most jurisdictions.

**Practical Relevance:** From a red team perspective, the goal is to blend in with normal traffic. A browser visiting a website is indistinguishable from thousands of legitimate users. From a blue team perspective, active probes surface in access logs, firewall logs, WAF events, and IDS alerts — the same data defenders monitor to catch reconnaissance early.

---

### Web Browser as a Reconnaissance Tool

**Definition:** The web browser is one of the most capable and least suspicious tools for active reconnaissance. Its traffic mimics normal user activity, making it difficult for defenders to distinguish from legitimate browsing.

**Why It Matters:** Browsers provide immediate access to response headers, JavaScript source files, cookies, storage, and certificate details — all without requiring specialised tools.

**Key Details:**

**TCP port defaults:**
- Port 80: Plain HTTP (rare today — almost all sites redirect to HTTPS)
- Port 443: HTTPS (dominant)
- Port 443 (UDP): HTTP/3 over QUIC — a newer transport protocol combining TCP and TLS functions
- Non-standard ports can be accessed by specifying them explicitly: `https://target.com:8443/`

**Developer Tools (Ctrl + Shift + I on Linux/Windows, Option + Command + I on macOS):**

| Tab | Reconnaissance Use |
|-----|-------------------|
| Network | Real-time request/response headers, timing, cookies, status codes |
| Console | Execute JavaScript in page context, view errors |
| Sources | Browse JS/CSS/HTML files — frequently contains hardcoded API endpoints, directory structures, and developer comments |
| Application → Storage | Inspect cookies, Local Storage, Session Storage — may contain tokens or API keys |
| Security | Certificate details including SANs (Subject Alternative Names) that reveal additional subdomains |

**Browser Extensions:**
- **FoxyProxy:** Switch between proxies (Burp Suite, ZAP, SOCKS5)
- **User-Agent Switcher and Manager:** Emulate different browsers/devices to discover mobile-specific endpoints
- **Wappalyzer:** Automatically fingerprints CMS, web server, JS frameworks, CDNs, and analytics tools while browsing

**Practical Relevance:** The Sources tab in Developer Tools is one of the most productive browser-based recon techniques — JavaScript files frequently contain API endpoints, internal URLs, and developer comments never intended to be public. This discovery is often the difference between finding a hidden admin endpoint and missing it entirely.

---

### ping

**Definition:** `ping` sends ICMP Echo Request packets to a target and measures whether it responds (ICMP Echo Reply) and how long it takes.

**Why It Matters:** Confirms whether a host is alive and reachable before investing time in further enumeration. Also provides TTL values that suggest the target operating system.

**Key Details:**
- On Linux/macOS: `ping -c 10 10.10.0.1` (count limited)
- On Windows: `ping -n 10 10.10.0.1`
- IPv6: `ping -6 TARGET_IPv6` or `ping6 TARGET_IPv6`
- TTL values as OS indicators: Windows typically returns 128, Linux/macOS typically 64, network devices often 255
- Many firewalls and CDN-fronted hosts block ICMP — a non-response does not confirm the host is down
- Modern networks using anycast (Cloudflare, Akamai) may return ICMP responses from a CDN node rather than the actual server

**Practical Relevance:** A failed ping does not mean the host is unreachable for other protocols — TCP services may still respond even when ICMP is blocked. Always verify with port-level probing.

---

### traceroute

**Definition:** `traceroute` maps the path packets take from your system to a target by exploiting the TTL (Time to Live) field in IP headers.

**Why It Matters:** Reveals intermediate routers, identifies network filtering points, maps latency distribution, and helps understand the network topology between you and the target.

**Key Details:**

**How it works:** By sending packets with incrementally increasing TTL values starting at 1, each router along the path decrements the TTL by one. When it hits 0, the router drops the packet and sends back an ICMP Time-to-Live Exceeded message. This reveals the router's IP address. TTL 1 reveals hop 1, TTL 2 reveals hop 2, and so on until the destination is reached.

- Linux/macOS: `traceroute TARGET_IP` (sends UDP by default)
- Windows: `tracert TARGET_IP` (sends ICMP by default)
- IPv6: `traceroute -6 TARGET_IPv6` or `traceroute6 TARGET_IPv6`
- TCP mode (for bypassing UDP filters): `traceroute -T TARGET_IP`
- ICMP mode: `traceroute -I TARGET_IP`
- Real-time continuous monitoring: `mtr TARGET_IP`

**Three packets per hop:** Traceroute sends three packets at each TTL value — the output shows up to three IP addresses and round-trip times per hop. Different IPs at the same hop number indicate load balancing.

**Asterisks (`*`):** Routers configured not to send ICMP TTL-Exceeded messages appear as `*`. This is common in secure environments trying to prevent topology reconnaissance.

**Route variability:** Consecutive traceroutes to the same target can take completely different paths due to load balancing, anycast routing, and BGP dynamic routing updates. This is expected and normal.

**Practical Relevance:** Traceroute helps identify where filtering occurs along the path (hops that appear as `*` after some visible hops), can reveal hosting infrastructure transitions, and provides context for understanding why some ports may be reachable while others are not.

---

### Telnet

**Definition:** Telnet (Teletype Network, 1969) is a cleartext application-layer protocol for remote terminal access on port 23.

**Why It Matters:** While Telnet servers are rare on modern systems (replaced by SSH), the Telnet *client* remains useful as a simple tool for connecting to any TCP port and observing the server's initial response (banner grabbing).

**Key Details:**
- Telnet sends all data including credentials in cleartext — completely insecure for remote administration
- SSH is the universal replacement for interactive remote access
- Finding open Telnet (port 23) during a pentest is itself a significant finding — indicates legacy systems or misconfiguration
- As a client tool, telnet can connect to any TCP port: `telnet TARGET_IP 80` connects to a web server
- The server's initial response (the "banner") often reveals software name and version

**Banner grabbing via telnet (HTTP example):**
```
telnet TARGET_IP 80
GET / HTTP/1.1
host: anything

[press Enter twice]
```

The `Server:` response header reveals web server software and version (e.g., `Server: nginx/1.6.2`).

**For encrypted services:** Telnet cannot handle TLS. Use `curl --head https://TARGET_IP` or `openssl s_client -connect TARGET_IP:443` for HTTPS services.

**Practical Relevance:** Banner grabbing is a core initial technique for identifying service versions. Software version information feeds directly into CVE research and vulnerability exploitation planning. Even in 2024, finding a telnet server is a valid pentest finding with significant risk implications.

---

### Netcat

**Definition:** Netcat (`nc`) is a versatile networking utility that can function as both a TCP/UDP client and server. It is one of the most flexible tools in a penetration tester's toolkit.

**Why It Matters:** Netcat can connect to any TCP/UDP port (like telnet, but more flexible), listen on any port for incoming connections, transfer files, and serve as the foundation for basic reverse shell setups. It has none of telnet's limitations around protocols.

**Key Details:**

**As a client (banner grabbing):**
```bash
nc TARGET_IP 80
GET / HTTP/1.1
host: anything

[press Shift+Enter after the GET line]
```

**As a server (listener):**
```bash
nc -lvnp 4444       # Listen on port 4444
nc -lp 4444         # Equivalent - flag order flexible
```

**Netcat options:**

| Option | Meaning |
|--------|---------|
| `-l` | Listen mode |
| `-p` | Specify port number |
| `-n` | Numeric only — no DNS resolution |
| `-v` | Verbose output |
| `-vv` | Very verbose output |
| `-k` | Keep listening after client disconnects |

**Key behaviour:** The `-p` flag must appear directly before the port number. Ports below 1024 require root privileges. For IPv6, add `-6`.

**ncat (Nmap project):** The modern, enhanced version of netcat from the Nmap project. Supports IPv6 and SSL (`ncat --ssl`), making it more versatile than legacy netcat for encrypted connections.

**Practical Relevance:** Netcat is the workhorse of manual service interaction during penetration tests. It is also the basis for basic reverse shell setups (covered in exploitation modules). Understanding how to use netcat as both client and server is foundational.

---

## Methodology

Active reconnaissance follows a logical progression from host discovery to service-level detail:

```
Host alive? → ping TARGET_IP
        |
        v
Network path? → traceroute TARGET_IP
        |
        v
Web application? → Browser + Developer Tools
  - Response headers (server software, versions, frameworks)
  - JavaScript source files (endpoints, credentials, comments)
  - Cookies and storage (tokens, session data)
  - Certificate SANs (additional subdomains)
        |
        v
Specific port/service? → nc TARGET_IP PORT
  - Banner grabbing
  - Protocol-level interaction
        |
        v
Intelligence consolidated:
  - Software versions → CVE research
  - Endpoints discovered → further manual testing
  - Network topology → scope management
  - Service inventory → active scanning targets defined
```

Each technique answers a specific question before moving to the next. Confirming a host is alive (ping) before attempting banner grabbing avoids wasted time. Understanding the network path (traceroute) before scanning informs expectations about filtering. Developer Tools reveal what automated scanners miss — hardcoded endpoints and credentials in JavaScript.

---

## Practical Activities

### Activity 1 — Browser Developer Tools Reconnaissance

**Objective:** Extract server technology, response headers, JavaScript endpoints, and certificate information from a web application.

**Steps:**
1. Navigate to the target URL in Firefox or Chrome
2. Open Developer Tools: Ctrl + Shift + I (Linux/Windows) or Option + Command + I (macOS)
3. **Network tab:** Reload the page; inspect the first response — review `Server:`, `X-Powered-By:`, `Content-Security-Policy:` headers
4. **Sources tab:** Browse loaded JavaScript files; search for `api`, `endpoint`, `config`, `/admin`, comments
5. **Application → Cookies:** Note session cookie names, HttpOnly/Secure flags
6. **Security tab:** Review certificate details and SANs for additional subdomains

**Findings:**
- Server software and version from `Server:` header
- Backend technology from `X-Powered-By:` (if present)
- JavaScript files revealing API endpoints, internal paths, developer comments
- Session cookie security attributes
- SANs in certificate revealing related domains

**Why It Matters:** JavaScript source inspection frequently surfaces endpoints that no scanner would find — they are only in the client-side code. These become high-priority manual testing targets.

---

### Activity 2 — Host Discovery with ping

**Objective:** Confirm whether the target host is alive and gather OS-type hints from TTL values.

**Commands:**
```bash
# Linux/macOS - limited to 10 packets
ping -c 10 TARGET_IP

# IPv6
ping -6 TARGET_IPv6
ping6 TARGET_IPv6
```

**Command Explanation:**
- `-c 10` limits to 10 ICMP Echo Requests; without this, ping runs indefinitely on Linux
- Each response includes the TTL of the reply — used to estimate operating system type
- Round-trip time (RTT) provides latency context

**Findings:** ICMP Echo Reply responses confirm host is alive. TTL value suggests OS type. Absence of response indicates ICMP filtering (host may still be reachable via TCP).

**Why It Matters:** Confirming host reachability before further enumeration avoids wasted time. TTL-based OS estimation provides initial context for payload selection.

---

### Activity 3 — Network Path Mapping with traceroute

**Objective:** Map the network path between your machine and the target, identify filtering points, and understand the infrastructure layer.

**Commands:**
```bash
# Linux/macOS
traceroute TARGET_IP

# Windows
tracert TARGET_IP

# TCP mode (bypass UDP filtering)
traceroute -T TARGET_IP

# ICMP mode
traceroute -I TARGET_IP

# IPv6
traceroute -6 TARGET_IPv6
traceroute6 TARGET_IPv6

# Real-time continuous monitoring
mtr TARGET_IP
```

**Command Explanation:**
- Default Linux traceroute uses UDP datagrams; `-T` switches to TCP, `-I` to ICMP
- Each TTL value is incremented by 1, starting at 1, until the destination responds
- Three packets per hop — multiple IPs at the same hop indicate load balancing
- `*` at a hop means that router is not sending ICMP TTL-Exceeded responses

**Findings:**
- Number of hops between attacker and target
- Intermediate router IPs (may belong to third parties)
- Points where filtering occurs (runs of `*` followed by more visible hops suggest stateful firewalls)
- CDN/anycast infrastructure revealing the target is behind a proxy layer

**Why It Matters:** Understanding where filtering occurs helps explain why some services are unreachable. Identifying CDN infrastructure early prevents wasting effort scanning CDN nodes.

---

### Activity 4 — Banner Grabbing with Telnet

**Objective:** Connect to a TCP service and read its banner to identify software and version.

**Commands:**
```bash
# Connect to web server on port 80
telnet TARGET_IP 80
```

Then type:
```
GET / HTTP/1.1
host: anything

[Enter twice]
```

**Command Explanation:**
- `telnet TARGET_IP 80` opens a raw TCP connection to port 80
- The HTTP request tells the web server which resource to return
- The `Server:` header in the response reveals web server software and version

**Findings:**
- `Server: nginx/1.6.2` — reveals web server type and specific version for CVE research

**Why It Matters:** Specific version information from banners enables immediate vulnerability research. `nginx/1.6.2` can be cross-referenced against NVD/Exploit-DB for known CVEs. Version disclosure in server headers is itself a security finding worth noting in a report.

---

### Activity 5 — Banner Grabbing with Netcat

**Objective:** Use netcat as a more flexible alternative to telnet for banner grabbing and port probing.

**Commands:**
```bash
# Banner grabbing (client mode)
nc TARGET_IP 80
```

Then type:
```
GET / HTTP/1.1
host: anything

[Shift+Enter]
```

```bash
# Set up a listener (server mode)
nc -lvnp 4444

# Connect to the listener from another machine
nc TARGET_IP 4444
```

**Command Explanation:**
- `nc TARGET_IP 80` opens a TCP connection to port 80 — identical result to telnet but more flexible
- `nc -lvnp 4444`: `-l` listens, `-v` verbose output, `-n` no DNS, `-p 4444` on port 4444
- Once the listener connection is established, text typed on either side is transmitted to the other

**Findings:**
- Same banner information as telnet: `Server: nginx/1.6.2`
- Listener mode confirms bidirectional TCP connectivity — useful for testing reverse shell setups

**Why It Matters:** Netcat is preferred over telnet for banner grabbing in practice — it is more versatile, better maintained, and supports IPv6 and SSL (with ncat). The listener mode establishes the concept of reverse shell connectivity that becomes central in exploitation modules.

---

## Observations and Analysis

- **HTTP/3 and QUIC:** The appearance of `h3` in the browser's Network tab Protocol column indicates HTTP/3 over QUIC/UDP port 443. This is increasingly common on major sites. Traditional active recon tools that focus on TCP may miss UDP-based services.

- **Rotating paths:** Running traceroute twice to the same target produced paths of 14 hops and 26 hops respectively — a clear demonstration that network paths are not stable or predictable. This matters when attributing network latency or identifying where filtering occurs.

- **CDN detection:** Traceroute reaching Cloudflare infrastructure and Shodan identifying CDN tags are consistent signals that the real origin server is hidden. This is important to note before investing effort in scanning the discovered IPs directly.

- **Asterisks in traceroute output:** Hops showing `*` represent routers that suppress ICMP TTL-Exceeded messages. This is a common defensive configuration. It does not mean the path is broken — the route continues through those routers even though they don't reveal themselves.

- **Banner suppression as a security measure:** Security-conscious administrators configure web servers to suppress version information in the `Server:` header (e.g., returning `Server: nginx` without the version). Finding a suppressed banner is itself a useful observation — it suggests the target has some hardening awareness. Finding an unsuppressed banner is a reportable finding.

- **Non-standard ports:** Any service can run on any port. Using `-p-` in Nmap (full port scan) is required to find services on non-standard ports. The telnet and netcat skills from this room apply to any discovered port, not just the standards.

---

## Tools and Technologies Used

### Web Browser (Firefox/Chrome/Edge)
- **Purpose:** Initial web application reconnaissance via human-readable interface
- **Common Usage:** Navigate to target URL; use Developer Tools for deep inspection
- **In This Room:** Headers, JS source files, certificate SANs, cookie attributes

### ping
- **Purpose:** ICMP host reachability testing and TTL-based OS estimation
- **Common Usage:** `ping -c 10 TARGET_IP`
- **In This Room:** Confirming host is alive before further enumeration

### traceroute / tracert
- **Purpose:** Network path mapping using TTL-based hop discovery
- **Common Usage:** `traceroute TARGET_IP` (Linux) / `tracert TARGET_IP` (Windows)
- **In This Room:** Mapping hops, identifying filtering points, observing route variability

### mtr
- **Purpose:** Real-time continuous traceroute with packet loss statistics
- **Common Usage:** `mtr TARGET_IP`
- **In This Room:** Reference tool for continuous path monitoring

### telnet (client)
- **Purpose:** Raw TCP connection for banner grabbing and protocol-level interaction
- **Common Usage:** `telnet TARGET_IP PORT`
- **In This Room:** Demonstrated for HTTP banner grabbing and protocol fundamentals

### netcat (nc / ncat)
- **Purpose:** Flexible TCP/UDP client and server for port probing, banner grabbing, and connectivity testing
- **Common Usage:** `nc TARGET_IP PORT` (client), `nc -lvnp PORT` (server)
- **In This Room:** Banner grabbing and bidirectional TCP communication demonstration

---

## Key Learnings

- Active reconnaissance leaves footprints — every probe can appear in logs, IDS alerts, or WAF events
- The web browser with Developer Tools is a powerful reconnaissance platform that mimics normal user traffic
- JavaScript source files are consistently the most productive browser-based recon target — they contain API endpoints, internal paths, and developer comments that scanners miss
- `ping` confirms host liveness; absence of ICMP response does not confirm the host is down
- `traceroute` uses TTL exploitation to map network hops — routes are not stable and can differ between consecutive runs
- Banner grabbing with `telnet` or `nc` reveals software versions for CVE research before any formal scanning
- Netcat's dual client/server functionality makes it the most versatile manual TCP probing tool
- Version disclosure in server banners (e.g., `Server: nginx/1.6.2`) is a reportable security finding

---

## Real-World Relevance

**Penetration Testing:** Active recon provides service-level intelligence that passive methods cannot — running software versions, actual response behaviour, and technology stack details. These inform vulnerability selection before formal scanning.

**Bug Bounty Hunting:** Browser Developer Tools for JavaScript endpoint discovery is a high-value technique used by experienced hunters to find API routes and admin paths that are accessible but never publicly documented.

**Red Team Operations:** Slow, methodical active recon that mimics legitimate user behaviour is harder to detect than automated scanning. A red team using only browser-based techniques can build a detailed application map without triggering a single WAF alert.

**Enterprise Environments:** Understanding how active probes appear in logs helps security engineers build better detection rules. Knowing that traceroute generates ICMP TTL-Exceeded traffic or that banner grabbing produces unusual TCP connections helps tune IDS signatures.

---

## Things Worth Remembering

**Quick Reference:**

| Command | Platform | Purpose |
|---------|---------|---------|
| `ping -c 10 TARGET_IP` | Linux/macOS | Host liveness (count-limited) |
| `ping -n 10 TARGET_IP` | Windows | Host liveness (count-limited) |
| `ping -6 TARGET_IPv6` | Linux/macOS | IPv6 host liveness |
| `traceroute TARGET_IP` | Linux/macOS | Network path mapping |
| `tracert TARGET_IP` | Windows | Network path mapping |
| `traceroute -T TARGET_IP` | Linux | TCP-based traceroute (bypass UDP filters) |
| `traceroute -6 TARGET_IPv6` | Linux | IPv6 traceroute |
| `mtr TARGET_IP` | Linux/macOS | Real-time path monitoring |
| `telnet TARGET_IP PORT` | Cross-platform | Raw TCP + banner grabbing |
| `nc TARGET_IP PORT` | Linux/macOS | Client: raw TCP + banner grabbing |
| `nc -lvnp PORT` | Linux/macOS | Server: listen for incoming connections |
| `nc -6 TARGET_IPv6 PORT` | Linux/macOS | IPv6 netcat client |
| `curl -I http://TARGET_IP` | Cross-platform | HTTP headers (preferred over telnet) |

**Developer Tools shortcut:**
- Linux/Windows: `Ctrl + Shift + I`
- macOS: `Option + Command + I`

**Key concepts:**
- Active recon = direct interaction = traceable footprints
- Telnet and netcat are protocol-agnostic TCP clients — they work against any text-based TCP service
- TTL in ping responses suggests OS type (Windows ≈ 128, Linux ≈ 64)
- Route variability in traceroute is normal — CDN/anycast environments produce especially variable paths
- Banner suppression in headers = hardening awareness; full version in headers = reportable misconfiguration

---

## Conclusion

Active reconnaissance bridges the gap between passive intelligence gathering and targeted exploitation. This room establishes the foundational manual techniques — browser-based header and source inspection, ICMP host discovery, TTL-based path mapping, and TCP-level banner grabbing — that give a penetration tester direct, service-specific intelligence about a target. These skills build the intuition for understanding what automated scanners will later find and why, and they remain essential even as tooling evolves. The discipline of manually interacting with individual services before running full scans is what separates methodical testers from those who blindly launch Nmap and miss the context.
