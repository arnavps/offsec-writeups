# Nmap Post Port Scans

## Overview

Knowing that a port is open is only the beginning. The real investigative value comes from understanding what service is running, which version it is, what operating system is behind it, and how all these pieces connect to each other across the network. This room covers the post-port-scan capabilities of Nmap: service version detection (`-sV`), OS fingerprinting (`-O`), traceroute (`--traceroute`), the Nmap Scripting Engine (NSE), and the three practical output formats for saving and processing scan results.

**Main objectives:**
- Understand how Nmap identifies service versions and why this matters for vulnerability research
- Understand how OS detection works through TCP/IP fingerprinting
- Use Nmap's traceroute and how it differs from the standard traceroute command
- Understand the Nmap Scripting Engine structure, categories, and how to select and run scripts
- Save scan output in Normal, Grepable, and XML formats

**Skills introduced:** `-sV` version detection, `-O` OS detection, `--traceroute`, NSE (`-sC`, `--script`), `-oN`/`-oG`/`-oX`/`-oA` output formats

---

## Concepts Covered

### Service and Version Detection (`-sV`)

**Definition:** Service and version detection probes open ports to identify exactly what software is running and which version.

**Why It Matters:** An open port reveals a service category. Version detection reveals the specific software version — enabling cross-referencing against CVE databases and Exploit-DB for known vulnerabilities. The difference between `22/tcp open ssh` and `22/tcp open ssh OpenSSH 6.7p1 Debian 5+deb8u8` is the difference between a category and an actionable attack surface.

**Key Details:**
- Enabled with `-sV`
- Intensity control: `--version-intensity LEVEL` (0 lightest to 9 most complete)
- `-sV --version-light` = intensity 2 (quick, common probes only)
- `-sV --version-all` = intensity 9 (exhaustive, all probes)
- **Critical behaviour:** `-sV` forces a full TCP 3-way handshake and establishes a real connection to probe the service — stealth SYN scan is not possible when `-sV` is used
- The service column (e.g., `ssh`) is a guess based on port number; the version column is obtained by actual banner interaction

**Example output comparison:**
- Without `-sV`: `22/tcp open ssh`
- With `-sV`: `22/tcp open ssh OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)`
- Without `-sV`: `80/tcp open http`
- With `-sV`: `80/tcp open http nginx 1.6.2`

**Practical Relevance:** Version detection is one of the highest-value Nmap capabilities. `OpenSSH 6.7p1` can be searched directly in CVE databases. The combination of service version + OS information narrows exploitable vulnerabilities from hundreds to a handful of candidates.

---

### OS Detection (`-O`)

**Definition:** Nmap analyses TCP/IP characteristics of responses to estimate the target's operating system without asking the target directly.

**Why It Matters:** Knowing the OS helps select appropriate exploits, understand available attack vectors, and tailor payloads. Windows and Linux have different privilege escalation paths, different file system structures, and different service ecosystems.

**Key Details:**

**How OS detection works — fingerprinting signals:**
- **TTL (Time to Live):** Values in IP responses suggest OS type
  - Linux systems typically respond with TTL ≈ 64
  - Windows systems commonly use TTL ≈ 128
  - Network devices often use TTL ≈ 255
- **TCP sequence behaviour:** How a system generates and increments TCP sequence numbers varies between OS families and kernel versions
- **Service responses:** Banner text, protocol quirks, and timing patterns differ between OS families

**Limitations:**
- Virtualisation, cloud networking layers, and firewalls can alter TCP/IP behaviour, making exact identification harder
- Kernel customisation and network middleboxes modify packet characteristics
- Result: "No exact OS matches for host" is common — the absence of a match is not a failure, it is still useful fingerprint data
- OS detection results should always be treated as indicative and confirmed with additional techniques

**Command:** `sudo nmap -sS -O TARGET_IP`

**Practical Relevance:** Even when `-O` returns "No exact match," the TTL and TCP sequence behaviour information is still useful. A TTL of 64 strongly suggests Linux; a TTL of 128 strongly suggests Windows. This guides tool selection and payload construction for subsequent phases.

---

### Traceroute (`--traceroute`)

**Definition:** Nmap can append a traceroute to its scan to map the network hops between the scanner and the target.

**Why It Matters:** Understanding the network path reveals intermediate infrastructure (routers, firewalls, load balancers), network segmentation, and geographic distribution of infrastructure. This information supports scope decisions and lateral movement planning.

**Key Details:**

**How Nmap traceroute differs from standard traceroute:**
- Standard traceroute: starts with TTL=1 and increments upward until reaching the destination
- Nmap traceroute: starts with a high TTL and decrements downward
- Both reveal the same hops — just in different scanning order

**Hop count observation:** Running `nmap -sS --traceroute TARGET` against a remote host shows the number of hops. A hop count of 3 indicates the target is relatively close (same data centre or local network segment).

**Command:** `sudo nmap -sS --traceroute TARGET_IP`

**Practical Relevance:** During external assessments, traceroute reveals hosting providers and CDN infrastructure. During internal assessments, it maps the internal network topology between your position and the target.

---

### Nmap Scripting Engine (NSE)

**Definition:** NSE is a Lua interpreter embedded in Nmap that allows running automated scripts against discovered hosts and services to gather additional information, test for vulnerabilities, or perform brute-force authentication.

**Why It Matters:** NSE dramatically extends Nmap's capability beyond port scanning. With 500+ built-in scripts and the ability to run custom ones, NSE can perform reconnaissance, vulnerability detection, and in some cases exploitation — all within a single tool.

**Key Details:**

**Script location:** `/usr/share/nmap/scripts/` — scripts are named by protocol prefix (e.g., `http-*`, `ssh-*`, `ftp-*`)

**Script categories:**

| Category | Description |
|----------|-------------|
| `auth` | Authentication-related testing |
| `brute` | Brute-force password auditing |
| `default` | Default scripts run with `-sC`; safe and informative |
| `discovery` | Retrieves additional information (DB tables, DNS names) |
| `dos` | Tests for DoS-vulnerable services |
| `exploit` | Attempts service exploitation |
| `external` | Uses third-party services (VirusTotal, Geoplugin) |
| `fuzzer` | Fuzzing attacks |
| `intrusive` | Aggressive scripts including brute force and exploitation |
| `malware` | Scans for backdoors |
| `safe` | Scripts safe to run without crashing the target |
| `version` | Service version retrieval |
| `vuln` | Checks for known vulnerabilities |

**Critical caution:** Some NSE script categories (`exploit`, `brute`, `dos`, `intrusive`) are aggressive and can crash services, lock accounts, or cause damage. Always confirm authorisation and understand what a script does before running it — read the script file if uncertain.

**Running scripts:**
```bash
# Default scripts (same as -sC)
sudo nmap -sS --script=default TARGET

# Shorthand for default scripts
sudo nmap -sS -sC TARGET

# Specific script by name
sudo nmap -sS --script "http-date" TARGET

# Pattern matching
sudo nmap -sS --script "ftp*" TARGET  # runs all ftp- scripts

# Vulnerability category
sudo nmap -sS --script=vuln TARGET
```

**What default scripts reveal:**
- SSH public keys (`-sC` against port 22)
- HTTP default page titles (`-sC` against port 80)
- SMB host information
- SSL/TLS certificate details
- Service-specific configuration information

**Practical Relevance:** Running `-sC` alongside `-sV` is the standard combination for initial service enumeration. It fills in details that version banners alone miss — the SSH host key fingerprint, the HTTP page title (reveals default installations), and SSL certificate SANs (reveals additional hostnames).

---

### The `-A` Flag

**Definition:** `-A` is a convenience flag that enables multiple detection features simultaneously.

**Why It Matters:** Rather than specifying `-sV -O -sC --traceroute` individually, `-A` runs all four in a single flag. Useful for comprehensive scans where you want the full picture.

**Key Details:**
- `-A` = `-sV` (version detection) + `-O` (OS detection) + `-sC` (default scripts) + `--traceroute`
- More aggressive and more detectable than individual flags
- Ideal for internal assessments or practice environments where stealth is not critical

**Command:** `sudo nmap -A TARGET_IP`

---

### Output Formats

**Definition:** Nmap can save scan results in multiple formats for documentation, processing, and future reference.

**Why It Matters:** Scan results are working documents. Saving them enables review without re-scanning, post-engagement reporting, grep-based analysis, and import into other tools. Good output file management is a professional habit.

**Key Details:**

**Normal format (`-oN`):**
- Human-readable, similar to terminal output
- Easy to read; less efficient for programmatic processing
- Command: `nmap -oN scan.nmap TARGET`

**Grepable format (`-oG`):**
- Each line is self-contained with host IP + all findings
- Designed for filtering with `grep`
- Significantly more compact than normal format (21 lines normal vs 4 lines grepable for same data)
- Normal grep returns `80/tcp open http nginx 1.6.2` — missing the host IP, useless when analysing multiple targets
- Grepable grep returns a complete line including IP, allowing multi-host correlation
- Command: `nmap -oG scan.gnmap TARGET`

**XML format (`-oX`):**
- Machine-readable structured data
- Most convenient for import into other tools (Metasploit, vulnerability management platforms)
- Command: `nmap -oX scan.xml TARGET`

**All formats simultaneously (`-oA`):**
- Saves Normal + Grepable + XML in one command: `nmap -oA scan TARGET`
- Creates three files: `scan.nmap`, `scan.gnmap`, `scan.xml`
- Best practice for comprehensive scan documentation

**Script Kiddie format (`-oS`):**
- Transforms output to "l33t speak" — novelty format with no practical value
- Not recommended for any professional or learning purpose

**Practical Relevance:** `-oA FILENAME` should be a habit in every scan. The time investment is zero, and it preserves results for reporting, evidence documentation, and future reference without needing to re-scan.

---

## Methodology

```
Open ports identified from previous scan phases
        |
        v
Service and version detection:
  sudo nmap -sV TARGET         (standard)
  sudo nmap -sV --version-light TARGET  (quick)
  sudo nmap -sV --version-all TARGET   (exhaustive)
        |
        v
Version information → cross-reference CVE databases:
  nvd.nist.gov
  exploit-db.com
  searchsploit [software version]
        |
        v
OS detection:
  sudo nmap -sS -O TARGET
  Analyse TTL values, TCP sequence patterns
        |
        v
Network path mapping:
  sudo nmap -sS --traceroute TARGET
        |
        v
NSE default scripts for additional detail:
  sudo nmap -sS -sC TARGET
  Review:
    - SSH host keys
    - HTTP page titles (default installations?)
    - SSL certificate details (SANs = additional hostnames)
        |
        v
Targeted NSE scripts based on findings:
  sudo nmap -sS --script "http-*" TARGET   (HTTP services)
  sudo nmap -sS --script "vuln" TARGET     (vulnerability check)
        |
        v
Save all results:
  sudo nmap -A -oA scan TARGET
        |
        v
Consolidate intelligence:
  - Software versions + CVEs → exploitation candidates
  - OS type → privilege escalation path selection
  - Network hops → infrastructure mapping
  - Script output → configuration details and default credentials
```

---

## Practical Activities

### Activity 1 — Service Version Detection

**Objective:** Identify specific software and version information running on open ports.

**Commands:**
```bash
sudo nmap -sV --version-light 10.201.118.127
sudo nmap -sV --version-all TARGET_IP
```

**Command Explanation:**
- `-sV` forces full TCP connection to each open port to read banners and service responses
- `--version-light` (intensity 2): runs the most likely probes for faster results
- `--version-all` (intensity 9): runs all available probes for maximum coverage

**Findings:**
- Port 22: `OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)` — specific version, OS family
- Port 80: `nginx 1.6.2` — web server name and version
- Service column (`ssh`, `http`) is based on port number; version column is based on actual banner interaction

**Why It Matters:** `OpenSSH 6.7p1` and `nginx 1.6.2` are specific, searchable version strings. Vulnerability databases (NVD, Exploit-DB) can immediately return known CVEs for these exact versions.

---

### Activity 2 — OS Detection

**Objective:** Determine the target's operating system through TCP/IP fingerprinting.

**Commands:**
```bash
sudo nmap -sS -O 10.48.160.237
```

**Findings:**
- "No exact OS matches for host" — common result in virtualised and cloud environments
- TTL values and TCP sequence patterns still confirm a Linux target (TTL ≈ 64)
- OS detection fingerprint data available even without an exact database match

**Why It Matters:** TTL of 64 strongly indicates Linux. This immediately guides tool selection, exploitation path, and privilege escalation technique choices. An exact match is valuable but not required for actionable intelligence.

---

### Activity 3 — Traceroute

**Objective:** Map the network path from scanner to target.

**Commands:**
```bash
sudo nmap -sS --traceroute 10.48.160.237
```

**Findings:**
- Hop count of 3 — target is 3 routers away
- Intermediate router IPs visible
- Traceroute data appended to standard scan output

**Why It Matters:** 3 hops suggests the target is in the same data centre or network region. For internal assessments, low hop counts indicate targets on the same or adjacent network segment — relevant for lateral movement planning.

---

### Activity 4 — Default NSE Scripts

**Objective:** Run Nmap's default scripts to extract additional service details.

**Commands:**
```bash
sudo nmap -sS -sC 10.201.118.127
sudo nmap -sS --script=default TARGET
```

**Findings:**
- Port 22 (SSH): all SSH public keys associated with the server are listed
- Port 80 (HTTP): page title extracted — confirms default configuration (page left as default)
- Additional protocol details that banner-only scanning misses

**Why It Matters:** Default page titles confirm default installations — a direct finding. SSH host keys confirm the server's identity and can be tracked across sessions.

---

### Activity 5 — Specific NSE Script

**Objective:** Run a targeted NSE script by name.

**Commands:**
```bash
sudo nmap -sS -n --script "http-date" 10.48.160.237
```

**Command Explanation:**
- `--script "http-date"` runs the specific script that retrieves HTTP server date/time
- `-n` disables DNS resolution (faster, quieter)
- Script output appears in its own section below the port table

**Findings:**
- HTTP server date and time
- Comparison between server time and local time — useful for understanding time sync configuration and for timestamp-based attack timing

**Why It Matters:** Server time information is used in various attack techniques (Kerberos authentication in Active Directory requires time synchronisation within 5 minutes). Identifying time skew is a useful finding.

---

### Activity 6 — Comprehensive Scan with Output

**Objective:** Run a full scan with all detection features and save results in all formats.

**Commands:**
```bash
sudo nmap -A -oA scan TARGET_IP
# Creates: scan.nmap (normal), scan.gnmap (grepable), scan.xml (XML)

# Verify output
cat scan.nmap
grep http scan.gnmap   # IP-aware grep - includes host IP in each line
```

**Command Explanation:**
- `-A` = `-sV -O -sC --traceroute` (all detection features)
- `-oA scan` saves to three files simultaneously with base name "scan"
- `grep http scan.gnmap` returns complete lines including host IP — useful when analysing multiple targets

---

## Observations and Analysis

- **`-sV` breaks SYN stealth:** Adding `-sV` to a SYN scan forces full TCP connections because version detection requires actual service interaction. This is a trade-off: you get version information but lose stealth. Plan scan phases accordingly — run SYN-only first for port discovery, then `-sV` for version detection.

- **OS detection limitations are informative:** A "no exact match" result from `-O` is not a failure — it often indicates a virtualised environment, cloud instance, or hardened system that has been configured to minimise OS fingerprinting. This is itself valuable intelligence.

- **NSE scripts require trust:** Running NSE scripts from unknown authors carries risk. Before running any script, read the script file to understand what it does. Scripts in the `brute`, `exploit`, and `dos` categories can cause significant damage to target services.

- **Default script reveals default configurations:** When `-sC` returns an HTTP page title of "Apache2 Ubuntu Default Page" or "Welcome to nginx," this is a direct finding — the web server was never configured. Default configurations often retain default credentials and expose unnecessary functionality.

- **Grepable output for multi-host correlation:** When analysing scans against many hosts, grepable format (`-oG`) is far more useful than normal format because each line contains the host IP. `grep "443/open" scan.gnmap` immediately returns all hosts with HTTPS, along with their IPs.

- **`-oA` as standard practice:** Adopting `-oA FILENAME` as a default habit requires no extra thought and ensures results are always preserved. File naming convention matters — use descriptive names including target and date: `nmap_external_10.0.0.0-24_2024-01-15`.

---

## Tools and Technologies Used

### Nmap (post-port capabilities)
- **Purpose:** Service version detection, OS fingerprinting, traceroute, scripting, output formatting
- **Common Usage:** `sudo nmap -sV -sC -O --traceroute -oA FILENAME TARGET`
- **In This Room:** `-sV`, `-O`, `--traceroute`, NSE (`-sC`, `--script`), `-oN`/`-oG`/`-oX`/`-oA`

---

## Key Learnings

- `-sV` forces full TCP connections — stealth SYN scan is incompatible with version detection
- Version information is the bridge between "port is open" and "this specific CVE applies"
- `-O` produces "no exact match" in virtualised/cloud environments but TTL values still indicate OS family
- Nmap's traceroute starts with high TTL and decrements (opposite of standard traceroute)
- NSE scripts extend Nmap from port scanner to reconnaissance and vulnerability detection platform
- Default scripts (`-sC`) reveal SSH keys, HTTP titles, and SSL certificates without extra configuration
- `intrusive`/`exploit`/`brute` NSE categories can damage services — always verify authorisation
- `-oA FILENAME` saves Normal + Grepable + XML simultaneously — adopt as standard practice
- Grepable format includes host IP in each line — essential for multi-host analysis
- `-A` flag combines `-sV -O -sC --traceroute` in a single convenience flag

---

## Real-World Relevance

**Penetration Testing:** The combination `sudo nmap -sV -sC -O -oA scan TARGET` is executed on every engagement. Version information feeds directly into CVE research and Metasploit module selection. NSE vulnerability scripts (`--script=vuln`) provide a quick sweep for obvious exploitable services.

**Bug Bounty Hunting:** Version detection on in-scope infrastructure identifies outdated software — common high-value findings. Older nginx/Apache versions with known CVEs are a reliable target for quick wins.

**Enterprise Security:** Regular internal Nmap scans with `-sV -oA` create a service inventory baseline. Changes between scans reveal new services (potential compromises or policy violations) and version changes (confirms patching).

**Vulnerability Management:** XML output (`-oX`) is the standard format for importing Nmap results into vulnerability management platforms (Tenable, Qualys, OpenVAS) for automated CVE correlation and prioritisation.

**Incident Response:** Quick `nmap -sV -T4 TARGET` scans against suspicious hosts provide immediate intelligence about what services are running and what versions — accelerating investigation without waiting for a full Nessus scan.

---

## Things Worth Remembering

**Core post-scan commands:**
```bash
# Service version detection
sudo nmap -sV TARGET
sudo nmap -sV --version-light TARGET    # faster
sudo nmap -sV --version-all TARGET      # exhaustive

# OS detection
sudo nmap -sS -O TARGET

# Traceroute
sudo nmap -sS --traceroute TARGET

# Default NSE scripts
sudo nmap -sS -sC TARGET

# Specific script
sudo nmap -sS --script "SCRIPT_NAME" TARGET

# Pattern match scripts
sudo nmap -sS --script "http*" TARGET

# All detection features
sudo nmap -A TARGET

# Save output (all three formats)
sudo nmap -A -oA FILENAME TARGET
```

**Output formats:**
```bash
-oN filename.nmap     # Normal (human-readable)
-oG filename.gnmap    # Grepable (IP in each line)
-oX filename.xml      # XML (machine-parseable)
-oA filename          # All three simultaneously
```

**NSE categories (risk-ordered):**
- `safe`, `discovery`, `version` — safe to run
- `default` — generally safe; standard with `-sC`
- `auth`, `vuln` — moderate risk; useful, check scope
- `brute`, `exploit`, `dos`, `intrusive` — aggressive; explicit permission required

**Option reference table:**

| Option | Meaning |
|--------|---------|
| `-sV` | Service and version detection |
| `-sV --version-light` | Intensity 2 (quick) |
| `-sV --version-all` | Intensity 9 (exhaustive) |
| `-O` | OS detection |
| `--traceroute` | Network path mapping |
| `-sC` | Default scripts |
| `--script=SCRIPT` | Run specific script(s) |
| `-A` | `-sV -O -sC --traceroute` combined |
| `-oN` | Normal output |
| `-oG` | Grepable output |
| `-oX` | XML output |
| `-oA` | All three formats |

---

## Conclusion

Nmap Post Port Scans transforms raw port discovery into actionable intelligence. Service version detection bridges the gap between "port is open" and "this specific software version has these specific CVEs." OS fingerprinting guides payload selection. The Nmap Scripting Engine extends the tool into a comprehensive service interrogation platform, revealing configuration details, cryptographic material, and vulnerability indicators without requiring additional tools. Saving results in appropriate formats ensures findings are preserved, shareable, and processable by downstream tools. Together, these capabilities represent the complete Nmap workflow — from initial host discovery through to the intelligence that drives exploitation planning.
