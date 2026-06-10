# Length Extension Attacks

## Overview

Length extension attacks exploit a structural property of hash functions built on the Merkle-Damgård construction — specifically MD5, SHA-1, and SHA-256. An attacker who knows the hash of an original message can compute a valid hash for a modified version of that message with extra data appended, without knowing the original input or any secret key. This room covers hash function internals, padding mechanics, the Merkle-Damgård state model, and exactly how length extension attacks exploit these properties.

---

## Topics Covered

- Hash function properties: pre-image resistance, second pre-image resistance, collision resistance
- Where hash functions are used
- How hash functions process data in blocks
- Padding mechanics and why they matter
- Internal state and the Merkle-Damgård construction
- SHA-256 deep dive: padding, message schedule, compression rounds, final hash
- How length extension attacks work
- Which hash functions are vulnerable

---

## Key Concepts

### Hash Function Security Properties

A cryptographic hash function must satisfy three properties to be considered secure:

| Property | Description |
|----------|-------------|
| **Pre-image resistance** | Given a hash output, it must be computationally infeasible to find the original input |
| **Second pre-image resistance** | Given a message and its hash, it must be infeasible to find a different message producing the same hash |
| **Collision resistance** | It must be computationally infeasible to find any two different inputs that produce the same hash |

These properties make hashes useful for password storage, file integrity verification, digital signatures, and message authentication codes.

---

### Where Hash Functions Are Used

- **Password storage** — the hash is stored, not the plaintext password. On login, the input is hashed and compared
- **File integrity checks** — a hash of a downloaded file is compared against the publisher's expected hash
- **Digital signatures** — a document's hash is signed with a private key, allowing anyone to verify it with the public key
- **HMAC** — a hash combined with a secret key to verify message authenticity and integrity

---

### How Hash Functions Process Data

Hash functions like MD5, SHA-1, and SHA-256 do not process input in one step. They divide input into fixed-size blocks and process one block at a time, updating an internal state after each block.

| Algorithm | Block size | Internal state size | Rounds per block |
|-----------|-----------|--------------------|--------------------|
| MD5 | 512 bits | 128 bits (4 registers: A, B, C, D) | 64 |
| SHA-1 | 512 bits | 160 bits (5 words: A–E) | 80 |
| SHA-256 | 512 bits | 256 bits (8 words: A–H) | 64 |

The final hash output is simply the internal state after all blocks have been processed.

---

### Padding

When a message does not fill the final block exactly, padding is applied. Padding serves two purposes:
1. Aligns the message to the required block size
2. Embeds the original message length in the final 64 bits of the last block

**SHA-256 padding rules:**
1. Append a single `1` bit
2. Append `0` bits until the message is 448 bits mod 512
3. Append the original message length as a 64-bit big-endian integer

**Example — padding "TryHackMe" (72 bits):**

```
Original:   72 bits of data
Step 1:     Append 1 bit          → 73 bits
Step 2:     Append 375 zero bits  → 448 bits
Step 3:     Append 64-bit length  → 512 bits (one full block)
```

The length field at the end is critical — it is what makes padding deterministic and predictable, which is exactly what enables length extension attacks.

---

### SHA-256 Internals

#### Initial Hash Values

SHA-256 is initialised with eight 32-bit constants derived from the fractional parts of the square roots of the first eight prime numbers:

```
H0 = 6a09e667    H1 = bb67ae85    H2 = 3c6ef372    H3 = a54ff53a
H4 = 510e527f    H5 = 9b05688c    H6 = 1f83d9ab    H7 = 5be0cd19
```

These are assigned to working variables `a` through `h` before processing begins.

#### Message Schedule Expansion

The 512-bit padded message is split into 16 × 32-bit words (W[0]–W[15]). The schedule is then expanded to 64 words (W[0]–W[63]) using:

```
W[i] = σ1(W[i-2]) + W[i-7] + σ0(W[i-15]) + W[i-16]

where:
σ0(x) = ROTR(x, 7)  XOR ROTR(x, 18) XOR SHR(x, 3)
σ1(x) = ROTR(x, 17) XOR ROTR(x, 19) XOR SHR(x, 10)
```

#### Compression: 64 Rounds

Each round updates the 8 working variables using two temporary values:

```
T1 = h + Σ1(e) + Ch(e,f,g) + K[i] + W[i]
T2 = Σ0(a) + Maj(a,b,c)

h=g, g=f, f=e, e=d+T1, d=c, c=b, b=a, a=T1+T2
```

Where:
- `Ch(e,f,g) = (e AND f) XOR (NOT e AND g)` — choice function
- `Maj(a,b,c) = (a AND b) XOR (a AND c) XOR (b AND c)` — majority function
- `Σ0(a) = ROTR(a,2) XOR ROTR(a,13) XOR ROTR(a,22)`
- `Σ1(e) = ROTR(e,6) XOR ROTR(e,11) XOR ROTR(e,25)`
- `K[i]` — 64 round constants (cube root fractional parts of first 64 primes)

#### Final Hash

After 64 rounds, the working variable values are added to the initial hash values:

```
H0 += a,  H1 += b,  H2 += c,  H3 += d
H4 += e,  H5 += f,  H6 += g,  H7 += h
```

The final 256-bit hash is the concatenation of H0–H7 in hexadecimal.

For "TryHackMe":
```
9e897fb7e8832e7d6e5f63a4bdbd0cd6496f53cb3f54b5abf84217d9f4ca5397
```

---

### The Length Extension Attack

#### Why It Works

After processing all blocks of the original message, the hash function halts and outputs its current internal state as the hash. This internal state **completely represents** where the algorithm stopped.

An attacker who knows this final hash can load it as the starting internal state and continue processing additional blocks. The hash function does not know or care that the message it is continuing from was previously "finished."

#### What the Attacker Needs

1. The hash of the original message `H(message)`
2. The length of the original message (or an estimate)
3. Knowledge of the padding rules for that hash function

The attacker does **not** need to know the original message or any secret key.

#### Step-by-Step

**Normal hashing:**
```
Message: "user=test"
         ↓ pad to block size
         ↓ hash entire padded block
Result:  H("user=test" + padding)
```

**Length extension:**
```
Known:   H("user=test" + padding)    ← load as internal state
Append:  "&admin=true"               ← new data
         ↓ hash new data as next block
Result:  H("user=test" + padding + "&admin=true")
         ← valid hash, computed without knowing "user=test" or any key
```

The attacker can present the server with the message `"user=test" + padding + "&admin=true"` and the computed hash, and a naive MAC implementation will accept it as valid.

#### Visual Summary

| Scenario | Message | Hash computation |
|----------|---------|-----------------|
| Normal | `user=test` | Hash from initial state → final state |
| Length extension | `user=test` + padding + `&admin=true` | Load known hash as state → continue hashing |

The padding between the original message and the appended data is non-negotiable — it was applied when the original message was hashed, and the attacker must include it exactly.

---

### Vulnerable Hash Functions

| Algorithm | Vulnerable | Reason |
|-----------|-----------|--------|
| MD5 | Yes | Merkle-Damgård construction, predictable internal state |
| SHA-1 | Yes | Same construction |
| SHA-256 | Yes (if used without HMAC) | Same construction |
| SHA-512 | Yes (if used without HMAC) | Same construction |
| SHA-3 (Keccak) | No | Sponge construction — fundamentally different design |
| HMAC-SHA256 | No | Secret key is mixed at both start and end, blocking state extension |

**HMAC is the fix:** HMAC computes `H(key XOR opad || H(key XOR ipad || message))`. The outer hash wraps the inner hash — even if the inner state is known, the outer key application makes extension impossible.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Merkle-Damgård | Construction underlying MD5, SHA-1, SHA-256 — processes data block by block, updating internal state |
| Internal state | The set of working variables (A–H in SHA-256) that carry computation between blocks |
| Padding oracle | A system that reveals whether padding is valid — enables padding oracle attacks (separate from length extension) |
| HMAC | Hash-based Message Authentication Code — uses a secret key to prevent length extension |
| Length extension | Attack where an attacker extends a known hash with new data without knowing the original message or key |
| Pre-image resistance | Hash security property: cannot reverse a hash to find the input |
| Collision resistance | Hash security property: cannot find two inputs with the same hash |
| Message schedule | The 64-word expansion of a 512-bit block used in SHA-256's compression rounds |
| Round constant K[i] | Predefined 32-bit values used per round in SHA-256, derived from cube roots of primes |

---

## Workflow / Process

```
Attacker knows H(secret || message) and message length
        |
        v
Reconstruct the padding that was applied to the original message
(using known length and hash function's padding rules)
        |
        v
Load H(secret || message) as the internal state of the hash function
        |
        v
Continue hashing with appended data (&admin=true, etc.)
        |
        v
Output: valid H(secret || message || padding || appended_data)
        |
        v
Submit to server: message = original + padding + appended_data
                  hash    = computed extension hash
        |
        v
Server verifies: H(secret || submitted_message) == submitted_hash
                 → match → accepted as valid
```

---

## Real-World Relevance

- Length extension attacks are directly applicable wherever a server uses `H(secret || message)` as a MAC and verifies it naively — this pattern appears in early API authentication schemes, file signature validation, and web application session tokens
- The hash_extender tool by Ron Bowes is the standard CTF and pentest tool for automating length extension attacks
- Flickr, Vimeo, and numerous early web APIs were found to use hash-based MACs vulnerable to this attack before HMAC adoption became standard
- SHA-3 was designed with the sponge construction specifically to eliminate Merkle-Damgård vulnerabilities, including length extension
- HMAC is the correct and standard fix — it is available in every major cryptographic library and should always be used when authenticating messages with a shared secret

---

## Key Learnings

- MD5, SHA-1, SHA-256 are all Merkle-Damgård — all vulnerable to length extension
- The final hash of any Merkle-Damgård function is its resumable internal state
- An attacker can resume hashing from a known state to produce a valid hash for an extended message
- The attacker must include the original padding in the extended message — it was baked in during the initial hash
- HMAC prevents length extension by wrapping the inner hash with a keyed outer hash
- SHA-3 (Keccak) is immune — its sponge construction does not expose a resumable state

---

## Additional Notes

- The `hashpumpy` Python library and `hash_extender` CLI tool automate length extension for MD5, SHA-1, and SHA-256
- When implementing API authentication, never use `H(secret + data)` — always use `HMAC(secret, data)`
- HMAC-SHA256 is the current standard for message authentication in most protocols (JWT, OAuth, AWS Signature V4)
- SHA-256 is not broken in general — it is only vulnerable to length extension when used *without* HMAC in a secret-prepend construction

---

## Conclusion

Length extension attacks expose a fundamental design consequence of the Merkle-Damgård construction: the final hash is a resumable computation state. Any system that relies on `H(secret || data)` as a MAC is vulnerable to an attacker appending data and producing a valid authentication tag without knowing the secret. The fix — HMAC — has been standardised for decades. Understanding this attack at the structural level reinforces why cryptographic primitives must be composed correctly, not just used individually.
