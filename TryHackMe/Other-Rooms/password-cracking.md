# Password Cracking

## Overview

A deep-dive into password cracking techniques — covering hash identification, dictionary attacks, rule-based attacks, mask attacks, and brute force. The room uses both Hashcat and John the Ripper across MD5, SHA-256, NTLM, and bcrypt hashes. Covers the theory behind why some hashes are easy to crack and others aren't.

**Room:** https://tryhackme.com/room/passwordcracking

---

## Topics Covered

- Hash identification by visual characteristics
- Dictionary attacks
- Rule-based attacks
- Mask attacks
- Brute-force attacks
- Hashcat and John the Ripper usage
- Salting and preimage resistance
- bcrypt and key stretching

---

## Key Concepts

### Hash Identification

Before cracking, identify the algorithm by inspecting the hash format:

| Hash Type | Length | Prefix / Format |
|---|---|---|
| MD5 | 32 hex chars | None |
| SHA-1 | 40 hex chars | None |
| SHA-256 | 64 hex chars | None |
| NTLM | 32 hex chars | None |
| bcrypt | ~60 chars | `$2a$`, `$2b$`, or `$2y$` |

### Tool Mode Reference

| Algorithm | Hashcat `-m` | John `--format=` |
|---|---|---|
| MD5 | 0 | raw-md5 |
| SHA-1 | 100 | raw-sha1 |
| SHA-256 | 1400 | raw-sha256 |
| SHA-512 | 1700 | raw-sha512 |
| NTLM | 1000 | nt |
| bcrypt | 3200 | bcrypt |

---

### Attack Types

#### Dictionary Attack

Tests a pre-built wordlist against the hash one candidate at a time. The fastest first step — always start here.

The most effective wordlist is `rockyou.txt` — 14 million real passwords from the 2009 RockYou breach. Pre-installed at `/usr/share/wordlists/rockyou.txt`.

**Limitation:** Only cracks passwords that appear verbatim in the wordlist.

#### Rule-Based Attack

Applies transformations to each word in a wordlist, generating common mutations:

- Capitalise first letter: `password` → `Password`
- Append number: `password` → `password1`
- Add special character: `password` → `password!`
- Substitute characters: `password` → `p@ssw0rd`

Exploits predictable patterns users apply to meet password policies.

**Hashcat rule files** (at `/opt/hashcat/rules/`):

| Rule File | Description |
|---|---|
| `best64.rule` | 64 highly effective mutations — good first choice |
| `rockyou-30000.rule` | 30,000 rules derived from RockYou analysis |
| `d3ad0ne.rule` | Large community-built rule set |
| `dive.rule` | Extensive rule set covering wide range of mutations |

#### Mask Attack

A structured brute-force where you define the password pattern. Generates only candidates matching the specified structure — far more efficient than full brute force when the pattern is known.

**Hashcat mask placeholders:**

| Placeholder | Character Set |
|---|---|
| `?l` | Lowercase (a-z) |
| `?u` | Uppercase (A-Z) |
| `?d` | Digits (0-9) |
| `?s` | Special characters |
| `?a` | All printable ASCII |

Example — password like `Summer2026!`:
```
?u?l?l?l?l?l?d?d?d?d?s
```

#### Brute Force

Tries every possible combination up to a specified length. Guaranteed to work given enough time, but impractical beyond 6–7 characters.

- 6-char lowercase: `26^6` = 308 million combinations
- 8-char mixed case + digits: `62^8` = 218 trillion combinations

Use only when the search space is genuinely small (e.g., 4-digit PIN).

### Choosing the Right Approach

| Scenario | Best Approach |
|---|---|
| No information about the password | Dictionary attack with rockyou.txt |
| Dictionary fails, password likely mutated | Dictionary + rules (e.g., best64.rule) |
| Known password pattern or enforced policy | Mask attack |
| Short password, small character set | Brute force |
| Target likely used company-specific terms | Custom wordlist + rules |

---

### Cryptographic Concepts

**Preimage resistance** — given a hash output `h`, it is computationally infeasible to find an input `x` where `H(x) = h`. This is why hashes can't be reversed directly.

**Salting** — a unique random value added to a password before hashing. Prevents rainbow table attacks and ensures identical passwords produce different hashes.

**bcrypt** — a password hashing algorithm designed specifically for password storage. Intentionally slow, uses key stretching, includes built-in salting, and is resistant to GPU brute-force attacks.

---

## Practical Examples / Demonstrations

### Setup

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > demo.txt
echo "0571749e2ac330a7455809c6b0e7af90" > demo2.txt
echo "37b4e2d82900d5e94b8da524fbeb33c0" > demo3.txt
```

### John the Ripper

**Dictionary attack:**
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt demo.txt
```

**Rule-based attack:**
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt --rules=wordlist demo2.txt
```

**View cracked passwords:**
```bash
john --show --format=raw-md5 demo.txt
```

Always specify `--format` — without it, John may auto-detect incorrectly and crack nothing.

### Hashcat

**Dictionary attack:**
```bash
hashcat -m 0 -a 0 demo.txt /usr/share/wordlists/rockyou.txt
```

**Rule-based attack:**
```bash
hashcat -m 0 -a 0 demo2.txt /usr/share/wordlists/rockyou.txt -r /opt/hashcat/rules/best64.rule
```

**Mask attack:**
```bash
hashcat -m 0 -a 3 demo3.txt '?l?l?l?l?l?l?l?l'
```

**Save output to file:**
```bash
hashcat -m 0 -a 0 demo.txt /usr/share/wordlists/rockyou.txt -o cracked.txt
```

**View previously cracked hashes:**
```bash
hashcat -m 0 demo.txt --show
```

**Session management (for long runs):**
```bash
# Start named session
hashcat -m 3200 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt --session=bcrypt_crack

# Resume if interrupted
hashcat --session=bcrypt_crack --restore
```

---

## Lab Hash Walkthroughs

**Hash 1 — MD5**
```
e10adc3949ba59abbe56e057f20f883e
Answer: 123456
```

**Hash 2 — SHA-256**
```
5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8
Command: hashcat -m 1400 hash2.txt /usr/share/wordlists/rockyou.txt
Answer: password
```

**Hash 3 — NTLM**
```
8846f7eaee8fb117ad06bdd830b7586c
Answer: password
```

**Hash 4 — bcrypt**
```
$2b$05$9I7YCSrgm6aLO7J5YPC9x.Kp08LQ7cSJTmkALhFTgm5UMFAwBr5.e
Prefix: $2b$ = bcrypt v2b, cost factor 05
Command: hashcat -m 3200 hash4.txt /usr/share/wordlists/rockyou.txt
Answer: hayden07
```

bcrypt at cost factor 5 processes only a few hundred candidates per second — compared to billions per second for MD5. This is by design.

---

## Tool Comparison

| | John the Ripper | Hashcat |
|---|---|---|
| Acceleration | CPU (primarily) | GPU (primarily, CPU fallback) |
| Speed (MD5/SHA) | Fast | Very fast |
| Format detection | Good auto-detect | Explicit mode required |
| Non-standard formats | Excellent | Good |
| Best for | Quick attempts, varied formats, shadow files | Sustained attacks, GPU-accelerated cracking |

Neither is strictly better. John is good for quick auto-detect attempts; Hashcat is the choice for sustained high-speed attacks.

---

## Real-World Relevance

- Password cracking is a standard post-exploitation step after obtaining a credential dump from a compromised system or database
- NTLM hashes are extracted from Windows systems via tools like Mimikatz, secretsdump, or by reading the SAM database
- Rule-based attacks are highly effective against corporate passwords — users predictably capitalise the first letter and append `1` or `!` to meet policy requirements
- bcrypt is the recommended algorithm for password storage — its cost factor makes large-scale cracking impractical
- RockYou2024 (~9.9 billion passwords) illustrates why dictionary attacks remain so effective — attackers draw from real passwords people have actually used

---

## Key Learnings

- Always start with a dictionary attack — it's fast and covers the most common passwords
- Rule-based attacks extend dictionary coverage by generating predictable mutations
- Mask attacks are efficient when the password structure is known
- Brute force is only viable for very short or constrained passwords
- bcrypt is slow by design — this is a feature, not a bug
- Salting defeats rainbow tables; preimage resistance prevents direct reversal
- Always specify `--format` in John and `-m` in Hashcat — incorrect format detection wastes the entire run

---

## Additional Notes

- Hashcat potfile: `/usr/local/hashcat/hashcat.potfile` — stores all cracked hashes; re-running skips already-cracked entries
- John potfile: `/usr/local/john/run/john.pot`
- SecLists wordlists: `/usr/share/wordlists/SecLists/Passwords/`
- On the AttackBox without a GPU, Hashcat runs in CPU mode — MD5/SHA attacks are still fast; bcrypt will be slow regardless

---

## Conclusion

Password cracking is a core offensive skill and a critical area of defensive awareness. The speed at which a hash can be cracked is entirely determined by the algorithm — MD5 falls in seconds, bcrypt takes hours. Understanding attack techniques (dictionary, rules, masks) and the cryptographic properties that make some hashes resistant (salting, key stretching) is essential for both breaking credentials and designing systems that store them securely.
