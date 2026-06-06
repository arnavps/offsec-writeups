# JavaScript Essentials

## Overview

JavaScript (JS) is the scripting language that powers dynamic behaviour on the web. It runs in the browser, handles user interactions, validates input, and communicates with servers. For a security professional, JS is important from two angles: understanding how web applications use it, and recognising how attackers exploit it. This room covers the core JS concepts needed to read, reason about, and eventually audit client-side web application code.

---

## Topics Covered

- Variables and data types
- Functions and loops
- Internal vs external JS
- Browser interaction functions: `alert`, `prompt`, `confirm`
- Control flow: conditionals and loops
- How attackers exploit client-side JS
- JS minification and obfuscation
- Secure coding practices for JavaScript

---

## Key Concepts

### Variables

Variables store data values and are referenced by name. JavaScript has three declaration keywords:

| Keyword | Scope | Reassignable | Notes |
|---------|-------|--------------|-------|
| `var` | Function-scoped | Yes | Legacy; can cause bugs due to hoisting |
| `let` | Block-scoped | Yes | Preferred for mutable values |
| `const` | Block-scoped | No | Preferred for constants and fixed references |

`let` and `const` are preferred in modern JS because block scoping gives better control over where a variable is accessible.

---

### Data Types

| Type | Example | Description |
|------|---------|-------------|
| `string` | `"hello"` | Text values |
| `number` | `42`, `3.14` | Integers and decimals |
| `boolean` | `true`, `false` | Logical values |
| `null` | `null` | Intentional absence of a value |
| `undefined` | (unassigned variable) | Variable declared but not assigned |
| `object` | `{}`, `[]` | Complex structures including arrays and objects |

---

### Functions

A function is a named, reusable block of code that performs a specific task. Grouping logic into functions avoids repetition and makes code maintainable.

```javascript
function PrintResult(rollNum) {
    alert("Roll number " + rollNum + " has passed the exam");
}
```

Functions can accept **parameters** (inputs) and can be called as many times as needed.

---

### Loops

Loops execute a block of code repeatedly while a condition remains true. Common loop types: `for`, `while`, `do...while`.

```javascript
const rollNumbers = [101, 102, 103];

for (let i = 0; i < rollNumbers.length; i++) {
    PrintResult(rollNumbers[i]);
}
```

This calls `PrintResult()` once for each item in the array instead of manually writing multiple calls.

---

### Control Flow

Control flow determines the order of execution based on conditions.

**if-else** — execute different code blocks depending on a condition:

```javascript
if (username === "admin" && password === "secret") {
    // grant access
} else {
    // deny access
}
```

**switch** — multi-branch conditional, cleaner than chained `if-else` for matching a single value.

---

### Browser Interaction Functions

JS provides built-in functions to interact with the user through the browser. These are useful for understanding client-side behaviour and spotting potential abuse vectors.

| Function | Behaviour | Return Value |
|----------|-----------|-------------|
| `alert("message")` | Displays a message in a popup with an OK button | `undefined` |
| `prompt("question")` | Displays an input box asking the user for text | User input string, or `null` if cancelled |
| `confirm("question")` | Displays a Yes/No popup | `true` (OK) or `false` (Cancel) |

**How attackers exploit these:**
If an attacker can inject JS into a page (XSS), they can abuse these functions to harass or trick users. For example:

```javascript
// This would trigger three consecutive alert boxes
for (let i = 0; i < 3; i++) {
    alert("Hacked");
}
```

This is a simple demonstration of what's possible — real XSS payloads can steal cookies, redirect users, or exfiltrate data.

---

### Internal vs External JavaScript

**Internal JS** — embedded directly inside an HTML file using `<script>` tags:

```html
<script>
    alert("Hello from internal JS");
</script>
```

Can be placed in `<head>` (loaded before page renders) or `<body>` (loaded as elements appear).

**External JS** — stored in a separate `.js` file and linked in HTML:

```html
<script src="app.js"></script>
```

This keeps HTML clean and allows the script to be cached. The external file can be hosted on the same server or a third-party CDN.

---

### Minification and Obfuscation

**Minification** compresses JS by removing whitespace, comments, and newlines, and shortening variable names. This reduces file size and improves load time.

- Minified code is harder for humans to read but functionally identical
- A minified file often has a `.min.js` extension (e.g. `jquery.min.js`)

**Obfuscation** deliberately makes code harder to understand by renaming variables to meaningless strings, inserting dummy code, and transforming logic. It's commonly used to protect intellectual property or slow down reverse engineering.

Neither minification nor obfuscation is a security control — a determined attacker can reverse both. They add friction, not protection.

---

## Secure Coding Practices

### 1. Never rely solely on client-side validation

JS validation runs in the browser and can be disabled or bypassed by the user. Always re-validate all input on the server side.

```javascript
// Client-side check — easy to bypass
if (username.length > 3) { ... }
```

Server-side validation is mandatory. Client-side validation is only a UX enhancement.

### 2. Do not include untrusted libraries

Including a third-party script via `<script src="...">` gives that script full access to the page. Attackers publish packages with names that closely resemble legitimate ones (typosquatting in the npm ecosystem). Always verify the source and integrity of any library you include.

Use **Subresource Integrity (SRI)** when loading from a CDN:

```html
<script src="https://cdn.example.com/lib.js"
        integrity="sha384-abc123..."
        crossorigin="anonymous"></script>
```

### 3. Never hardcode secrets in JS

Client-side JS is visible to anyone who opens browser DevTools. Hardcoding API keys, tokens, or passwords in JS exposes them publicly.

```javascript
// Bad — anyone can read this in the browser
const privateAPIKey = 'pk_TryHackMe-1337';
```

Secrets must be kept server-side and accessed through authenticated API calls.

### 4. Minify and obfuscate in production

While not a security guarantee, minifying and obfuscating production code raises the bar for attackers trying to understand and exploit application logic.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Scope | Where a variable is accessible in code |
| Function | A named, reusable block of code |
| Loop | Repeated execution while a condition is true |
| XSS | Cross-Site Scripting — injecting malicious JS into a web page |
| DOM | Document Object Model — the browser's representation of an HTML page |
| Minification | Removing unnecessary characters to reduce file size |
| Obfuscation | Transforming code to make it harder to read/reverse engineer |
| SRI | Subresource Integrity — cryptographic verification of external scripts |
| CDN | Content Delivery Network — distributed hosting for static assets |

---

## Practical Examples

### Bypassing a client-side login check

If a developer implements authentication entirely in JS:

```javascript
if (username === "admin" && password === "hardcoded_pass") {
    grantAccess();
}
```

An attacker can open the browser console and call `grantAccess()` directly, or modify the JS to skip the check entirely. This is why server-side authentication is non-negotiable.

### XSS via alert abuse

```javascript
// Injected into a vulnerable input field
<script>alert(document.cookie)</script>
```

If the application doesn't sanitise input and renders it in the DOM, this executes in any visitor's browser.

---

## Real-World Relevance

- **XSS (Cross-Site Scripting)** is consistently in the OWASP Top 10 and directly exploits JavaScript execution in the browser
- **Client-side only validation** is a common developer mistake exploited in web app pentests and bug bounties
- **Hardcoded API keys** in JS files are regularly found via GitHub searches and automated scanning tools
- **Malicious npm packages** that mimic legitimate libraries (e.g. `event-stream` incident) are an active supply chain threat
- Understanding JS is required to read, manipulate, and craft payloads when testing web applications manually with Burp Suite

---

## Key Learnings

- Use `let` and `const` instead of `var` for better scoping control
- Functions and loops are the core building blocks of JS logic
- `alert`, `prompt`, and `confirm` are browser-native functions that attackers can abuse via XSS
- Client-side validation is easily bypassed — server-side validation is always required
- Never put secrets in client-side JavaScript
- Minification reduces file size; obfuscation adds reverse engineering friction — neither is a security boundary

---

## Additional Notes

- The browser console (F12 → Console tab) lets anyone execute arbitrary JS on any page they visit — this is expected behaviour, not a vulnerability in itself
- Minified JS can be de-minified/prettified using browser DevTools or tools like `js-beautify`
- Obfuscated JS can be deobfuscated using tools like [deobfuscate.io](https://deobfuscate.io) or manual analysis
- The `<script>` tag's `src` attribute can load external JS from any domain — this is a significant trust decision

---

## Conclusion

JavaScript is the language of the web's front end and a central concern in web application security. Understanding how it handles variables, functions, and user interaction builds the foundation for recognising and exploiting client-side vulnerabilities. The secure coding practices covered here — server-side validation, no hardcoded secrets, trusted libraries only — are directly applicable to both building and auditing web applications.
