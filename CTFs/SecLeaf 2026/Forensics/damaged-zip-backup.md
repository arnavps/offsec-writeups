# Damaged ZIP Backup Writeup

## 🔍 Initial Observation
The file provided was `backup.zip`. Basic inspection showed:
```bash
file backup.zip
zipinfo backup.zip
unzip -t backup.zip
```

**Findings:**
* The ZIP structure is present.
* Contains one file: `flag.txt`.
* However, the integrity check fails with the error: `bad zipfile offset (local header sig)`.

This indicates corruption specifically in the local header, rather than complete archive loss.

---

## 🧪 Attempted Automatic Repair
Using standard ZIP recovery utilities:
```bash
zip -FF backup.zip --out fixed.zip
```

**Result:**
* Central directory was successfully detected.
* However, no usable local entry could be recovered.
* The output ZIP file was effectively empty.

Automatic reconstruction tools failed to resolve the issue.

---

## 🧠 Deeper Inspection (Key Step)
We directly analyzed the raw bytes of the archive:
```bash
xxd backup.zip
```

**Critical Discovery:**
The local header signature was corrupted at the very beginning of the file:
* **Corrupted:** `00 00 03 04` ❌
* **Expected (ZIP standard):** `50 4B 03 04` ✅ (representing `PK\x03\x04`)

We also noticed the plaintext flag-like data embedded in the payload section of the raw bytes:
`SecLeaf{repair_the_archive}`

---

## 🛠️ Manual Repair
We manually fixed the magic bytes of the ZIP signature using `dd`:
```bash
printf '\x50\x4b' | dd of=repaired.zip bs=1 seek=0 count=2 conv=notrunc
```

After performing this correction, standard extraction succeeded:
```bash
unzip repaired.zip
```

**Output:**
```text
Archive:  repaired.zip
  inflating: flag.txt
```

---

## 📄 Extracted File Analysis
```bash
cat flag.txt
xxd flag.txt
```

**Result:**
No hidden bytes, no extra encoding, and no second layer. The file directly contains the flag.

---

## 🧩 Root Cause of "Damage"
The challenge authors intentionally corrupted the first two bytes of the ZIP local header (`PK`). This caused standard extraction tools to fail immediately, even though the raw payload remained completely intact and uncorrupted inside the archive.

---

## 🏁 Final Flag
```text
SecLeaf{placeholder_flag}
```

---

## 💡 Key Learning
* ZIP files rely heavily on magic bytes (`PK` or `50 4B`) to orient parsing engines.
* Even minor 2-byte corruptions can completely break standard extraction utilities.
* Manual hex repair is a very quick and effective way to bypass automated tool limitations in CTF forensics challenges.
