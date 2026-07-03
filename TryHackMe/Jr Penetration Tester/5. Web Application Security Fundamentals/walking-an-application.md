# Walking An Application

## Overview

Before running a single automated tool, a skilled penetration tester manually walks through a web application using only a browser. This room establishes the discipline of methodical manual exploration — mapping application features, reading page source, leveraging browser Developer Tools, and understanding how client-side storage works. The emphasis is on building observation habits that automated scanners cannot replace.

**Main objectives:**
- Systematically map a web application's features and endpoints
- Extract intelligence from page source: comments, hidden links, framework information, directory listings
- Use browser Developer Tools (Inspector, Debugger, Network, Storage) for security analysis
- Understand what each storage mechanism (cookies, Local Storage, Session Storage) reveals about application behaviour

**Skills introduced:** Manual application mapping, page source review, HTML comment analysis, browser DevTools for security, cookie inspection, client-side storage analysis

---

## Concepts Covered

### Application Mapping

**Definition:** The process of systematically recording every page, endpoint, feature, and input point of a web application before any security testing begins.

**Why It Matters:** Without a map, testing is incomplete and opportunistic. A documented feature inventory ensures every attack surface is eventually tested and provides the framework for report findings.

**Key Details:** A well-structured application map records the endpoint URL, the feature it provides, any input fields, and notes about interesting behaviour or potential vulnerability classes.

**Example application map structure:**

| Feature | Endpoint | Summary |
|---------|---------|---------|
| Home Page | `/` | Company landing page |
| Contact Form | `/contact` | Name, email, message fields + file upload |
| Customer Login | `/customers/login` | Username/password authentication |
| Customer Dashboard | `/customers` | Ticket list, Create Ticket button |
| Create Ticket | `/customers/ticket/new` | Text input + file upload |
| Password Reset | `/customers/reset` | Email field, reset mechanism |

---

### Page Source Analysis

**Definition:** The raw HTML/CSS/JavaScript returned by a web server, viewable via right-click → View Page Source or by prepending `view-source:` to the URL.

**Why It Matters:** The rendered page in the browser hides information that is visible in the source. Developer comments, hidden links, framework identification, and directory structure are all present in source but invisible to normal browsing.

**Key Details — what to look for:**

- **HTML comments (`<!-- ... -->`):** Developer notes, TODO items, references to development versions or staging URLs, temporary pages, and version information
- **Hidden links in anchor tags (`<a href="...">`):** Pages not linked from navigation — `/secr3t-area`, admin paths, internal tools
- **External file references:** CSS, JS, and image `src` and `href` paths reveal the directory structure
- **Directory listing:** If the files/ directory is navigable, directory listing may be enabled — check for `403 Forbidden` vs actual file listing
- **Framework identification:** Copyright notices, `generator` meta tags, script paths (e.g., `/wp-content/` for WordPress), and comments referencing specific frameworks and version numbers

**Framework version significance:** Once a framework and version are identified, checking the framework's changelog or CVE database can reveal whether the version is outdated and what known vulnerabilities apply.

---

### Browser Developer Tools — Inspector

**Definition:** The Inspector tab provides a live, editable DOM view of what is currently rendered in the browser — distinct from the static page source.

**Why It Matters:** JavaScript, CSS, and user interaction can modify the DOM after page load. The Inspector shows the current state, not the initial source. It also allows real-time DOM manipulation — useful for bypassing client-side controls.

**Key Details:**
- Right-click any element → Inspect to open Developer Tools focused on that element
- CSS properties can be directly modified in the Styles panel
- `display: block` on a paywall element → change to `display: none` to hide it and reveal underlying content
- Changes are local to the browser session and reset on page refresh
- Reveals class names, IDs, and structure that can be cross-referenced with page source

**Security implications:** Client-side controls (paywalls, disabled buttons, hidden form fields) are entirely bypassable via Inspector. Any restriction that exists only in the DOM is not a security control.

---

### Browser Developer Tools — Debugger

**Definition:** The Debugger (called Sources in Chrome) allows inspection of JavaScript files and insertion of breakpoints that pause execution at specific lines.

**Why It Matters:** Minimised and obfuscated JavaScript files may contain hardcoded API endpoints, secret keys, logic for authentication decisions, or logic that removes content from the page.

**Key Details:**
- Access the file list in the left panel → find JS files under assets/
- **Minification:** All whitespace removed, everything on one line — use Pretty Print `{}` to add readable formatting
- **Obfuscation:** Variable renaming and logic scrambling to resist analysis — Pretty Print helps but cannot fully reverse it
- **Breakpoints:** Click a line number to set a breakpoint; the browser pauses execution at that line, preserving the state at that moment
- Useful for examining the exact state of the page when a specific script runs

**Practical application:** A script that removes a popup or paywall element can be paused with a breakpoint on its last line (`flash['remove']();`), preventing the removal and revealing the content underneath.

---

### Browser Developer Tools — Network Tab

**Definition:** Records all HTTP requests made by a page, including AJAX/XHR requests that happen in the background without page navigation.

**Why It Matters:** Dynamically loaded content, form submissions, and API calls are all invisible in the page source but appear in the Network tab. These reveal API endpoints, request formats, cookies sent with requests, and server response data.

**Key Details:**
- Refresh the page with Network tab open to capture all initial requests
- Submit forms to see the form submission request and its response
- Each request shows: request headers, response headers, cookies, and response body
- **AJAX/XHR requests** (like contact form submissions) appear as entries without page navigation
- Examining these reveals the endpoint receiving the data, the exact fields transmitted, and any cookies included in the request

---

### Browser Developer Tools — Storage Tab

**Definition:** The Storage tab displays all client-side data stored by the website: cookies, Local Storage, Session Storage, and Cache Storage.

**Why It Matters:** Authentication tokens, session identifiers, API keys, and user preferences are often stored client-side. Understanding what is stored — and what security attributes are set — reveals how the application manages authentication and what can be manipulated.

**Key Details — storage types:**

| Storage Type | Persistence | Scope | Common Use |
|-------------|------------|-------|------------|
| Local Storage | Persistent (survives browser close) | Domain | User preferences, cached data |
| Session Storage | Temporary (single tab/session) | Tab | Single-session state |
| Cookies | Configurable (Expires/Max-Age) | Domain + Path | Sessions, authentication |
| Cache Storage | Persistent | Domain | Cached resources for offline use |

**Cookie security flags:**

| Flag | Effect | Attack Protection |
|------|--------|------------------|
| `HttpOnly` | JavaScript cannot read the cookie | Prevents XSS-based cookie theft |
| `Secure` | Cookie only transmitted over HTTPS | Prevents interception on HTTP |
| `SameSite=Strict` | Cookie not sent on cross-site requests | Mitigates CSRF |
| `SameSite=Lax` | Cookie sent on same-site + top-level navigation | Partial CSRF mitigation |

**Practical relevance:** Missing `HttpOnly` means a successful XSS vulnerability can steal the session cookie via `document.cookie`. Missing `Secure` means the cookie is transmitted over HTTP. Absence of `SameSite` leaves the application more vulnerable to CSRF.

---

## Methodology

```
Visit the application as a normal user:
  Navigate every page linked from the navigation
  Follow every link discovered
  Create an account and explore authenticated features
        |
        v
Build application map:
  Record: endpoint, feature, input fields, interesting behaviour
        |
        v
Review page source for each page:
  Look for HTML comments (<!-- ... -->)
  Look for hidden href links not in navigation
  Note file paths in external resource references
  Identify framework from meta tags, generator comments, footer
  Check if asset directories are browsable
        |
        v
Use Developer Tools:
  Inspector: examine DOM elements blocking content
  Debugger: find and inspect JavaScript files; set breakpoints
  Network: submit forms, observe AJAX requests and responses
  Storage: inspect cookies, check security flags, review Local/Session Storage
        |
        v
Record all findings with context:
  What was found?
  Where was it found?
  What does it indicate?
  What is the security implication?
```

---

## Practical Activities

### Activity 1 — Page Source Review

**Objective:** Extract intelligence from raw HTML source.

**Actions:**
- Right-click the page → View Page Source, or prefix URL with `view-source:`
- Search for `<!--` to find HTML comments
- Search for `href=` to find all links including hidden ones
- Identify framework version from footer comments or meta tags

**Findings:**
- Developer comment referencing a temporary homepage with a link to a development version
- A hidden link to a `/secr3t-` prefixed page not visible in the navigation
- Framework name and version number in a footer comment; visiting the framework website confirmed the site was running an outdated version with known issues
- Asset directory (`/assets`) browsable with directory listing enabled — revealing a `flag.txt` file

**Why It Matters:** HTML comments and hidden links are immediate, zero-effort findings that expose paths an attacker can visit directly. Framework version information is the starting point for CVE research. An accessible directory with directory listing enabled can expose sensitive files with no authentication required.

---

### Activity 2 — Inspector: Bypassing a Paywall

**Objective:** Use the Inspector to reveal content hidden behind a client-side paywall.

**Actions:**
1. Navigate to the News section
2. Right-click the premium subscription blocker element → Inspect
3. Locate the `div` element with class `premium-customer-blocker`
4. In the Styles panel, find `display: block`
5. Click the value `block` and change it to `none`

**Findings:**
- The paywall disappeared, revealing the premium article content
- The control was entirely client-side — the server sent the content regardless; the DIV just blocked the view

**Why It Matters:** This is a direct demonstration that client-side restrictions are not security controls. The server sent the protected content unconditionally. Only server-side access control (checking session/role before returning data) would actually protect it.

---

### Activity 3 — Debugger: Intercepting JavaScript Execution

**Objective:** Use a breakpoint to pause a JavaScript removal function and observe what it hides.

**Actions:**
1. Open Debugger tab
2. Navigate to the assets folder and open `flash.min.js`
3. Use Pretty Print `{}` to make the minified code more readable
4. Scroll to the line containing `flash['remove']();`
5. Click that line number to set a breakpoint
6. Refresh the page

**Findings:**
- With the breakpoint set, the red flash element was preserved on screen instead of being removed
- The element contained content (a flag) that the JavaScript was designed to hide immediately on page load

**Why It Matters:** JavaScript that hides, disables, or removes content client-side can always be paused or removed. Breakpoints reveal what the code was about to do and what content it was preventing the user from seeing.

---

### Activity 4 — Network Tab: Capturing Form Submission

**Objective:** Observe an AJAX form submission to discover the receiving endpoint and transmitted data.

**Actions:**
1. Open the Network tab
2. Navigate to the Contact page
3. Fill in the contact form and click Send Message
4. Observe the new request entry in the Network tab
5. Examine: request URL, request body, response body, cookies transmitted

**Findings:**
- The form was submitted via AJAX — a POST request appeared in the Network tab without any page navigation
- The request went to `/contact` with the form data in the body
- Response included confirmation and, in this case, a flag embedded in the server's response

**Why It Matters:** Network tab analysis reveals the exact API contract behind every form. This is how a penetration tester identifies parameters to fuzz, understands what the server expects, and finds endpoints not visible in the page source.

---

### Activity 5 — Storage Tab: Cookie Security Audit

**Objective:** Inspect cookies set after authentication and evaluate their security attributes.

**Actions:**
1. Register an account and log in
2. Open Storage tab → Cookies → select the domain
3. Record all cookie names, values, and flags
4. Note which cookies are missing `HttpOnly`, `Secure`, or `SameSite` attributes

**Findings:**
- Session cookie `PHPSESSID` visible with its value
- Missing `HttpOnly` flag on the session cookie — JavaScript can read it via `document.cookie`
- Missing `SameSite` attribute — cross-site requests include the cookie

**Why It Matters:** A missing `HttpOnly` flag makes the session cookie vulnerable to theft if any XSS vulnerability exists. A missing `SameSite` attribute means CSRF attacks are possible. Both are reportable security findings with direct exploit paths.

---

## Observations and Analysis

- **Client-side content restriction is not access control:** The premium content was delivered by the server regardless of subscription status — only the display was blocked. This is a fundamental web security principle: all access control must be enforced server-side.

- **Minimised JavaScript hides logic, not secrets:** The obfuscated `flash.min.js` was still analysable with Pretty Print. Developers sometimes believe obfuscation makes client-side code "secure" — it does not. It merely adds friction.

- **Framework version in source = attack surface research starting point:** A comment identifying the framework and version is immediately actionable. Checking the framework's official change log, CVE databases, or NVD for that version number often surfaces exploitable vulnerabilities.

- **Directory listing on asset folders exposes unlinked files:** The `/assets` directory being browsable revealed files that were never linked from any page. On a real application, this could expose backup files, configuration dumps, or internal documentation.

- **Cookie flags are not optional in production:** `HttpOnly` and `Secure` should be present on every session cookie. `SameSite=Lax` or `Strict` should be explicitly set. Absent flags are consistently reportable findings in web application assessments.

---

## Tools and Technologies Used

### Web Browser (Firefox/Chrome)
- **Purpose:** Primary manual testing tool for web application walking
- **Common Usage:** Navigate pages, view source, use Developer Tools
- **In This Room:** Application mapping, page source analysis, DOM manipulation, JS debugging, network traffic inspection, storage analysis

---

## Key Learnings

- Manual application walking precedes all automated testing — it builds the map that gives automated tools context
- HTML comments, hidden links, and framework comments in page source are immediate, zero-effort findings
- Browser Developer Tools are as important for security testing as any dedicated tool
- Inspector allows real-time DOM manipulation — client-side restrictions are bypassable
- Debugger breakpoints pause JavaScript execution — revealing hidden content and logic
- Network tab captures AJAX requests that are invisible to page source review
- Cookie `HttpOnly`, `Secure`, and `SameSite` flags are the primary defence against XSS-based session theft and CSRF

---

## Real-World Relevance

**Penetration Testing:** Manual walking is the first step on every web application assessment. Automated scanners find known patterns; manual inspection finds logic flaws, hidden features, and misconfigurations specific to the application.

**Bug Bounty Hunting:** Hidden endpoints discovered via page source and JS files are frequently high-value targets. Many bounty programmes reward finding `.js` files with hardcoded API keys or undocumented endpoints.

**Enterprise Environments:** Cookie security flag audits are a standard deliverable in web application penetration test reports. Finding missing `HttpOnly` or `Secure` flags is a consistent, reportable finding with direct exploit scenarios.

---

## Things Worth Remembering

- View page source: right-click → View Page Source, or `view-source:URL`
- Search source for `<!--` (comments), `href=` (links), and framework references
- Inspector: change `display: block` to `display: none` to bypass client-side paywalls
- Debugger: set breakpoints on removal/hide functions to preserve hidden content
- Network tab: submit forms with Network tab open to capture AJAX requests
- Storage tab: audit cookies for `HttpOnly`, `Secure`, `SameSite` flags
- Client-side controls = no security — enforce access control server-side only
- Missing `HttpOnly` on session cookie = XSS can steal sessions
- Missing `SameSite` = CSRF is possible

---

## Conclusion

Walking An Application establishes the foundational discipline of manual web application reconnaissance. The methodical combination of application mapping, page source review, and Developer Tools usage builds an understanding of how the application works that no automated scanner can replicate. The key insight from this room is that browsers are already powerful security testing tools — long before Burp Suite or any specialised software is opened. The habits built here — reading source, using DevTools, inspecting storage — directly inform every subsequent web application technique.
