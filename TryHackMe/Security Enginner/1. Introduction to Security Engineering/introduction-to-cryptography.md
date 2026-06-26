# Introduction to Cryptography

## Overview

Cryptography is the science of securing communication so that only intended parties can read it. This room covers the foundational concepts every security engineer must understand: symmetric and asymmetric encryption, hashing, HMAC, Diffie-Hellman key exchange, PKI, and practical password security. These are not abstract maths problems — they underpin TLS, SSH, VPNs, code signing, password storage, and virtually every other secure protocol in use today.

**Why it matters:** You cannot audit, design, or break secure systems without understanding the cryptographic primitives they are built on. Every TLS handshake, every password hash, every digital signature relies on these concepts.

**Where these concepts apply:** TLS/HTTPS, SSH, VPNs, code signing, certificate authorities, password databases, HMAC-based API authentication, JWT signing, disk encryption.

---

## Core Concepts

---

### Concept 1 — Foundational Terminology

| Term | Definition |
|------|-----------|
| **Plaintext** | The original, readable message before encryption |
| **Ciphertext** | The encrypted output — unreadable without the key |
| **Cipher** | The algorithm that defines the encryption and decryption processes |
| **Key** | The secret value used by the cipher to transform plaintext to ciphertext and back |
| **Keyspace** | The total number of valid keys for a given cipher |
| **Encryption** | The process of converting plaintext to ciphertext using a cipher and key |
| **Decryption** | The reverse — converting ciphertext back to plaintext using the cipher and key |

---

### Concept 2 — Classical Ciphers (Historical Context)

#### Caesar Cipher (Substitution)

Shifts every letter in the plaintext by a fixed number of positions in the alphabet.

- Key range: 1–25 (keyspace = 25)
- Key = 3 right shift: A→D, B→E, T→W
- Trivially broken by brute force — only 25 possible keys
- "TRY HACK ME" with key 3 → "WUB KDFN PH"

#### Transposition Cipher

Rearranges the order of letters rather than substituting them.

- The message is written column by column, then columns are reordered by the key, then read row by row
- Does not change the letters — only their positions

#### Mono-Alphabetic Substitution Cipher

Maps each letter to a different letter — a 26-character key string.

- Keyspace = 26! ≈ 4 × 10²⁶ — brute force is infeasible
- **Weakness:** Letter frequency analysis — in English, 'e' appears 13%, 't' 9.1%, 'a' 8.2% of the time. Analysing frequency in the ciphertext reveals the mapping. Tools like quipqiup can break it automatically.

#### Security Requirement for Modern Cryptography

For an encryption algorithm to be considered secure:
- It must be computationally infeasible to recover the plaintext without the key
- "Infeasible" means not solvable in polynomial time for any practical input size
- Breaking an encrypted message should take millions of years, not days or weeks

---

### Concept 3 — Symmetric Encryption

#### Definition

A symmetric encryption algorithm uses the **same key** for both encryption and decryption. Both parties must agree on and securely share this key before communicating.

#### AES — Advanced Encryption Standard

Published by NIST in 2001. Currently the dominant symmetric encryption standard.

| Property | Value |
|----------|-------|
| Key sizes | 128, 192, or 256 bits |
| Block size | 128 bits (16 bytes) |
| Status | Still considered secure; widely used |

**AES Four Transformations (per round):**

| Step | Operation |
|------|-----------|
| **SubBytes** | Each byte is substituted using a lookup table (S-box) |
| **ShiftRows** | Rows of the 4×4 state array are shifted left by 0,1,2,3 positions |
| **MixColumns** | Each column is multiplied by a fixed matrix — diffuses data across columns |
| **AddRoundKey** | Round key XORed into the state |

Number of rounds: 10 (128-bit key), 12 (192-bit), 14 (256-bit).

#### Block Ciphers vs Stream Ciphers

| Type | How It Works | Examples |
|------|-------------|---------|
| **Block cipher** | Encrypts fixed-size blocks (typically 128 bits) at a time | AES, 3DES, Blowfish, Camellia |
| **Stream cipher** | Encrypts one byte at a time using a keystream | RC4 (deprecated), ChaCha20 |

#### Commonly Used Symmetric Algorithms

| Algorithm | Notes |
|-----------|-------|
| AES-128, AES-192, AES-256 | Current standard; all key sizes considered secure |
| 3DES (Triple DES) | Deprecated 2023; disallowed 2024 — use AES instead |
| IDEA | International Data Encryption Algorithm |
| Blowfish | Designed by Bruce Schneier |
| Twofish | Successor to Blowfish; also Schneier |
| Camellia | Designed by Mitsubishi & NTT |

#### What Symmetric Encryption Provides

| Property | How |
|----------|-----|
| **Confidentiality** | Intercepted ciphertext is unreadable without the key |
| **Integrity** | Any modification to ciphertext prevents correct decryption (or produces gibberish) |
| **Authenticity** | Successful decryption proves the sender knew the shared key |

#### The Key Scaling Problem

Symmetric encryption requires every pair of communicating parties to share a unique key:

```
2 users  →  1 key
3 users  →  3 keys
100 users → 4,950 keys  (formula: n(n-1)/2)
```

Managing 4,950 separate secret keys across 100 users is operationally infeasible. This is the primary motivation for asymmetric encryption.

---

### Concept 4 — Practical Symmetric Encryption Tools

#### GPG (GNU Privacy Guard)

```bash
# Encrypt a file symmetrically
gpg --symmetric --cipher-algo AES256 message.txt
# Output: message.txt.gpg (binary OpenPGP format)

# Encrypt to ASCII text (readable in any text editor)
gpg --armor --symmetric --cipher-algo AES256 message.txt

# Decrypt
gpg --output original_message.txt --decrypt message.txt.gpg
```

#### OpenSSL

```bash
# Encrypt with AES-256-CBC
openssl aes-256-cbc -e -in message.txt -out encrypted_message

# Decrypt
openssl aes-256-cbc -d -in encrypted_message -out original_message.txt

# With PBKDF2 key derivation (more secure against brute force)
openssl aes-256-cbc -pbkdf2 -iter 10000 -e -in message.txt -out encrypted_message

# Decrypt with PBKDF2
openssl aes-256-cbc -pbkdf2 -iter 10000 -d -in encrypted_message -out original_message.txt
```

**PBKDF2 (Password-Based Key Derivation Function 2):** Applies the hash function many times to derive the encryption key from a password. This makes brute-force attacks exponentially slower — an attacker must repeat 10,000 iterations per password guess instead of one.


---

### Concept 5 — Asymmetric Encryption

#### Definition

Asymmetric encryption uses a **mathematically linked key pair**: a public key and a private key. Data encrypted with one key can only be decrypted with the other. The public key is freely shared; the private key is kept secret and never transmitted.

#### How the Key Pair Works

```
Encrypt with PUBLIC key  →  only PRIVATE key can decrypt  (confidentiality)
Encrypt with PRIVATE key →  only PUBLIC key can decrypt   (authenticity / non-repudiation)
```

#### Security Properties Provided

| Property | Mechanism |
|----------|----------|
| **Confidentiality** | Alice encrypts with Bob's public key → only Bob's private key decrypts it |
| **Integrity** | Any tampering with the ciphertext causes decryption to fail or produce garbage |
| **Authenticity** | Bob encrypts (signs) with his private key → anyone can verify with his public key that it came from Bob |
| **Non-repudiation** | Bob cannot deny sending a message signed with his private key |

#### Solving the Key Scaling Problem

Symmetric encryption: 100 users need **4,950 unique keys**
Asymmetric encryption: 100 users only need **100 key pairs** (one per person)

Each user publishes their public key. Anyone who wants to send a message to that user encrypts it with the recipient's public key. No prior shared secret needed.

#### Limitation

Asymmetric encryption is **significantly slower** than symmetric encryption for bulk data. In practice, asymmetric encryption is used to securely exchange a symmetric key, then symmetric encryption handles the actual data. This hybrid approach underpins TLS/HTTPS.

---

### Concept 6 — RSA

#### Definition

RSA (Rivest-Shamir-Adleman) is the most widely used asymmetric encryption algorithm. Its security relies on the mathematical difficulty of factoring large numbers — multiplying two large primes is easy; finding the original primes from the product is computationally infeasible at sufficient key sizes.

#### How RSA Works (Mathematical Overview)

1. Choose two large random primes **p** and **q**
2. Calculate **N = p × q** (the modulus)
3. Calculate **ϕ(N) = N − p − q + 1**
4. Choose **e** and **d** such that **e × d = 1 mod ϕ(N)**
5. **Public key** = (N, e); **Private key** = (N, d)
6. Encrypt: **y = x^e mod N**
7. Decrypt: **x = y^d mod N**

**Security basis:** Given N, finding p and q is computationally infeasible when p and q are large (1024+ bits each). This is the integer factorisation problem.

#### RSA Key Generation with OpenSSL

```bash
# Generate a 2048-bit RSA private key
openssl genrsa -out private-key.pem 2048

# Extract the public key from the private key
openssl rsa -in private-key.pem -pubout -out public-key.pem

# View RSA parameters (p, q, N, e, d as prime1, prime2, modulus, publicExponent, privateExponent)
openssl rsa -in private-key.pem -text -noout

# Encrypt a file with the recipient's public key
openssl pkeyutl -encrypt -in plaintext.txt -out ciphertext -inkey public-key.pem -pubin

# Decrypt with the private key
openssl pkeyutl -decrypt -in ciphertext -inkey private-key.pem -out decrypted.txt
```

#### Practical RSA Key Sizes

| Key Size | Security Status |
|----------|----------------|
| 512 bits | Broken — avoid |
| 1024 bits | Weak — avoid |
| 2048 bits | Current minimum — acceptable |
| 4096 bits | Strong — recommended for long-term use |

---

### Concept 7 — Diffie-Hellman Key Exchange

#### Definition

Diffie-Hellman (DH) is an asymmetric protocol that allows two parties to establish a shared secret key over an **insecure, public channel** without ever transmitting the secret itself. Eavesdroppers who observe all transmitted values cannot derive the shared secret.

#### How It Works

Both parties agree publicly on two values: a large prime **q** and a generator **g** (smaller than q).

```
Alice                               Bob
─────                               ───
Choose secret a                     Choose secret b
A = g^a mod q  ──── sends A ────>
                <──── sends B ────  B = g^b mod q

Shared key = B^a mod q              Shared key = A^b mod q
           = (g^b)^a mod q                     = (g^a)^b mod q
           = g^(ab) mod q           =           g^(ab) mod q
           ✓ SAME VALUE             ✓ SAME VALUE
```

An eavesdropper sees q, g, A, and B — but cannot derive a, b, or the shared key without solving the discrete logarithm problem, which is computationally infeasible for large q (256+ bits in practice).

#### Generate DH Parameters with OpenSSL

```bash
# Generate Diffie-Hellman parameters (2048-bit)
openssl dhparam -out dhparams.pem 2048

# View the prime P and generator G
openssl dhparam -in dhparams.pem -text -noout
```

#### The MITM Vulnerability

DH alone does **not** authenticate the parties — it only establishes a secret. A Man-in-the-Middle (MITM) attacker (Mallory) can:

1. Intercept Alice's A value — replace it with her own M value sent to Bob
2. Intercept Bob's B value — replace it with her own M value sent to Alice
3. Establish separate shared secrets with Alice and with Bob
4. Decrypt, read, modify, and re-encrypt all traffic in both directions

Alice and Bob believe they are talking to each other but are both talking to Mallory.

**Solution:** PKI and digital certificates — authenticate the public key so you know it actually belongs to the intended party.

---

### Concept 8 — Cryptographic Hash Functions

#### Definition

A hash function takes input of **arbitrary size** and produces a fixed-size output called a **message digest** or **checksum**. It is a one-way function — you cannot reverse the output to recover the input.

#### Properties of a Secure Hash Function

| Property | Description |
|----------|-------------|
| **Deterministic** | Same input always produces the same output |
| **One-way (pre-image resistant)** | Cannot compute input from output |
| **Avalanche effect** | A single bit change in input produces a completely different output |
| **Collision resistant** | Computationally infeasible to find two different inputs with the same hash |
| **Fixed output size** | Output length is constant regardless of input size |

#### Common Hash Algorithms

| Algorithm | Output Size | Status |
|-----------|-------------|--------|
| **SHA-256** | 256 bits (64 hex chars) | Secure — widely used |
| **SHA-384** | 384 bits | Secure |
| **SHA-512** | 512 bits | Secure |
| **SHA-224** | 224 bits | Secure |
| **RIPEMD-160** | 160 bits | Secure |
| **MD5** | 128 bits | **Broken** — collision attacks possible |
| **SHA-1** | 160 bits | **Broken** — collision attacks possible |

#### Use Cases

**Password storage:**
```
Store: hash(password) in database instead of plaintext password
On login: hash(submitted_password) — compare to stored hash
```

**File integrity verification:**
```bash
sha256sum file.txt          # Generate checksum
sha256sum -c checksums.txt  # Verify against stored checksum
```

Even a single bit flip in a file produces a completely different hash — tampering is immediately detectable.

**Digital signatures:** Hash the message first, then sign the hash with the private key (signing the full message directly would be too slow with asymmetric algorithms).

---

### Concept 9 — HMAC (Hash-Based Message Authentication Code)

#### Definition

HMAC combines a cryptographic hash function with a **secret key** to produce a Message Authentication Code (MAC). Unlike a plain hash, HMAC cannot be computed by anyone who doesn't know the secret key — providing both integrity and authenticity.

#### How HMAC Works

```
HMAC(K, message) = H( (K ⊕ opad) || H( (K ⊕ ipad) || message ) )

Where:
  H    = hash function (e.g., SHA-256)
  K    = secret key (zero-padded to block size B)
  ipad = 0x36 repeated B times
  opad = 0x5C repeated B times
  ⊕    = XOR
  ||   = concatenation
```

#### Using HMAC on Linux

```bash
# Using hmac256
hmac256 s3cr3tkey message.txt

# Using sha256hmac
sha256hmac message.txt --key s3cr3tkey
```

#### HMAC vs Plain Hash

| | Plain Hash | HMAC |
|-|-----------|------|
| Key required | No | Yes |
| Provides integrity | Yes | Yes |
| Provides authenticity | No | Yes |
| Can be forged without key | Yes (anyone can hash) | No |
| Common use | File checksums, password storage | API authentication, message authentication |

**Real-world HMAC uses:** AWS Signature Version 4 (API request signing), JWT HS256 signing, TLS record MAC, IPsec authentication.

---

### Concept 10 — PKI (Public Key Infrastructure)

#### Definition

PKI is the system of policies, processes, and technologies that manages the creation, distribution, validation, and revocation of digital certificates. PKI solves the MITM problem in key exchange by providing a trusted mechanism to verify that a public key genuinely belongs to the claimed entity.

#### The Problem PKI Solves

Without PKI, when you connect to `bank.com` and receive a public key, you have no way to verify that key actually belongs to `bank.com` and not an attacker. PKI introduces a trusted third party — a **Certificate Authority (CA)** — whose job is to verify identities and sign certificates.

#### Certificate Chain of Trust

```
Root CA (self-signed, trusted by OS/browser by default)
    |
    └── Intermediate CA (signed by Root CA)
              |
              └── End-entity Certificate (signed by Intermediate CA)
                  e.g., bank.com's TLS certificate
```

Your browser trusts the Root CA. It verifies the Intermediate CA's signature using the Root CA's public key. It verifies bank.com's certificate using the Intermediate CA's public key. If all signatures are valid → the chain is trusted.

#### How TLS Uses PKI

1. Browser connects to `https://bank.com`
2. Server presents its TLS certificate (signed by a trusted CA)
3. Browser verifies the certificate chain up to a trusted Root CA
4. Identity confirmed → proceed with TLS handshake
5. TLS handshake: agree on cipher suite, use asymmetric crypto to exchange a symmetric session key
6. All subsequent traffic encrypted with the symmetric session key (fast)

#### Generating a CSR and Self-Signed Certificate

```bash
# Generate a Certificate Signing Request (CSR)
openssl req -new -nodes -newkey rsa:4096 -keyout key.pem -out cert.csr
# Options:
#  req -new         = create new CSR
#  -nodes           = save private key without passphrase
#  -newkey rsa:4096 = generate new 4096-bit RSA key
#  -keyout key.pem  = save private key here
#  -out cert.csr    = save CSR here

# Generate a self-signed certificate (for testing only)
openssl req -x509 -newkey -nodes rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365
# -x509 = self-signed (not a CSR)
# -sha256 = sign with SHA-256
# -days 365 = valid for 1 year

# View a certificate
openssl x509 -in cert.pem -text
```

**Self-signed certificates** are not trusted by browsers — they generate security warnings. They are only appropriate for testing and internal use where you manually add the certificate to trust stores.

---

### Concept 11 — Password Security and Hashing

#### Why Not Store Passwords in Plaintext

Any database breach immediately exposes all user passwords. Attackers can then use those passwords on other services (credential stuffing — most users reuse passwords).

#### Evolution of Password Storage

**Level 1 — Plaintext (never acceptable):**
```
username | password
alice    | qwerty
bob      | dragon
```

**Level 2 — Hashed (better, but vulnerable to rainbow tables):**
```
username | hash(password)
alice    | d8578edf8458ce06fbc5bb76a58c5ca4   ← MD5 of "qwerty"
bob      | 8621ffdbc5698829397d97767ac13db3
```
A **rainbow table** is a precomputed lookup of hash → password. The attacker looks up the hash and immediately recovers the password.

**Level 3 — Salted hash (correct approach):**
```
username | hash(password + salt) | salt
alice    | 8a43db01d06107fcad... | 12742
bob      | aab2b680e6a1cb43c7... | 22861
```
A **salt** is a random value appended to the password before hashing. Each user has a unique salt stored alongside their hash. Rainbow tables are useless — the attacker would need a separate table for every possible salt value.

**Level 4 — Key derivation functions (best practice):**

Use **PBKDF2**, **bcrypt**, or **Argon2** instead of a raw hash:
- These functions are intentionally slow — they apply the hash thousands or millions of times
- Makes brute-force attacks exponentially slower
- bcrypt and Argon2 are self-salting — they generate and store the salt automatically

```bash
# OpenSSL PBKDF2 example (used in file encryption)
openssl aes-256-cbc -pbkdf2 -iter 100000 -e -in plaintext.txt -out encrypted.bin
```

#### Password Hashing Best Practices Summary

| Practice | Reason |
|----------|--------|
| Use bcrypt, Argon2, or PBKDF2 | Intentionally slow; resists brute force |
| Always use unique per-user salts | Defeats rainbow tables |
| Never store plaintext passwords | Any breach immediately exposes all passwords |
| Never use MD5 or SHA-1 for passwords | Fast and broken — trivially cracked |
| Iteration count should be tunable | Increase as hardware gets faster |

---

## Architecture and Relationships

### How Cryptographic Primitives Combine in TLS

```
TLS Handshake:
  1. Client → Server: ClientHello (supported ciphers)
  2. Server → Client: Certificate (RSA/ECDSA public key, signed by CA)
  3. Client: Verify certificate chain against trusted Root CAs
  4. Client → Server: Pre-master secret (encrypted with server's public key)
     OR Diffie-Hellman key exchange (authenticated by certificate signature)
  5. Both derive: session keys (symmetric AES key + HMAC key)

Data Transfer:
  6. All data encrypted with AES (symmetric — fast)
  7. Each record includes HMAC (integrity + authenticity)
```

### Symmetric vs Asymmetric — When to Use Each

| Use Case | Algorithm Type | Why |
|----------|---------------|-----|
| Bulk data encryption (files, disk, traffic) | Symmetric (AES) | Fast; efficient for large data |
| Key exchange | Asymmetric (RSA, DH, ECDH) | No shared secret needed upfront |
| Digital signatures | Asymmetric (RSA, ECDSA) | Only private key holder can sign |
| Authentication | Asymmetric or HMAC | Depends on context |
| Password storage | KDF (bcrypt, Argon2) | Intentionally slow; salted |
| File integrity | Hash (SHA-256) | Fast; one-way |
| API authentication | HMAC-SHA256 | Symmetric — requires shared key |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **AES** | Advanced Encryption Standard — symmetric block cipher with 128/192/256-bit keys |
| **Asymmetric Encryption** | Encryption using a public/private key pair |
| **Block Cipher** | Encrypts fixed-size data blocks |
| **CA** | Certificate Authority — trusted third party that signs digital certificates |
| **Caesar Cipher** | Substitution cipher shifting letters by a fixed number |
| **Certificate** | Digitally signed document binding a public key to an identity |
| **Ciphertext** | Encrypted output |
| **CSR** | Certificate Signing Request — sent to a CA to get a certificate signed |
| **DH / Diffie-Hellman** | Key exchange protocol establishing a shared secret over a public channel |
| **Digital Signature** | Hash of a message encrypted with the signer's private key |
| **Hash Function** | One-way function producing a fixed-size digest from arbitrary input |
| **HMAC** | Hash-based MAC — combines hash function with a secret key |
| **KDF** | Key Derivation Function — derives cryptographic keys from passwords (bcrypt, Argon2, PBKDF2) |
| **Key** | Secret value used by a cipher for encryption/decryption |
| **Keyspace** | Total number of valid keys for a cipher |
| **MITM** | Man-in-the-Middle attack — attacker intercepts and potentially modifies communication |
| **Non-repudiation** | The inability to deny having performed an action |
| **PBKDF2** | Password-Based Key Derivation Function 2 — applies hash many times to slow brute force |
| **PKI** | Public Key Infrastructure — system for managing digital certificates and CAs |
| **Plaintext** | Original, readable message before encryption |
| **Private Key** | Secret key in asymmetric pair — never shared |
| **Public Key** | Shareable key in asymmetric pair |
| **Rainbow Table** | Precomputed hash-to-password lookup table |
| **RSA** | Rivest-Shamir-Adleman — asymmetric algorithm based on integer factorisation |
| **Salt** | Random value added to a password before hashing to defeat rainbow tables |
| **SHA-256** | Secure Hash Algorithm producing 256-bit digest — currently secure |
| **Stream Cipher** | Encrypts data byte by byte |
| **Symmetric Encryption** | Same key for encryption and decryption |
| **TLS** | Transport Layer Security — protocol securing HTTPS using hybrid cryptography |

---

## Exam and Interview Revision

### Must Remember

- Caesar cipher keyspace = 25; broken by brute force or frequency analysis
- Symmetric: same key both ways; fast; scaling problem (n(n-1)/2 keys for n users)
- AES key sizes: **128, 192, 256 bits**; still considered secure; block size = 128 bits
- AES rounds: **10** (128-bit), **12** (192-bit), **14** (256-bit)
- Asymmetric: public/private key pair; encrypt with public → decrypt with private; sign with private → verify with public
- RSA security: integer factorisation hardness; minimum 2048-bit keys in practice
- DH: establishes shared secret over public channel; vulnerable to MITM without authentication
- Hash: one-way, fixed-size output, avalanche effect; SHA-256 secure; MD5/SHA-1 broken
- HMAC = hash + secret key → provides integrity AND authenticity
- PKI: CA signs certificates to bind public keys to identities; solves DH MITM problem
- Password storage: never plaintext → hash → salted hash → bcrypt/Argon2/PBKDF2
- Salt defeats rainbow tables; KDFs defeat brute force via intentional slowness
- TLS hybrid: asymmetric for key exchange, symmetric for bulk data, HMAC for integrity

### Common Interview Questions

| Question | Key Points |
|----------|-----------|
| What is the difference between symmetric and asymmetric encryption? | Symmetric: same key both sides, fast, needs secure key exchange. Asymmetric: public/private pair, slower, no prior shared secret needed. |
| Why is asymmetric encryption not used for bulk data? | It is significantly slower than symmetric encryption. In practice, asymmetric is used to exchange a symmetric key (hybrid approach — TLS). |
| How does RSA work at a high level? | Key pair from two large primes; security relies on integer factorisation hardness. Public key encrypts; private key decrypts. |
| What is a digital signature? | Hash of the message encrypted with the sender's private key. Anyone can verify with the public key — proves authenticity and non-repudiation. |
| What is the difference between a hash and HMAC? | A hash has no key — anyone can compute it. HMAC requires a secret key — only the key holder can produce a valid HMAC. HMAC provides authenticity; plain hash does not. |
| Why is salting passwords important? | Salts are unique per user. Even if two users have the same password, their hashes differ. Defeats precomputed rainbow table attacks. |
| Why use bcrypt/Argon2 over SHA-256 for passwords? | SHA-256 is designed to be fast — bcrypt/Argon2 are intentionally slow. Slower hash = millions of times harder to brute force. |
| What is PKI and why is it needed? | System of CAs and certificates that binds public keys to verified identities. Needed to prevent MITM attacks where an attacker substitutes their own public key. |
| What is the MITM attack on Diffie-Hellman? | Attacker intercepts A and B, substitutes their own values, establishes separate secrets with each party. Prevented by authenticating the DH exchange with certificates (as in TLS). |
| What is PBKDF2? | Password-Based Key Derivation Function 2 — applies a hash function thousands of times to derive an encryption key from a password, making brute force attacks significantly slower. |
