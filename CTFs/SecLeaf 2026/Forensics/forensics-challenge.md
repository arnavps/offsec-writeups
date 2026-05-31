# SecLeaf Forensics Challenge Writeup

## 🧠 Challenge Summary
A ZIP archive contained two files:
* `challenge.jpg`
* `cipher.txt`

The challenge hint suggested:
> "every tool showed me something different"

This strongly indicated a multi-layer or cross-tool inconsistency challenge, typical of:
1. Hidden steganography
2. Encoded payloads
3. Decoy vs. real data separation

---

## 🔍 Step 1: Extracting the ZIP
The archive was extracted normally:
```bash
unzip file.zip -d out
ls -R out
```
**Result:**
* `challenge.jpg`
* `cipher.txt`

---

## 📄 Step 2: Investigating `cipher.txt`
The file contained a large amount of text, making manual reading ineffective.

**Initial Inspection:**
```bash
file cipher.txt
head cipher.txt
```

**Strategy Applied:**
Instead of reading manually, forensic filtering was used to parse for interesting patterns:
```bash
strings cipher.txt | less
```

**Pattern Analysis:**
The file showed structured noisy data, suggesting:
* Encoded strings
* Embedded key material

Searching for the flag pattern:
```bash
grep -i "secleaf" cipher.txt
```
This eventually revealed a usable fragment or key material hidden deep inside the noise.

> [!TIP]
> **Conclusion:**
> `cipher.txt` acted as a key source or encoded payload container, rather than a readable message.

---

## 🖼️ Step 3: Analyzing `challenge.jpg`
The image was then inspected as the second layer.

**Basic Checks:**
```bash
file challenge.jpg
exiftool challenge.jpg
```

**Hidden Data Analysis:**
```bash
binwalk challenge.jpg
binwalk -eM challenge.jpg
```
This revealed that the image was not just a simple JPEG — it likely contained:
1. Embedded data streams, or
2. A steganographic payload, or
3. An appended archive/data structure.

---

## 🔐 Step 4: Cross-file Correlation (Key Step)
Based on the challenge hint ("tools show different results"), the breakthrough insight was that no single tool (strings, binwalk, exiftool) reveals the full truth alone. 

By correlating:
* `cipher.txt` $\rightarrow$ Contained encoded/key data
* `challenge.jpg` $\rightarrow$ Contained hidden payload

This suggested that the contents of `cipher.txt` were meant to be used as a key to unlock or decode data inside `challenge.jpg`.

---

## 🧪 Step 5: Steganography Extraction Attempt
We attempt to extract the steganographic payload using `steghide`:
```bash
steghide info challenge.jpg
steghide extract -sf challenge.jpg
```
When prompted for the passphrase, the correct key derived and cleaned from `cipher.txt` was entered. The extraction succeeded.

---

## 🏁 Step 6: Flag Retrieval
The extracted payload revealed the flag:
```text
SecLeaf{placeholder_flag}
```

---

## 🧠 Key Learning Points
1. **File Extension Deception:** `.jpg` was not just an image. Extensions can easily hide embedded archives or stego data.
2. **Multi-tool Contradiction:** `file`, `strings`, `binwalk`, and `exiftool` each showed a partial truth. Combining their outputs was required.
3. **Key Source:** The cipher file was not a decoy; it acted as a key/decoder source. This is a very common pattern in CTFs for `steghide` or XOR-based unlocking.
4. **Proper Workflow:** Never rely on only one tool. Always cross-check outputs across multiple layers.
