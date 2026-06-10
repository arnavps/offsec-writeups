# Attacking ECB Oracles

## Overview

AES is the global standard for symmetric encryption — but AES alone is not sufficient for security. How blocks are chained together (the cipher mode) determines whether patterns in the plaintext survive into the ciphertext. Electronic Codebook (ECB) mode encrypts each block independently, which means identical plaintext blocks always produce identical ciphertext blocks. This structural weakness enables chosen-plaintext attacks against ECB encryption oracles. This room covers symmetric encryption fundamentals, cipher block modes, ECB's core flaw, and how to exploit it.

---

## Topics Covered

- Encryption vs encoding
- Core principles of secure encryption: confusion, diffusion, key-only reliance
- AES and block ciphers
- Cipher block modes: why they exist
- ECB mode: how it works and why it is insecure
- ECB oracle attacks: chosen-plaintext exploitation

---

## Key Concepts

### Encryption vs Encoding

A common confusion in security:

| | Encoding | Encryption |
|--|---------|-----------|
| Uses a key? | No | Yes |
| Purpose | Format transformation | Confidentiality |
| Reversible without key? | Yes | No |
| Example | Base64, URL encoding | AES, RSA |

Encryption without a key is not encryption — it is encoding. The security of encryption is entirely dependent on the key remaining secret. The algorithm itself is assumed to be public knowledge.

---

### Principles of Secure Encryption

Three principles, established through NIST's AES competition (1997), now define what makes an encryption algorithm trustworthy:

| Principle | Description |
|-----------|-------------|
| **Confusion** | The relationship between the key and ciphertext must be as complex as possible — knowing the ciphertext should reveal nothing about the key |
| **Diffusion** | Each bit of the plaintext should influence many bits of the ciphertext — spreading the effect of input changes throughout the output |
| **Key-only reliance** | Security must depend entirely on the key, not on keeping the algorithm secret — the algorithm can be freely published |

AES (Rijndael cipher, by Vincent Rijmen and Joan Daemen) satisfies all three. It is the dominant symmetric cipher used worldwide.

---

### Block Ciphers and AES

AES is a block cipher — it operates on fixed-size blocks of data (16 bytes / 128 bits per block). Messages longer than one block must be split into multiple blocks. The last block is padded to reach the required 16 bytes if it falls short.

The question of how multiple blocks are processed and linked together is answered by the **cipher mode**.

---

### Cipher Block Modes

| Mode | Description | Security |
|------|-------------|---------|
| **ECB** (Electronic Codebook) | Each block encrypted independently with the same key | Insecure — identical plaintext blocks produce identical ciphertext blocks |
| **CBC** (Cipher Block Chaining) | Each plaintext block is XORed with the previous ciphertext block before encryption | Secure if authenticated |
| **CTR** (Counter Mode) | Each block encrypted with a unique nonce+counter pair — no chaining | Secure, parallelisable |

---

### ECB Mode: How It Works

ECB takes each plaintext block and encrypts it directly with the key. No state is carried between blocks. The final ciphertext is the concatenation of all independently encrypted blocks.

**Example: encrypting `CryptographyAndAESIsALotOfFun!!!` with key `secretpassword!!`:**

```
Plaintext:
  Block 1: "CryptographyAndA"  → 43 72 79 70 74 6F 67 72 61 70 68 79 41 6E 64 41
  Block 2: "ESIsALotOfFun!!!"  → 45 53 49 73 41 4C 6F 74 4F 66 46 75 6E 21 21 21

Encrypt each independently:
  Block 1 ciphertext: 92 FF C7 FE CD 0D 04 13 E8 B2 63 6D F4 38 BF 2A
  Block 2 ciphertext: FE 7C 80 45 6D 0A 87 C7 7A 20 61 78 B9 7A 19 33

Final ciphertext: 92 FF C7 FE CD 0D 04 13 E8 B2 63 6D F4 38 BF 2A FE 7C 80 45 6D 0A 87 C7 7A 20 61 78 B9 7A 19 33
```

**Python (pycryptodome):**
```python
from Crypto.Cipher import AES
import binascii

key = b"secretpassword!!"
plaintext = b"CryptographyAndAESIsALotOfFun!!!"

cipher = AES.new(key, AES.MODE_ECB)
ciphertext = binascii.hexlify(cipher.encrypt(plaintext)).decode()
print(ciphertext)
```

The output looks random. The flaw is not immediately apparent — until you consider what happens with repeated blocks.

---

### ECB's Core Flaw: Pattern Preservation

Because every 16-byte block is encrypted identically with the same key, **identical plaintext blocks always produce identical ciphertext blocks**.

**Classic demonstration — the ECB penguin:**

Encrypting a bitmap image with ECB mode preserves the visual structure of the original image. Regions of solid colour in the original produce repeated ciphertext blocks in the encrypted version — the outline of the penguin remains visible in the ciphertext. This visually illustrates why ECB is fundamentally insecure for any data with repeating structure.

**Implication:** An attacker who can observe two ciphertext blocks that are identical knows the corresponding plaintext blocks are identical, even without the key. This leaks information about the plaintext.

---

### ECB Oracle Attacks

An **encryption oracle** is any system that will encrypt arbitrary attacker-controlled input. When ECB is used, the oracle can be manipulated to reveal secret data one byte at a time.

#### The Attack Principle

The oracle encrypts: `attacker_controlled_prefix || secret_data`

Because ECB encrypts each 16-byte block independently, the attacker can craft input so that a known portion and one unknown byte end up in the same block. By comparing the oracle's output for all 256 possible values of that byte, the attacker identifies the match — revealing the unknown byte.

#### Step-by-Step (one byte)

Assume the oracle encrypts: `attacker_input || secret`

The secret's first byte is unknown. The attacker wants to discover it.

**Step 1 — Align the target byte at the block boundary:**

Submit 15 bytes of known padding (`AAAAAAAAAAAAAAA`). Now block 1 is:
```
[AAAAAAAAAAAAAAA | ?]   ← 15 known + 1 unknown (first byte of secret)
```

Record the ciphertext of block 1: `C_target`

**Step 2 — Brute-force the unknown byte:**

For each possible byte value `b` (0x00 to 0xFF), submit:
```
AAAAAAAAAAAAAAA + b
```

Compare the ciphertext of this block against `C_target`. When they match:
```
b = first byte of secret
```

**Step 3 — Repeat for the next byte:**

Submit 14 bytes of padding. Block 1 is now:
```
[AAAAAAAAAAAAAA | b1 | ?]   ← 14 known + 1 recovered + 1 unknown
```

Brute-force the second unknown byte with 15 known bytes followed by each candidate.

Continue until the entire secret is recovered.

#### Attack Complexity

- 256 oracle queries per byte (worst case)
- Scales linearly with secret length
- No key required — only access to the encryption oracle

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Block cipher | Encryption algorithm operating on fixed-size data blocks |
| Cipher mode | Method defining how blocks are chained together during encryption |
| ECB | Electronic Codebook — each block encrypted independently; insecure |
| CBC | Cipher Block Chaining — each block XORed with previous ciphertext before encryption |
| Encryption oracle | A system that encrypts attacker-supplied input, potentially revealing secrets |
| Chosen-plaintext attack | Attack where the attacker can encrypt arbitrary inputs and observe the output |
| Padding | Extra bytes appended to the last block to reach the required block size |
| Diffusion | Cryptographic property ensuring plaintext changes spread throughout the ciphertext |
| Confusion | Cryptographic property obscuring the relationship between key and ciphertext |

---

## Practical Example

### ECB Oracle Attack (Python — conceptual)

```python
from Crypto.Cipher import AES
import os

SECRET = b"MySecretValue123"
KEY = os.urandom(16)

def oracle(attacker_input: bytes) -> bytes:
    plaintext = attacker_input + SECRET
    # pad to block size
    while len(plaintext) % 16 != 0:
        plaintext += b'*'
    cipher = AES.new(KEY, AES.MODE_ECB)
    return cipher.encrypt(plaintext)

# Recover first byte of SECRET
target_block = oracle(b'A' * 15)[:16]

for b in range(256):
    test = b'A' * 15 + bytes([b])
    if oracle(test)[:16] == target_block:
        print(f"First byte: {chr(b)} (0x{b:02x})")
        break
```

---

## Workflow / Process

```
Identify ECB mode in use:
  Submit two identical 16-byte blocks as input
  If ciphertext blocks 1 and 2 are identical → ECB confirmed

        |
        v
Align target byte at block boundary:
  Submit (block_size - 1 - known_bytes) padding
        |
        v
Record target block ciphertext from oracle output
        |
        v
Brute-force unknown byte:
  For b in 0..255:
    Submit (padding + known_bytes + b) to oracle
    Compare block ciphertext against target
    Match found → byte revealed
        |
        v
Shift window by one byte, repeat for next unknown byte
        |
        v
Concatenate recovered bytes → plaintext secret
```

---

## Real-World Relevance

- ECB is found in legacy systems, embedded devices, and poorly implemented encryption libraries — it is still encountered in real-world assessments
- The ECB penguin is a canonical demonstration of why cipher mode matters — used in cryptography education worldwide
- ECB oracle attacks are a standard CTF challenge type and appear in real-world API assessments where an application encrypts user-controlled data server-side
- The Padding Oracle attack (covered in the next room) is the analogous attack for CBC mode — together, ECB and CBC oracle attacks cover the two most commonly encountered insecure AES modes
- AES-GCM is the modern standard — it provides both confidentiality and authenticity, eliminating ECB-style pattern leakage and CBC-style bit-flipping/oracle vulnerabilities

---

## Key Learnings

- ECB encrypts every block with the same key independently — identical plaintext blocks always produce identical ciphertext blocks
- This pattern preservation makes ECB unsuitable for any data with repeating structure
- An ECB oracle (any service that encrypts attacker-controlled input) can be exploited to recover secret data one byte at a time using chosen-plaintext attacks
- ECB mode can be confirmed by submitting two identical 16-byte blocks and observing whether the ciphertext blocks match
- CBC and CTR modes prevent pattern leakage by introducing chaining and per-block uniqueness
- AES-GCM is the recommended modern mode for authenticated encryption

---

## Additional Notes

- Padding in this room uses `*` for simplicity — in real AES implementations, PKCS#7 padding is standard (see the Padding Oracles room)
- The ECB oracle attack described here is the "byte-at-a-time ECB decryption" attack from the Cryptopals challenges (Set 2, Challenge 12)
- Detecting ECB in a black-box scenario: submit 48 bytes of identical input (3 identical 16-byte blocks). If two ciphertext blocks are identical in the output, ECB is in use
- In image encryption, ECB mode visually preserves structure; other modes like CBC produce visually indistinguishable ciphertext

---

## Conclusion

ECB mode represents a fundamental cipher design failure — its independence of blocks is its flaw. The moment identical plaintext blocks produce identical ciphertext, an attacker gains structural information about the plaintext without ever decrypting anything. ECB oracle attacks exploit this to systematically recover secret data one byte at a time using only access to an encryption endpoint. The lesson is clear: choosing the right cipher mode is as critical as choosing the right cipher.
