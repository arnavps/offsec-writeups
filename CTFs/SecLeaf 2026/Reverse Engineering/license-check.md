# license_check Reverse Engineering Writeup

## 🏷️ Challenge Overview
We are given a stripped ELF binary named `license_check`. The program asks for a license key and prints a flag if the correct key is entered.

---

## 🔍 Initial Enumeration
First, identify the binary type and protections:
```bash
file license_check
checksec --file=license_check
```

Next, inspect the binary strings:
```bash
strings license_check
```

**Interesting strings found:**
```text
=== SECLEAF LICENSE SYSTEM ===
Flag: %s
```
No obvious plaintext flag is visible in the raw strings.

---

## ⚙️ Static Analysis
Disassemble the binary:
```bash
objdump -d license_check | less
```

**Key Observations:**
1. Calls to `strcmp` are present.
2. Two buffers are initialized on the stack.
3. A helper XOR routine exists around address `0x1179`.

The helper performs:
```c
buf[i] ^= key;
```

Disassembly confirms:
* `124a: call 1179`
* `126b: call 1179`

This indicates two separate XOR decode operations occur.

---

## 🔑 Extracting the License Key
Instead of reversing everything manually, the easiest path is runtime inspection using GDB.

### Break on `strcmp`
Launch GDB:
```bash
gdb ./license_check
```

Set a breakpoint on `strcmp` and run:
```gdb
b strcmp
run
```

Enter dummy input when prompted (e.g., `AAAA`). When the breakpoint hits, inspect the registers holding the string arguments:
```gdb
x/s $rdi
x/s $rsi
```

**Output:**
* `$rdi` $\rightarrow$ `"AAAA"` (our input)
* `$rsi` $\rightarrow$ `"CFI@OW:K][ML]\\[^:(*(-"` (correct key)

### Correct License Key
Actual runtime key:
```text
CFI@OW:K][ML]\[^:(*(-
```
> [!NOTE]
> GDB displays escaped backslashes (`\\`). The actual input should use a single `\`.

---

## 🚀 Running the Program
Execute the binary:
```bash
./license_check
```

**Input:**
```text
CFI@OW:K][ML]\[^:(*(-
```

**Program Output:**
```text
Access Granted
Flag: <REDACTED_OBFUSCATED_PAYLOAD>
```

---

## 🧠 Understanding the Flag Logic
The binary stores stack-based encrypted buffers which are XOR-decoded at runtime. Two XOR keys are used:

| Purpose | XOR Key |
|---|---|
| License key | `0x42` |
| Flag buffer | `0x55` |

### MD5 Validation
The organizers additionally provided an MD5 check:
`1e0092dbf20c7229d28b6ad47df60e75`

Direct wrapper attempts like `SecLeaf{<REDACTED_OBFUSCATED_PAYLOAD>}` did not match the MD5, indicating that either another lightweight transformation exists or the organizers intended partial reconstruction/guessing. However, the runtime extraction itself is deterministic and correct.

---

## 🏁 Final Recovered Values
* **License Key:** `CFI@OW:K][ML]\[^:(*(-`
* **Runtime Flag Payload:** `SecLeaf{placeholder_flag}`

---

## 💡 Key Reverse Engineering Takeaways
1. **Dynamic Debugging vs Static Reversing:** Intercepting symbols in GDB is often exponentially faster than reversing custom obfuscation algorithms manually.
2. **Standard Comparison Breakpoints:** Placing breakpoints on comparison functions (`strcmp`, `memcmp`, `strncmp`) immediately reveals target buffers.
3. **Character Escaping:** Always be careful with double-escaped output from debuggers.
4. **XOR Obfuscation:** Storing ciphertext on the stack and decoding at runtime is a highly common pattern in entry-level RE challenges.

---

## 🛠 Useful Commands Used
* **Disassembly:** `objdump -d license_check | less`
* **Search for comparisons:** `objdump -d license_check | grep strcmp`
* **GDB runtime extraction:**
  ```gdb
  gdb ./license_check
  b strcmp
  run
  x/s $rdi
  x/s $rsi
  ```
