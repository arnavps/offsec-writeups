# SecLeaf Q2 CTF 2026 — Writeups

![SecLeaf Q2 CTF 2026 Banner](image_bfb6de.jpg)

> **CAPTURE. EXPLOIT. DOMINATE.**
> A repository dedicated to my solutions, scripts, and detailed writeups for the challenges solved during the **SecLeaf Q2 CTF 2026**.

---

## 🌐 Event Overview

SecLeaf Q2 CTF 2026 is a fast-paced, jeopardy-style cybersecurity competition designed to test real-world hacking skills across multiple domains, from exploiting web vulnerabilities to reverse-engineering binaries.

* **Official Website:** [SecLeaf CTF Portal](https://ctf.secleaf.com)
* **Community & Updates:** [Discord Server](https://discord.gg/6y7phme2En) | [WhatsApp Group](https://chat.whatsapp.com/JGB9ObwYof1CHjBnY19FBn)

---

## 🛠️ Challenge Categories

The competition featured challenges across these core cybersecurity disciplines. The categories I've documented writeups for are marked below:

- [ ] 🕸️ **Web Security** — Flaws in web applications (XSS, SQLi, IDOR, SSRF, etc.)
- [x] 🔐 **Cryptography** — Breaking ciphers, hashing flaws, and mathematical exploits
- [x] 🔍 **Digital Forensics** — Analyzing memory dumps, packet captures (PCAPs), and log files
- [x] ⚙️ **Reverse Engineering** — Decompiling binaries, analyzing control flow, and bypassing checks
- [x] 💥 **Binary Exploitation** — Stack overflows, heap exploitation, and pwnables
- [ ] 🕵️‍♂️ **OSINT** — Open-source intelligence gathering and tracking digital footprints
- [x] 🖼️ **Steganography** — Extracting hidden data from images, audio, and file formats
- [ ] 🧩 **Miscellaneous Puzzles** — Scripting, automation, and general logic puzzles

---

## 📂 Repository Structure

The writeups are organized cleanly by CTF category. Each category directory contains deep-dive step-by-step forensic analysis, methodology descriptions, tools used, and solution highlights.

```text
SecLeaf 2026/
├── Cryptography/
│   └── layered-transmission.md          # Multi-layer Hex, Base64, and ROT13 decoding
├── Forensics/
│   ├── damaged-zip-backup.md            # Manual magic byte repair of corrupted ZIP archives
│   ├── force-push-wont-save-you.md      # Git forensics, dangling commit/blob extraction
│   ├── forensic-log-recovery.md         # Hex fragment matching and semantic log reconstruction
│   ├── forensics-challenge.md           # Multi-tool steganography and steghide extraction
│   ├── hidden-file-format-analysis.md   # Spoofed metadata extension analysis (JPG to ZIP)
│   ├── look-at-the-structure.md         # Transition point analysis and ASCII art geometry
│   ├── one-billion-tries.md             # PKZIP PIN cracking optimization & mask logic
│   ├── theinvoiceincident.md            # Macro phishing & endpoint event correlation report
│   └── vector-asset-steganography.md    # Extracting base64 streams from SVG XML nodes
├── Pwn/
│   └── ret2win.md                       # Stack overflow & alignment gadget exploitation
└── Reverse Engineering/
    ├── license-check.md                 # strcmp intercept & runtime XOR decryption in GDB
    └── wrong-turn.md                    # Executable unpacking (UPX) and static decoy analysis
```

---

## 🚀 Solved Writeups Index

### 🔐 Cryptography
* [layered-transmission.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Cryptography/layered-transmission.md) — Walkthrough of multi-step decoding of a hexadecimal stream into standard Base64 and a ROT13 cipher.

### 🔍 Digital Forensics & Steganography
* [forensic-log-recovery.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/forensic-log-recovery.md) — Analyzing authentications and debugging log corpora to filter hex data from decoy logs.
* [force-push-wont-save-you.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/force-push-wont-save-you.md) — Extracting hidden dangling objects and trees in Git DB after multiple force-pushes.
* [hidden-file-format-analysis.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/hidden-file-format-analysis.md) — Revealing underlying file extensions by checking JPG signatures and extracting embedded files.
* [forensics-challenge.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/forensics-challenge.md) — Multi-tool stego decryption pairing structural ciphertext arrays with image payloads.
* [damaged-zip-backup.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/damaged-zip-backup.md) — Recovering unreadable zip file local header offsets through binary signature correction.
* [look-at-the-structure.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/look-at-the-structure.md) — Structural coordinate plotting of rasterized ASCII-art diagonal boundaries.
* [vector-asset-steganography.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/vector-asset-steganography.md) — Locating hidden Base64 payloads and hidden CSS layer properties inside SVG files.
* [one-billion-tries.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/one-billion-tries.md) — Extracting zip hash arrays and optimizing mask brute forcing using John the Ripper.
* [theinvoiceincident.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Forensics/theinvoiceincident.md) — Incident report tracking VBA macros, base64-encoded powershell spawning, and C2 staging.

### ⚙️ Reverse Engineering
* [license-check.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Reverse%20Engineering/license-check.md) — Intercepting arguments on standard library comparators inside GDB for a stripped ELF.
* [wrong-turn.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Reverse%20Engineering/wrong-turn.md) — Decompressing UPX binaries and extracting target XOR arrays to bypass static decoys.

### 💥 Binary Exploitation (Pwn)
* [ret2win.md](file:///c:/Users/Arnav%20Shirwadkar/Desktop/OSC/writeups/CTFs/SecLeaf%202026/Pwn/ret2win.md) — Stack overflow exploitation utilizing RIP hijacking and a custom stack-alignment ret gadget.

---

## 🏆 Key Takeaways & Highlights

* **Favorite Challenge:** *wrong_turn* — Loved the clever static honey-pot packed string overlay design, which highlighted the danger of trusting static analysis outputs directly on packed binaries.
* **New Tools & Techniques Learned:** Advanced dynamic interception of standard library calls in GDB, scriptable ZIP/PNG repair, and extracting coordinate-transition runs from custom visual layouts.
* **Core Takeaway:** Verify all assumptions; extensions lie, Git histories are easily poisoned, and complex-looking challenges often yield to systematic, step-by-step structural inspection.

---

## 📜 Disclaimer

The writeups and scripts hosted in this repository are strictly for educational and instructional purposes. Do not run any automated exploit scripts against infrastructure you do not own or have explicit, written permission to test.
