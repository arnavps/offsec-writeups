# Cryptography Basics

## Overview

Cryptography is the foundation of secure digital communication. It ensures that data cannot be read or altered by unauthorised parties as it travels across networks or sits in storage. This room covers the core concepts — plaintext, ciphertext, keys, symmetric and asymmetric encryption — and the mathematical operations (XOR and modulo) that underpin modern cryptographic algorithms.

---

## Topics Covered

- Why cryptography matters
- Core terminology: plaintext, ciphertext, cipher, key
- Caesar cipher as a teaching example
- Symmetric encryption: DES, 3DES, AES
- Asymmetric encryption: RSA, Diffie-Hellman, ECC
- XOR operation
- Modulo operation

---

## Key Concepts

### Why Cryptography Matters

Without cryptography:
- Credentials sent over the network would be readable by anyone with access to the path
- Files could be modified in transit without detection
- You couldn't verify you're talking to the real server

Cryptography provides **confidentiality** (data can't be read), **integrity** (data can't be altered), and **authenticity** (you're talking to who you think you are).

---

### Core Terminology

| Term | Definition |
|---|---|
| Plaintext | The original, readable data before encryption |
| Ciphertext | The scrambled, unreadable output after encryption |
| Cipher | The algorithm that converts plaintext to ciphertext and back |
| Key | The secret value that controls how the cipher encrypts/decrypts |
| Encryption | Converting plaintext to ciphertext using a cipher and key |
| Decryption | Converting ciphertext back to plaintext using a cipher and key |

```
Encryption:  plaintext + cipher + key  →  ciphertext
Decryption:  ciphertext + cipher + key  →  plaintext
```

Security comes from keeping the **key** secret — not the algorithm. Algorithms are public and well-studied.

---

### Caesar Cipher (Teaching Example)

Shifts each letter by a fixed number of positions. The shift amount is the key.

**Example with key = 3:**
- `HELLO` → `KHOOR`
- H+3=K, E+3=H, L+3=O, L+3=O, O+3=R

**Why it's insecure:** Only 25 possible keys. A computer tries all of them in milliseconds. Never used in real systems — used here to illustrate the key/algorithm concept.

---

### Symmetric Encryption

Uses the **same key** for both encryption and decryption. Both parties must share the secret key securely.

**Characteristics:**
- Fast and efficient — suitable for bulk data
- Key distribution problem: how do you share the key securely?

**Common algorithms:**

| Algorithm | Key Size | Notes |
|---|---|---|
| DES | 56-bit | Broken in 1999 — do not use |
| 3DES | 168-bit (effective 112-bit) | Deprecated 2019 — legacy systems only |
| AES | 128, 192, or 256-bit | Current standard — AES-256 recommended |

---

### Asymmetric Encryption

Uses **two mathematically linked keys**: a public key (shareable) and a private key (secret). Data encrypted with the public key can only be decrypted with the private key.

**Characteristics:**
- Solves the key distribution problem
- Slower than symmetric encryption
- Used for key exchange and digital signatures, not bulk data

**Common algorithms:**

| Algorithm | Key Size | Notes |
|---|---|---|
| RSA | 2048-bit minimum (3072/4096 recommended) | Based on integer factorisation difficulty |
| Diffie-Hellman | 2048-bit minimum | Key exchange protocol |
| ECC | 256-bit ≈ 3072-bit RSA | Equivalent security with smaller keys |

**Hybrid approach (used in HTTPS):**
1. Asymmetric encryption establishes a shared symmetric key
2. Symmetric encryption handles all subsequent data transfer

---

### XOR Operation

XOR (exclusive OR) compares two bits and returns 1 if they differ, 0 if they are the same.

| A | B | A ⊕ B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

**Key properties:**
- `A ⊕ A = 0`
- `A ⊕ 0 = A`
- Commutative: `A ⊕ B = B ⊕ A`
- Associative: `(A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)`

**XOR as symmetric encryption:**
```
C = P ⊕ K          (encrypt)
P = C ⊕ K          (decrypt)
```

XOR is the building block of many stream ciphers and block cipher modes.

---

### Modulo Operation

The modulo operator (`%` or `mod`) returns the remainder of division.

```
25 % 5 = 0    (25 = 5×5 + 0)
23 % 6 = 5    (23 = 3×6 + 5)
23 % 7 = 2    (23 = 3×7 + 2)
```

**Key property:** Modulo is not reversible. Given `x % 5 = 4`, infinite values of x satisfy this. This one-way property is exploited in asymmetric cryptography.

The result of `a % n` is always in the range `0` to `n-1`.

---

## Real-World Relevance

- AES-256 is the standard for encrypting data at rest (full-disk encryption, file encryption)
- RSA and ECC are used in TLS certificates, SSH keys, and code signing
- XOR is used in stream ciphers and as a component in block cipher modes (CBC, CTR)
- Weak or deprecated algorithms (DES, 3DES, RC4, MD5) are common findings in security assessments
- The key distribution problem is why HTTPS uses a hybrid approach — asymmetric for key exchange, symmetric for data

---

## Key Learnings

- Security comes from keeping the key secret, not the algorithm
- Symmetric encryption is fast but requires secure key distribution
- Asymmetric encryption solves key distribution but is slower
- AES is the current symmetric standard; RSA and ECC are the dominant asymmetric algorithms
- XOR and modulo are the mathematical foundations of many cryptographic operations
- DES and 3DES are deprecated — AES should be used instead

---

## Conclusion

Cryptography is the mathematical backbone of digital security. Understanding the difference between symmetric and asymmetric encryption, the role of keys, and the mathematical operations that make cryptography work provides the foundation for understanding TLS, SSH, digital signatures, and every other security protocol built on top of these concepts.
