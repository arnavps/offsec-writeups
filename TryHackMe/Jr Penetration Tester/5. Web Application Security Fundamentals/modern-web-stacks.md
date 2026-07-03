# Modern Web Stacks

## Overview

Understanding how modern web applications are built is essential for testing them effectively. This room covers the key technologies, frameworks, and architectural patterns that make up contemporary web stacks — from client-side rendering frameworks and single-page applications to server-side logic, API design, and cloud deployment. The goal is not to become a developer, but to understand what you're targeting when you walk into a web application penetration test.

Modern web applications look fundamentally different from the PHP/HTML sites of the early web. Knowing the architecture tells you where to probe, what tools to use, and what vulnerability classes are relevant.

---

## Topics Covered

- Client-side vs server-side rendering
- Single-Page Applications (SPAs) and JavaScript frameworks
- REST APIs and API-first architecture
- Common backend technologies (Node.js, Python, Java)
- Databases — SQL and NoSQL
- Authentication patterns (sessions, tokens, JWTs)
- Cloud infrastructure and containerisation (Docker, Kubernetes)
- CDNs, WAFs, and load balancers

---

## Key Concepts

### Client-Side vs Server-Side Rendering

**Server-Side Rendering (SSR):**
The server generates complete HTML pages and sends them to the browser. The browser renders what it receives without executing additional logic. Every page navigation triggers a full request-response cycle.

- Technologies: PHP, Ruby on Rails, Django (Python), ASP.NET, Express with template engines
- Testing perspective: traditional — view page source gives a complete picture, forms submit to visible endpoints, redirects are straightforward

**Client-Side Rendering (CSR):**
The server sends a minimal HTML shell and a JavaScript bundle. The browser executes the JavaScript, which fetches data from APIs and builds the UI dynamically. Page content is rendered in the browser, not the server.

- Technologies: React, Vue.js, Angular, Svelte
- Testing perspective: `view-source:` shows the shell only — not useful. APIs are the real interface. Network tab is essential. Content is dynamically injected into the DOM.

**Hybrid (Next.js, Nuxt.js):**
Some pages are server-rendered (for SEO and performance), others are client-rendered. Testing requires understanding which routes are which.

---

### Single-Page Applications (SPAs)

**Definition:** An SPA loads a single HTML document and dynamically updates the content as the user interacts with it — without full page reloads. Navigation happens via JavaScript (client-side routing), not HTTP redirects.

**Why It Matters for Testing:**
- Page source reveals nothing about rendered content
- All data exchange happens via API (usually JSON over HTTP/HTTPS)
- The entire API surface must be mapped by intercepting browser traffic — not by reading page source
- Standard Gobuster `dir` mode may miss content loaded dynamically — Ajax Spider in ZAP is required
- Authentication is typically token-based (JWT stored in localStorage or cookie), not session-based

**Identifying an SPA:**
- Single large `.js` bundle file loaded on page load
- URL changes without full page refresh (e.g., hash routing: `/#/dashboard`)
- No server-rendered content visible in View Page Source beyond a `<div id="root">` or `<div id="app">`
- All data appears in Network tab XHR/Fetch requests

---

### REST APIs

**Definition:** REST (Representational State Transfer) is the dominant API design pattern. RESTful APIs use HTTP methods to perform operations on resources, with resources identified by URL paths.

**HTTP Methods and Semantics:**

| Method | Typical Use | Idempotent |
|--------|------------|-----------|
| GET | Retrieve a resource | Yes |
| POST | Create a resource | No |
| PUT | Replace a resource entirely | Yes |
| PATCH | Partially update a resource | Yes |
| DELETE | Remove a resource | Yes |

**REST Security Testing Points:**
- **Method confusion:** Does a `GET` endpoint behave differently if called as `POST`? Does `OPTIONS` reveal allowed methods?
- **Parameter tampering:** Change resource IDs in URLs (`/api/users/123` → `/api/users/124`) for IDOR testing
- **Verb tampering:** Does the server enforce methods, or does it accept unexpected verbs?
- **Endpoint discovery:** APIs often follow naming conventions (`/api/v1/users`, `/api/v1/admin`) — wordlists based on REST conventions find more endpoints

**API Documentation Exposure:**
- Swagger/OpenAPI: `/swagger.json`, `/api-docs`, `/openapi.json`
- When found, these documents list all endpoints, parameters, and expected request/response formats — gold for a tester

---

### GraphQL

**Definition:** An alternative to REST where the client specifies exactly what data it needs in a query, and the server returns exactly that. All requests typically go to a single endpoint (e.g., `/graphql`).

**GraphQL Security Testing:**
- **Introspection:** By default, GraphQL allows clients to query the schema itself (`__schema`). Introspection reveals all types, queries, and mutations — full API documentation without needing external docs
- **Overposting/Mass assignment:** Mutations (write operations) may accept fields that shouldn't be user-settable
- **Batching attacks:** GraphQL allows multiple queries in a single request — useful for brute-force bypass if rate limiting only counts per HTTP request

```bash
# Introspection query to enumerate the schema
curl -X POST http://TARGET/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name } } }"}'
```

---

### Backend Technologies

**Node.js / Express:**
- JavaScript runtime on the server
- Async, event-driven — non-blocking I/O
- Commonly used for REST APIs and real-time applications (WebSockets)
- Security considerations: prototype pollution, `eval()` injection, insecure deserialisation of JSON/BSON

**Python (Django / Flask / FastAPI):**
- Django: full-featured MVC framework with ORM, admin panel, built-in CSRF protection
- Flask: minimal framework — security depends heavily on the developer adding it
- FastAPI: modern async framework designed for APIs with automatic OpenAPI doc generation
- Security considerations: SSTI (Server-Side Template Injection) in Jinja2 templates, SQL injection if raw queries used

**Java (Spring Boot):**
- Common in enterprise environments
- Spring Security provides comprehensive auth/authz framework
- Security considerations: XXE (XML External Entity) injection, deserialisation vulnerabilities (Java object deserialisation), SSRF

**PHP:**
- Still widely deployed in older applications and CMSs (WordPress, Drupal)
- Security considerations: file inclusion (LFI/RFI), command injection via shell functions, SQL injection, object injection

---

### Databases — SQL vs NoSQL

**SQL Databases (MySQL, PostgreSQL, MSSQL, SQLite):**
- Structured data, predefined schema, relational tables
- Query language: SQL
- Security: SQL injection if input is concatenated into queries; use parameterised queries

**NoSQL Databases (MongoDB, Redis, Cassandra, DynamoDB):**
- Flexible schema, document or key-value stores
- Query format is typically JSON/BSON or API calls
- Security: NoSQL injection — operators like `$gt`, `$ne`, `$regex` can manipulate queries if user input is unsanitised
- MongoDB example: supplying `{"$gt": ""}` as a password value can bypass authentication if the backend doesn't sanitise the JSON body

---

### Authentication Patterns

**Session-Based Authentication:**
- Server creates a session record after login; session ID stored in a cookie
- Server maintains session state — stateful
- Security: session fixation, session hijacking, missing HttpOnly/Secure flags

**Token-Based Authentication (JWT):**
- After login, server issues a signed JSON Web Token
- Token stored client-side (localStorage or cookie)
- Each request sends the token in the `Authorization: Bearer <token>` header
- Server is stateless — no session storage required
- Security issues:
  - `alg: none` attack — removing the signature algorithm to bypass verification
  - Weak or hardcoded signing secret — allows token forgery
  - Sensitive data in payload — JWT body is base64-encoded, not encrypted; readable without the secret

**OAuth 2.0 / OIDC:**
- Delegated authorisation — allows "Login with Google" style flows
- Security: open redirect vulnerabilities, state parameter not validated (CSRF), token leakage in redirect URI

---

### Cloud Infrastructure and Containers

**Docker:**
- Packages applications and dependencies into containers
- Containers share the host OS kernel — less isolated than VMs
- Security concerns: running containers as root, exposed Docker daemon socket, privileged containers, unpatched base images

**Kubernetes:**
- Container orchestration platform managing clusters of Docker containers
- Security concerns: misconfigured RBAC, exposed dashboard, secrets in environment variables, unauthenticated API server

**Cloud Services (AWS, GCP, Azure):**
- Web applications often use cloud storage (S3), serverless functions (Lambda), managed databases, and IAM
- Security concerns: publicly accessible S3 buckets, over-privileged IAM roles, metadata endpoint SSRF (accessing `169.254.169.254` for cloud credentials)

---

### WAFs, CDNs, and Load Balancers

**Web Application Firewall (WAF):**
- Sits in front of the application; filters malicious requests based on rules or signatures
- Security testing implication: detect WAF presence early (error messages, response headers, Cloudflare/Akamai/AWS WAF headers); understand that it may block some payloads but not all
- Bypass approaches: encoding, case variation, unusual HTTP methods, fragmented payloads

**CDN (Content Delivery Network):**
- Caches and serves static content from edge servers geographically closer to users
- Security implication: the real origin IP may be hidden behind the CDN — finding the origin IP directly may bypass WAF protection

**Load Balancer:**
- Distributes requests across multiple backend servers
- Security implication: inconsistent responses between backends (if only some are patched), session state issues if sessions aren't shared

---

## Workflow / Process

```
Identify the web stack:
  View page source — is there a full HTML document or just a <div id="root">?
  Check Network tab — are all requests XHR/Fetch (SPA) or full page loads?
  Run Wappalyzer — identify frameworks, servers, CDNs
  Check HTTP headers — Server, X-Powered-By, CF-Ray (Cloudflare)
        |
        v
Map the API surface:
  Check /swagger.json, /api-docs, /openapi.json for API documentation
  Intercept all traffic in Burp Proxy while browsing the application
  Run Ajax Spider in ZAP for SPA content
  Try GraphQL introspection if a /graphql endpoint is discovered
        |
        v
Identify authentication mechanism:
  Check cookies — session cookie with PHPSESSID (PHP session) or JWT-format value?
  Check localStorage in DevTools Storage tab — JWT stored there?
  Check Authorization header in Network tab requests
        |
        v
Technology-specific testing:
  PHP: file inclusion, object injection
  Node.js: prototype pollution, eval injection
  Django/Flask: SSTI in Jinja2 templates
  Spring Boot: XXE, deserialisation
  MongoDB: NoSQL injection operators
  SQL databases: SQLi via parameterisation check
        |
        v
Infrastructure checks:
  WAF detection: send simple payload, observe response format
  CDN detection: check response headers for CDN identifiers
  S3 buckets: look for bucket URLs in JS files and requests
  Cloud metadata: test SSRF to 169.254.169.254 if SSRF is possible
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SPA | Single-Page Application — client-side rendered web app that updates the DOM dynamically |
| REST | Representational State Transfer — HTTP-based API design pattern using resource URLs and methods |
| GraphQL | Query language for APIs with a single endpoint and client-defined data selection |
| JWT | JSON Web Token — signed token used for stateless authentication |
| SSR | Server-Side Rendering — server generates complete HTML before sending to browser |
| CSR | Client-Side Rendering — browser renders UI by executing JavaScript |
| WAF | Web Application Firewall — layer that filters malicious HTTP requests |
| CDN | Content Delivery Network — distributed caching layer for web content |
| NoSQL Injection | Manipulating NoSQL queries via unsanitised JSON operators |
| SSTI | Server-Side Template Injection — injecting code into server-side template engines |
| SSRF | Server-Side Request Forgery — making the server send requests on the attacker's behalf |
| Introspection | GraphQL feature that returns the full API schema when queried |

---

## Real-World Relevance

- Modern penetration tests almost exclusively target SPAs with REST or GraphQL backends — understanding these architectures is fundamental
- JWT vulnerabilities (`alg: none`, weak secrets) appear regularly in bug bounty reports and penetration test findings
- GraphQL introspection being enabled in production is a common misconfiguration that exposes the full API schema to unauthenticated users
- Cloud misconfiguration (S3 public access, SSRF to metadata endpoint) is a top finding category in cloud-native application assessments
- WAF bypass techniques are required knowledge for any professional web application penetration tester

---

## Key Learnings

- View Page Source is insufficient for SPAs — all meaningful content is in the Network tab
- Every API endpoint is an attack surface — map the full API before testing individual endpoints
- JWT `alg: none` and weak secret attacks are specific to token-based authentication — understand the auth mechanism before testing it
- Technology identification (Wappalyzer, headers, page source) narrows the vulnerability hypothesis to technology-specific classes
- Cloud metadata endpoints and public storage buckets are first-class findings in cloud-native application assessments

---

## Conclusion

Modern web stacks have moved well beyond static HTML and traditional server-rendered applications. Single-page applications, REST and GraphQL APIs, JWT-based authentication, and cloud infrastructure each introduce their own attack surfaces and require specific testing approaches. Understanding how an application is built — the framework, the API design, the auth mechanism, and the infrastructure — is what allows a penetration tester to efficiently target the most relevant vulnerability classes rather than spraying generic payloads at a well-defended target.
