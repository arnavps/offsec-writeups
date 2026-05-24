# Operating System Security

## Overview

The operating system is the first and most fundamental layer of security on any computer. Before any third-party security tool is involved, the OS enforces authentication, permissions, and process isolation. This room covers the CIA triad as it applies to OS security, the three primary attack vectors targeting OS-level weaknesses, and a practical demonstration of exploiting weak authentication and poor security hygiene on a Linux system.

---

## Topics Covered

- The CIA triad in the context of OS security
- Authentication and weak passwords
- Weak file permissions
- Malicious programs (Trojans, ransomware)
- Practical: SSH login, privilege escalation via command history

---

## Key Concepts

### The CIA Triad

OS security is built around protecting three core properties:

| Property | Description |
|---|---|
| Confidentiality | Sensitive files and data are accessible only to authorised users |
| Integrity | Files cannot be tampered with — on disk or in transit |
| Availability | The system remains accessible and functional when needed |

Attacks on an OS target one or more of these properties.

---

### Authentication and Weak Passwords

Authentication verifies identity before granting access. The three factors of authentication:

| Factor | Description | Examples |
|---|---|---|
| Something you know | Knowledge-based | Password, PIN |
| Something you are | Biometric | Fingerprint, face recognition |
| Something you have | Possession-based | Phone (SMS OTP), hardware token |

Passwords are the most common authentication method and therefore the most attacked. Common weaknesses:

- Using simple, guessable passwords
- Reusing the same password across multiple accounts
- Writing passwords down in visible locations (sticky notes)

**Top commonly used (and attacked) passwords:**

```
123456
123456789
qwerty
password
111111
12345678
abc123
```

---

### Weak File Permissions

The principle of least privilege states that users and processes should only have access to what they need — nothing more.

Weak file permissions break this principle and enable two types of attacks:

- **Confidentiality attack** — an unauthorised user can read files they shouldn't have access to
- **Integrity attack** — an unauthorised user can modify files they shouldn't be able to edit

Proper permission management ensures that sensitive files are readable and writable only by the appropriate users or processes.

---

### Malicious Programs

Malware attacks one or more of the CIA triad properties depending on its type:

| Malware Type | CIA Impact | Description |
|---|---|---|
| Trojan Horse | Confidentiality, Integrity | Disguises itself as legitimate software; gives attacker remote access to read and modify files |
| Ransomware | Availability | Encrypts user files, making them unreadable; attacker demands payment for the decryption key |
| Spyware | Confidentiality | Silently collects and exfiltrates user data |

---

## Practical Examples / Demonstrations

### Scenario: Weak Password + SSH Login

A sticky note on a screen revealed credentials: username `sammie`, password `dragon`.

**Connect via SSH:**

```bash
ssh sammie@10.48.148.68
```

On first connection, SSH will prompt about the server's authenticity:

```
Are you sure you want to continue connecting (yes/no)?
```

Type `yes` to accept and store the server's key.

Enter the password when prompted. Note: SSH does not display any characters while typing the password — this is normal behaviour.

**Confirm identity:**

```bash
whoami
```

**List files in current directory:**

```bash
ls
```

**Read a file:**

```bash
cat filename.txt
```

---

### Privilege Escalation via Command History

The `history` command prints previously executed commands for the current user:

```bash
history
```

Users with poor security hygiene sometimes accidentally type passwords as commands. If a user typed their password (or another user's password) directly into the terminal, it appears in the history output.

**Switch to another user:**

```bash
su - johnny
```

**Switch to root (after finding the root password in history):**

```bash
su - root
```

**Read a file in the root directory:**

```bash
cat /root/flag.txt
```

---

### Challenge Walkthrough Summary

| Step | Command | Result |
|---|---|---|
| Login as sammie | `ssh sammie@10.48.148.68` | Password: `dragon` (from sticky note) |
| Confirm identity | `whoami` | Returns `sammie` |
| Switch to johnny | `su - johnny` | Password found by trying common passwords (`abc123`) |
| Check history | `history` | Root password accidentally typed as a command |
| Switch to root | `su - root` | Password: `happyHack!NG` |
| Read flag | `cat /root/flag.txt` | `THM{YouGotRoot}` |

---

## Workflow / Process

### Attack Chain: Weak Credentials → Root Access

```
Discover credentials (sticky note / OSINT / guessing)
        |
SSH into target with weak password
        |
Enumerate users and switch accounts (su -)
        |
Check command history for accidentally typed passwords
        |
Escalate to root using discovered password
        |
Full system access
```

---

## Real-World Relevance

- Weak passwords are the most common initial access vector — credential stuffing, password spraying, and brute force attacks all exploit this
- SSH with password authentication is a common attack surface on internet-facing Linux servers — key-based authentication should be used instead
- Command history (`~/.bash_history`) is a standard post-exploitation artefact — analysts check it during forensics; attackers check it for credentials
- The principle of least privilege is a core hardening measure — overly permissive file permissions are a frequent finding in security audits
- Ransomware attacks on availability are among the most financially damaging cyber incidents — backups and endpoint protection are the primary defences
- Sticky notes with passwords are a real and common physical security failure

---

## Key Learnings

- OS security maps directly to the CIA triad: confidentiality, integrity, and availability
- Weak passwords are the most exploited authentication weakness
- File permissions must follow the principle of least privilege
- Malware targets different CIA properties depending on its type
- Command history can expose credentials — a simple but impactful operational security failure
- SSH is the standard remote access protocol for Linux — understanding it is essential

---

## Additional Notes

- `passwd` changes a user's password on Linux
- `/etc/shadow` stores hashed passwords on Linux — readable only by root; a primary target during privilege escalation
- SSH key-based authentication is significantly more secure than password authentication and should be the default for any internet-facing server
- `chmod` and `chown` are the Linux commands for managing file permissions and ownership

---

## Conclusion

OS security starts with the basics: strong authentication, proper file permissions, and avoiding malicious software. This room demonstrates how a chain of simple failures — a weak password on a sticky note, a password accidentally typed into a terminal — can lead to full root compromise. These aren't theoretical scenarios; they reflect real-world attack patterns seen in penetration tests and incident investigations.
