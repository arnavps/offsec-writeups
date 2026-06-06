# Burp Suite: The Basics

## Overview

Burp Suite is the industry-standard tool for web application security testing. It acts as an intercepting proxy that sits between your browser and a target web server, allowing you to capture, inspect, modify, and replay every HTTP/HTTPS request and response. Whether you're doing a formal penetration test, bug bounty hunting, or learning web security, Burp Suite is the primary tool in the workflow. This room introduces the core modules, navigation, and the proxy setup that underpins everything else.

---

## Topics Covered

- What Burp Suite is and why it matters
- Burp Suite Community vs Professional
- Core modules: Proxy, Repeater, Intruder, Decoder, Comparer, Sequencer
- Dashboard layout
- Navigation and keyboard shortcuts
- Burp Proxy: intercepting and logging traffic
- Target tab and scope management

---

## Key Concepts

### What is Burp Suite?

Burp Suite is a Java-based web application testing framework developed by PortSwigger. Its core function is capturing all HTTP/HTTPS traffic between a browser and a web server — allowing a tester to intercept, inspect, modify, and replay requests.

It supports testing of:
- Standard web applications
- Mobile application backends
- REST and GraphQL APIs
- WebSocket-based applications

---

### Burp Suite Editions

This room focuses on **Burp Suite Community** — the free tier. It provides a strong set of tools sufficient for learning and manual testing, with some limitations:

- Intruder is rate-limited (slower brute-forcing)
- No automated scanner (available in Professional)
- No saved projects between sessions

---

## Core Modules

### Proxy

The central component of Burp Suite. It intercepts all traffic between the browser and the target server. When a request is intercepted, it is held in the Proxy tab — you can forward it, drop it, edit it, or route it to another module.

- Even with interception off, Burp logs all traffic passing through
- Supports both HTTP and WebSocket traffic
- **Match and Replace** — use regex rules to automatically modify requests/responses (e.g. swap User-Agent, inject headers)

### Repeater

Captures a single request and allows you to modify and resend it as many times as needed. Essential for:
- Manual payload testing (SQLi, XSS, etc.)
- Iterating on a request to understand how the server responds to different inputs
- Testing API endpoints

### Intruder

Automates request spraying — sends a large number of modified requests to a single endpoint. Used for:
- Brute-force attacks (login forms, PINs)
- Fuzzing parameters to find unexpected behaviour
- Payload enumeration

> **Note:** In Community edition, Intruder is heavily rate-limited, making it slow for large attacks. Tools like `ffuf` or `hydra` are typically used instead for speed.

### Decoder

Transforms data between encodings. Useful for:
- Decoding Base64, URL encoding, HTML entities
- Encoding payloads before sending to a target
- Quickly converting captured data without leaving Burp

### Comparer

Compares two pieces of data at word or byte level. Useful for:
- Spotting differences between two responses (e.g. valid vs invalid login)
- Comparing large response bodies quickly

### Sequencer

Analyses the randomness of tokens — session cookies, CSRF tokens, password reset tokens. If tokens are predictable, they can be forged. Sequencer tests the statistical randomness of a sample of captured tokens.

### Extender / BApp Store

Burp supports extensions written in Java, Python (via Jython), or Ruby (via JRuby). The **BApp Store** provides community and PortSwigger-developed extensions. A notable Community-compatible extension:
- **Logger++** — extends Burp's built-in logging with more granular filtering and display

---

## Dashboard Layout

The Burp Dashboard is divided into four quadrants:

| Quadrant | Purpose |
|----------|---------|
| **Tasks** | Background tasks Burp runs during your session. Community default: "Live Passive Crawl" (logs visited pages) |
| **Event Log** | Shows Burp's own activity — proxy start, connections made through Burp, errors |
| **Issue Activity** | Professional only — displays vulnerabilities found by the automated scanner |
| **Advisory** | Professional only — detailed info on identified vulnerabilities with references and remediation suggestions |

---

## Navigation and Keyboard Shortcuts

Burp Suite's interface is organised with a top menu bar for module selection and a secondary bar for sub-tabs within each module.

- **Module Selection** — top bar; click to switch between Proxy, Repeater, Intruder, etc.
- **Sub-tabs** — appear below the module bar; module-specific settings and views
- **Detach tabs** — go to `Window > Detach` to open a tab in a separate window

| Shortcut | Tab |
|----------|-----|
| `Ctrl + Shift + D` | Dashboard |
| `Ctrl + Shift + T` | Target |
| `Ctrl + Shift + P` | Proxy |
| `Ctrl + Shift + I` | Intruder |
| `Ctrl + Shift + R` | Repeater |

---

## Workflow / Process

### Setting Up the Proxy

Burp Suite operates as a local HTTP proxy. To route browser traffic through it:

1. Launch Burp Suite — it starts a proxy listener on `127.0.0.1:8080` by default
2. Configure your browser to use `127.0.0.1:8080` as its HTTP proxy
3. For HTTPS traffic, install the **Burp CA certificate** in your browser (exported from `Proxy > Options > Import/Export CA Certificate`) to avoid TLS errors

**Recommended approach:** Use the **FoxyProxy** browser extension to quickly toggle between Burp proxy and direct connection.

### Intercepting Requests

When `Intercept is on` in the Proxy tab, every outbound request is paused and displayed for review. From here you can:
- **Forward** — send the request as-is
- **Drop** — discard the request
- **Edit** — modify any part of the request before forwarding
- **Send to Repeater/Intruder** — route to another module for further testing

Toggle interception off when you just want to browse and log traffic without being prompted for every request.

### Using Repeater

1. In the Proxy HTTP history, right-click a request → `Send to Repeater`
2. Switch to the Repeater tab
3. Modify the request (change parameters, headers, body)
4. Click `Send` to fire it at the target
5. The response appears on the right — compare, iterate, and test

### Setting Scope

To avoid noise from unrelated traffic:

1. Go to `Target > Site Map`
2. Right-click the target domain → `Add To Scope`
3. Burp will ask if you want to stop logging out-of-scope traffic — select **Yes**

All Burp tools (history, scanner, Intruder) will then focus on the in-scope target.

---

## Target Tab

The Target tab has three sub-tabs:

| Sub-tab | Purpose |
|---------|---------|
| **Site Map** | Tree-view of the target application built from browsed traffic. Captures all visited URLs, parameters, and API endpoints |
| **Issue Definitions** | Full list of vulnerability classes Burp's scanner checks for — accessible in Community too. Useful as a reference during manual testing |
| **Scope Settings** | Define included/excluded domains and IP ranges. Controls what Burp tracks and processes |

The Site Map is particularly useful for mapping API endpoints — any API call made while browsing will appear here automatically.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Intercepting Proxy | A proxy that sits between client and server to capture and modify traffic |
| HTTPS Interception | Requires installing Burp's CA certificate to decrypt TLS traffic |
| Scope | The defined set of targets Burp focuses on |
| Site Map | Auto-generated tree of all visited pages and endpoints |
| BApp Store | Burp Suite's extension marketplace |
| Repeater | Module for manually resending and modifying individual requests |
| Intruder | Module for automated request fuzzing and brute-forcing |
| Sequencer | Module for testing token randomness |
| Match and Replace | Proxy feature for automatic request/response modification using regex |
| CA Certificate | Certificate Authority cert; must be installed to intercept HTTPS without errors |

---

## Real-World Relevance

- Burp Suite is the standard tool used in **OSCP**, **web application penetration tests**, and **bug bounty hunting**
- The Proxy and Repeater are used together in virtually every manual web vulnerability test
- Intruder (or `ffuf`/`hydra` for speed) powers credential brute-forcing and parameter fuzzing
- Site Map is used during reconnaissance to enumerate all accessible endpoints in a target application
- Sequencer is used when testing session management — weak token randomness can lead to session hijacking
- Decoder is used when payloads need encoding (e.g. URL-encoding a SQLi payload to bypass WAF rules)

---

## Key Learnings

- Burp Suite is a full web application testing framework, not just a proxy
- The Proxy is the core — everything flows through it
- Repeater is the primary tool for manual, iterative vulnerability testing
- Scope configuration keeps your testing focused and logs clean
- The Site Map passively maps the application just from normal browsing
- Community edition is sufficient for manual testing and learning; Professional adds scanning and speed

---

## Additional Notes

- Burp's default proxy port is `8080` — change it in `Proxy > Options` if there's a port conflict
- The Burp browser (built-in Chromium) has the CA cert pre-installed and the proxy pre-configured — useful for quick testing without manual browser setup
- WebSocket messages are captured in `Proxy > WebSockets History`, separate from HTTP history
- Keyboard shortcuts significantly speed up the workflow — worth memorising `Ctrl+Shift+R` (Repeater) and `Ctrl+Shift+P` (Proxy) early

---

## Conclusion

Burp Suite is the central tool of web application security testing. This room establishes the fundamentals: understanding what each module does, how to route traffic through the proxy, how to intercept and modify requests, and how to scope your testing. Every technique that follows in web hacking — testing for SQLi, XSS, IDOR, authentication bypass — is carried out with Burp Suite as the primary interface. Getting comfortable with it early is one of the highest-leverage skills you can build.
