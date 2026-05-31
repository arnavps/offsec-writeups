# Forensic Log Recovery

## 🧠 Objective
Recovered logs contained fragmented debug entries, corrupted metadata, and decoy `temporary_*` flags. The goal was to reconstruct a valid flag in the format: `SecLeaf{}`

---

## 1. Initial Extraction
The archive was unpacked:
```bash
unzip logs.zip -d logs
ls -R logs
```

This revealed a large structured dataset consisting of:
* `auth_*.log` (main corpus)
* `cache_*.log`
* `debug_*.log`
* `error_*.log`
* `notes*.log`
* `final_*.log`
* `recovered_*.log`
* `temp_*.log`

At first glance, this looked like authentication logs with injected fragments.

---

## 2. First Signal: Flag-like Strings
A broad search:
```bash
grep -R "SecLeaf{" logs
```
returned many entries like: `SecLeaf{temporary_XXXXX}`

> [!WARNING]
> **Important observation:**
> These were decoy placeholders, not real flags.
> * **Pattern:** Always `temporary_<number>`
> * **Occurrence:** Appeared in many unrelated auth logs
> 
> **Conclusion:** Noise layer injected for misdirection.

---

## 3. Real Signal Detection (Hex Fragments)
Next step was identifying structured corruption:
```bash
grep -R "[0-9a-f]\{8\}" logs
```

This revealed consistent patterns:
```text
fragment: 4addf9bb
cache fragment: 5365634c
orphan data: 636f6e74
sync failed: 6578745f
...
```

**Key insight:** These are 32-bit hex chunks representing ASCII text.

---

## 4. Decoding Core Fragments
Important hex values decoded:

| Hex | ASCII |
|---|---|
| `5365634c` | `SecL` |
| `6561667b` | `eaf{` |
| `636f6e74` | `cont` |
| `6578745f` | `ext_` |
| `69735f74` | `is_t` |
| `68655f72` | `he_r` |
| `65616c5f656e656d797d` | `eal_enemy}` |

---

## 5. Structural Reconstruction

### Step 1: Fixed header (trusted sources)
* `debug_183.log` $\rightarrow$ `SecL`
* `error_29.log` $\rightarrow$ `eaf{`

Resulting prefix: `SecLeaf{`

### Step 2: Closing block
`65616c5f656e656d797d` $\rightarrow$ `eal_enemy}`

This confirms the flag ends with `}` and the final word contains `enemy`.

---

## 6. Key Challenge: Ordering Problem
At this point, fragments were:
* `is_t`
* `he_r`
* `cont`
* `ext_`

However:
1. File order was meaningless.
2. Auth logs were shuffled.
3. Temporary values were decoys.

---

## 7. Critical Insight: XOR / ECB Hint
From notes:
* XOR patterns detected
* Possible ECB encryption

**Meaning:** Data was split into fixed blocks and shuffled or indexed. Order $\neq$ file order. Solution required reconstructing meaning, not file sequence.

---

## 8. Logical Reconstruction Strategy
Instead of linear joining, group by semantic units:
* `is_t` + `he_r` $\rightarrow$ `is_the_r`
* `cont` + `ext_` $\rightarrow$ `context_`
* `eal_enemy` $\rightarrow$ `real_enemy`

---

## 9. Final Assembly Logic
Putting the structure together semantically:
`SecLeaf{ context_is_the_real_enemy }`

---

## 10. Final Flag
```text
SecLeaf{placeholder_flag}
```

---

## 🔥 Key Learning Points
1. **Decoy Handling:** `temporary_*` values were intentional noise. Always validate frequency + consistency.
2. **Multi-layer Forensics:** Logs contained decoys (auth logs), real payload (hex fragments), and structural anchors (debug/error/cache).
3. **Hex-to-ASCII Reconstruction:** Scan for 4-byte aligned hex chunks and convert them before guessing their meaning.
4. **Ordering Trap:** File numbering was irrelevant. Reconstruction required semantic grouping, not sorting.
5. **Encryption Hint Usage:** "ECB/XOR patterns" indicates block permutation. No full decryption was needed, just rearrangement logic.
