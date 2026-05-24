# Client-Server Basics

## Overview

The client-server model is the fundamental architecture behind almost every networked application. Understanding how clients and servers communicate — through requests, responses, protocols, and ports — is essential for web security, network analysis, and understanding how attacks like man-in-the-middle or service exploitation work.

---

## Topics Covered

- The client-server model
- Requests and responses
- Protocols
- Ports
- DNS in the client-server context
- HTTP methods

---

## Key Concepts

### The Client-Server Model

In any networked interaction, there are two roles:

- **Client** — initiates the request. Always the one that starts the conversation.
- **Server** — listens for requests and responds with the requested resource or an error.

A browser visiting a website is a classic example: the browser is the client, the web server is the server. The client always initiates — the server never reaches out unprompted in standard communication.

---

### Requests and Responses

A client sends a request to a server asking for a specific resource. The server processes it and returns a response.

If the request is malformed or the resource doesn't exist, the server returns an error response. The client must format its request correctly according to the protocol the server expects.

---

### Protocols

A protocol is a defined set of rules that governs how a client and server communicate. It specifies:

- Which commands/methods the client and server understand
- How a request must be structured
- What syntax is used
- What response should be returned for each type of request
- How to handle invalid or unsupported requests

Without a shared protocol, two systems cannot communicate meaningfully. HTTP, FTP, SSH, and SMTP are all examples of protocols — each defines a specific communication contract.

---

### Ports

A port is a numerical identifier (0–65535) that tells the server which service a client wants to reach. A single server can run multiple services simultaneously, each listening on a different port.

Common port assignments:

| Protocol | Port |
|---|---|
| FTP | 21 |
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| SMB | 445 |
| RDP | 3389 |

When a client connects to a server, it must specify both the IP address (which machine) and the port (which service on that machine).

---

### DNS in Context

Before a client can connect to a server by name (e.g., `tryhackme.com`), it needs the server's IP address. DNS (Domain Name System) resolves the human-readable domain name to an IP address — similar to looking up a street address from a business name.

```
Client wants to reach tryhackme.com
        |
DNS query: "What is the IP for tryhackme.com?"
        |
DNS response: "104.26.10.229"
        |
Client connects to 104.26.10.229 on port 80 (HTTP) or 443 (HTTPS)
```

---

### HTTP Methods

HTTP defines 9 core methods (also called commands) that specify the intended action of a request:

| Method | Purpose |
|---|---|
| GET | Retrieve a resource from the server |
| POST | Submit data to the server (create or process) |
| PUT | Update an existing resource |
| DELETE | Remove a resource |
| PATCH | Apply a partial update to a resource |
| HEAD | Same as GET but returns headers only, no body |
| OPTIONS | Returns the HTTP methods supported by the server for a given URL |
| CONNECT | Establishes a tunnel (used for HTTPS through proxies) |
| TRACE | Diagnostic method — echoes the received request back to the client |

**GET** is the most common — used every time a browser loads a page.

**GET request example:**

```http
GET https://tryhackme.com/index.php HTTP/1.1
Host: tryhackme.com
```

Key fields in the request/response:
- **Scheme** — protocol used (`http` or `https`)
- **Host** — the server being requested
- **Filename** — the specific resource (e.g., `/index.php`; `/` maps to `index.html`)
- **Address** — the resolved IP of the server
- **Status** — outcome of the request (e.g., `200 OK`)

---

## Important Terminology

| Term | Definition |
|---|---|
| Client | The device or application that initiates a request |
| Server | The system that receives requests and returns responses |
| Protocol | A defined set of rules for communication between systems |
| Port | A numerical identifier for a specific service on a server |
| DNS | Domain Name System — resolves domain names to IP addresses |
| HTTP Method | A command that defines the intended action of an HTTP request |
| GET | HTTP method to retrieve a resource |
| POST | HTTP method to submit data to a server |
| Status Code | A 3-digit code in the HTTP response indicating the outcome |

---

## Workflow / Process

### Full Client-Server Request Flow

```
Client (browser) wants to load tryhackme.com
        |
DNS resolves tryhackme.com → IP address
        |
Client opens TCP connection to IP:443 (HTTPS)
        |
Client sends HTTP GET request
        |
Server receives and processes the request
        |
Server returns HTTP response (200 OK + HTML)
        |
Client renders the page
```

---

## Real-World Relevance

- Every web application vulnerability — SQLi, XSS, IDOR, SSRF — is exploited through the client-server request/response cycle
- Understanding HTTP methods is essential for API security testing — PUT and DELETE on unprotected endpoints can modify or destroy data
- Port scanning (e.g., with `nmap`) maps which services are running on a server by probing ports
- Protocol knowledge is required for manual exploitation — crafting raw HTTP requests, fuzzing parameters, and intercepting traffic with tools like Burp Suite
- DNS is a recon target — enumerating DNS records reveals infrastructure, subdomains, and third-party services

---

## Key Learnings

- The client always initiates; the server always responds
- Protocols define the rules of communication — without them, systems can't talk
- Ports identify which service on a server a client wants to reach
- DNS translates domain names to IP addresses before a connection can be made
- HTTP has 9 methods — GET and POST are the most common; all are relevant to security testing

---

## Additional Notes

- The `OPTIONS` method is useful during recon — it reveals which HTTP methods a server supports on a given endpoint
- `TRACE` is often disabled on production servers as it can be used in cross-site tracing (XST) attacks
- Ports below 1024 are privileged ports — on Unix systems, only root can bind to them

---

## Conclusion

The client-server model underpins virtually all networked communication. Understanding how requests are formed, how protocols govern the exchange, and how ports direct traffic to the right service is foundational knowledge for anyone working in web or network security.
