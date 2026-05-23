# How Websites Work

## Overview

This room covers the fundamentals of how websites are built and served — from the technologies that make up a webpage to how browsers render content and how user input can become a security vulnerability. Understanding the client-server model and the role of HTML, CSS, and JavaScript is essential groundwork for web application security.

---

## Topics Covered

- Client-server model
- Frontend technologies: HTML, CSS, JavaScript
- Sensitive data exposure
- HTML injection

---

## Key Concepts

### The Client-Server Model

When you visit a website, your browser (the client) sends a request to a web server. The server processes the request and returns data — HTML, CSS, JavaScript, images — which the browser uses to render the page.

- **Frontend (Client-Side)** — everything the browser downloads and renders: HTML structure, CSS styling, JavaScript behaviour
- **Backend (Server-Side)** — the server logic that processes requests, queries databases, and generates responses

---

### Frontend Technologies

**HTML (HyperText Markup Language)**
Defines the structure and content of a webpage using elements (tags). It is the skeleton of every webpage.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Example Page</title>
  </head>
  <body>
    <h1>Hello World</h1>
    <p>This is a paragraph.</p>
  </body>
</html>
```

**CSS (Cascading Style Sheets)**
Controls the visual presentation of HTML elements — colours, fonts, layout, spacing. CSS does not affect functionality, only appearance.

**JavaScript (JS)**
Adds interactivity and dynamic behaviour to web pages. JS runs in the browser and can:
- Modify page content in real time without reloading
- Respond to user events (clicks, input, scrolling)
- Make requests to the server in the background (AJAX/fetch)

Without JavaScript, web pages would be entirely static.

---

### Sensitive Data Exposure

Sensitive data exposure occurs when a website inadvertently includes sensitive information in its frontend source code — visible to anyone who views the page source.

Common examples:
- HTML comments containing credentials or internal notes
- Hardcoded API keys or tokens in JavaScript files
- Internal IP addresses or server paths in source code

Since the browser downloads all frontend code, anything in HTML, CSS, or JS is accessible to the user — including attackers. Developers sometimes leave debug information, temporary credentials, or internal comments in production code.

**How to check:** Right-click any webpage → "View Page Source" or use browser developer tools.

---

### HTML Injection

HTML injection is a vulnerability that occurs when user-supplied input is rendered directly on the page without sanitisation. If a website takes user input and inserts it into the HTML without filtering, an attacker can inject arbitrary HTML tags.

**Example — vulnerable behaviour:**

If a site displays: `Welcome, [username]!` and the username field is not sanitised, an attacker could input:

```html
<h1>Hacked</h1>
```

And the page would render that as actual HTML, displaying a large "Hacked" heading.

Taking it further, if JavaScript is also injectable, this becomes Cross-Site Scripting (XSS):

```html
<script>alert('XSS')</script>
```

**Root cause:** Lack of input sanitisation — the application trusts user input and renders it as markup rather than plain text.

---

## Important Terminology

| Term | Definition |
|---|---|
| Client-Side | Code that runs in the user's browser (HTML, CSS, JS) |
| Server-Side | Code that runs on the web server (PHP, Python, Node.js, etc.) |
| HTML | Markup language defining the structure of a webpage |
| CSS | Stylesheet language controlling visual presentation |
| JavaScript | Scripting language adding interactivity to web pages |
| Sensitive Data Exposure | Unintentional disclosure of sensitive info in frontend source code |
| Input Sanitisation | Filtering and validating user input before processing or displaying it |
| HTML Injection | Injecting HTML tags via unsanitised user input |
| XSS | Cross-Site Scripting — injecting JavaScript via unsanitised input |

---

## Practical Examples / Demonstrations

### Viewing page source

In any browser:
```
Right-click → View Page Source
```
or navigate to:
```
view-source:https://example.com
```

### HTML injection payload

```html
<img src=x onerror="alert('injected')">
```

If this input is reflected on the page without sanitisation, the browser executes the `onerror` handler.

### Basic XSS test payload

```html
<script>alert(1)</script>
```

If an alert box appears, the input is being rendered as HTML/JS rather than plain text.

---

## Workflow / Process

### How a Browser Renders a Page

```
User enters URL in browser
        |
Browser sends HTTP GET request to web server
        |
Server returns HTML document
        |
Browser parses HTML → builds DOM
        |
Browser fetches linked CSS → applies styles
        |
Browser fetches and executes JavaScript → adds interactivity
        |
Page is fully rendered and displayed
```

---

## Real-World Relevance

- Sensitive data exposure is one of the OWASP Top 10 — hardcoded credentials and API keys in source code are a common finding in bug bounty and pentests
- HTML injection is the entry point to understanding XSS, which is one of the most prevalent web vulnerabilities
- JavaScript files often contain API endpoints, authentication logic, and internal paths — reviewing JS source is a standard recon step
- Input sanitisation failures lead to a wide range of vulnerabilities: XSS, HTML injection, SQLi, command injection

---

## Key Learnings

- Websites consist of a frontend (client-side) and a backend (server-side)
- HTML structures content, CSS styles it, JavaScript makes it interactive
- All frontend code is visible to the user — never store sensitive data there
- Unsanitised user input rendered on a page leads to HTML injection and XSS
- Viewing page source and JS files is a standard first step in web recon

---

## Additional Notes

- Browser developer tools (F12) provide far more detail than view-source — including network requests, cookies, local storage, and the DOM
- Content Security Policy (CSP) headers can mitigate XSS by restricting which scripts are allowed to execute
- HTML encoding user input (converting `<` to `&lt;`) is the primary defence against HTML injection

---

## Conclusion

Understanding how websites are built — and how browsers process and render that code — is the foundation of web application security. The gap between what developers intend and what the browser actually does with user input is where most client-side vulnerabilities live.
