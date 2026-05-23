# HTTP in Detail

## Overview

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the web. Every time a browser loads a page, it uses HTTP (or its encrypted counterpart HTTPS) to request and receive content from a web server. Understanding how HTTP works — requests, responses, headers, status codes — is essential for web application security testing.

---

## Topics Covered

- HTTP vs HTTPS
- URL structure
- HTTP methods
- HTTP status codes
- Request and response headers

---

## Key Concepts

### HTTP and HTTPS

**HTTP** is the protocol used to transfer web content (HTML, images, video, etc.) between a client and a web server. Developed by Tim Berners-Lee between 1989–1991.

**HTTPS** is the encrypted version of HTTP. It uses TLS (Transport Layer Security) to:
- Encrypt data in transit, preventing interception
- Verify the identity of the web server, preventing impersonation

Any sensitive data (credentials, payment info, session tokens) should only ever be transmitted over HTTPS.

---

### URL Structure

A URL (Uniform Resource Locator) is the full address used to access a resource on the web. Each component has a specific role:

```
http://user:password@tryhackme.com:80/path/page?id=1#section
  |       |              |          |     |       |     |
Scheme  Credentials     Host      Port  Path   Query  Fragment
```

| Component | Description | Example |
|---|---|---|
| Scheme | Protocol to use | `http`, `https`, `ftp` |
| User | Optional credentials embedded in the URL | `user:password@` |
| Host | Domain name or IP of the server | `tryhackme.com` |
| Port | Port to connect on (default: 80/HTTP, 443/HTTPS) | `:8080` |
| Path | Location of the resource on the server | `/blog/post` |
| Query String | Additional parameters passed to the resource | `?id=1&page=2` |
| Fragment | Reference to a specific section within the page | `#comments` |

---

### HTTP Methods

HTTP methods define the intended action of a request:

| Method | Purpose |
|---|---|
| GET | Retrieve data from the server |
| POST | Submit data to the server (e.g., form submission, creating records) |
| PUT | Update an existing resource on the server |
| DELETE | Remove a resource from the server |

GET and POST are by far the most common in web applications.

---

### HTTP Status Codes

The first line of every HTTP response contains a status code indicating the outcome of the request.

**Status code ranges:**

| Range | Category | Meaning |
|---|---|---|
| 100–199 | Informational | Request received, continue sending |
| 200–299 | Success | Request completed successfully |
| 300–399 | Redirection | Client must take further action (follow redirect) |
| 400–499 | Client Error | Problem with the request |
| 500–599 | Server Error | Problem on the server side |

**Common status codes:**

| Code | Name | Description |
|---|---|---|
| 200 | OK | Request succeeded |
| 201 | Created | Resource successfully created |
| 301 | Moved Permanently | Resource has permanently moved; update bookmarks/links |
| 302 | Found | Temporary redirect |
| 400 | Bad Request | Malformed or missing request data |
| 401 | Unauthorised | Authentication required |
| 403 | Forbidden | Authenticated but not permitted to access this resource |
| 404 | Not Found | Resource does not exist |
| 405 | Method Not Allowed | HTTP method not supported for this endpoint |
| 500 | Internal Server Error | Unhandled server-side error |
| 503 | Service Unavailable | Server overloaded or under maintenance |

---

### HTTP Headers

Headers carry metadata about the request or response. They are key-value pairs sent alongside the HTTP message.

**Common Request Headers (Client → Server):**

| Header | Description |
|---|---|
| `Host` | Specifies which website to serve when a server hosts multiple domains |
| `User-Agent` | Identifies the client software (browser name and version) |
| `Content-Length` | Size of the request body in bytes; used when sending data |
| `Accept-Encoding` | Compression formats the client supports (e.g., `gzip`, `br`) |
| `Cookie` | Session data sent back to the server on each request |

**Common Response Headers (Server → Client):**

| Header | Description |
|---|---|
| `Set-Cookie` | Instructs the browser to store a cookie and send it on future requests |
| `Cache-Control` | Defines how long the browser should cache the response |
| `Content-Type` | Tells the client what type of content is being returned (e.g., `text/html`, `application/json`) |
| `Content-Encoding` | Compression method used on the response body |

---

## Important Terminology

| Term | Definition |
|---|---|
| HTTP | HyperText Transfer Protocol — the protocol for web communication |
| HTTPS | Encrypted version of HTTP using TLS |
| URL | Uniform Resource Locator — the full address of a web resource |
| Query String | Parameters appended to a URL after `?` |
| HTTP Method | Defines the action of an HTTP request (GET, POST, PUT, DELETE) |
| Status Code | A 3-digit code in the HTTP response indicating the outcome |
| Request Header | Metadata sent from client to server |
| Response Header | Metadata sent from server to client |
| Cookie | Small piece of data stored by the browser and sent with each request |

---

## Practical Examples / Demonstrations

### Basic HTTP GET Request

```http
GET /index.html HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0
Accept-Encoding: gzip, deflate
```

### HTTP POST Request (form submission)

```http
POST /login HTTP/1.1
Host: tryhackme.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30

username=admin&password=secret
```

### HTTP Response

```http
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=abc123; Path=/; HttpOnly
Content-Length: 1024

<html>...</html>
```

---

## Workflow / Process

### HTTP Request/Response Cycle

```
Client (Browser)
    |
    |-- [HTTP Request: GET /index.html] -->
    |                                       Web Server
    |                                           |
    |                                    Process request
    |                                           |
    | <-- [HTTP Response: 200 OK + HTML] -------
    |
Browser renders the page
```

---

## Real-World Relevance

- HTTP methods are directly relevant to REST API security testing — PUT and DELETE on unprotected endpoints can modify or destroy data
- Status codes reveal application behaviour during fuzzing and enumeration (403 vs 404 vs 200 tells you a lot)
- The `Host` header is the basis for virtual host enumeration and Host header injection attacks
- Cookies set without `HttpOnly` or `Secure` flags are vulnerable to theft via XSS or interception over HTTP
- Query strings are a primary injection point for SQLi, XSS, and parameter tampering
- `User-Agent` and other headers can be manipulated to bypass basic access controls or WAF rules

---

## Key Learnings

- HTTP is the protocol powering the web; HTTPS adds encryption and server verification via TLS
- URLs have distinct components — each can be a potential attack vector
- HTTP methods define intent: GET retrieves, POST submits, PUT updates, DELETE removes
- Status codes communicate the outcome of every request — knowing them speeds up testing significantly
- Headers carry critical metadata and are frequently targeted in web application attacks

---

## Additional Notes

- HTTP/1.1 is still widely used; HTTP/2 and HTTP/3 improve performance but the core request/response model remains the same
- Cookies should always have `HttpOnly` (prevents JS access) and `Secure` (HTTPS only) flags set
- The `OPTIONS` HTTP method can reveal which methods a server supports — useful during recon
- Tools like Burp Suite intercept and modify HTTP requests/responses in real time

---

## Conclusion

HTTP is the language of the web. Every web application security concept — from authentication to injection attacks — operates over HTTP. Understanding the structure of requests and responses, what each header does, and how status codes behave is a prerequisite for effective web application testing.
