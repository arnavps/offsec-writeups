# NoSQL Injection

## Overview

NoSQL databases like MongoDB have become increasingly popular, but the fundamental principle of injection attacks applies to them as much as to SQL databases. When user input is incorporated into a NoSQL query without proper handling, an attacker can manipulate the query's logic. This room covers MongoDB's data model, NoSQL query structure, and two primary attack types: operator injection (authentication bypass, account enumeration, password extraction) and syntax injection.

---

## Topics Covered

- MongoDB: documents, collections, query structure
- NoSQL query operators
- Injection types: syntax injection vs operator injection
- Operator injection: authentication bypass with `$ne`
- Account enumeration with `$nin`
- Password extraction with `$regex`
- Syntax injection: identification and data extraction

---

## Key Concepts

### MongoDB Data Model

MongoDB stores data in **documents** — JSON-like key-value structures — rather than rows in tables.

```json
{
  "_id": ObjectId("5f077332de2cdf808d26cd74"),
  "username": "lphillips",
  "first_name": "Logan",
  "last_name": "Phillips",
  "age": "65",
  "email": "lphillips@example.com"
}
```

Multiple related documents are grouped into **collections** — the equivalent of SQL tables.

### MongoDB Query Filters

Instead of SQL `WHERE` clauses, MongoDB uses associative array filters:

```javascript
// Find all users named John with age 30
db.people.find({ "first_name": "John", "age": 30 })
```

MongoDB supports operators for complex conditions:

| Operator | Meaning |
|----------|---------|
| `$eq` | Equal to |
| `$ne` | Not equal to |
| `$gt` / `$lt` | Greater / less than |
| `$nin` | Not in a list |
| `$regex` | Matches a regular expression |

---

### Injection Types

#### Syntax Injection

Breaking out of the query structure to inject arbitrary query code — analogous to SQL injection. Less common because most libraries apply filters that prevent direct syntax manipulation.

#### Operator Injection

Injecting MongoDB query operators into the query to alter its behaviour — without breaking out of the query syntax. More common because user input that varies in format can bypass operator-level filtering.

---

### How to Inject NoSQL

PHP (and many other languages) allow passing array variables via HTTP request parameters using bracket notation:

```
user[$ne]=xxxx&pass[$ne]=yyyy
```

The server-side PHP code receives:
```php
$user = ['$ne' => 'xxxx']
$pass = ['$ne' => 'yyyy']
```

The resulting MongoDB filter:
```json
{ "username": { "$ne": "xxxx" }, "password": { "$ne": "yyyy" } }
```

---

## Practical Examples / Demonstrations

### Operator Injection: Authentication Bypass

**Vulnerable PHP login code:**
```php
$q = new MongoDB\Driver\Query(['username' => $user, 'password' => $pass]);
```

Where `$user` and `$pass` come directly from POST parameters.

**Attack:** Submit the following in the POST body:
```
user[$ne]=xxxx&pass[$ne]=yyyy
```

**Resulting filter:**
```json
{ "username": { "$ne": "xxxx" }, "password": { "$ne": "yyyy" } }
```

This returns any document where the username is not `xxxx` AND the password is not `yyyy` — which returns all users. The application logs in as the first user returned.

**Via Burp Suite:** Intercept the login request, modify the POST body from `user=xxxx&pass=yyyy` to `user[$ne]=xxxx&pass[$ne]=yyyy`, and forward.

---

### Operator Injection: Account Enumeration with `$nin`

The `$nin` operator excludes specific values. By iteratively adding known usernames to the exclusion list, an attacker cycles through all accounts.

**Round 1 — skip admin:**
```
user[$nin][]=admin&pass[$ne]=anything
```
Filter: `{ "username": { "$nin": ["admin"] }, "password": { "$ne": "anything" } }`
Logs in as the first non-admin user.

**Round 2 — skip admin and previous user:**
```
user[$nin][]=admin&user[$nin][]=jude&pass[$ne]=anything
```

Repeat until all user accounts have been accessed.

---

### Operator Injection: Password Extraction with `$regex`

The `$regex` operator enables pattern matching. Using it against the password field allows character-by-character extraction.

**Step 1 — determine password length:**
```
user=admin&pass[$regex]=^.{5}$
```
If the response is successful (user found), the password is exactly 5 characters. Iterate from 1 upward until a match is found.

**Step 2 — extract first character:**
```
user=admin&pass[$regex]=^a....
```
If successful, the password starts with `a`. Iterate through `a–z`, `A–Z`, `0–9`, and special characters until the first character is confirmed.

**Step 3 — extract remaining characters:**
Extend the known prefix one character at a time:
```
user=admin&pass[$regex]=^ab...
user=admin&pass[$regex]=^abc..
```

Continue until the full password is recovered. Maximum queries per character: character set size (typically 62–95). Fully automatable with a script.

---

### Syntax Injection: Identification and Data Extraction

Syntax injection is possible when the application constructs a MongoDB query using JavaScript string concatenation — specifically in `$where` queries.

**Vulnerable server-side code (Python):**
```python
for x in mycol.find({"$where": "this.username == '" + username + "'"}):
```

**Step 1 — Identify the vulnerability:**
Inject a single quote `'`. If the application returns a JavaScript error, the input is being concatenated into a `$where` expression.

**Step 2 — Confirm with boolean conditions:**
- `admin' && 0 && 'x` → no result (false condition)
- `admin' && 1 && 'x` → admin's data returned (true condition)

**Step 3 — Extract all records:**
```
admin'||1||'
```

The `||1||` is always true, so the filter matches all documents. Output: every user's data.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Document | A MongoDB record — a JSON-like key-value structure |
| Collection | A group of related documents — equivalent to a SQL table |
| Operator | MongoDB query modifier like `$ne`, `$nin`, `$regex` |
| Operator injection | Injecting MongoDB operators into query parameters to alter query logic |
| Syntax injection | Breaking out of the query structure to inject arbitrary query code |
| `$ne` | Not equal — returns documents where a field does not match the specified value |
| `$nin` | Not in — returns documents where a field is not in the specified list |
| `$regex` | Regular expression match — used for pattern-based field matching |
| `$where` | MongoDB operator allowing JavaScript expressions in queries — dangerous |

---

## Workflow / Process

```
Identify input fields used in MongoDB queries (login, search, etc.)
        |
        v
Test operator injection: submit user[$ne]=x&pass[$ne]=y
Does the application authenticate? → Operator injection confirmed
        |
        v
Authentication bypass: log in without valid credentials
        |
        v
Account enumeration: use $nin to iterate through users
        |
        v
Password extraction: use $regex to recover passwords character by character
        |
        v
Syntax injection test: inject single quote → observe JavaScript errors
Confirm with boolean conditions → extract all data with ||1||
```

---

## Real-World Relevance

- NoSQL injection is commonly overlooked during assessments focused on SQL databases — but the attack surface is equivalent when user input reaches query filters
- MongoDB is widely used in Node.js, PHP, and Python applications — all three support passing array parameters in HTTP requests
- Operator injection is particularly effective against login forms because many developers assume that MongoDB's structured filter format prevents injection
- Password extraction via `$regex` demonstrates that credential harvesting is possible even without direct data extraction capabilities
- The `$where` clause is deprecated in modern MongoDB versions for exactly this reason — never use it with user input

---

## Key Learnings

- NoSQL injection exploits improper handling of user input in NoSQL queries, following the same root cause as SQL injection
- MongoDB supports array parameters via bracket notation (`field[$operator]=value`) — enabling operator injection from HTTP requests
- `$ne` bypasses authentication by returning any document where credentials do not match the provided values
- `$nin` enables account enumeration by systematically excluding known usernames
- `$regex` enables blind data extraction one character at a time without direct output
- Syntax injection via `$where` is possible when user input is concatenated into JavaScript expressions
- Use library-provided filter methods with parameterised inputs — never concatenate user input into query strings

---

## Additional Notes

- Mongoose (Node.js ODM for MongoDB) sanitises operators by default in recent versions — older applications or those using raw MongoDB drivers remain vulnerable
- The `mongoose-sanitize` or `express-mongo-sanitize` middleware for Node.js strips `$` prefixes from request inputs, blocking operator injection
- JSON-based POST bodies can also carry operator injection if the server deserialises user JSON directly into filter objects: `{"username": {"$ne": null}}` in a JSON body achieves the same result
- Always validate that input fields containing user credentials accept only strings, not objects or arrays

---

## Conclusion

NoSQL injection demonstrates that injection attacks are not limited to SQL — any system that incorporates user input into a query language without proper sanitisation is vulnerable. MongoDB's operator-based query model creates a unique attack surface where an attacker can inject comparison operators to manipulate query logic, bypass authentication, enumerate users, and extract credentials without ever breaking out of the expected data format. The defence is the same principle as parameterised SQL: ensure user input is treated strictly as data values, never as query structure.
