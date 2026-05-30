# Public Key Cryptography Basics

## Overview

Public key cryptography solves the fundamental problem of establishing secure communication between parties who have never met. This room covers RSA, Diffie-Hellman key exchange, SSH key pairs, digital signatures, certificates, and GPG — the practical tools that implement asymmetric cryptography in the real world.

---

## Topics Covered

- Authentication, authenticity, integrity, and confidentiality
- RSA — how it works and the math behind it
- Diffie-Hellman key exchange
- SSH key authentication
- Digital signatures
- Certificates and the chain of trust
- GPG (GNU Privacy Guard)
- Cryptanalysis terminology

---

## Key Concepts

### The Four Security Properties

| Property | Description |
|---|---|
| Authentication | Verifying the identity of who you're communicating with |
| Authenticity | Verifying that a message genuinely comes from the claimed sender |
| Integrity | Ensuring data has not been altered in transit |
| Confidentiality | Ensuring only authorised parties can access the data |

---

### RSA

RSA is a public-key encryption algorithm based on the mathematical difficulty of factoring large numbers. Multiplying two large primes is easy; finding the factors of their product is computationally infeasible for sufficiently large numbers.

**Simplified RSA example:**

1. Bob chooses primes p = 157, q = 199 → n = 31243
2. Calculates ϕ(n) = 30888, selects e = 163, d = 379
3. Public key: (n=31243, e=163) | Private key: (n=31243, d=379)
4. Alice encrypts x=13: y = 13^163 mod 31243 = 16341
5. Bob decrypts: x = 16341^379 mod 31243 = 13

**Real-world key sizes:** RSA 2048-bit minimum; 3072 or 4096-bit recommended.

---

### Diffie-Hellman Key Exchange

Allows two parties to establish a shared secret over an insecure channel without transmitting the secret itself. The shared secret can then be used as a symmetric encryption key.

**Process:**

1. Alice and Bob agree on public values: prime p and generator g
2. Each generates a private key (a and b)
3. Each calculates a public key: A = g^a mod p, B = g^b mod p
4. They exchange public keys
5. Both calculate the shared secret: Alice: B^a mod p, Bob: A^b mod p → both equal g^ab mod p

An observer sees p, g, A, and B but cannot compute the shared secret without solving the discrete logarithm problem.

---

### SSH Key Authentication

SSH supports key-based authentication as a more secure alternative to passwords.

**Key generation:**

```bash
ssh-keygen                          # Default RSA key
ssh-keygen -t ed25519               # Ed25519 key (recommended)
ssh-keygen -t ecdsa -b 521          # ECDSA key
```

**Connecting with a key:**

```bash
ssh -i privatekey.pem username@hostname
```

**Key storage:**
- Private key: `~/.ssh/id_rsa` (or `id_ed25519`) — never share this
- Public key: `~/.ssh/id_rsa.pub` — copy to server's `~/.ssh/authorized_keys`
- Permissions: private key must be `600` or stricter

**Copying public key to server:**

```bash
ssh-copy-id username@hostname
```

**SSH key algorithms:**

| Algorithm | Notes |
|---|---|
| RSA | Default; use 4096-bit minimum |
| ECDSA | Elliptic curve variant of DSA |
| Ed25519 | Modern, fast, recommended |
| DSA | Legacy; avoid |

**Important:** The passphrase protecting a private key decrypts the key locally — it is never transmitted. The passphrase protects the key file if it is stolen.

---

### Digital Signatures

Digital signatures verify the authenticity and integrity of a document or message using asymmetric cryptography.

**How it works:**
1. Sender hashes the document
2. Sender encrypts the hash with their **private key** → this is the signature
3. Recipient decrypts the signature with the sender's **public key** → recovers the hash
4. Recipient hashes the received document and compares — if they match, the document is authentic and unmodified

Only the holder of the private key can create a valid signature. Anyone with the public key can verify it.

---

### Certificates and Chain of Trust

Certificates bind a public key to an identity and are signed by a Certificate Authority (CA).

**How HTTPS certificate trust works:**

```
Root CA (trusted by OS/browser at install time)
    ↓ signs
Intermediate CA
    ↓ signs
Website Certificate (e.g., tryhackme.com)
```

Your browser trusts the website's certificate because it traces back to a root CA that your browser already trusts.

**Getting a TLS certificate:**
- Paid: from commercial CAs (DigiCert, Sectigo, etc.)
- Free: Let's Encrypt — automated, widely used

---

### GPG (GNU Privacy Guard)

GPG is an open-source implementation of the OpenPGP standard. Used for encrypting files, signing emails, and verifying software integrity.

**Generate a key pair:**

```bash
gpg --gen-key
```

**Import a key:**

```bash
gpg --import backup.key
```

**Decrypt a file:**

```bash
gpg --decrypt confidential_message.gpg
```

GPG private keys can be protected with a passphrase. If passphrase-protected, John the Ripper can attempt to crack it using `gpg2john`.

---

## Important Terminology

| Term | Definition |
|---|---|
| RSA | Public-key algorithm based on integer factorisation |
| Diffie-Hellman | Key exchange protocol for establishing a shared secret |
| ECC | Elliptic Curve Cryptography — equivalent security with smaller keys |
| Digital Signature | A hash encrypted with a private key to prove authenticity and integrity |
| Certificate | A document binding a public key to an identity, signed by a CA |
| CA | Certificate Authority — trusted entity that signs certificates |
| Chain of Trust | The hierarchy from root CA to intermediate CA to end-entity certificate |
| GPG | GNU Privacy Guard — open-source OpenPGP implementation |
| `authorized_keys` | File on SSH server listing public keys permitted to authenticate |
| Passphrase | Password protecting a private key file — never transmitted |
| Cryptanalysis | The study of breaking or bypassing cryptographic systems |
| Brute-Force Attack | Trying every possible key or password |
| Dictionary Attack | Trying words from a wordlist as candidate passwords |

---

## Real-World Relevance

- RSA and Ed25519 SSH keys are used for server authentication in every enterprise environment
- Certificates and the chain of trust are what make HTTPS trustworthy — certificate validation failures enable MITM attacks
- Digital signatures are used to verify software packages, code commits, and email authenticity
- GPG is used to sign Linux package repositories — verifying GPG signatures before installing software is a security best practice
- Diffie-Hellman is used in TLS to provide Perfect Forward Secrecy (PFS) — even if the server's private key is compromised, past sessions cannot be decrypted
- SSH private keys found on compromised systems are high-value targets — they may provide access to other systems

---

## Key Learnings

- RSA security relies on the difficulty of factoring large numbers
- Diffie-Hellman establishes a shared secret over an insecure channel without transmitting the secret
- SSH key authentication is more secure than passwords — Ed25519 is the recommended algorithm
- Digital signatures use the private key to sign and the public key to verify
- Certificates chain back to trusted root CAs — this is the basis of HTTPS trust
- GPG implements OpenPGP for file encryption and email signing

---

## Conclusion

Public key cryptography solves the fundamental challenge of secure communication between strangers. RSA, Diffie-Hellman, digital signatures, and certificates are the building blocks of TLS, SSH, and code signing — the protocols that secure virtually all modern digital communication. Understanding how they work is essential for both implementing security correctly and identifying where it can fail.
