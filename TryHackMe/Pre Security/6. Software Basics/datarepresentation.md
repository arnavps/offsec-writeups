# Data Representation

## Overview

Computers don't understand text, images, or numbers the way humans do — everything is ultimately stored and processed as binary. This room covers the numbering systems computers use to represent data, how those systems relate to each other, and how colour is encoded at the binary level. This knowledge underpins everything from reading hex dumps to understanding memory addresses and colour values in web security.

---

## Topics Covered

- Decimal, binary, hexadecimal, and octal numbering systems
- Bits and bytes
- Hex colour representation

---

## Key Concepts

### Numbering Systems

| System | Base | Digits Used | Notes |
|---|---|---|---|
| Decimal | 10 | 0–9 | The system humans use in everyday life |
| Binary | 2 | 0, 1 | The native language of computers — represents two electrical states |
| Hexadecimal | 16 | 0–9, A–F | Every 4 binary digits (bits) map to one hex digit |
| Octal | 8 | 0–7 | Every 3 binary digits map to one octal digit; less commonly used |

**Why hexadecimal?**
Binary values get long quickly. A single byte (8 bits) requires 8 binary digits but only 2 hex digits. Hex is a compact, human-readable way to represent binary data — used everywhere from memory addresses to colour codes to packet dumps.

**Example conversions:**

| Decimal | Binary | Hexadecimal |
|---|---|---|
| 0 | 0000 | 0 |
| 10 | 1010 | A |
| 15 | 1111 | F |
| 16 | 00010000 | 10 |
| 255 | 11111111 | FF |

---

### Bits and Bytes

**Bit** — the smallest unit of data in computing. Short for binary digit. Can be either `0` or `1`.

**Byte** — a group of 8 bits. Also called an octet. The standard unit of data storage and memory addressing on modern systems.

```
1 byte = 8 bits
Example: 01000001 = the letter 'A' in ASCII
```

---

### Hex Colour Representation

Colours on computer screens are represented using the RGB model — a combination of Red, Green, and Blue values. Each channel is assigned one byte (8 bits), giving a range of 0–255 per channel.

In hex colour notation, each channel is written as two hex digits:

```
#RRGGBB
```

| Colour | Hex Code | R | G | B |
|---|---|---|---|---|
| Red | `#FF0000` | 255 | 0 | 0 |
| Green | `#00FF00` | 0 | 255 | 0 |
| Blue | `#0000FF` | 0 | 0 | 255 |
| White | `#FFFFFF` | 255 | 255 | 255 |
| Black | `#000000` | 0 | 0 | 0 |

With 1 byte per channel and 3 channels, the total number of possible colours is:

```
256 × 256 × 256 = 16,777,216 (over 16 million colours)
```

---

## Important Terminology

| Term | Definition |
|---|---|
| Bit | Binary digit — the smallest unit of data; either 0 or 1 |
| Byte | 8 bits; the standard unit of memory and storage |
| Octet | Another name for a byte (8 bits) |
| Binary | Base-2 numbering system using only 0 and 1 |
| Hexadecimal | Base-16 numbering system using digits 0–9 and letters A–F |
| Octal | Base-8 numbering system using digits 0–7 |
| RGB | Red, Green, Blue — the colour model used by computer displays |
| Hex Colour | A 6-digit hex value representing an RGB colour (`#RRGGBB`) |

---

## Practical Examples / Demonstrations

### Binary to Hex conversion

Group binary digits into sets of 4, then convert each group:

```
Binary:  1010 1111
Hex:     A    F
Result:  0xAF
```

### Reading a hex colour

```
#1A2B3C
  R = 1A = 26
  G = 2B = 43
  B = 3C = 60
```

---

## Real-World Relevance

- Hex is used everywhere in security: memory addresses, shellcode, packet captures, hash values, and cryptographic keys are all displayed in hex
- Understanding binary and hex is required for reading disassembly output, analysing malware, and working with raw network data
- Hex colour codes appear in web application source code — understanding them helps when reviewing frontend code
- Binary representation of data is fundamental to understanding how buffer overflows, bit manipulation, and encoding attacks work

---

## Key Learnings

- Computers operate in binary — all data is ultimately 0s and 1s
- Hex is a compact representation of binary — 2 hex digits = 1 byte
- 1 byte = 8 bits; the standard unit of data on modern systems
- RGB colours use 1 byte per channel, producing over 16 million possible combinations
- Octal is less common but still appears in Unix file permission notation (e.g., `chmod 755`)

---

## Additional Notes

- The `0x` prefix denotes a hexadecimal value in most programming contexts (e.g., `0xFF`)
- Unix file permissions are expressed in octal: `rwxr-xr-x` = `755` in octal
- Network masks, port numbers, and IP addresses are all ultimately binary values

---

## Conclusion

Data representation is the lowest-level foundation of computing. Understanding how binary, hex, and decimal relate to each other — and how data like colours is encoded — makes it significantly easier to read technical output, understand memory layouts, and work with raw data in security contexts.
