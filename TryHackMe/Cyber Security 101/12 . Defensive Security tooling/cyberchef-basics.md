# CyberChef: The Basics

## Overview

CyberChef is a browser-based data transformation tool developed by GCHQ. It handles a wide range of encoding, decoding, encryption, hashing, extraction, and format conversion tasks — all without leaving the browser. In security work, it is constantly used to decode obfuscated data, manipulate artefacts, and understand encoding schemes found in malware, phishing payloads, and forensic evidence. This room introduces CyberChef's interface, its core operation categories, and the foundational concept of Base64 encoding.

---

## Topics Covered

- CyberChef's four interface areas
- Operations: encoding, decoding, extraction, date/time, data format
- Recipes: chaining operations
- Base64 encoding mechanics explained step by step
- URL encoding and decoding
- Extractors: IP addresses, URLs, email addresses
- UNIX timestamp conversion

---

## Key Concepts

### CyberChef Interface

CyberChef is structured around four areas:

| Area | Purpose |
|------|---------|
| **Operations** | A searchable repository of all available transformations, organised by category |
| **Recipe** | The working area where operations are selected, arranged, and configured. Operations execute in sequence from top to bottom |
| **Input** | Where data is entered — by typing, pasting, or dragging a file |
| **Output** | Displays the result of applying the recipe to the input |

The **Recipe** is the core of the tool. Each operation can be configured with specific arguments. Recipes can be saved, loaded, and shared.

---

### Operations Categories

#### Encoding / Decoding

| Operation | Description | Example |
|-----------|-------------|---------|
| From Morse Code | Translates Morse Code to uppercase alphanumeric characters | `- .... .-. . .- - ...` → `THREATS` |
| URL Encode | Encodes special characters to percent-encoding | `https://tryhackme.com/` → `https%3A%2F%2Ftryhackme%2Ecom%2F` |
| To Base64 | Encodes raw data to an ASCII Base64 string | `This is fun!` → `VGhpcyBpcyBmdW4h` |
| To Hex | Converts a string to hexadecimal bytes | `This` → `54 68 69 73` |
| To Decimal | Converts input to an ordinal integer array | `This` → `84 104 105 115` |
| ROT13 | Caesar cipher rotating characters by 13 positions | `Digital Forensics` → `Qvtvgny Sberafvpf` |

#### Data Format Operations

| Operation | Description | Example |
|-----------|-------------|---------|
| From Base64 | Decodes Base64 back to raw data | `V2VsY29tZSB0byB0cnloYWNrbWUh` → `Welcome to tryhackme!` |
| URL Decode | Converts percent-encoded characters back to raw values | `https%3A%2F%2Fgchq%2Egithub%2Eio` → `https://gchq.github.io` |
| From Base85 | Decodes Base85-encoded data (more efficient than Base64) | `BOu!rD]j7BEbo7` → `hello world` |
| From Base58 | Decodes Base58 (avoids visually ambiguous characters: `l`, `I`, `0`, `O`) | `AXLU7qR` → `Thm58` |
| To Base62 | Encodes to Base62 (compact, alphanumeric only) | `Thm62` → `6NiRkOY` |

**Base encodings** convert binary data into a text representation using a defined ASCII character set. They are not encryption — they are reversible transformations used to make binary data safe for text-based transmission.

#### Extractors

| Operation | Purpose |
|-----------|---------|
| Extract IP addresses | Pulls all valid IPv4 and IPv6 addresses from the input |
| Extract URLs | Extracts URLs — protocol (`http://`, `ftp://`, etc.) must be present to avoid false positives |
| Extract email addresses | Extracts all strings matching the `anything@domain.com` format |

Extractors are useful for quickly pulling indicators of compromise (IOCs) from raw logs, email headers, or document content.

#### Date and Time

| Operation | Description |
|-----------|-------------|
| To UNIX Timestamp | Parses a datetime string in UTC and returns the corresponding UNIX timestamp |
| From UNIX Timestamp | Converts a UNIX timestamp to a human-readable datetime string |

A UNIX timestamp is a 32-bit integer representing the number of seconds elapsed since January 1, 1970 UTC (the UNIX epoch). Example: `Fri Sep 6 20:30:22 +04 2024` → `1725654622`.

---

### URL Encoding Reference

Common characters and their percent-encoded equivalents (UTF-8):

| Character | Encoded |
|-----------|---------|
| `:` | `%3A` |
| `/` | `%2F` |
| `.` | `%2E` |
| `=` | `%3D` |
| `#` | `%23` |

---

### How Base64 Encoding Works (Manual Walkthrough)

Base64 is one of the most commonly encountered encodings in security work — found in email attachments, JWT tokens, malware payloads, and encoded commands. Understanding the mechanics helps when automated tools are unavailable.

**Example: Encode `THM` to Base64**

**Step 1 — Convert each character to binary using ASCII values:**

| Character | ASCII Decimal | Binary (8-bit) |
|-----------|--------------|----------------|
| T | 84 | 01010100 |
| H | 72 | 01001000 |
| M | 77 | 01001101 |

Concatenate: `010101000100100001001101` (24 bits total)

**Step 2 — Split into 6-bit groups:**

```
010101 | 000100 | 100001 | 001101
```

Convert each 6-bit group to decimal:

| Binary | Decimal |
|--------|---------|
| 010101 | 21 |
| 000100 | 4 |
| 100001 | 33 |
| 001101 | 13 |

**Step 3 — Look up each decimal value in the Base64 index table:**

| Index | Character |
|-------|-----------|
| 21 | V |
| 4 | E |
| 33 | h |
| 13 | N |

Result: **`THM`** in Base64 is **`VEhN`**

This is why Base64 is not encryption — given the output, the input can always be recovered by reversing the process.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Recipe | A sequence of operations in CyberChef applied to input data in order |
| Operation | A single transformation function (encode, decode, extract, convert, etc.) |
| Base64 | A binary-to-text encoding using 64 ASCII characters (A–Z, a–z, 0–9, +, /) |
| Base58 | Encoding that removes visually ambiguous characters (`l`, `I`, `0`, `O`) — used in Bitcoin addresses |
| Percent-encoding | URL encoding scheme where special characters are replaced with `%XX` hex codes |
| UNIX timestamp | Integer count of seconds since January 1, 1970 UTC |
| IOC | Indicator of Compromise — observable artefact indicating potential malicious activity (IP, URL, hash, etc.) |
| ROT13 | Caesar cipher shifting each letter 13 positions — self-inverse (applying it twice returns the original) |

---

## Practical Examples

### Decode a Base64 string

Input: `V2VsY29tZSB0byB0cnloYWNrbWUh`
Operation: `From Base64`
Output: `Welcome to tryhackme!`

### Decode a URL-encoded string

Input: `https%3A%2F%2Fgchq%2Egithub%2Eio%2FCyberChef%2F`
Operation: `URL Decode`
Output: `https://gchq.github.io/CyberChef/`

### Chain operations (Recipe example)

To decode a string that is Base64-encoded and then ROT13'd:
1. Add `ROT13` as the first operation
2. Add `From Base64` as the second operation
3. Input the encoded string — CyberChef applies both in sequence

### Extract IOCs from a log dump

Paste raw log content into the Input area, add `Extract IP addresses` and `Extract URLs` as operations — CyberChef returns all IPs and URLs found in the data.

---

## Workflow / Process

```
Receive encoded/obfuscated data (from malware, email, log, etc.)
        |
        v
Identify encoding type (Base64, hex, URL, ROT13, etc.)
        |
        v
Open CyberChef (online or offline)
        |
        v
Paste data into Input area
        |
        v
Add appropriate operation(s) to the Recipe
        |
        v
Configure operation arguments if needed
        |
        v
Review Output — chain additional operations if further decoding is needed
        |
        v
Save recipe if it will be reused
```

---

## Real-World Relevance

- Base64-encoded PowerShell commands are one of the most common obfuscation techniques in malware and phishing payloads — CyberChef decodes them in seconds
- URL encoding appears constantly in web application testing and log analysis — understanding percent-encoding is required to read URLs and HTTP requests accurately
- Extractors make IOC extraction from large blocks of raw text (email headers, log files, memory strings) fast and reliable
- UNIX timestamps appear in logs, forensic artefacts, and malware configuration files — `From UNIX Timestamp` makes them immediately readable
- Chained recipes in CyberChef replicate multi-stage deobfuscation — attackers often layer encodings (Base64 inside Base64, XOR then Base64, etc.) to evade detection
- CyberChef is used in CTFs extensively — nearly every encoding or decoding challenge can be solved in it

---

## Key Learnings

- CyberChef has four areas: Operations, Recipe, Input, Output
- Operations execute in order from top to bottom in the Recipe
- Base64 converts 8-bit binary groups into 6-bit groups mapped to a 64-character ASCII set — it is encoding, not encryption
- URL encoding replaces special characters with `%XX` hex sequences
- Extractors pull IPs, URLs, and email addresses from raw text automatically
- UNIX timestamps count seconds since January 1, 1970 UTC — `From UNIX Timestamp` converts them to readable dates

---

## Additional Notes

- CyberChef can be run offline by downloading the release from its GitHub repository — useful in air-gapped environments where internet access is restricted
- The `Magic` operation in CyberChef attempts to automatically detect the encoding and suggest the right recipe — useful when the encoding is unknown
- Base64 strings are identifiable by their character set (A–Z, a–z, 0–9, +, /) and the `=` padding at the end
- ROT13 is its own inverse — applying `ROT13` twice returns the original text

---

## Conclusion

CyberChef is an indispensable utility in security work. Its recipe-based approach to chaining data transformations makes it equally useful for quick one-off decoding and for building repeatable multi-step analysis workflows. Understanding the core encoding schemes it handles — particularly Base64 and URL encoding — is foundational knowledge for anyone working in malware analysis, incident response, web application security, or CTF competitions.
