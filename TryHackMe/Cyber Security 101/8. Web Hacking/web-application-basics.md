# Web Application Basics

## Overview

Web applications are the most common attack surface in modern security. Before exploiting them, you need to understand how they're built and how they communicate. This room covers the fundamental architecture of web applications — front end and back end — and dives deep into how HTTP works, including requests, responses, headers, and security-specific headers. This knowledge is a prerequisite for almost every web-based attack and defence technique.

---

## Topics Covered

- Front-end technologies: HTML, CSS, JavaScript
- Back-end components: databases, infrastructure, WAF
- URL structure and anatomy
- HTTP request and response structure
- HTTP methods and their security implications
- HTTP status codes
- Request headers and response headers
- HTTP security headers: CSP, HSTS, X-Content-Type-Options, Referrer-Policy

---

## Key Concepts

### Front End vs Back End

Web applications are split into two layers:

**Front End** — everything the user sees and interacts with in the browser:
- **HTML (HyperText Markup Language)** — defines the structure and content of a web page. It instructs the browser what to display and how to lay it out.
- **CSS (Cascading Style Sheets)** — controls the visual appearance: colours, fonts, spacing, layout.
- **JavaScript (JS)** — adds dynamic behaviour. While HTML is static instructions, JS enables logic, decisions, and real-time changes in the browser.

**Back End** — the server-side components that power the application:
- **Database** — stores and retrieves application data (user accounts, posts, preferences, etc.)
- **Infrastructure** — web servers, application servers, networking equipment, and supporting software
- **WAF (Web Application Firewall)** — an optional security layer that filters malicious requests before they reach the web server. Provides protection against common attacks like SQLi, XSS, and path traversal.

---

### URL Anatomy

A URL (Uniform Resource Locator) is a structured address pointing to a resource on the web. Each component has a specific role.

```
https://user@tryhackme.com:443/api/users?id=1#section
  |      |        |          |      |       |    |
scheme  user    host        port  path   query  fragment
```

| Component | Example | Description |
|-----------|---------|-------------|
| Scheme | `https` | Protocol used. HTTP is unencrypted; HTTPS is TLS-encrypted |
| User | `user@` | Optional credentials in the URL — insecure and uncommon |
| Host/Domain | `tryhackme.com` | The target server. Must be unique; registered through domain registrars |
| Port | `443` | Directs to the correct service on the server. HTTP default: 80, HTTPS default: 443 |
| Path | `/api/users` | Points to the specific resource or page on the server |
| Query String | `?id=1` | Starts with `?`. Passes parameters to the server. User-controllable |
| Fragment | `#section` | Client-side only. Points to a section within a page |

**Security notes:**
- **Typosquatting** — attackers register lookalike domains (e.g. `tryhackrne.com`) used in phishing attacks
- Query strings and fragments are user-controllable — always validate and sanitise server-side to prevent injection attacks
- Credentials in URLs (`user:pass@host`) are a security risk and should never be used in practice

---

### HTTP Messages

HTTP (HyperText Transfer Protocol) is the communication protocol between a client (browser) and a web server. All interaction happens through **HTTP messages**, which come in two types:

- **HTTP Request** — sent by the client to trigger an action on the server
- **HTTP Response** — sent by the server in reply

Every HTTP message shares the same general structure:

```
Start Line       ← Request line or status line
Headers          ← Key-value metadata pairs
                 ← Empty line (separates headers from body)
Body             ← Optional data payload
```

---

### HTTP Requests

The **request line** has three parts:

```
METHOD /path HTTP/version
```

Example:
```
GET /login HTTP/1.1
```

#### HTTP Methods

| Method | Purpose | Security Note |
|--------|---------|---------------|
| `GET` | Retrieve data — no body, no state change | Never put sensitive data (tokens, passwords) in GET params — they appear in URLs and logs |
| `POST` | Send data to create/update resources | Validate and sanitise all input to prevent SQLi and XSS |
| `PUT` | Replace a resource entirely | Verify authorisation before accepting |
| `DELETE` | Remove a resource | Restrict to authorised users only |
| `PATCH` | Partially update a resource | Validate data to avoid inconsistencies |
| `HEAD` | Like GET but returns headers only, no body | Used for metadata checks |
| `OPTIONS` | Lists allowed methods on a resource | Disable if not required — reduces attack surface |
| `TRACE` | Echoes the request back for debugging | Disable in production — can be abused in cross-site tracing attacks |
| `CONNECT` | Establishes a tunnel (used for HTTPS proxying) | Required for encrypted connections through proxies |

#### Common Request Headers

| Header | Example | Description |
|--------|---------|-------------|
| `Host` | `Host: tryhackme.com` | Specifies the target web server |
| `User-Agent` | `User-Agent: Mozilla/5.0` | Identifies the browser/client making the request |
| `Referer` | `Referer: https://google.com/` | The URL the user came from |
| `Cookie` | `Cookie: session=abc123` | Stores session data previously set by the server |
| `Content-Type` | `Content-Type: application/json` | Describes the format of the request body |

#### Request Body Formats

Used with `POST` and `PUT` requests to send data to the server.

**URL Encoded** (`application/x-www-form-urlencoded`):
```
POST /profile HTTP/1.1
Host: tryhackme.com
Content-Type: application/x-www-form-urlencoded

name=Aleksandra&age=27&country=US
```

**Multipart Form Data** (`multipart/form-data`) — used for file uploads:
```
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

aleksandra
----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="profile_pic"; filename="aleksandra.jpg"
Content-Type: image/jpeg

[Binary Data]
----WebKitFormBoundary7MA4YWxkTrZu0gW--
```

**JSON** (`application/json`):
```
POST /api/user HTTP/1.1
Content-Type: application/json

{"name": "Aleksandra", "age": 27, "country": "US"}
```

**XML** (`application/xml`):
```
POST /api/user HTTP/1.1
Content-Type: application/xml

<user>
  <name>Aleksandra</name>
  <age>27</age>
  <country>US</country>
</user>
```

---

### HTTP Responses

The **status line** format:

```
HTTP/version STATUS_CODE Reason Phrase
```

Example: `HTTP/1.1 200 OK`

#### Status Code Categories

| Range | Category | Meaning |
|-------|----------|---------|
| 100–199 | Informational | Request received, processing continues |
| 200–299 | Success | Request completed successfully |
| 300–399 | Redirection | Resource has moved; follow the new location |
| 400–499 | Client Error | Problem with the request (bad syntax, missing auth) |
| 500–599 | Server Error | The server failed to fulfil a valid request |

#### Common Status Codes

| Code | Meaning |
|------|---------|
| `100` | Continue |
| `200` | OK — request succeeded |
| `301` | Moved Permanently — update your bookmark |
| `404` | Not Found — resource doesn't exist at this path |
| `500` | Internal Server Error — something broke server-side |

#### Common Response Headers

| Header | Example | Description |
|--------|---------|-------------|
| `Date` | `Fri, 23 Aug 2024 10:43:21 GMT` | Timestamp of when the response was generated |
| `Content-Type` | `text/html; charset=utf-8` | Format and encoding of the response body |
| `Server` | `nginx` | Server software — often obscured to reduce info leakage |
| `Set-Cookie` | `sessionId=38af1337es7a8` | Sets a cookie on the client. Use `HttpOnly` and `Secure` flags |
| `Cache-Control` | `max-age=600` | Controls caching. Use `no-cache` for sensitive responses |
| `Location` | `/index.html` | Used in 3xx redirects. Validate to prevent open redirect attacks |

---

### HTTP Security Headers

Security headers instruct the browser to enforce additional protections. They're a low-effort, high-value defensive measure. Use [securityheaders.io](https://securityheaders.io) to analyse a site's header configuration.

#### Content-Security-Policy (CSP)

Mitigates **XSS** by defining which sources are allowed to load content (scripts, styles, images, etc.).

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com; style-src 'self'
```

- `default-src 'self'` — only load resources from the same origin by default
- `script-src 'self' https://cdn.tryhackme.com` — scripts allowed from own domain and a specific CDN
- `style-src 'self'` — stylesheets only from own domain

#### Strict-Transport-Security (HSTS)

Forces the browser to always connect over HTTPS, even if the user types `http://`.

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

- `max-age` — how long (in seconds) the browser should enforce HTTPS
- `includeSubDomains` — applies the rule to all subdomains
- `preload` — allows inclusion in browser preload lists so HSTS is enforced before even the first visit

#### X-Content-Type-Options

Prevents the browser from guessing (sniffing) the MIME type of a response, which can be abused in certain attacks.

```
X-Content-Type-Options: nosniff
```

- `nosniff` — browser must use the `Content-Type` header as-is and not infer the type

#### Referrer-Policy

Controls how much referrer information is sent when navigating from one page to another. Prevents leaking internal URLs or sensitive path information.

| Directive | Behaviour |
|-----------|-----------|
| `no-referrer` | No referrer information sent at all |
| `same-origin` | Referrer sent only for same-origin requests |
| `strict-origin` | Sends only the origin (not the full path) when protocol stays the same (HTTPS → HTTPS) |
| `strict-origin-when-cross-origin` | Full URL for same-origin; origin only for cross-origin; nothing on protocol downgrade |

---

## HTTP Versions

| Version | Year | Key Features |
|---------|------|-------------|
| HTTP/0.9 | 1991 | GET only |
| HTTP/1.0 | 1996 | Headers, content types, caching |
| HTTP/1.1 | 1997 | Persistent connections, chunked transfer, widely used today |
| HTTP/2 | 2015 | Multiplexing, header compression, performance improvements |
| HTTP/3 | 2022 | Built on QUIC protocol, faster and more secure than HTTP/2 |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| HTTP | HyperText Transfer Protocol — the web's communication protocol |
| HTTPS | HTTP over TLS/SSL — encrypted communication |
| URL | Uniform Resource Locator — structured web address |
| WAF | Web Application Firewall — filters malicious HTTP traffic |
| CSP | Content Security Policy — controls allowed content sources |
| HSTS | HTTP Strict Transport Security — enforces HTTPS |
| MIME type | Media type identifier (e.g. `text/html`, `application/json`) |
| Typosquatting | Registering lookalike domains to deceive users |
| Query string | URL parameters starting with `?` — user-controllable |
| Fragment | URL section starting with `#` — client-side only |

---

## Workflow / Process

```
User types URL in browser
        |
        v
Browser parses URL → resolves DNS for the host
        |
        v
TCP connection established (TLS handshake if HTTPS)
        |
        v
Browser sends HTTP Request (method, path, headers, body)
        |
        v
Server processes request → queries database if needed
        |
        v
Server sends HTTP Response (status line, headers, body)
        |
        v
Browser renders HTML/CSS/JS → displays the page
```

---

## Real-World Relevance

- **HTTP methods** misuse (e.g. accepting `DELETE` without auth) is a common API vulnerability
- **Query string injection** is the foundation of SQLi and XSS attacks
- **Missing security headers** (CSP, HSTS) are consistently flagged in web app penetration tests and bug bounties
- **Open redirect** via the `Location` header is an OWASP-recognised vulnerability used in phishing
- **Cookie flags** (`HttpOnly`, `Secure`) directly affect session hijacking risk
- Understanding request/response structure is a prerequisite for using tools like Burp Suite, OWASP ZAP, and curl for manual testing

---

## Key Learnings

- Web applications have a clear front end (HTML/CSS/JS) and back end (server, database, WAF) separation
- A URL has 7 distinct components — each carries security implications
- HTTP requests use methods to express intent; each method has specific security considerations
- HTTP responses include status codes grouped into five categories
- Security headers like CSP and HSTS are enforced browser-side and add meaningful protection
- Request bodies can be sent in multiple formats — understanding them is essential for crafting payloads

---

## Additional Notes

- The `Server` response header leaks software version info — often removed or replaced with a generic value in hardened deployments
- Fragments (`#`) are never sent to the server — they exist only in the browser
- Putting credentials in URLs (`user:pass@host`) is a legacy behaviour and a security risk — avoid it entirely
- HTTP/1.1 is still dominant despite HTTP/2 and HTTP/3 being available, due to compatibility with existing infrastructure

---

## Conclusion

Web application basics aren't just background knowledge — they're the lens through which every web vulnerability is understood and exploited. Knowing how HTTP works, what each header does, and where user input enters the request/response cycle is foundational for web application testing. Every technique covered later in web hacking — SQLi, XSS, IDOR, SSRF — traces back to these fundamentals.
