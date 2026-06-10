# Breaking Crypto the Simple Way

## Overview

Cryptography is only as strong as its implementation. Even mathematically sound algorithms fail when developers use weak keys, predictable randomness, hardcoded secrets, or unauthenticated encryption modes. This room covers the most common cryptographic implementation mistakes — weak key generation, RSA prime vulnerabilities, hashing algorithm selection, client-side key exposure, and unauthenticated encryption — and explains exactly why each one breaks security.

---

## Topics Covered

- Cryptographic key strength: entropy, length, uniqueness
- RSA fundamentals and why prime quality matters
- GCD-based factorisation attacks on shared RSA primes
- Hashing: vulnerabilities, salting, and algorithm selection
- Why SHA-256 is wrong for passwords
- Hardcoded keys in client-side code
- Unauthenticated encryption and bit-flipping attacks

---

## Key Concepts

### What Makes a Key Strong?

Cryptography is predicated on keys being computationally infeasible to guess. A 128-bit key has 2^128 possible combinations — brute-forcing this is impractical on any hardware. But key strength requires more than just length.

| Property | Description |
|----------|-------------|
| **Length** | Longer keys exponentially increase brute-force cost |
| **Entropy** | Keys must be truly random — not derived from timestamps, usernames, or other predictable inputs |
| **Uniqueness** | Keys must not be reused across different encryptions or systems — reuse enables correlation attacks |

Violating any of these three properties makes the key susceptible to brute-force or mathematical attacks.

---

### RSA: Math and Vulnerabilities

RSA is an asymmetric encryption algorithm whose security relies on the difficulty of factoring large numbers.

**Key components:**
- `n = p × q` — the public modulus, the product of two large prime numbers
- `e` — public exponent (typically 65537)
- `φ(n) = (p-1) × (q-1)` — Euler's totient function
- `d` — private exponent: the modular inverse of `e` modulo `φ(n)`, satisfying `e × d ≡ 1 (mod φ(n))`

Security depends on factoring `n` being computationally infeasible. As primes get larger, factorisation time grows exponentially:

```python
import time
from sympy.ntheory import factorint

n_small = 253          # 11 × 23
n_medium = 988027      # 941 × 1051
n_large = 2147483647   # large prime

for n in [n_small, n_medium, n_large]:
    start = time.time()
    factorint(n)
    print(f"Time to factor {n}: {time.time() - start:.6f} seconds")
```

Output:
```
Time to factor 253: 0.000019 seconds
Time to factor 988027: 0.000041 seconds
Time to factor 2147483647: 0.000094 seconds
```

For properly sized RSA keys (2048+ bits), factorisation is infeasible. The security assumption breaks down only when prime generation is flawed.

#### "P's and Q's" — RSA Prime Vulnerabilities

The paper by Ross Anderson and Serge Vaudenay identifies specific RSA weaknesses from poor prime generation:

| Vulnerability | Description |
|---------------|-------------|
| **Predictable primes** | If `p` or `q` is generated from a weak RNG (e.g. seeded with system time), an attacker can reproduce the generation process and derive the primes |
| **Shared primes across keys** | If two different RSA keys share a common prime `p`, an attacker computes `GCD(n1, n2) = p`, then factors both keys trivially |
| **Primes too close together** | If `p` and `q` are close in value, Fermat's factorisation method can quickly recover them |

**GCD attack on shared primes:**

```
n1 = p × q1
n2 = p × q2

GCD(n1, n2) = p
```

Once `p` is known, `q = n/p` and the private key can be fully derived. This attack is polynomial time — completely trivial once the shared prime is found.

---

### Hashing: Vulnerabilities and Algorithm Selection

A hash function transforms input into a fixed-size output. It is one-way — you cannot reverse a hash to recover the input. Hashes are used for password storage, data integrity verification, and message authentication (HMAC).

#### Common Hashing Vulnerabilities

| Vulnerability | Description |
|---------------|-------------|
| **Weak algorithms** | MD5 and SHA-1 are susceptible to collision attacks — two different inputs can produce the same hash |
| **No salting** | Without a unique random value added per input before hashing, attackers can use precomputed rainbow tables to reverse common hashes |
| **Insecure HMACs** | HMACs are only as strong as their underlying hash function and key quality |

#### Why SHA-256 Is Wrong for Passwords

SHA-256 is designed to be fast. For data integrity and digital signatures, speed is good. For password hashing, speed is the attacker's advantage.

| Algorithm | Approximate GPU speed |
|-----------|----------------------|
| MD5 | ~100 billion H/s |
| SHA-256 | ~1 billion H/s |
| bcrypt (cost=12) | ~1,000 H/s |
| Argon2id | ~100 H/s |

At 1 billion SHA-256 hashes per second, an attacker testing a wordlist of 1 million common passwords takes about 1 millisecond. bcrypt at cost=12 limits that to 1,000 seconds. The difference is the deliberate cost factor built into password hashing schemes.

bcrypt, Argon2, and PBKDF2 are specifically designed to be slow and adaptive — their cost parameters can be increased as hardware improves. SHA-256 has no such mechanism.

#### Choosing the Right Hash Function

| Purpose | Recommended Function | Reason |
|---------|---------------------|--------|
| Password storage | Argon2, bcrypt, PBKDF2 | Slow by design, resistant to GPU brute-force |
| Data integrity / checksums | SHA-256, SHA-3, BLAKE2 | Fast and efficient |
| Message authentication | HMAC-SHA256, HMAC-SHA3 | Verifies integrity with a shared secret |

Using SHA-256 for passwords does not provide immediate exposure but makes brute-force dramatically more feasible than using a proper password hashing scheme.

---

### Hardcoded Keys in Client-Side Code

Embedding encryption keys or API credentials in code that executes in the browser (JavaScript) exposes them to anyone who visits the page. Browser DevTools or a simple `curl` request retrieves the entire script — including any embedded secrets.

**Risks:**
- **Unauthorised access** — exposed API keys allow direct backend access without authentication
- **Data tampering** — attacker can use exposed keys to sign or decrypt data
- **Cryptographic bypass** — any encryption relying on a client-side key provides no real protection

**Common scenarios:**
- API keys hardcoded in JavaScript for convenience
- Encryption keys embedded in front-end frameworks for client-side decryption
- Secrets stored in config files bundled with the web application

The correct approach: secrets must stay server-side. Client-side code receives only what is needed for display — never keys, credentials, or secrets.

---

### Unauthenticated Encryption and Bit-Flipping

**Unauthenticated encryption** protects data confidentiality but provides no integrity guarantee. An attacker can modify the ciphertext, and when the application decrypts it, the tampered plaintext is processed as valid.

**AES-CBC without authentication** is the classic example. In CBC mode, each plaintext block is XORed with the previous ciphertext block before encryption. When a ciphertext block is modified, the corresponding plaintext block is altered in a predictable way.

#### How Bit-Flipping Works

Modifying a specific bit in a CBC ciphertext block causes:
1. The corresponding plaintext block to decrypt as garbage (lost)
2. The **next** plaintext block to have a specific, controlled bit changed

Because the XOR relationship is predictable, an attacker who knows the plaintext can calculate exactly which ciphertext bits to flip to produce a desired change in the decrypted output.

**Example:** An attacker flips bits in the ciphertext to change `user=alice` to `user=admin` in the decrypted plaintext — without knowing the encryption key.

**Prevention:** Use authenticated encryption modes that combine confidentiality with an integrity check:
- **AES-GCM** (Galois/Counter Mode) — industry standard
- **AES-CCM** — used in wireless protocols
- **ChaCha20-Poly1305** — fast, modern alternative

Any modification to the ciphertext is detected before decryption proceeds.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Entropy | The amount of unpredictability in a value — higher entropy = harder to guess |
| RSA | Asymmetric encryption algorithm based on the difficulty of factoring large primes |
| GCD | Greatest Common Divisor — used to find shared primes between two RSA keys |
| Fermat's factorisation | Algorithm that efficiently factors `n` when `p` and `q` are close in value |
| Salt | A unique random value added to each password before hashing to prevent rainbow table attacks |
| Rainbow table | Precomputed table of hashes for common inputs — defeated by salting |
| HMAC | Hash-based Message Authentication Code — verifies message integrity using a shared key |
| Unauthenticated encryption | Encryption that provides confidentiality but no integrity check |
| Bit-flipping attack | Modifying ciphertext to predictably alter decrypted plaintext — exploits unauthenticated CBC |
| AES-GCM | Authenticated encryption mode — detects any ciphertext modification |

---

## Practical Examples

### RSA Shared Prime GCD Attack (Python)

```python
from math import gcd

n1 = 3233    # shares prime p=61 with n2
n2 = 4087    # n2 = 61 × 67

p = gcd(n1, n2)
print(f"Shared prime: {p}")       # 61
print(f"q1 = {n1 // p}")          # 53
print(f"q2 = {n2 // p}")          # 67
```

### Bcrypt Password Hashing (Python)

```python
import bcrypt

password = b"hunter2"
hashed = bcrypt.hashpw(password, bcrypt.gensalt(rounds=12))

# Verification
print(bcrypt.checkpw(password, hashed))  # True
```

### Detecting a Hardcoded Key (JavaScript — what not to do)

```javascript
// Never do this — visible to any user
const SECRET_KEY = "my-secret-key-16";
const cipher = CryptoJS.AES.encrypt(data, SECRET_KEY);
```

---

## Real-World Relevance

- The "P's and Q's" shared prime attack was demonstrated on real-world RSA keys in 2012 by Lenstra et al. — scanning millions of public keys found thousands sharing primes, all trivially factored
- SHA-1 collision attacks were demonstrated practically in 2017 (SHAttered) — two different PDF files with identical SHA-1 hashes
- Hardcoded API keys in GitHub repositories are continuously harvested by automated scanners — this is a major real-world credential exposure vector
- AES-CBC without authentication is the mechanism behind BEAST and POODLE attacks against SSL/TLS
- Modern TLS (1.3) exclusively uses authenticated encryption modes (AES-GCM, ChaCha20-Poly1305) — removing CBC from the standard entirely

---

## Key Learnings

- Key strength requires length, true entropy, and uniqueness — predictable or reused keys break security regardless of algorithm
- RSA security fails if `p` and `q` share a prime with another key, are too close together, or are generated from a weak RNG
- MD5 and SHA-1 are broken for integrity — SHA-256 is broken for passwords (too fast)
- Use bcrypt, Argon2, or PBKDF2 for passwords; SHA-256 and BLAKE2 for checksums and data integrity
- Client-side code is public — never embed keys, credentials, or secrets in JavaScript
- AES-CBC without authentication enables bit-flipping — always use authenticated encryption (AES-GCM)

---

## Additional Notes

- Argon2id is the current OWASP recommendation for password hashing — it is memory-hard, making GPU attacks expensive
- Fermat's factorisation is fast when `|p - q|` is small — a well-implemented RSA key generator ensures `p` and `q` are sufficiently far apart
- Salting prevents rainbow table attacks but does not slow down brute force — salt is meaningless without a slow hash function
- The distinction between encoding (no key, reversible) and encryption (key required) is important: Base64 is encoding, not encryption

---

## Conclusion

Breaking crypto rarely requires attacking the algorithm itself. Most real-world cryptographic failures come from implementation mistakes: weak random number generation, shared primes, wrong hash function choices, hardcoded keys, and unauthenticated encryption modes. Understanding each failure mode and its root cause is essential for both implementing cryptography correctly and identifying where it goes wrong during security assessments.
