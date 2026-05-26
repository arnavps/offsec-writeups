# Cryptography Concepts

## Overview

Cryptography is the practical implementation of confidentiality and integrity — two pillars of the CIA Triad. It uses mathematical rules and secret keys to protect data in transit and at rest. Without cryptography, every password, banking transaction, and private message sent over the internet would be readable by anyone with access to the network. This room covers the core concepts: plaintext vs ciphertext, symmetric vs asymmetric encryption, and how they combine in real-world protocols like HTTPS.

---

## Topics Covered

- Plaintext, ciphertext, keys, and algorithms
- The Caesar cipher as a teaching example
- Symmetric encryption
- Asymmetric encryption
- How HTTPS combines both
- Symmetric vs asymmetric comparison

---

## Key Concepts

### Core Definitions

| Term | Definition |
|---|---|
| Plaintext | Readable data before encryption. Example: `HELLO` |
| Ciphertext | Scrambled, unreadable output after encryption. Example: `KHOOR` |
| Key | The secret value that controls how encryption and decryption work |
| Algorithm | The public set of steps that defines how the key is applied to the data |

The security of a cryptographic system comes from keeping the **key** secret — not the algorithm. Algorithms are public and well-studied.

```
Encryption:  plaintext + algorithm + key  →  ciphertext
Decryption:  ciphertext + algorithm + key  →  plaintext
```

---

### The Caesar Cipher

The Caesar cipher is a simple substitution cipher used to illustrate how keys and algorithms work together. It is not secure and is never used in real systems — it exists here purely as a teaching tool.

**How it works:** Each letter in the plaintext is shifted forward by a fixed number of positions in the alphabet. That number is the key.

**Example with key = 3:**

| Plaintext | Shift | Ciphertext |
|---|---|---|
| H | +3 | K |
| E | +3 | H |
| L | +3 | O |
| L | +3 | O |
| O | +3 | R |

`HELLO` → `KHOOR`

To decrypt, Bob shifts each letter backwards by 3: `KHOOR` → `HELLO`

**Why it's insecure:** There are only 25 possible keys (shifts 1–25). A computer can try all of them in milliseconds. Real algorithms like AES use key spaces so large that brute force is computationally infeasible.

---

### Symmetric Encryption

Symmetric encryption uses a **single key** for both encryption and decryption. Both parties must have a copy of the same secret key.

**Characteristics:**
- Fast and efficient — suitable for encrypting large amounts of data
- Used for encrypting files, hard drives, and bulk network traffic
- The key distribution problem: how do two parties securely share the key in the first place?

**Real-world examples:** AES (Advanced Encryption Standard), ChaCha20

**The key distribution problem:**
If Alice and Bob want to communicate using symmetric encryption, they need to share the key. But if they send it over an unencrypted channel, an attacker can intercept it. This is the fundamental limitation of symmetric encryption — and the reason asymmetric encryption exists.

---

### Asymmetric Encryption

Asymmetric encryption uses **two mathematically linked keys**:

- **Public key** — can be shared openly with anyone
- **Private key** — kept secret by the owner; never shared

**How it works:**
- Data encrypted with the public key can only be decrypted with the corresponding private key
- Data encrypted with the private key can be decrypted by anyone with the public key (used for digital signatures)

The two keys are linked by complex mathematics. Recovering the private key from the public key would take an ordinary computer hundreds to thousands of years — this computational difficulty is the security foundation.

**Real-world examples:** RSA, Elliptic Curve Cryptography (ECC)

**Analogy:** A mailbox — anyone can drop a letter in (encrypt with public key), but only the owner with the key can open it and read the contents (decrypt with private key).

---

### How HTTPS Combines Both

HTTPS uses a **hybrid approach** — asymmetric encryption to solve the key distribution problem, then symmetric encryption for the actual data transfer.

**The process when you visit `https://google.com`:**

1. Your browser requests the server's public key
2. The server sends its public key wrapped in a certificate
3. Browser and server use asymmetric encryption to securely agree on a shared symmetric key (session key)
4. All subsequent communication is encrypted using that symmetric session key

**Why the hybrid approach?**
- Asymmetric encryption solves key distribution but is slow
- Symmetric encryption is fast but requires a shared key
- Combining them gives you the security of asymmetric with the speed of symmetric

---

### Symmetric vs Asymmetric: Comparison

| Feature | Symmetric | Asymmetric |
|---|---|---|
| Number of keys | One (shared) | Two (public + private) |
| Key sharing | Both parties need the same secret key | Public key can be shared openly |
| Speed | Very fast | Slower |
| Primary use | Bulk data encryption (files, traffic) | Key exchange, digital certificates, signatures |
| Analogy | One key locks and unlocks a box | A mailbox: anyone posts, only the owner retrieves |
| Examples | AES, ChaCha20 | RSA, ECC |

---

## Important Terminology

| Term | Definition |
|---|---|
| Plaintext | Readable data before encryption |
| Ciphertext | Encrypted, unreadable data |
| Key | The secret value controlling encryption and decryption |
| Algorithm | The public method defining how the key is applied |
| Symmetric Encryption | Encryption using a single shared key |
| Asymmetric Encryption | Encryption using a public/private key pair |
| Public Key | The shareable half of an asymmetric key pair |
| Private Key | The secret half of an asymmetric key pair; never shared |
| HTTPS | HTTP with TLS encryption — uses a hybrid of asymmetric and symmetric encryption |
| TLS | Transport Layer Security — the protocol that secures HTTPS connections |
| Caesar Cipher | A simple substitution cipher that shifts letters by a fixed number |
| Key Distribution Problem | The challenge of securely sharing a symmetric key over an untrusted channel |
| Hybrid Encryption | Using asymmetric encryption to exchange a symmetric key, then symmetric for data |

---

## Workflow / Process

### HTTPS Handshake (Simplified)

```
Browser  →  [Request public key]  →  Server
Browser  ←  [Public key + certificate]  ←  Server
Browser  →  [Session key encrypted with server's public key]  →  Server
Server decrypts session key with private key
Both parties now share the session key
Browser  ↔  [All data encrypted with symmetric session key]  ↔  Server
```

---

## Real-World Relevance

- TLS/HTTPS is the most visible application of cryptography — the padlock in your browser represents this entire process
- Weak or outdated encryption (e.g., DES, RC4, MD5) is a common finding in security assessments
- Improper key management (hardcoded keys, weak key generation) is a frequent vulnerability in applications
- Certificate validation failures enable man-in-the-middle attacks — attackers present a fake certificate to intercept HTTPS traffic
- Understanding encryption is required for analysing network captures, assessing API security, and understanding token-based authentication (JWT, OAuth)
- Ransomware uses strong symmetric encryption (AES) to lock victim files — the attacker holds the decryption key

---

## Key Learnings

- Cryptography protects confidentiality (encryption) and integrity (hashing/signatures)
- Security comes from keeping the key secret, not the algorithm
- Symmetric encryption is fast but requires secure key distribution
- Asymmetric encryption solves key distribution using a public/private key pair
- HTTPS uses a hybrid approach: asymmetric for key exchange, symmetric for data
- The Caesar cipher illustrates the concept of keys and algorithms but is trivially broken

---

## Additional Notes

- AES-256 is the current gold standard for symmetric encryption
- RSA and ECC are the dominant asymmetric algorithms — ECC provides equivalent security with smaller key sizes
- Hashing (SHA-256, SHA-3) is related to cryptography but is one-way — used for integrity verification, not encryption
- Perfect Forward Secrecy (PFS) ensures that even if a private key is compromised, past session keys cannot be recovered

---

## Conclusion

Cryptography is the mathematical backbone of secure communication. Understanding the difference between symmetric and asymmetric encryption — and how they combine in protocols like HTTPS — is essential for both offensive and defensive security work. It's not magic; it's applied mathematics, and knowing where it can fail is just as important as knowing how it works.
