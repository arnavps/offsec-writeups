# Crack the Hash

## Overview

A hash-cracking challenge covering multiple hashing algorithms — MD5, SHA-1, SHA-256, bcrypt, MD4, NTLM, SHA-512crypt, and salted SHA-1. The goal is to recover the plaintext password from each hash using online tools and Hashcat. A solid introduction to hash identification and the practical use of Hashcat modes.

**Room:** https://tryhackme.com/room/crackthehash

---

## Topics Covered

- Hash identification
- Online hash cracking (CrackStation)
- Hashcat dictionary attacks
- Hashcat modes for different algorithms
- Salted hashes

---

## Key Concepts

### How Hash Cracking Works

Hash functions are one-way — you can't reverse them mathematically. Cracking works by hashing candidate passwords and comparing the output to the target hash. If they match, the plaintext is found.

Two main approaches:
- **Online tools** — fast for common hashes; databases of pre-computed hash:plaintext pairs
- **Hashcat** — local tool, flexible, supports hundreds of hash types, GPU-accelerated

### Hashcat Basics

```bash
hashcat -m <mode> <hashfile> <wordlist>
```

- `-m` — hash mode (algorithm)
- `-D 2` — use GPU (1 = CPU, 2 = GPU)
- Hash stored in a text file, one per line

---

## Task Walkthroughs

### Level 1

**Task 1-1 — MD5**
```
Hash: 48bb6e862e54f2a795ffc4e541caed4d
Mode: -m 0
Answer: easy
```

**Task 1-2 — SHA-1**
```
Hash: CBFDAC6008F9CAB4083784CBD1874F76618D2A97
Mode: -m 100
Answer: password123
```

**Task 1-3 — SHA-256**
```
Hash: 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
Mode: -m 1400
Answer: letmein
```

**Task 1-4 — bcrypt**
```
Hash: $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
Mode: -m 3200
Answer: bleh
```

bcrypt cannot be cracked with online tools — the cost factor makes it too slow for pre-computed databases. Hashcat required.

**Task 1-5 — MD4**
```
Hash: 279412f945939ba78ce0758d3fd83daa
Mode: -m 900
Answer: Eternity22
```

---

### Level 2

**Task 2-1 — SHA-256**
```
Hash: F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
Mode: -m 1400
Answer: paule
```

**Task 2-2 — NTLM**
```
Hash: 1DFECA0C002AE40B8619ECF94819CC1B
Mode: -m 1000
Answer: n63umy8lkf4i
```

**Task 2-3 — SHA-512crypt ($6$)**
```
Hash: $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
Mode: -m 1800
Answer: waka99
```

Takes significantly longer due to the algorithm's cost. Let it run.

**Task 2-4 — SHA-1 with salt**
```
Hash: e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
Mode: -m 110
Answer: 481616481616
```

The salt (`tryhackme`) is appended after the colon. Online tools won't work here — the salt makes pre-computed lookups useless.

---

## Practical Examples / Demonstrations

### Hashcat dictionary attack

```bash
# Store hash in a file
echo "48bb6e862e54f2a795ffc4e541caed4d" > hash.txt

# Run Hashcat (MD5)
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# GPU mode (Windows)
hashcat64.exe -D 2 -m 0 Hash/hash.txt Dict/rockyou.txt
```

### Hashcat mode reference

| Algorithm | Hashcat Mode |
|---|---|
| MD5 | 0 |
| SHA-1 | 100 |
| SHA-256 | 1400 |
| MD4 | 900 |
| NTLM | 1000 |
| bcrypt | 3200 |
| SHA-512crypt ($6$) | 1800 |
| SHA-1 + salt (sha1($p.$s)) | 110 |

### Identify an unknown hash

Use an online hash identifier if unsure of the algorithm:
- https://md5hashing.net/hash_type_checker
- `hash-identifier` (Linux tool)

---

## Real-World Relevance

- NTLM hashes are extracted from Windows systems during post-exploitation (via tools like Mimikatz or secretsdump) and cracked offline
- bcrypt is the recommended algorithm for password storage — its intentional slowness makes cracking impractical at scale
- Salted hashes prevent rainbow table attacks — each password produces a unique hash even if the plaintext is identical
- MD5 and SHA-1 are cryptographically broken and should never be used for password storage
- Hash cracking is a standard step after obtaining a credential dump from a compromised database

---

## Key Learnings

- Different hash algorithms require different Hashcat modes — always identify the hash type first
- Online tools work for unsalted, common hashes; Hashcat is required for bcrypt, salted hashes, and uncommon algorithms
- bcrypt is intentionally slow — cracking it takes orders of magnitude longer than MD5
- Salts defeat pre-computed (rainbow table) attacks by making each hash unique
- Run Hashcat on the host machine, not a VM — VMs can't access GPU resources

---

## Additional Notes

- CrackStation: https://crackstation.net
- Hashcat modes reference: https://hashcat.net/wiki/doku.php?id=example_hashes
- rockyou.txt is pre-installed on Kali/Parrot at `/usr/share/wordlists/rockyou.txt`
- For long bcrypt runs, use `--session=<name>` to allow resuming if interrupted

---

## Conclusion

Crack the Hash is a practical introduction to hash identification and Hashcat. The key takeaway is that not all hashes are equal — algorithm choice directly determines how resistant a hash is to cracking. MD5 falls in seconds; bcrypt can take hours or days. Understanding this distinction is fundamental to both offensive credential attacks and defensive password storage decisions.
