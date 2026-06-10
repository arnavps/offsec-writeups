# Padding Oracles

## Overview

The Padding Oracle attack is a chosen-ciphertext attack against CBC-mode encryption that allows an attacker to decrypt ciphertext without knowing the key. It works by sending modified ciphertexts to a server and using the server's response — specifically whether the padding is valid or invalid — as an oracle to recover the plaintext byte by byte. This room covers CBC mode encryption and decryption in full detail, PKCS#7 padding, and the step-by-step mechanics of the padding oracle attack.

---

## Topics Covered

- Block cipher modes overview
- PKCS#7 padding scheme
- CBC encryption: step-by-step with worked example
- CBC decryption: step-by-step with worked example
- The padding oracle attack: theory and mechanics
- Recovering plaintext without the key

---

## Key Concepts

### Block Ciphers and Modes of Operation

Block ciphers like AES require fixed-size input blocks (16 bytes for AES). When a message is longer than one block, a **cipher mode** defines how blocks are chained.

| Mode | Description | Security Notes |
|------|-------------|---------------|
| **ECB** | Each block encrypted independently | Insecure — patterns preserved |
| **CBC** | Each block XORed with previous ciphertext before encryption | Secure confidentiality but vulnerable to padding oracle if unauthenticated |
| **CTR** | Counter value combined with nonce per block | Secure, parallelisable |

---

### PKCS#7 Padding

When the last plaintext block is shorter than the block size, **PKCS#7 padding** fills the remaining bytes. Each padding byte has a value equal to the number of padding bytes added.

| Text length (mod 8) | Padding bytes added | Value of each padding byte |
|---------------------|--------------------|-----------------------------|
| 0 | 8 | `0x08` |
| 1 | 7 | `0x07` |
| 2 | 6 | `0x06` |
| 3 | 5 | `0x05` |
| 4 | 4 | `0x04` |
| 5 | 3 | `0x03` |
| 6 | 2 | `0x02` |
| 7 | 1 | `0x01` |

If the plaintext is already a multiple of the block size, an entire extra block of padding is added (all bytes set to the block size value).

**Examples (block size 8):**
- `"Try"` (3 bytes) → `54 72 79 05 05 05 05 05`
- `"Hack"` (4 bytes) → `48 61 63 6B 04 04 04 04`
- `"Me"` (2 bytes) → `4D 65 06 06 06 06 06 06`

During decryption, padding bytes are removed. If the padding is invalid (e.g. the last byte is `0x03` but the preceding two bytes are not also `0x03`), a padding error is raised.

---

### CBC Encryption — Worked Example

**Message:** `"TryHackMe"` | **Block size:** 8 bytes | **Key:** secret | **IV:** `01 01 01 01 01 01 01 01`

**Step 1 — Split and pad:**
```
Block 1: "TryHackM"  → 54 72 79 48 61 63 6B 4D  (8 bytes, no padding)
Block 2: "e"         → 65 07 07 07 07 07 07 07  (1 byte + 7 bytes PKCS#7 padding)
```

**Step 2 — XOR Block 1 with IV:**
```
54 72 79 48 61 63 6B 4D
XOR
01 01 01 01 01 01 01 01
=
55 73 78 49 60 62 6A 4C
```

**Step 3 — Encrypt XOR result → Ciphertext Block 1 (C1):**
```
Encrypt(55 73 78 49 60 62 6A 4C) = A3 3C 9F 12 58 44 76 10
C1 = A3 3C 9F 12 58 44 76 10
```

**Step 4 — XOR Block 2 with C1, then encrypt:**
```
65 07 07 07 07 07 07 07
XOR
A3 3C 9F 12 58 44 76 10
=
C6 3B 98 15 5F 43 71 17

Encrypt(C6 3B 98 15 5F 43 71 17) = B7 9F 2D 5A 11 66 4F 7A
C2 = B7 9F 2D 5A 11 66 4F 7A
```

**Final ciphertext:** `A3 3C 9F 12 58 44 76 10 B7 9F 2D 5A 11 66 4F 7A`

**CBC encryption diagram:**
```
Plaintext[i] → XOR with C[i-1] (or IV for first block) → Encrypt with key → C[i]
```

---

### CBC Decryption — Worked Example

Decryption is the reverse. The core formula is:

```
P[i] = Dk(C[i]) XOR C[i-1]
```

Where:
- `P[i]` = plaintext block i
- `Dk(C[i])` = intermediate value from decrypting C[i] with key k
- `C[i-1]` = previous ciphertext block (IV for the first block)

**Step 1 — Decrypt C1:**
```
Dk(A3 3C 9F 12 58 44 76 10) = 55 73 78 49 60 62 6A 4C  (intermediate)
XOR IV:
55 73 78 49 60 62 6A 4C XOR 01 01 01 01 01 01 01 01 = 54 72 79 48 61 63 6B 4D
= "TryHackM"
```

**Step 2 — Decrypt C2:**
```
Dk(B7 9F 2D 5A 11 66 4F 7A) = C6 3B 98 15 5F 43 71 17  (intermediate)
XOR C1:
C6 3B 98 15 5F 43 71 17 XOR A3 3C 9F 12 58 44 76 10 = 65 07 07 07 07 07 07 07
= "e" + padding (remove 7 bytes of 0x07)
= "e"
```

**Recovered plaintext:** `"TryHackMe"`

---

### The Padding Oracle Attack

A **padding oracle** is any server that tells an attacker whether the padding of a submitted ciphertext is valid or not — either through:
- Different error messages for padding error vs. decryption error
- Timing differences in the response

This single bit of information (valid/invalid padding) is enough to recover the entire plaintext.

#### Foundation: the Decryption Formula

```
P[i] = Dk(C[i]) XOR C[i-1]
```

`Dk(C[i])` is the intermediate value — the raw output of the block cipher's decryption function before XOR. The attacker's goal is to determine `Dk(C[i])` for each block. Once `Dk(C[i])` is known:

```
P[i] = Dk(C[i]) XOR C[i-1]    (using the real C[i-1])
```

The intermediate value is the bridge between the ciphertext and the plaintext.

#### Recovering One Block: Byte-by-Byte

**Setup:**
```
IV:         31 31 31 31 31 31 31 31 31 31 31 31 31 31 31 31
Ciphertext: 88 12 4e 09 e2 0e ab 43 f7 c3 23 2d 92 5a 1a ee (C1, 16 bytes)
```

The attack targets `Dk(C1)` byte by byte, working from the **last byte backwards**.

---

**Step 1 — Target the last byte (position 16)**

A valid padding of `0x01` requires `P1[16] = 0x01`, meaning:
```
Dk(C1)[16] XOR test[16] = 0x01
```

The attacker submits modified IVs (`test`) with `test[16]` ranging from `0x00` to `0xFF`. All other bytes of the IV are zeroed. For each value, the oracle is queried. When valid padding is returned:

```
Dk(C1)[16] = test[16] XOR 0x01
```

If the valid guess is `test[16] = 0x37`:
```
Dk(C1)[16] = 0x37 XOR 0x01 = 0x36
```

Now recover the actual plaintext byte using the real IV:
```
P1[16] = Dk(C1)[16] XOR IV[16] = 0x36 XOR 0x31 = 0x07
```

---

**Step 2 — Target the second-to-last byte (position 15)**

For valid padding of `0x02 0x02`:
- `test[16]` must be set to produce `P1[16] = 0x02` → `test[16] = Dk(C1)[16] XOR 0x02 = 0x36 XOR 0x02 = 0x34`
- `test[15]` is iterated from `0x00` to `0xFF`

When valid padding is found:
```
Dk(C1)[15] = test[15] XOR 0x02
P1[15] = Dk(C1)[15] XOR IV[15]
```

---

**Step 3 — Repeat for all 16 bytes**

Work backwards from byte 16 to byte 1. For each position `i`, set the already-recovered bytes to produce the target padding value `(16 - i + 1)`, then iterate the current target byte.

After all 16 bytes: the full plaintext block is recovered. Remove padding to get the original plaintext.

---

**Example recovery:**

After iterating all 16 bytes of `Dk(C1)` and XORing with the IV:
```
P1 = 54 72 79 48 61 63 6b 4d 65 06 06 06 06 06 06 06
```

Remove 6 bytes of `0x06` padding:
```
P1 = "TryHackMe"
```

---

### Query Count

For each of the 16 bytes, at most 256 oracle queries are needed (one per possible byte value). Total worst-case queries per 16-byte block: **256 × 16 = 4,096**. For multi-block ciphertexts, multiply by the number of blocks. Fully automatable.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Padding oracle | A server that reveals whether submitted ciphertext has valid padding — enough to decrypt everything |
| PKCS#7 | Padding scheme where each padding byte value equals the number of padding bytes added |
| IV | Initialisation Vector — random value XORed with the first plaintext block in CBC; equivalent to C[0] |
| Intermediate value `Dk(C[i])` | Raw output of the block cipher decryption before XOR — the target of the oracle attack |
| CBC | Cipher Block Chaining — each plaintext block XORed with the previous ciphertext block before encryption |
| Chosen-ciphertext attack | Attack where the attacker submits modified ciphertexts and observes responses |
| Padding error | Server response indicating the decrypted padding bytes are invalid |

---

## Real-World Relevance

- The ASP.NET padding oracle vulnerability (MS10-070, 2010) allowed decryption of ViewState and authentication tokens on millions of .NET web applications — fully exploitable using the POET tool
- POODLE (CVE-2014-3566) exploited SSL 3.0's CBC padding validation to decrypt HTTPS cookies
- Lucky13 (2013) demonstrated that timing differences in CBC MAC verification are sufficient to mount a padding oracle attack even without explicit error messages
- The `padbuster` tool automates padding oracle attacks against CBC-mode encrypted cookies and tokens — commonly used in web application penetration testing
- Prevention: use authenticated encryption (AES-GCM) — the authentication tag is checked before any decryption occurs, eliminating the oracle entirely
- If CBC must be used: implement MAC-then-Encrypt or Encrypt-then-MAC patterns, and ensure all padding errors return identical responses with identical timing

---

## Key Learnings

- PKCS#7 padding adds bytes whose value equals the count of padding bytes; an entire extra block is added if the message is already aligned
- CBC decryption: `P[i] = Dk(C[i]) XOR C[i-1]` — each plaintext block depends on the previous ciphertext block (or IV)
- A padding oracle is any server that distinguishes between valid and invalid padding in submitted ciphertext
- The attack recovers the intermediate value `Dk(C[i])` by iterating possible byte values and using the oracle's response
- Once `Dk(C[i])` is known, XOR with the real `C[i-1]` recovers the plaintext
- The attack requires at most 256 × block_size oracle queries per block — fully automated and practical
- Prevention: authenticated encryption (AES-GCM) makes padding oracles impossible by design

---

## Additional Notes

- The IV must be known or attacker-controlled for the attack to produce the correct plaintext — in most attack scenarios, the IV is transmitted with the ciphertext
- For multi-block ciphertexts, each block is attacked independently by using the previous ciphertext block as the modifiable "IV"
- A timing-based oracle (no error message difference, only timing) is harder to exploit but still feasible — see Lucky13 for the proof-of-concept
- `padbuster`, `POET`, and `PadBusterPy` are commonly used tools for automating this attack
- The attack also allows **ciphertext modification** — after recovering `Dk(C[i])`, you can set the preceding IV or ciphertext block to any value to produce any desired plaintext in `P[i]`

---

## Conclusion

The padding oracle attack transforms a single bit of server feedback — valid or invalid padding — into a complete plaintext decryption capability without the key. The attack works because CBC mode's decryption formula creates a predictable mathematical relationship between the ciphertext, the intermediate decryption value, and the plaintext. Authenticated encryption eliminates this attack class entirely: if the authentication tag is invalid, decryption never proceeds and no oracle feedback is ever generated.
