# John the Ripper: Basics

## Overview

John the Ripper (JtR) is a versatile, widely-used hash cracking tool that combines fast cracking speed with support for an extensive range of hash types. It is used in penetration testing and CTFs to crack password hashes from Linux shadow files, Windows SAM databases, password-protected archives, and SSH keys. This room covers John's core syntax, cracking modes, and specialised conversion tools.

---

## Topics Covered

- Basic syntax and automatic cracking
- Hash identification
- Format-specific cracking
- Cracking NTHash/NTLM (Windows)
- Cracking `/etc/shadow` (Linux) with `unshadow`
- Single crack mode and word mangling
- Custom rules
- Cracking ZIP and RAR archives
- Cracking SSH private key passphrases

---

## Key Concepts

### Basic Syntax

```bash
john [options] [file]
```

**Automatic cracking (John detects hash type):**

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Format-specific cracking:**

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**List all supported formats:**

```bash
john --list=formats
john --list=formats | grep -iF "md5"
```

**View cracked passwords:**

```bash
john --show hash.txt
john --show --format=raw-md5 hash.txt
```

---

### Hash Identification

John doesn't always auto-detect correctly. Use `hash-identifier` or online tools to identify the hash type first.

```bash
python3 hash-id.py
```

Or use online tools like https://hashes.com/en/tools/hash_identifier

---

### Cracking NTHash / NTLM (Windows)

NTLM is the hash format used by modern Windows to store passwords in the SAM database. Obtained via Mimikatz, secretsdump, or by reading NTDS.dit.

```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

### Cracking Linux `/etc/shadow` with `unshadow`

John requires the `/etc/passwd` and `/etc/shadow` files to be combined before cracking.

```bash
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
```

Or with specific files:

```bash
unshadow local_passwd local_shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
```

---

### Single Crack Mode

Uses the username and GECOS field information to generate candidate passwords through word mangling. Useful when you know the username.

```bash
john --single --format=raw-sha256 hashes.txt
```

**File format for single mode** — prepend the hash with the username:

```
mike:1efee03cdcb96d90ad48ccc7b8666033
```

**Word mangling examples for username "Markus":**
- `Markus1`, `Markus2`, `Markus!`
- `MArkus`, `MARKus`
- `markus`, `MARKUS`

---

### Custom Rules

Custom rules are defined in `john.conf` (usually `/opt/john/john.conf` or `/etc/john/john.conf`).

**Rule syntax:**

```
[List.Rules:RuleName]
cAz"[0-9][!£$%@]"
```

- `c` — capitalise first letter
- `Az` — append to end
- `[0-9]` — a digit 0–9
- `[!£$%@]` — one of these symbols

**Example — generate `Polopassword1!` from `polopassword`:**

```
[List.Rules:PoloPassword]
cAz"[0-9][!£$%@]"
```

**Use a custom rule:**

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rule=PoloPassword hash.txt
```

---

### Cracking ZIP Archives

```bash
zip2john zipfile.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
```

---

### Cracking RAR Archives

```bash
rar2john rarfile.rar > rar_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt
```

---

### Cracking SSH Private Key Passphrases

```bash
ssh2john id_rsa > id_rsa_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
```

On some systems:

```bash
python3 /opt/john/ssh2john.py id_rsa > id_rsa_hash.txt
```

---

## Practical Examples / Demonstrations

### Lab walkthrough (from the referenced walkthrough)

**Crack hash1.txt (MD5):**

```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash1.txt
john --show --format=raw-md5 hash1.txt
```

**Crack hash2.txt (SHA-1):**

```bash
john --format=raw-sha1 --wordlist=/usr/share/wordlists/rockyou.txt hash2.txt
```

**Crack hash3.txt (SHA-256):**

```bash
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash3.txt
```

**Crack hash4.txt (Bcrypt):**

```bash
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash4.txt
```

**Crack hash5.txt (MD4):**

```bash
john --format=raw-md4 --wordlist=/usr/share/wordlists/rockyou.txt hash5.txt
```

**Crack hash6.txt (NTLM/NTHash):**

```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash6.txt
```

**Crack hash7.txt (sha512crypt):**

```bash
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash7.txt
```

**Unshadow and crack:**

```bash
unshadow passwd.txt shadow.txt > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt
```

**Single crack mode:**

```bash
# Prepend username to hash in file: joker:hash_value
john --single --format=raw-md5 hash8.txt
```

**Crack ZIP:**

```bash
zip2john secure.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
john --show zip_hash.txt
```

**Crack RAR:**

```bash
rar2john secure.rar > rar_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt
john --show rar_hash.txt
```

**Crack SSH key:**

```bash
ssh2john id_rsa > id_rsa_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
john --show id_rsa_hash.txt
```

---

## Important Terminology

| Term | Definition |
|---|---|
| John the Ripper | CPU-based hash cracking tool with broad format support |
| `unshadow` | John utility combining `/etc/passwd` and `/etc/shadow` for cracking |
| `zip2john` | Converts ZIP file to crackable hash format |
| `rar2john` | Converts RAR file to crackable hash format |
| `ssh2john` | Converts SSH private key to crackable hash format |
| Single crack mode | Generates candidates from username/GECOS data via word mangling |
| Word mangling | Applying transformations to a base word to generate password candidates |
| GECOS | Unix user info field (full name, etc.) used by John in single mode |
| Custom rules | User-defined transformation rules in `john.conf` |
| Potfile | John's file storing previously cracked hashes (`john.pot`) |
| NTLM | Windows password hash format stored in the SAM database |

---

## Real-World Relevance

- John is used in penetration tests to crack hashes obtained from compromised systems
- `unshadow` + John is the standard workflow for cracking Linux password hashes after gaining root access
- SSH key passphrase cracking is relevant when a private key is found on a compromised system
- ZIP and RAR cracking is common in CTFs and forensic investigations
- Single crack mode exploits the human tendency to base passwords on usernames — a realistic attack against real accounts
- Custom rules exploit predictable password patterns (capitalise first letter, append number and symbol) that users apply to meet complexity requirements

---

## Key Learnings

- Always specify `--format` — John's auto-detection is unreliable for many hash types
- `unshadow` must be run before cracking Linux shadow hashes
- Single crack mode generates candidates from username data — prepend `username:` to the hash
- `zip2john`, `rar2john`, and `ssh2john` convert protected files to crackable hash format
- Custom rules in `john.conf` exploit predictable password patterns
- `--show` displays previously cracked hashes from the potfile

---

## Conclusion

John the Ripper is a flexible and powerful tool for cracking a wide variety of hash types. Its conversion utilities (`unshadow`, `zip2john`, `rar2john`, `ssh2john`) make it applicable far beyond simple hash files. Understanding its modes — wordlist, single, and custom rules — and knowing when to use each is a practical skill for both CTF challenges and real-world penetration testing engagements.
