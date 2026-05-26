# Data Encoding

## Overview

Encoding is how computers represent text as numbers. ASCII was the original standard for English text, but it couldn't handle the world's languages. Unicode solved this by creating a universal character set, with UTF-8, UTF-16, and UTF-32 as the encoding formats that implement it. Understanding encoding is relevant to security — encoding mismatches cause garbled text, and encoding schemes are frequently abused in injection attacks and obfuscation.

---

## Topics Covered

- ASCII and its limitations
- Extended ASCII and the chaos it caused
- Unicode as the universal solution
- UTF-8, UTF-16, and UTF-32
- Emoji encoding
- What causes garbled/gibberish characters

---

## Key Concepts

### ASCII

ASCII (American Standard Code for Information Interchange) is a 7-bit character encoding standard that defines 128 characters. It covers:

- English uppercase and lowercase letters (A–Z, a–z)
- Digits (0–9)
- Basic punctuation and control characters

**Key ASCII values:**

| Decimal | Hex | Character |
|---|---|---|
| 48 | 30 | `0` |
| 57 | 39 | `9` |
| 65 | 41 | `A` |
| 90 | 5A | `Z` |
| 97 | 61 | `a` |
| 122 | 7A | `z` |
| 127 | 7F | DEL |

ASCII uses 7 bits, giving exactly 128 possible values (0–127).

---

### The Problem with ASCII

ASCII only covers English. It has no room for characters like `ñ`, `€`, `あ`, or `ب`. Extended ASCII attempted to fix this by using the 8th bit to add 128 more characters — but different regions created different standards:

- ISO-8859-1 (Latin 1) — Western European
- ISO-8859-2 (Latin 2) — Central European
- Windows-1252 — Microsoft's Western European variant

**The problem:** These standards assigned different characters to the same byte values. If a file was saved using ISO-8859-1 and opened with ISO-8859-2, characters would display incorrectly. For example, `Ø` (ISO-8859-1) would appear as `Ř` (ISO-8859-2).

This created a fundamental rule: the encoding used to read a file must match the encoding used to write it.

---

### Why a Universal Standard Was Needed

The scale of the problem becomes clear when you consider character counts across languages:

| Language | Character Count |
|---|---|
| English | 52 (upper + lowercase) |
| Arabic | 250+ (ligatures and diacritics) |
| Japanese (daily use Kanji) | 2,136 (JIS X 0208 defines 6,879) |
| Chinese (educated recognition) | ~8,000 (GB 18030-2022 defines 87,887+) |

Add emoji, mathematical symbols, historical scripts, and currency symbols — no regional encoding standard could cover all of this.

---

### Unicode

Unicode is a universal character encoding standard that assigns a unique **code point** to every character across all modern and historical writing systems. A code point is written as `U+` followed by a hex value.

**Examples:**

| Code Point | Character | Description |
|---|---|---|
| U+0041 | A | Latin uppercase A |
| U+03A9 | Ω | Greek capital Omega |
| U+3042 | あ | Japanese Hiragana |
| U+0628 | ب | Arabic letter Ba |
| U+1F600 | 😀 | Grinning face emoji |

Unicode defines over 1.1 million possible code points, of which around 150,000 are currently assigned.

**Key benefit:** Because Unicode is a universal standard, both sender and recipient use the same encoding. There's no need to specify or match regional encoding variants.

---

### UTF-8, UTF-16, and UTF-32

Unicode defines the character set (what code points exist). UTF encodings define how those code points are stored as bytes.

| Encoding | Unit Size | Notes |
|---|---|---|
| UTF-8 | Variable (1–4 bytes) | Backward compatible with ASCII; most efficient for English text; dominant on the web |
| UTF-16 | Variable (2 or 4 bytes) | Used internally by Windows and Java; more efficient for non-Latin scripts |
| UTF-32 | Fixed (4 bytes) | Simple but space-inefficient; every character uses 4 bytes regardless |

**UTF-8** is the most widely used encoding on the internet. ASCII characters (U+0000 to U+007F) are stored as a single byte in UTF-8, making it fully backward compatible with ASCII.

---

### Emoji Encoding

Emoji are standard Unicode characters with assigned code points. They are encoded like any other character using UTF-8, UTF-16, or UTF-32.

Example: 😀 = U+1F600

In UTF-8, this encodes to 4 bytes: `F0 9F 98 80`

The display of emoji depends on the font and platform — the encoding is standardised, but the visual rendering varies.

---

### What Causes Garbled Characters?

Garbled or gibberish characters (often called mojibake) occur when text encoded in one format is decoded using a different format.

Common causes:
- A file saved as UTF-8 opened as ISO-8859-1
- A database storing UTF-8 data but configured to use Latin-1
- An email client misidentifying the encoding of a message

The fix is always to ensure the reading encoding matches the writing encoding — or to use UTF-8 universally.

---

## Important Terminology

| Term | Definition |
|---|---|
| ASCII | 7-bit encoding standard for English characters (128 values) |
| Extended ASCII | 8-bit regional variants of ASCII — caused encoding incompatibility |
| Unicode | Universal character set assigning unique code points to all characters |
| Code Point | A unique number assigned to a character in Unicode (e.g., U+0041) |
| UTF-8 | Variable-width Unicode encoding; backward compatible with ASCII |
| UTF-16 | Variable-width Unicode encoding using 2 or 4 bytes per character |
| UTF-32 | Fixed-width Unicode encoding using 4 bytes per character |
| Mojibake | Garbled text caused by encoding/decoding mismatch |
| BOM | Byte Order Mark — a marker at the start of a file indicating encoding and byte order |

---

## Real-World Relevance

- Encoding is directly relevant to injection attacks — URL encoding (`%41` = `A`), HTML encoding (`&lt;` = `<`), and Unicode encoding are all used to bypass input filters and WAF rules
- SQL injection and XSS payloads are frequently encoded to evade detection
- Understanding UTF-8 vs ASCII is important when analysing malware strings, log files, and network captures
- Encoding mismatches in web applications can cause data corruption and unexpected behaviour — a common source of bugs and occasionally exploitable conditions
- Base64 is another encoding scheme (not covered here but closely related) used extensively in web tokens, email attachments, and obfuscated payloads

---

## Key Learnings

- ASCII is a 7-bit standard covering 128 English characters — insufficient for global use
- Extended ASCII created regional variants that caused encoding incompatibility
- Unicode is the universal solution — assigns a unique code point to every character in every language
- UTF-8, UTF-16, and UTF-32 are encoding formats that implement Unicode
- UTF-8 is the dominant encoding on the web and is backward compatible with ASCII
- Garbled characters result from encoding/decoding mismatches

---

## Additional Notes

- UTF-8 is the recommended encoding for all new systems and files
- The HTML `<meta charset="UTF-8">` tag tells the browser which encoding to use when rendering a page
- Python 3 uses Unicode strings by default; Python 2 used ASCII by default — a common source of encoding bugs when migrating code

---

## Conclusion

Encoding is how text becomes bytes. ASCII worked for English but failed globally. Unicode solved the problem by creating a universal character set, and UTF-8 became the standard implementation. For security professionals, encoding knowledge is practical — it's the basis for understanding how injection payloads are obfuscated and how text-based attacks bypass filters.
