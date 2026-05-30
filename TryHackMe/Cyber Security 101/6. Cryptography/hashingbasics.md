# Hashing Basics

## Overview

Hashing is a one-way cryptographic function that converts data of any size into a fixed-length output. Unlike encryption, there is no key and no way to reverse the process. Hashing is used to verify file integrity, store passwords securely, and detect tampering. This room covers how hash functions work, why collisions matter, password storage best practices, rainbow tables, salting, and HMAC.

---

## Topics Covered

- What hash functions are and how they work
- Common hash algorithms: MD5, SHA-1, SHA-256, SHA-512
- Hash collisions and the pigeonhole effect
- Password storage: plaintext, encryption, and hashing
- Rainbow tables and salting
- Secure password storage with Argon2, Bcrypt, Scrypt
- Linux password storage (`/etc/shadow`)
- Integrity checking with hashes
- HMAC

---

## Key Concepts

### What is a Hash Function?

A hash function takes input of any size and produces a fixed-size output (the hash value). Key properties:

- **One-way** — computationally infeasible to reverse (preimage resistance)
- **Deterministic** — same input always produces the same output
- **Avalanche effect** — a single bit change in input causes a completely different output
- **Fixed output size** — regardless of input size

Hash functions are not encryption — there is no key, and the output cannot be reversed.

**Example — single bit difference:**
- `T` (0x54 = 01010100) and `U` (0x55 = 01010101) differ by one bit
- Their MD5, SHA-1, and SHA-256 hashes are completely different

---

### Common Hash Algorithms

| Algorithm | Output Size | Status |
|---|---|---|
| MD5 | 128-bit (32 hex chars) | Broken — collisions demonstrated |
| SHA-1 | 160-bit (40 hex chars) | Broken — collisions demonstrated |
| SHA-256 | 256-bit (64 hex chars) | Secure — widely used |
| SHA-512 | 512-bit (128 hex chars) | Secure — stronger than SHA-256 |
| Bcrypt | ~60 chars (with prefix) | Designed for passwords — intentionally slow |

Hash output is typically encoded as hexadecimal. `md5sum`, `sha1sum`, `sha256sum`, `sha512sum` produce hex output.

---

### Hash Collisions

A collision occurs when two different inputs produce the same hash output. Because the number of possible inputs is unlimited but outputs are fixed-size, collisions are mathematically inevitable (pigeonhole effect).

**MD5 and SHA-1 are broken** — collisions can be engineered intentionally. They should not be used for password storage or integrity verification.

---

### Password Storage

**Storing passwords in plaintext** — catastrophic if the database is breached. The RockYou breach (2009) exposed 14 million plaintext passwords — the source of `rockyou.txt`.

**Storing passwords encrypted** — still requires storing the encryption key. If the key is compromised, all passwords are exposed.

**Storing password hashes** — the correct approach. The server stores the hash; when a user logs in, it hashes the submitted password and compares. The original password is never stored.

**Problem with unsalted hashes:** If two users have the same password, they have the same hash. Rainbow tables can reverse common hashes instantly.

---

### Rainbow Tables

A rainbow table is a precomputed lookup table mapping hash values to their plaintext inputs. Given a hash, you look it up in the table and instantly find the password — no cracking required.

**Defence:** Salting.

---

### Salting

A salt is a unique random value added to each password before hashing. Even if two users have the same password, their salts differ, so their hashes differ. Rainbow tables become useless because they would need to be recomputed for every possible salt.

Salts are stored alongside the hash — they don't need to be secret.

**Secure password storage process:**

1. Choose a secure algorithm: Argon2, Scrypt, Bcrypt, or PBKDF2
2. Generate a unique random salt for this user
3. Concatenate: `password + salt`
4. Hash the combined string
5. Store: `hash + salt`

---

### Linux Password Storage

Passwords are stored in `/etc/shadow` (root-readable only). Format:

```
$prefix$options$salt$hash
```

**Common prefixes:**

| Prefix | Algorithm |
|---|---|
| `$y$` | yescrypt (default on modern systems) |
| `$2b$`, `$2y$` | bcrypt |
| `$6$` | sha512crypt |
| `$1$` | md5crypt (legacy) |

The prefix makes it easy to identify the algorithm used.

---

### Integrity Checking

Hashing verifies that a file hasn't been modified. If even one bit changes, the hash changes completely.

```bash
sha256sum filename.iso
sha256sum ~/Hashing-Basics/Task-7/libgcrypt-1.11.0.tar.bz2
```

Compare the output against the official hash published by the software vendor. If they match, the file is identical to the original.

Hashing can also find duplicate files — identical files have identical hashes.

---

### HMAC (Hash-based Message Authentication Code)

HMAC combines a hash function with a secret key to verify both **authenticity** (the message came from someone with the key) and **integrity** (the message hasn't been modified).

```
HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))
```

Where K = key, M = message, H = hash function.

HMAC is used in TLS, JWT tokens, and API authentication.

---

## Practical Examples / Demonstrations

### Generate file hashes

```bash
md5sum file.txt
sha1sum file.txt
sha256sum file.txt
sha512sum file.txt
```

### Crack hashes with Hashcat

```bash
# Bcrypt
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# SHA-256
hashcat -m 1400 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# SHA-512crypt ($6$)
hashcat -m 1800 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# MD5
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### Lab hash answers

| Hash | Algorithm | Answer | Command |
|---|---|---|---|
| `$2a$06$7yoU3Ng8...` | Bcrypt | `85208520` | `hashcat -m 3200 -a 0 hash1.txt rockyou.txt` |
| `9eb7ee7f551d2f0a...` | SHA-256 | `halloween` | `hashcat -m 1400 -a 0 hash2.txt rockyou.txt` |
| `$6$GQXVvW4EuM$...` | SHA-512crypt | `spaceman` | `hashcat -m 1800 -a 0 hash3.txt rockyou.txt` |
| `b6b0d451bbf6fed6...` | MD5 | `funforyou` | `hashcat -m 0 -a 0 hash4.txt rockyou.txt` |

**SHA256 of libgcrypt-1.11.0.tar.bz2:**

```bash
sha256sum ~/Hashing-Basics/Task-7/libgcrypt-1.11.0.tar.bz2
# Answer: 09120c9867ce7f2081d6aaa1775386b98c2f2f246135761aae47d81f58685b9c
```

**Hashcat mode for HMAC-SHA512 (key = $pass):** `1750`

---

## Important Terminology

| Term | Definition |
|---|---|
| Hash function | One-way function producing a fixed-size output from any input |
| Hash value | The fixed-size output of a hash function |
| Preimage resistance | Property making it infeasible to find the input given a hash output |
| Collision | Two different inputs producing the same hash output |
| Salt | A unique random value added to a password before hashing |
| Rainbow table | Precomputed hash-to-plaintext lookup table |
| HMAC | Hash-based Message Authentication Code — combines hashing with a secret key |
| Bcrypt | Password hashing algorithm — intentionally slow, includes built-in salting |
| `/etc/shadow` | Linux file storing hashed passwords (root-readable only) |

---

## Real-World Relevance

- Password databases store hashes, not plaintext — when a database is breached, attackers crack the hashes offline
- MD5 and SHA-1 are broken and should never be used for password storage or security-critical integrity checks
- Bcrypt, Argon2, and Scrypt are designed for password storage — their intentional slowness makes large-scale cracking impractical
- Unsalted hashes are vulnerable to rainbow table attacks — always use salted algorithms
- File integrity verification (comparing SHA-256 hashes) is used to verify downloaded software and detect tampering
- HMAC is used in JWT tokens, API authentication, and TLS — understanding it is relevant for web security testing

---

## Key Learnings

- Hash functions are one-way — you cannot reverse them, only crack them by trying inputs
- MD5 and SHA-1 are broken — use SHA-256 or stronger for integrity; use Bcrypt/Argon2 for passwords
- Salting defeats rainbow tables — every user gets a unique salt
- `/etc/shadow` stores Linux password hashes with the algorithm identified by the prefix
- HMAC provides both authenticity and integrity using a shared secret key
- Hashcat `-m` specifies the hash type; `-a 0` is dictionary attack mode

---

## Conclusion

Hashing is the foundation of password security and data integrity verification. Understanding why MD5 and SHA-1 are broken, how salting defeats rainbow tables, and how to use tools like Hashcat to crack hashes is essential knowledge for both offensive security (credential attacks) and defensive security (implementing secure password storage).
