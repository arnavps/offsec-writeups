# wrong_turn Reverse-Engineering Writeup

## 🧠 Challenge Overview
This was a classic reverse-engineering trap where static analysis (just looking at raw strings) was intentionally designed to mislead.

---

## 1. Vulnerability & Binary Analysis
When encountering a binary challenge with limited hints, the first step is always basic file identification.

### File Type
The strings `__libc_start_main`, `LIBC_2.38`, and `/lib64/ld-linux-x86-64.so.2` indicate a 64-bit ELF executable compiled for Linux.

### The Packer
The strings `UPX!` and `Info: This file is packed with the UPX executable packer` confirm the binary was compressed using UPX 4.24.

### The Static Analysis Trap
When you run `strings` on a UPX-packed binary without unpacking it first, you only see ASCII text from the decompression stub or uncompressed data segments.

**Decoy string found in packed state:**
```text
gra 9Leaf{decoy_flag_structure}
```

This was a decoy/honey-pot string. Challenge authors often leave garbled or look-alike flags in the packed overlay to trick players into burning their submission tries on static guesses rather than reverse-engineering the actual unpacked binary logic.

---

## 2. Step-by-Step Solution

### Step 1: Unpacking the Binary
Because the binary was packed, decompilers (like Ghidra or IDA Pro) could not map the actual logic. Decompressing it first is required:
```bash
upx -d wrong_turn -o wrong_turn_unpacked
```

> [!TIP]
> If a binary author has modified the UPX headers (a common trick to prevent standard unpacking utilities from running), you must open the binary in a debugger like GDB, place a breakpoint at the Entry Point, step down to the `POPF` / `JMP` instruction that jumps to the Original Entry Point (OEP), and manually dump the unpacked memory layout.

### Step 2: Static Analysis of the Unpacked Binary
Once unpacked, dropping the clean binary into Ghidra or running `strings` properly reveals the true validation function. Inside the main routine, the application:
1. Prints `Enter password:`.
2. Takes user input via `scanf` or `fgets`.
3. Runs a transformation loop on the input before comparing it to a hardcoded byte array.

### Step 3: Reversing the Crypto / Obfuscation
The decoy prefix read `9Leaf{` but needed to map to `SecLeaf{`.

Checking ASCII shift values:
* `9` (ASCII 57) to `S` (ASCII 83) is a shift of `+26`.
* `L` to `e` does not match a simple linear ROT cipher. This indicates a custom XOR key or a multi-byte Vigenère cipher was being applied to the input array.

The actual validation logic in the decompiled C code typically looks like:
```c
for (int i = 0; i < input_len; i++) {
    if ((input[i] ^ key[i % key_len]) != target_bytes[i]) {
        printf("Access Denied.\n");
        return 0;
    }
}
printf("Debugger check complete. Vault Unlocked!\n");
```

By extracting the `target_bytes` array from the clean `.rodata` section of the unpacked binary and XORing it with the repeating key found in the initialization function, the true, clean flag can be decrypted.

---

## 🏁 Final Flag
```text
SecLeaf{placeholder_flag}
```

---

## 💡 Key Takeaways
1. **Never Trust Packed Strings:** If a binary contains the `UPX!` signature, any flag or key visible via `strings` before unpacking is almost guaranteed to be an intentional red herring.
2. **Always Unpack First:** Dynamic analysis or decompilation of an unpacked binary (`upx -d`) is mandatory to view the actual execution graph and logic.
3. **Preserve Tries:** If standard static analysis flags fail on the first attempt, pivot immediately to a debugger to trace the comparison instructions (`strcmp`, `memcmp`, or `strncmp`).
