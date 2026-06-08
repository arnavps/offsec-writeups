# Logs Fundamentals

## Overview

Logs are the digital equivalent of footprints — a record of activity left behind by every process, user, system, and application. In security, logs are the primary source of evidence for detecting attacks, investigating incidents, and performing forensic analysis. This room covers what logs are, why they matter, the major log types, Windows event logs and their key event IDs, web server access logs, and how to perform basic log analysis using Linux command-line tools.

---

## Topics Covered

- Why logs matter in security operations
- Major log types and their uses
- Windows event log categories
- Key Windows Event IDs
- Web server access log structure (Apache)
- Linux command-line log analysis: `cat`, `grep`, `less`

---

## Key Concepts

### Why Logs Matter

Attackers work to minimise the traces they leave behind — but they can never eliminate them entirely. Logs capture the trail. Security teams use that trail to detect ongoing attacks, investigate past incidents, and build a timeline of what happened.

| Use Case | Description |
|----------|-------------|
| Security Events Monitoring | Real-time log analysis enables detection of anomalous activity as it happens |
| Incident Investigation and Forensics | Logs provide a detailed record of activity during an incident — essential for root cause analysis |
| Troubleshooting | Application and system errors recorded in logs help diagnose and fix issues |
| Performance Monitoring | Logs provide insight into application and system performance over time |
| Auditing and Compliance | Logs establish an activity trail required by compliance frameworks (PCI-DSS, HIPAA, ISO 27001, etc.) |

---

### Log Types

| Log Type | Primary Use | Examples |
|----------|-------------|---------|
| **System Logs** | OS-level troubleshooting and monitoring | Startup/shutdown events, driver loading, hardware errors, service status |
| **Security Logs** | Security incident detection and investigation | Authentication events, authorisation changes, security policy modifications, user account changes, abnormal activity |
| **Application Logs** | Application-specific event tracking | User interactions, application errors, update events, configuration changes |
| **Audit Logs** | Compliance and detailed activity tracking | Data access events, system changes, user activity, policy enforcement |
| **Network Logs** | Network traffic monitoring and investigation | Inbound/outbound traffic, connection logs, firewall logs |
| **Access Logs** | Resource access tracking | Web server access, database access, API access |

> Different applications and services generate additional log types specific to their function.

---

### Windows Event Logs

Windows stores event logs in categorised files. The three most important categories from a security perspective are:

| Log File | Contents |
|----------|---------|
| **Security** | The most security-relevant log. Records authentication events, user account changes, privilege use, security policy modifications |
| **System** | OS-level events — driver issues, hardware events, service start/stop, system startup and shutdown |
| **Application** | Events from applications running on the OS — errors, warnings, compatibility issues |

#### Windows Event Log Fields

Each Windows event log entry contains:

| Field | Description |
|-------|-------------|
| **Description** | Detailed information about the logged activity |
| **Log Name** | The log file category (Security, System, Application, etc.) |
| **Logged** | Timestamp of when the event occurred |
| **Event ID** | A unique numeric identifier for the specific type of activity |

Event IDs are the most efficient way to search for specific activity — instead of reading through thousands of log lines, you filter by the Event ID for the action you are investigating.

#### Key Windows Event IDs

| Event ID | Description |
|----------|-------------|
| 4624 | Successful user login |
| 4625 | Failed user login attempt |
| 4634 | Successful user logoff |
| 4720 | A user account was created |
| 4722 | A user account was enabled |
| 4724 | An attempt was made to reset an account's password |
| 4725 | A user account was disabled |
| 4726 | A user account was deleted |

> These are the most commonly referenced IDs in security investigations. Many more exist — it is not necessary to memorise all of them, but knowing these critical IDs significantly speeds up investigations.

---

### Web Server Access Logs (Apache)

Apache web server access logs are stored at `/var/log/apache2/access.log`. Every HTTP request to the server generates one log entry.

#### Log Entry Fields

Example log line:
```
172.16.0.1 - - [06/Jun/2024:13:58:44] "GET / HTTP/1.1" 200 - "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36"
```

| Field | Example Value | Description |
|-------|--------------|-------------|
| IP Address | `172.16.0.1` | IP address of the client making the request |
| Timestamp | `[06/Jun/2024:13:58:44]` | Date and time the request was received |
| HTTP Method | `GET` | The action requested (GET, POST, PUT, DELETE, etc.) |
| URL | `/` | The resource path requested |
| Status Code | `200` | Server response code (200 = OK, 404 = Not Found, 500 = Server Error) |
| User-Agent | `Mozilla/5.0 ...` | Information about the client's OS and browser |

Web server access logs are invaluable during incident investigation — they show exactly what requests were made to the server, from which IP, at what time, and what the server returned.

---

## Practical Examples / Demonstrations

### Linux Command-Line Log Analysis

#### `cat` — Display a log file

```bash
cat /var/log/apache2/access.log
```

Outputs the entire contents of the log file to the terminal. Useful for smaller log files.

#### `grep` — Search for a pattern in a log

Search for a specific IP address in the access log:
```bash
grep "192.168.1.1" /var/log/apache2/access.log
```

Returns all lines containing `192.168.1.1`. `grep` is the primary tool for targeted log searching — filter by IP, URL, status code, user agent, keyword, or any other string.

Search for all POST requests:
```bash
grep "POST" /var/log/apache2/access.log
```

Search for 404 errors:
```bash
grep " 404 " /var/log/apache2/access.log
```

#### `less` — Page through a large log file

```bash
less /var/log/apache2/access.log
```

Displays the log one page at a time — essential for large files that would flood the terminal with `cat`.

| Key | Action |
|-----|--------|
| `Space` | Move to the next page |
| `b` | Move to the previous page |
| `/pattern` | Search for a string — type `/` then the search term and press Enter |
| `n` | Jump to the next occurrence of the search |
| `N` | Jump to the previous occurrence of the search |
| `q` | Quit |

---

## Workflow / Process

```
Security event occurs (login attempt, file access, network connection, etc.)
        |
        v
Activity is recorded in the relevant log file
(Security log, access log, application log, etc.)
        |
        v
Log is ingested by SIEM or available for manual analysis
        |
        v
Analyst searches logs for specific Event ID, IP, timestamp, or pattern
using grep, SIEM query, or a forensics tool
        |
        v
Relevant log entries are extracted and correlated with other sources
        |
        v
Timeline of events is reconstructed
        |
        v
Findings documented in incident report or investigation notes
```

---

## Real-World Relevance

- Logs are the primary data source for every SIEM platform — no logs, no detection
- Windows Event ID 4625 (failed login) is one of the most commonly searched IDs in security investigations — brute force attacks generate thousands of these
- Web server access logs are analysed during web application incident investigations to reconstruct what an attacker requested, which paths they accessed, and when
- Centralised log management (sending all logs to a SIEM or a dedicated log aggregator) is a security baseline requirement — if logs only exist on the host, they can be deleted by an attacker
- Log retention policies directly affect how far back an investigation can go — organisations with short retention windows (30 days) may not have logs for a breach discovered months later
- Compliance frameworks (PCI-DSS, ISO 27001, HIPAA) mandate specific log types and retention periods

---

## Key Learnings

- Logs record every significant activity across systems, networks, and applications — they are the primary evidence source in security investigations
- The six major log types are: System, Security, Application, Audit, Network, Access
- Windows Security logs are the most relevant for security investigations
- Windows Event IDs uniquely identify specific activity types — 4624 (login success), 4625 (login failure), 4720 (account created) are the most critical
- Apache access logs contain: IP, timestamp, HTTP method, URL, status code, and User-Agent
- `cat` displays files, `grep` filters by pattern, `less` pages through large files

---

## Additional Notes

- Log timestamps are critical — ensure all systems are synchronised to a consistent time source (NTP). Inconsistent timestamps make cross-source correlation unreliable
- Log forwarding to a centralised SIEM protects logs from being deleted by an attacker who compromises the source system
- Failed login attempts (Event ID 4625) followed closely by a successful login (4624) from the same source is a common indicator of a brute force attack followed by a successful compromise
- `grep -v` excludes lines matching a pattern — useful for filtering out known-good sources and reducing noise
- Combining `grep` with `sort` and `uniq -c` is a powerful pattern for counting occurrences (e.g. counting requests per IP in an access log)

---

## Conclusion

Logs are the foundational evidence layer in defensive security. Without logs, detection is blind, investigations have no data, and compliance cannot be demonstrated. Understanding the major log types, how Windows event logs are structured and searched, how web server access logs are read, and how to query logs from the command line builds the practical skills needed for SOC analysis, incident response, and digital forensics work.
