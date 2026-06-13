# LDAP Injection

## Overview

LDAP (Lightweight Directory Access Protocol) is widely used for centralised user authentication and directory management — including Microsoft Active Directory and OpenLDAP. When web applications construct LDAP queries from user input without proper sanitisation, attackers can manipulate the query to bypass authentication, access unauthorised data, or enumerate directory entries. This room covers LDAP structure, query syntax, and injection techniques including tautology-based injection, wildcard injection, and blind LDAP injection.

---

## Topics Covered

- What LDAP is and where it is used
- LDAP directory structure: DNs, OUs, attributes
- LDAP search query syntax and filters
- LDAP injection fundamentals
- Authentication bypass: tautology-based and wildcard injection
- Blind LDAP injection
- Mitigation strategies

---

## Key Concepts

### What is LDAP?

LDAP is a network protocol for accessing and managing distributed directory services over TCP/IP. It enables centralised management of users, groups, and other directory objects.

Common services using LDAP:
- **Microsoft Active Directory** — Windows domain user and resource management
- **OpenLDAP** — open-source cross-platform directory service

LDAP typically listens on:
- Port **389** — unencrypted or StartTLS
- Port **636** — SSL/TLS (LDAPS)

---

### LDAP Directory Structure

LDAP directories are hierarchical, similar to a file system tree.

| Concept | Description | Example |
|---------|-------------|---------|
| **TLD (Top-Level Domain)** | Root of the LDAP tree | `dc=ldap,dc=thm` |
| **Organisational Unit (OU)** | Category grouping within the tree | `ou=people`, `ou=groups` |
| **Distinguished Name (DN)** | Unique path identifying an entry | `cn=John Doe,ou=people,dc=example,dc=com` |
| **Relative DN (RDN)** | Single level within the hierarchy | `cn=John Doe` |
| **Attribute** | Property of a directory entry | `mail=john@example.com`, `uid=jdoe` |

---

### LDAP Query Syntax

An LDAP search query has four components:
1. **Base DN** — starting point in the directory tree
2. **Scope** — `base` (base DN only), `one` (immediate children), `sub` (base DN + all descendants)
3. **Filter** — conditions entries must match
4. **Attributes** — which attributes to return

#### Filter Syntax

LDAP filters use parenthesised expressions:

| Operator | Example | Meaning |
|----------|---------|---------|
| `=` | `(cn=John)` | Equality |
| `=*` | `(cn=*)` | Presence (any value) |
| `*` wildcard | `(cn=J*)` | Starts with J |
| `&` | `(&(obj=user)(cn=J*))` | AND |
| `\|` | `(\|(cn=John)(cn=Jane))` | OR |
| `!` | `(!(cn=admin))` | NOT |

**Command-line query example:**
```bash
ldapsearch -x -H ldap://TARGET:389 -b "dc=ldap,dc=thm" "(ou=People)"
```

---

### LDAP Injection Fundamentals

LDAP injection occurs when user input is directly concatenated into an LDAP query filter without sanitisation, allowing an attacker to alter the filter's logic.

**Vulnerable PHP authentication code:**
```php
$filter = "(&(uid=" . $userInput . ")(userPassword=" . $passInput . "))";
```

If `$userInput` is not sanitised, the attacker can inject additional filter conditions.

**Common attack vectors:**
- Authentication bypass — log in without valid credentials
- Unauthorised data access — retrieve entries not intended for the user
- Data manipulation — modify directory attributes

---

### Authentication Bypass Techniques

#### Tautology-Based Injection

A tautology is a condition that is always true. Injecting one into the LDAP filter ensures the query always matches.

**Vulnerable filter:**
```
(&(uid={userInput})(userPassword={passInput}))
```

**Injected values:**
- `userInput`: `*)(|(& `
- `passInput`: `pwd)`

**Resulting filter:**
```
(&(uid=*)(|(&)(userPassword=pwd)))
```

**Why this works:**
- `(uid=*)` — matches any user (wildcard)
- `(|(&)(userPassword=pwd))` — OR of two conditions:
  - `(&)` — an empty AND is always true in LDAP
  - `(userPassword=pwd)` — irrelevant, the first condition already makes the OR true

The empty `(&)` condition evaluates to true, making the entire OR true, which bypasses the password check regardless of what `pwd` is.

#### Wildcard Injection

Using `*` for both username and password:

**Injected values:**
- `uid = *`
- `userPassword = *`

**Resulting filter:**
```
(&(uid=*)(userPassword=*))
```

This matches any entry that has both a `uid` and `userPassword` attribute — effectively any user — and validates regardless of the actual password value.

---

### Blind LDAP Injection

Blind LDAP injection is used when the application does not directly return LDAP query results — the attacker must infer information from indirect signals:
- Different responses for true vs false conditions
- Error messages revealing query structure
- Response timing differences

**Use case:** Enumerate valid usernames or extract attribute values character by character by testing conditions and observing whether the application behaves differently (e.g. login succeeds vs fails, different page content).

---

## Practical Examples / Demonstrations

### Authentication Bypass Lab

Target: `http://TARGET/normal.php`

**Inject into username field:** `*`
**Inject into password field:** `*`

The LDAP filter becomes `(&(uid=*)(userPassword=*))` — matches any authenticated user and returns the first entry, bypassing authentication entirely.

### ldapsearch Enumeration (External Access)

When LDAP is exposed on port 389:
```bash
ldapsearch -x -H ldap://TARGET:389 -b "dc=ldap,dc=thm" "(ou=People)"
```

This retrieves all entries under the `People` organisational unit without authentication (if anonymous bind is allowed).

---

## Important Terminology

| Term | Meaning |
|------|---------|
| LDAP | Lightweight Directory Access Protocol — protocol for directory service access |
| DN | Distinguished Name — unique path identifying a directory entry |
| RDN | Relative Distinguished Name — a single level in the DN hierarchy |
| OU | Organisational Unit — a grouping within the LDAP directory tree |
| Filter | Criteria used to match entries in an LDAP search |
| Tautology | A logical condition that is always true — used to force query success |
| Wildcard | `*` in LDAP filters matches any value for that attribute |
| Blind injection | Injection where results are not directly visible — inferred from app behaviour |
| Anonymous bind | Connecting to an LDAP server without credentials — often misconfigured to allow access |

---

## Mitigation

- **Escape special characters:** Sanitise LDAP filter metacharacters before embedding user input: `(`, `)`, `*`, `\`, `/`, NUL
- **Input validation:** Accept only expected characters (e.g. alphanumeric for usernames)
- **Use LDAP libraries safely:** Use library functions that accept parameters separately from filter structures — equivalent to parameterised queries for SQL
- **Disable anonymous bind:** Require authentication for all LDAP connections
- **Least privilege:** LDAP service accounts used by applications should have the minimum permissions needed — read-only where write is not required
- **Encrypt LDAP connections:** Use LDAPS (port 636) or StartTLS to prevent credential interception

---

## Real-World Relevance

- LDAP injection is most impactful in enterprise environments where Active Directory is the authentication backend — a successful bypass grants access to any application relying on that directory
- Tautology injection (`*)(|(&)...`) is the LDAP equivalent of SQL's `' OR '1'='1`
- Misconfigured anonymous bind on internal LDAP servers is a common finding in internal network assessments — allows unrestricted directory enumeration without credentials
- LDAP injection tools like `ldapdomaindump` are commonly used post-exploitation to map Active Directory structure after obtaining LDAP access

---

## Key Learnings

- LDAP queries use filter syntax with parenthesised conditions, logical operators (`&`, `|`, `!`), and wildcards (`*`)
- LDAP injection inserts extra filter conditions by manipulating how user input is embedded in the filter string
- Tautology injection uses the always-true `(&)` empty AND to bypass password checks
- Wildcard injection uses `*` to match any value, bypassing specific value checks
- Blind LDAP injection infers directory information from application behaviour without direct output
- Mitigation requires escaping LDAP metacharacters in user input and using library-level parameterisation

---

## Additional Notes

- LDAP special characters that must be escaped: `\`, `*`, `(`, `)`, NUL (`\00`)
- OWASP provides an LDAP injection prevention cheat sheet with language-specific escaping functions
- Active Directory's LDAP implementation may respond differently from OpenLDAP to certain injection payloads
- When testing externally, tools like `ldapsearch`, `nmap --script ldap-*`, and `enum4linux` are standard for LDAP enumeration

---

## Conclusion

LDAP injection is the directory-service equivalent of SQL injection — user input incorporated into a query language without sanitisation allows an attacker to alter query logic, bypass authentication, and access unauthorised data. The tautology and wildcard techniques work by exploiting LDAP's filter semantics to create conditions that always evaluate to true. Proper input sanitisation, escaping of LDAP metacharacters, and disabling anonymous bind are the key defensive measures.
