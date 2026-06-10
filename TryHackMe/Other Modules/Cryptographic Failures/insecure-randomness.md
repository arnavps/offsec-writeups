# Insecure Randomness

## Overview

Randomness underpins almost every security-critical operation in modern applications — key generation, session tokens, password reset links, nonces, and cryptographic operations all depend on values that an attacker cannot predict. When randomness is weak, predictable, or generated incorrectly, these values can be guessed or reproduced — enabling session hijacking, authentication bypass, and token forgery. This room covers the types of random number generators, the concept of entropy, and the specific differences between statistical and cryptographically secure PRNGs, along with best practices for both developers and penetration testers.

---

## Topics Covered

- What randomness means in a security context
- Entropy and why it matters
- True Random Number Generators (TRNG)
- Pseudorandom Number Generators (PRNG)
- Statistical PRNGs vs Cryptographically Secure PRNGs (CSPRNG)
- Seeding: how it creates predictability
- Common attack patterns against weak randomness
- Best practices for pentesters and developers

---

## Key Concepts

### Why Randomness Matters

Cryptographic systems depend on values that attackers cannot predict. These values include:

| Use Case | Example | Risk if Predictable |
|----------|---------|---------------------|
| Cryptographic keys | AES key, RSA prime | Attacker reconstructs the key and decrypts all data |
| Session tokens | PHPSESSID, JWT nonce | Session hijacking |
| Password reset tokens | Magic links, reset URLs | Account takeover |
| Nonces | IV, CSRF tokens | Replay attacks, forgery |
| Unique identifiers | Order IDs, user IDs | Enumeration, correlation |

---

### Entropy

**Entropy** measures the unpredictability of a value. Higher entropy means more randomness and a larger space of possible values — harder to guess or brute-force.

- **High entropy:** A 256-bit value generated from hardware noise has 2^256 possible states — computationally infeasible to guess
- **Low entropy:** A timestamp-seeded value from 2024 has a few billion possible seeds — guessable in seconds with a modern computer

Low entropy is the root cause of most randomness vulnerabilities. Even an algorithm that appears random can be broken if its seed space is small enough to enumerate.

---

### True Random Number Generators (TRNG)

TRNGs derive randomness from physical, unpredictable phenomena:
- Thermal noise in electronic circuits
- Radioactive decay timing
- Photon arrival timing in optical sensors

**Properties:**
- Genuinely non-deterministic — the same physical process never produces the same sequence twice
- Not reproducible by an attacker who lacks access to the physical source
- Typically slower than algorithmic generators — require specialised hardware or OS entropy pools
- Used for the most security-critical operations: RSA/ECC key generation, master key seeding

**Examples:** Intel RDRAND instruction, `/dev/random` (Linux hardware entropy), hardware security modules (HSMs)

---

### Pseudorandom Number Generators (PRNG)

PRNGs generate sequences that **appear** random but are entirely deterministic. The same seed always produces the same sequence. They are fast and efficient — suitable for high-volume random number needs.

**Critical property:** If an attacker knows or can guess the seed, they can reproduce the entire output sequence.

#### Statistical PRNG

Designed to produce numbers that pass statistical randomness tests — no obvious patterns, good distribution. **Not** designed for security.

- Used in: simulations, gaming, statistical sampling, non-security testing
- Examples: `rand()` in C, `mt_rand()` in PHP (Mersenne Twister), `random.random()` in Python

**Security risk:** Predictable if the seed can be guessed (e.g. seeded with a timestamp). Cannot be used for tokens, keys, or session IDs.

#### Cryptographically Secure PRNG (CSPRNG)

A PRNG specifically designed for security-sensitive applications. Must satisfy stronger requirements:
- **Computationally infeasible to reverse:** Even knowing portions of the output, the seed and future values cannot be deduced
- **Forward secrecy:** Compromise of the current state does not reveal past outputs
- **Backtracking resistance:** Compromise does not allow prediction of future outputs

- Used in: session tokens, password reset links, cryptographic key generation, nonces
- Examples: `random_bytes()` in PHP, `java.security.SecureRandom` in Java, `os.urandom()` in Python, `/dev/urandom` on Linux

**CSPRNGs are the only acceptable choice** for any security-sensitive random value generation.

---

### Seeding

A seed is the initial value given to a PRNG. The seed determines the entire output sequence. The same seed always produces the same sequence.

**Predictable seeds create predictable output:**
- System timestamp at startup → attacker knows the approximate time → brute-forces the seed space
- User ID as seed → direct correlation to a specific user
- Process ID as seed → limited to a few thousand possibilities

**Secure seeding:** The seed must come from a high-entropy source — typically the OS entropy pool (`/dev/urandom`, `CryptGenRandom` on Windows) which gathers entropy from hardware events, network timing, and other unpredictable sources.

---

### Common Attack Patterns

| Attack | Description |
|--------|-------------|
| **Seed bruteforce** | If the seed space is small (e.g. millisecond timestamps within a 60-second window), enumerate all possible seeds and reproduce token values |
| **PRNG state prediction** | For Mersenne Twister (mt_rand), 624 consecutive 32-bit outputs are sufficient to fully reconstruct the internal state and predict all future values |
| **Token exhaustion** | For short tokens generated with low entropy, enumerate all possible values and replay them against the target endpoint |
| **Magic link forgery** | Password reset or login links with predictable token generation allow an attacker to forge valid links for arbitrary accounts |

**Tool:** `php_mt_seed` — reconstructs the Mersenne Twister seed from a known output value, enabling full state recovery and prediction of past and future outputs.

---

## Practical Examples

### Weak PRNG Token Generation (PHP — insecure)

```php
// DO NOT USE: timestamp seed is predictable
srand(time());
$token = rand(100000, 999999);
// Attacker knows approximate time → brute-forces seed → recovers token
```

### Secure Token Generation (PHP — correct)

```php
// CSPRNG: output is computationally unpredictable
$token = bin2hex(random_bytes(32));  // 256 bits of entropy
```

### Weak Token in Python (insecure)

```python
import random
import time

random.seed(int(time.time()))  # timestamp seed — low entropy
token = random.randint(0, 2**32)
```

### Secure Token in Python (correct)

```python
import os
import secrets

token = secrets.token_hex(32)  # uses os.urandom() — CSPRNG
```

### Java Secure Random

```java
// Insecure — java.util.Random is a statistical PRNG
Random rng = new Random();
int token = rng.nextInt();

// Secure — java.security.SecureRandom is a CSPRNG
SecureRandom csprng = new SecureRandom();
byte[] token = new byte[32];
csprng.nextBytes(token);
```

---

## Best Practices

### For Developers

| Practice | Description |
|----------|-------------|
| Always use CSPRNG for security-sensitive values | `random_bytes()`, `SecureRandom`, `os.urandom()`, `secrets` module |
| Never use predictable seed values | No timestamps, user IDs, process IDs, or IP addresses as seeds |
| Regenerate tokens per operation | Do not reuse tokens across users or requests |
| Use strong key generation functions | PHP: `openssl_pkey_new()`, Java: `KeyPairGenerator` with `SecureRandom` |
| Minimum token length | 128 bits (16 bytes) minimum for session tokens; 256 bits for cryptographic material |

### For Penetration Testers

| Practice | Description |
|----------|-------------|
| Identify weak PRNG usage in code | Look for `mt_rand()`, `rand()`, `random()` generating security-sensitive values |
| Test for seed predictability | Collect multiple tokens and check for sequential or timestamp-derived patterns |
| Use php_mt_seed | Reconstruct PHP Mersenne Twister seeds from a known token value |
| Test token exhaustion | For short tokens (< 64 bits), attempt brute-force against reset link or session endpoints |
| Analyse token generation timing | Tokens generated at known times have predictable timestamp-derived seeds |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| TRNG | True Random Number Generator — uses physical unpredictable phenomena |
| PRNG | Pseudorandom Number Generator — algorithmic, deterministic, seed-dependent |
| CSPRNG | Cryptographically Secure PRNG — designed for security, computationally unpredictable |
| Statistical PRNG | PRNG designed to pass statistical tests only — not suitable for security |
| Seed | Initial value given to a PRNG that determines its entire output sequence |
| Entropy | Measure of unpredictability in a value — higher entropy = more secure |
| Mersenne Twister | The `mt_rand()` PRNG in PHP — predictable from 624 consecutive outputs |
| php_mt_seed | Tool for recovering Mersenne Twister seeds from known PHP PRNG outputs |
| Nonce | Number used once — must be unpredictable to prevent replay attacks |
| Session token | Credential issued after authentication — must be generated with CSPRNG |

---

## Real-World Relevance

- In 2008, a Debian OpenSSL bug seeded the PRNG with only the process ID (PID) — a maximum of 32,768 possible seeds — allowing all SSL keys generated over two years to be reproduced and compromised
- PHP's `mt_rand()` (Mersenne Twister) is used in many frameworks and CMS platforms for token generation — attackers with access to any predictable output can reconstruct the state and forge tokens
- Multiple e-commerce platforms have been found generating order IDs and discount codes using `rand()` or timestamp-based seeds, allowing sequential enumeration
- Password reset link vulnerabilities are consistently found in web application assessments — timestamp-seeded or short tokens are trivially brute-forced
- Bug bounty programmes frequently reward findings related to predictable session tokens, weak password reset mechanisms, and CSRF token predictability

---

## Key Learnings

- TRNGs use physical randomness — non-reproducible, high entropy, used for key generation
- Statistical PRNGs are deterministic and predictable from the seed — never use for security
- CSPRNGs are the only acceptable choice for session tokens, keys, reset links, and nonces
- Predictable seeds (timestamps, PIDs, user IDs) collapse the entropy of any PRNG to a small, enumerable space
- The Mersenne Twister (mt_rand, Python's random) can be fully reconstructed from 624 consecutive outputs
- Minimum security: 128 bits from a CSPRNG for tokens; 256 bits for cryptographic material

---

## Additional Notes

- `secrets` (Python 3.6+) and `random_bytes()` (PHP 7+) are drop-in CSPRNG functions specifically added to discourage misuse of statistical PRNGs
- `/dev/urandom` on Linux is the OS CSPRNG — it is non-blocking and suitable for all security-sensitive random number generation; `/dev/random` is blocking and rarely necessary
- Forward secrecy in CSPRNGs means even if an attacker recovers the current state, they cannot deduce past outputs — important for session token security
- Unique identifiers (UUIDs) are often mistaken for secure tokens — UUID v4 has only 122 bits of randomness and is derived from PRNG in many implementations; always verify the underlying generator

---

## Conclusion

Insecure randomness is one of the most consistently overlooked vulnerabilities in web application development. The distinction between a statistical PRNG and a CSPRNG is often poorly understood — developers assume any "random" function is secure when in fact most are deterministic and predictable from their seed. The correct approach is simple: always use the platform's designated CSPRNG for any security-sensitive value generation, never seed from predictable inputs, and ensure sufficient bit length. The consequences of getting this wrong — session hijacking, authentication bypass, forged tokens — justify treating randomness as a fundamental security control rather than an implementation detail.
