# Hidden File Format Analysis

## 🧩 Challenge Description
A suspicious image file was recovered during an investigation. At first glance, it appears to be a normal JPEG, but analysts suspect hidden data or file manipulation.

**Objective:** Identify the true file format and extract the hidden flag.

**Flag Format:** `SecLeaf{}`

---

## 📁 Initial Recon
We start by analyzing the provided file:
```bash
file Important.jpg
ls -lh Important.jpg
```
The file initially appears to be a JPEG image based on its extension, but a deeper inspection is required.

---

## 🧪 Metadata Analysis
Using ExifTool:
```bash
exiftool Important.jpg
```

**Key Finding:**
* **File Type:** `ZIP`
* **MIME Type:** `application/zip`
* **Zip File Name:** `flag.txt`

This is our critical clue: the file is not actually a JPEG. It is a ZIP archive disguised as an image.

---

## 🧠 Analysis Insight
This is a classic file extension spoofing attack:
* The extension is `.jpg`.
* The internal structure is a `ZIP` archive.
* Likely used to bypass casual inspection or email/upload filters.

This technique is extremely common in CTF challenges, malware delivery, and basic evasion tricks.

---

## 📦 Extraction
We rename and extract the archive:
```bash
mv Important.jpg Important.zip
unzip Important.zip
```

**Output:**
```text
inflating: flag.txt
```

---

## 🚩 Flag Retrieval
```bash
cat flag.txt
```

**Flag:**
```text
SecLeaf{placeholder_flag}
```

---

## 🧾 Conclusion
This challenge demonstrates the importance of verifying file types programmatically rather than relying on file extensions.

**Key Takeaways:**
1. Always verify file type using commands like `file` and `exiftool`.
2. File extensions can be completely misleading.
3. ZIP/JPEG spoofing is a common CTF trick to hide archives inside image structures.
4. Metadata often reveals the true structure of spoofed files.

---

## 🛠 Useful Commands Used
* `file <file>`
* `exiftool <file>`
* `strings <file>`
* `binwalk -e <file>`
* `unzip <file>`
