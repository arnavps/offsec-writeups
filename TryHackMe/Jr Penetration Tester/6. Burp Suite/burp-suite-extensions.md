# Burp Suite: Extensions

## Overview

Burp Suite's functionality does not end with its built-in tools. The Extensions system allows users to load additional modules — either from the official BApp Store, community GitHub repositories, or custom-written code — that add new capabilities to Burp's existing toolchain. Extensions can add new scanner checks, automate repetitive tasks, integrate with external tools, decode custom encodings, and dramatically speed up specific assessment types.

This room covers the Extensions tab, the BApp Store, and the Burp Suite API for writing custom extensions. Understanding extensions is what separates an intermediate Burp user from a proficient one — many real-world assessments require capabilities that the base product does not include out of the box.

**Skills introduced:** BApp Store navigation, installing and managing extensions, Jython/JRuby setup for non-Java extensions, awareness of high-value extensions, introductory Burp Extension API concepts.

---

## Topics Covered

- The Extensions tab and its panels
- The BApp Store — browsing and installing community extensions
- Extension language support: Java, Python (Jython), Ruby (JRuby)
- High-value extensions and what they add
- Introduction to the Burp Suite Extender API
- Writing a basic extension

---

## Key Concepts

### The Extensions Tab

The Extensions tab (formerly called Extender in older Burp versions) is the central management interface for all installed extensions.

**Four sub-tabs:**

| Sub-tab | Purpose |
|---------|---------|
| **Extensions** | List of all installed extensions — enable/disable, remove, view output and errors |
| **BApp Store** | Official marketplace for community-built extensions |
| **APIs** | Documentation for the Burp Extender API — reference for extension development |
| **Python Environment / Ruby Environment** | Configure Jython (Python) or JRuby (Ruby) interpreters for non-Java extensions |

#### Extensions Panel

Each installed extension shows:
- Name and version
- Loaded status (checkbox to enable/disable without uninstalling)
- Output tab — any `stdout` output the extension prints
- Errors tab — any exceptions or error messages from the extension

**Practical note:** When an extension behaves unexpectedly, the first troubleshooting step is checking the Errors tab — most issues produce traceable exception output here.

---

### The BApp Store

**Definition:** The BApp Store is the official repository of community-contributed and PortSwigger-developed extensions for Burp Suite. It is accessible directly from within Burp under Extensions → BApp Store.

**How to install an extension:**
1. Open Burp Suite → Extensions → BApp Store
2. Browse or search by name or category
3. Click an extension to see its description, author, and user ratings
4. Click **Install** — the extension downloads and loads automatically
5. Some extensions require additional configuration after installation (e.g., API keys, Jython setup)

**Extension ratings:** BApp Store entries show a popularity rating — higher-rated extensions are generally more stable and maintained.

**Pro vs Community:** Some BApp extensions are marked as requiring Burp Suite Pro. Community edition users can still install them but certain features may be disabled or locked.

---

### Language Support — Jython and JRuby

Burp Suite is built in Java. Extensions can be written in:

| Language | Runtime Required | Notes |
|----------|-----------------|-------|
| **Java** | None (runs natively) | Best performance; most extensions are Java |
| **Python** | Jython (Python on JVM) | Popular for scripting; requires Jython JAR |
| **Ruby** | JRuby (Ruby on JVM) | Less common but supported |

#### Setting Up Jython (Python Extensions)

1. Download the Jython standalone JAR from `jython.org`
2. In Burp: Extensions → Python Environment → set the **Jython standalone JAR file** path
3. Python extensions can now be loaded

**Why this matters:** Many community extensions and custom scripts are written in Python. Without Jython configured, Python extensions will fail to load with a "Python environment not set up" error.

---

### High-Value Extensions

These are the extensions most commonly used in professional web application penetration testing:

#### Active Testing / Scanning

| Extension | Function |
|-----------|---------|
| **Active Scan++** | Adds additional active scan checks beyond Burp's built-in scanner — includes SSRF, XXE, header injection, and more |
| **Backslash Powered Scanner** | Uses backslash and other unusual characters to find server-side template injection and related vulnerabilities |
| **J2EEScan** | Additional scan checks targeting Java EE applications — useful for enterprise Java stacks |
| **Freddy, Deserialization Bug Finder** | Scans for Java and PHP deserialisation vulnerabilities |

#### Recon and Information Gathering

| Extension | Function |
|-----------|---------|
| **GAP (Get All Parameters)** | Harvests all parameter names seen across all in-scope requests — creates a comprehensive parameter wordlist |
| **JS Link Finder** | Extracts links and endpoints from JavaScript files — finds hidden API endpoints |
| **Content Type Converter** | Converts request body between JSON, XML, and form-encoded formats — useful for testing alternative content types |

#### Specific Vulnerability Classes

| Extension | Function |
|-----------|---------|
| **CSRF Scanner** | Passive and active checks for missing or bypassable CSRF tokens |
| **Param Miner** | Discovers hidden parameters and headers via smart wordlist-based fuzzing — one of the most used extensions in bug bounty |
| **HTTP Request Smuggler** | Tests for HTTP/1.1 and HTTP/2 request smuggling vulnerabilities |
| **HUNT** | Tags requests and highlights parameters associated with specific vulnerability classes (SQLi, SSRF, etc.) |
| **SQLiPy** | SQLMap integration — pass requests directly from Burp to SQLMap |

#### Utility and Workflow

| Extension | Function |
|-----------|---------|
| **Logger++** | Advanced logging with filtering, search, and colour coding — more powerful than the built-in HTTP history |
| **Turbo Intruder** | High-speed Intruder replacement using a Python script — designed for race conditions and high-volume fuzzing |
| **Autorize** | Tests for broken access control — replays requests with a lower-privileged account's cookie automatically |
| **AuthMatrix** | Builds a matrix of user roles vs endpoints to map access control systematically |
| **Bypass WAF** | Adds encoding and obfuscation headers to bypass WAF detection |

#### Burp Collaborator Clients

| Extension | Function |
|-----------|---------|
| **Burp Collaborator Everywhere** | Injects Collaborator payloads into all parameters automatically to detect out-of-band interactions (blind SSRF, XXE, etc.) |

---

## The Burp Suite Extender API

### What the API Provides

The Burp Extender API is a Java interface that gives extensions programmatic access to almost every part of Burp's functionality:

| API Interface | What It Exposes |
|--------------|----------------|
| `IBurpExtenderCallbacks` | Core callback interface — the entry point for all extensions |
| `IHttpListener` | Hook into every HTTP request and response passing through Burp |
| `IProxyListener` | Specifically hook into Proxy traffic |
| `IScannerCheck` | Add custom passive or active scanner checks |
| `IContextMenuFactory` | Add items to the right-click context menu |
| `IMessageEditorTab` | Add custom tabs to the HTTP message editor |
| `IExtensionStateListener` | Handle extension load and unload events |
| `IHttpRequestResponse` | Represent and manipulate individual HTTP request/response pairs |

### Extension Entry Point

Every Burp extension must implement `IBurpExtender` and define a `registerExtenderCallbacks` method:

```java
import burp.IBurpExtender;
import burp.IBurpExtenderCallbacks;

public class BurpExtension implements IBurpExtender {
    @Override
    public void registerExtenderCallbacks(IBurpExtenderCallbacks callbacks) {
        // Called when the extension is loaded
        callbacks.setExtensionName("My Extension");
        // Register listeners, scanners, menu items here
    }
}
```

In Python (Jython):
```python
from burp import IBurpExtender

class BurpExtender(IBurpExtender):
    def registerExtenderCallbacks(self, callbacks):
        callbacks.setExtensionName("My Python Extension")
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        print("Extension loaded successfully")
```

### Implementing an HTTP Listener

To intercept and process every request and response:

```python
from burp import IBurpExtender, IHttpListener

class BurpExtender(IBurpExtender, IHttpListener):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        callbacks.setExtensionName("HTTP Logger")
        callbacks.registerHttpListener(self)

    def processHttpMessage(self, toolFlag, messageIsRequest, messageInfo):
        # Called for every request and response
        if messageIsRequest:
            request = self._helpers.analyzeRequest(messageInfo)
            print("Request to: " + str(request.getUrl()))
        else:
            response = self._helpers.analyzeResponse(
                messageInfo.getResponse()
            )
            print("Response status: " + str(response.getStatusCode()))
```

### Implementing a Passive Scanner Check

```python
from burp import IBurpExtender, IScannerCheck, IScanIssue
import re

class BurpExtender(IBurpExtender, IScannerCheck):
    def registerExtenderCallbacks(self, callbacks):
        self._callbacks = callbacks
        self._helpers = callbacks.getHelpers()
        callbacks.setExtensionName("Custom Passive Check")
        callbacks.registerScannerCheck(self)

    def doPassiveScan(self, baseRequestResponse):
        # Analyse the response for patterns
        response = baseRequestResponse.getResponse()
        response_str = self._helpers.bytesToString(response)
        issues = []

        # Example: flag if response contains "password" in plaintext
        if re.search(r'password\s*=\s*["\'][^"\']+["\']', response_str, re.I):
            # Create and return a scan issue
            # (IScanIssue implementation omitted for brevity)
            pass
        return issues

    def doActiveScan(self, baseRequestResponse, insertionPoint):
        return []  # No active checks in this example

    def consolidateDuplicateIssues(self, existingIssue, newIssue):
        return -1  # -1 = keep both
```

---

## Workflow / Process

### Installing and Using an Extension (BApp Store)

```
Open Burp Suite
     |
     v
Extensions → BApp Store
     |
     v
Search or browse for required extension
(e.g., "Param Miner", "Autorize", "Logger++")
     |
     v
Click extension → review description and requirements
     |
     v
Click Install → extension loads automatically
     |
     v
Check Extensions → Extensions list:
  - Verify extension shows as Loaded
  - Check Output tab for any initialisation messages
  - Check Errors tab if the extension doesn't behave as expected
     |
     v
Configure extension if required
  (Some extensions add a new tab in Burp's main interface)
     |
     v
Use during assessment — extension hooks into Proxy, Scanner, etc. automatically
```

### Loading a Custom Extension

```
Write extension code (Java .jar, Python .py, or Ruby .rb)
     |
     v
If Python: configure Jython JAR first
  Extensions → Python Environment → set Jython standalone JAR path
     |
     v
Extensions → Extensions → Add
     |
     v
Select extension type (Java / Python / Ruby)
Select file path
Click Next
     |
     v
Check Output / Errors tabs for load status
     |
     v
Extension is now active
```

---

## Real-World Usage

- **Param Miner** is used on virtually every web application assessment — hidden parameters are a consistent finding in bug bounty and pentests
- **Autorize** dramatically speeds up broken access control testing — manually testing every endpoint with every privilege level is time-prohibitive without it
- **Turbo Intruder** is the go-to for race condition testing — standard Intruder is too slow for the millisecond windows most race conditions require
- **Logger++** is typically installed immediately on any serious engagement — Burp's built-in history lacks the filtering and search depth needed for large-scale assessments
- **HTTP Request Smuggler** is used when assessing applications behind reverse proxies or load balancers — request smuggling remains a high-severity finding
- Custom extensions are written when a target uses proprietary encoding, custom authentication tokens, or non-standard data formats that built-in tools cannot handle natively

---

## Security Engineer Perspective

- Extensions execute inside Burp's JVM with full access to all traffic — only install extensions from trusted sources (BApp Store, well-known researchers)
- Custom extensions for internal use can implement organisation-specific detection logic — flagging custom header values, internal IP addresses in responses, or proprietary error strings
- Extensions can automate evidence collection — automatically screenshot or log findings matching specific patterns during a scan
- Understanding the Extender API enables building tooling that integrates Burp into CI/CD pipelines for automated security testing

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **BApp Store** | Official Burp Suite extension marketplace |
| **Extension** | A plugin that adds functionality to Burp Suite |
| **IBurpExtender** | Core Java interface all extensions must implement |
| **IBurpExtenderCallbacks** | Callback interface providing access to Burp's internals |
| **IHttpListener** | API interface to hook into all HTTP traffic through Burp |
| **IScannerCheck** | API interface to add custom passive or active scanner checks |
| **Jython** | Python implementation running on the JVM — required for Python-based Burp extensions |
| **JRuby** | Ruby implementation running on the JVM — required for Ruby-based Burp extensions |
| **Param Miner** | Extension for discovering hidden parameters and headers |
| **Autorize** | Extension for automated broken access control testing |
| **Turbo Intruder** | High-speed Python-scriptable replacement for Intruder — used for race conditions |
| **Logger++** | Advanced logging extension with filtering and search |
| **Active Scan++** | Extension adding additional active scanner checks |

---

## Key Learnings

- Extensions are loaded and managed under Extensions → Extensions; the BApp Store is under its own sub-tab
- Python extensions require Jython standalone JAR configured under Python Environment before they will load
- Check the Errors tab first when an extension fails to behave as expected
- Param Miner, Autorize, Logger++, and Turbo Intruder are essential installs for any serious web application assessment
- The Burp Extender API exposes hooks for HTTP traffic, scanner checks, context menus, and message editor tabs
- All extensions implement `IBurpExtender` and define `registerExtenderCallbacks` as the entry point
- `IHttpListener.processHttpMessage()` is called for every request and response passing through Burp

---

## Conclusion

Extensions transform Burp Suite from a capable proxy into a customisable security testing platform. The BApp Store provides immediate access to community-built tools covering everything from automated access control testing to request smuggling detection. For assessments with unique requirements, the Extender API provides direct access to Burp's internals — allowing custom detection logic, automated evidence collection, and integration with external tooling. Knowing what extensions exist and when to reach for them is a core competency in professional web application security testing.
