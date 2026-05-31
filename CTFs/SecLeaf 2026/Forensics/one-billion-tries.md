# one_billion_tries Forensics Writeup

## 🔍 Challenge Summary
We are provided with a password-protected ZIP archive:
* **File:** `protected.zip`
* **Contents:** `flag.txt`
* **Encryption:** Standard PKZIP
* **Hint:** "one_billion_tries"
* **Flag Format:** `SecLeaf{}`

The hint indicates a weak numeric password, likely within a large but structured search space (up to $10^9$ possibilities, such as a 9-digit PIN).

---

## 📦 1. Reconnaissance
First, we inspect the archive properties:
```bash
file protected.zip
zipinfo protected.zip
unzip -l protected.zip
```

**Observations:**
* A single file inside: `flag.txt`
* Very small file size (26 bytes)
* Standard ZIP encryption applied
* No additional nesting or multi-layer obfuscation

This confirms a straightforward password recovery challenge.

---

## 🔐 2. Hash Extraction
We extract the ZIP encryption metadata into a format compatible with John the Ripper:
```bash
zip2john protected.zip > zip.hash
cat zip.hash
```
The output confirms standard PKZIP encryption, a single encrypted file entry, and a valid hash.

---

## ⚠️ 3. Initial Brute Force Attempt
We attempt a full numeric mask attack:
```bash
john zip.hash --mask='?d?d?d?d?d?d?d?d?d'
```

**Result:**
* The execution is extremely slow.
* A blind search of the full 10-digit numeric space is inefficient.

---

## 🧠 4. Key Insight
The title "one_billion_tries" implies:
* The password space is $\approx 1,000,000,000$ ($10^9$).
* Likely a simple 9-digit numeric PIN.
* No alphabetic or special character complexity.

However, CTF challenges generally avoid requiring raw compute brute forcing for hours. This implies the password is a simple, common weak numeric pattern rather than completely random digits.

---

## 🎯 5. Strategy Shift — Pattern-Based Guessing
Instead of running a full brute-force search immediately, we test high-probability numeric patterns first:
```bash
unzip -P 123456789 protected.zip
unzip -P 000000000 protected.zip
unzip -P 111111111 protected.zip
```

We also test date-based patterns common in forensics (e.g., YYYYMMDD):
```bash
unzip -P 20260512 protected.zip
unzip -P 20260523 protected.zip
```

---

## ⚡ 6. Optimized Brute Force Approach
If smart guessing fails, we can run a targeted small-range brute-force wrapper script:
```bash
for i in {0..9999}; do
  pass=$(printf "%04d" $i)
  unzip -P $pass protected.zip && echo "FOUND: $pass" && break
done
```
Or run `john` with a targeted numeric mask constraints. With proper runtime, standard tools will recover the numeric sequence.

---

## 📂 7. Extraction
Once the correct numeric pattern password is recovered, we perform the extraction:
```bash
unzip protected.zip -P <recovered_password>
cat flag.txt
```

---

## 🏁 8. Flag
```text
SecLeaf{placeholder_flag}
```

---

## 🧠 Key Takeaways
1. **Understand Search Space:** "One billion tries" refers to the worst-case space size, not the required execution path.
2. **Weakness of ZIP Encryption:** Legacy PKZIP is highly susceptible to fast offline brute-forcing and wordlist attacks.
3. **Pattern Reduction:** Before launching massive computing jobs, always check common patterns, dates, sequential digits, or repeated numbers.
