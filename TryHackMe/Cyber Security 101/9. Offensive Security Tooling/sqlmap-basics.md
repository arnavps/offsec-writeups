# SQLMap: The Basics

## Overview

SQL Injection (SQLi) is one of the most prevalent and impactful vulnerabilities in web application security. SQLMap is an automated tool that detects and exploits SQL injection vulnerabilities, turning what would otherwise be a highly manual process into a streamlined workflow. This room introduces the fundamentals of SQL injection, how SQLMap works, and how to use it to extract data from vulnerable databases.

---

## Topics Covered

- What SQL injection is and why it matters
- How authentication bypass via SQLi works
- What SQLMap is and what it automates
- Key SQLMap flags for enumeration and extraction
- HTTP GET-based testing
- Cookie-based authenticated testing

---

## Key Concepts

### How Web Applications Use Databases

Web applications store and retrieve data from databases constantly. When a user logs in, the application constructs an SQL query using the input and checks it against the database:

```sql
SELECT * FROM users WHERE username = 'John' AND password = 'Un@detectable444';
```

If the credentials match a record, the user is authenticated. This query structure is the foundation of how SQLi works — because the user's input is directly embedded into the query.

---

### What is SQL Injection?

SQL injection occurs when user-supplied input is embedded into an SQL query **without proper sanitisation**, allowing an attacker to inject SQL syntax that changes the query's logic.

#### Authentication Bypass Example

An attacker who doesn't know a user's password submits:
- Username: `John`
- Password: `abc' OR 1=1;-- -`

The application constructs:
```sql
SELECT * FROM users WHERE username = 'John' AND password = 'abc' OR 1=1;-- -';
```

Breaking this down:
- `AND password = 'abc'` — fails, because `abc` is not the real password
- `OR 1=1` — always evaluates to true
- Since `OR` requires only one condition to be true, the overall query succeeds
- `-- -` comments out everything after `1=1`, preventing a syntax error from the trailing quote

The database returns the user record, and the attacker is logged in as John without ever knowing the password.

**Why the single quote matters:**
Without the `'` after `abc`, the entire string `abc OR 1=1;-- -` would be treated as the password value. The single quote closes the string literal in the SQL syntax, allowing the injected SQL to be interpreted as logic rather than data.

---

### Impact of SQL Injection

A successful SQLi attack can result in:
- Authentication bypass
- Full database enumeration (all tables, all data)
- Extraction of credentials, PII, and sensitive records
- In some cases: reading/writing files on the server, executing OS commands

SQLi is classified as a critical vulnerability and appears consistently in the OWASP Top 10.

---

### What is SQLMap?

SQLMap is an open-source command-line tool that automates the detection and exploitation of SQL injection vulnerabilities. It handles:
- Detecting whether a parameter is injectable
- Identifying the database type and version
- Extracting database names, table names, and column data
- Bypassing common WAF/filter patterns

It is built into Kali Linux and is also available on most penetration testing distributions.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SQL Injection | Inserting SQL syntax into user input to manipulate database queries |
| Sanitisation | Validating and escaping user input so it cannot alter SQL query logic |
| GET parameter | A value passed in the URL query string (e.g. `?cat=1`) |
| Session cookie | A cookie that identifies an authenticated session (e.g. `PHPSESSID`) |
| DBMS | Database Management System — e.g. MySQL, PostgreSQL, MSSQL |
| Enumeration | Listing available databases, tables, and columns |
| Dump | Extracting all records from a specified table |

---

## Practical Examples / Demonstrations

### Getting Started

```bash
sqlmap --help
```

For guided, interactive usage (useful for beginners):
```bash
sqlmap --wizard
```

---

### HTTP GET-Based Testing

When a web application passes parameters in the URL (e.g. `http://sqlmaptesting.thm/search?cat=1`), the URL itself can be passed to SQLMap for testing:

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1"
```

SQLMap will test the `cat` parameter for injectable patterns.

---

### Enumerating Databases

Once a vulnerable parameter is identified, extract all database names:

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" --dbs
```

---

### Enumerating Tables in a Database

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" -D database_name --tables
```

Replace `database_name` with the name returned from `--dbs`.

---

### Dumping Table Contents

```bash
sqlmap -u "http://sqlmaptesting.thm/search?cat=1" -D database_name -T table_name --dump
```

This extracts all records from the specified table — usernames, passwords, emails, or whatever is stored there.

---

### Cookie-Based Authenticated Testing

Many web applications require authentication before certain pages or parameters are reachable. SQLMap supports injecting session cookies to test authenticated endpoints:

```bash
sqlmap -u "http://sqlmaptesting.thm/dashboard" --cookie="PHPSESSID=abcdef123456"
```

**Why this matters:**
Without providing a valid session cookie, SQLMap would test the application as an unauthenticated user. Authenticated pages may have different, more privileged injection points that would be missed otherwise. You can capture the session cookie from your browser's DevTools after logging in manually.

---

### Key SQLMap Flags

| Flag | Description |
|------|-------------|
| `-u` | Target URL to test |
| `--dbs` | Enumerate all database names |
| `-D <name>` | Specify the database to query |
| `--tables` | Enumerate tables in the specified database |
| `-T <name>` | Specify the table to query |
| `--dump` | Extract all records from the specified table |
| `--cookie` | Include a session cookie for authenticated testing |
| `--wizard` | Interactive guided mode — prompts for options step by step |
| `--help` | List all available flags and options |
| `--batch` | Non-interactive mode — automatically accepts default answers |
| `--level` | Depth of testing (1–5). Higher levels test more parameters |
| `--risk` | Risk of payloads (1–3). Higher risk includes more aggressive tests |

---

## Workflow / Process

```
Identify a potentially injectable parameter
(GET param in URL, POST field, or cookie value)
        |
        v
Run SQLMap against the target with -u and the parameter
        |
        v
SQLMap confirms vulnerability and identifies the DBMS type
        |
        v
Run --dbs to enumerate available databases
        |
        v
Run -D <db> --tables to enumerate tables in a target database
        |
        v
Run -D <db> -T <table> --dump to extract all records
        |
        v
Analyse extracted data (credentials, PII, etc.)
```

---

## Real-World Relevance

- SQL injection remains one of the most common vulnerabilities found in web application penetration tests and bug bounty programs
- The authentication bypass technique (`OR 1=1`) directly mirrors how real attackers bypass login forms on unprotected applications
- Credential databases extracted via SQLi are commonly used for credential stuffing attacks against other services
- SQLMap is a standard tool in OSCP, real-world web app assessments, and CTF challenges
- Parameterised queries (prepared statements) are the primary defence against SQLi — SQLMap's success directly demonstrates the consequence of not using them
- Cookie-based testing is particularly relevant for modern web apps where almost all sensitive functionality is behind authentication

---

## Key Learnings

- SQL injection works by injecting SQL logic into input fields that are not properly sanitised
- The `OR 1=1` condition is always true, allowing authentication bypass when injected into a password field
- The single quote `'` is critical — it closes the string literal and allows SQL to be injected as logic
- `-- -` comments out the rest of the query, preventing syntax errors from the injected code
- SQLMap automates detection and exploitation: `--dbs` → `-D --tables` → `-D -T --dump`
- Use `--cookie` to test parameters that are only accessible after authentication

---

## Additional Notes

- SQLMap may trigger WAF rules or rate limiting on real targets — use `--delay`, `--random-agent`, or `--tamper` scripts for evasion in authorised tests
- The `--dump` command can extract large amounts of data — always confirm scope and authorisation before running it
- SQLMap also supports testing POST parameters, HTTP headers, and JSON/XML request bodies — not just GET URLs
- Running with `--batch` suppresses interactive prompts, useful in scripted or automated assessments
- Always ensure explicit written authorisation before running SQLMap against any target

---

## Conclusion

SQL injection is a critical vulnerability class that every web security practitioner needs to understand. SQLMap makes the exploitation process systematic and efficient, but the real learning is in understanding why the injection works — how user input flows into SQL queries, why unsanitised input breaks query logic, and what an attacker can access once that logic is subverted. Parameterised queries and input validation are the direct mitigations, and understanding the attack makes those defences meaningful.
