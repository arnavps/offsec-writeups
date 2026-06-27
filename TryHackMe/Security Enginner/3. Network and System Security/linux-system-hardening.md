# Linux System Hardening

## Executive Summary

Linux hardening is the systematic process of reducing a Linux system's attack surface by configuring, restricting, and securing every layer of the system — from the physical boot process to running services to user accounts. This room applies the defence-in-depth principle to Linux: no single control is sufficient; layered controls together create a significantly more resilient system.

**Major concepts covered:** Physical security and GRUB hardening, LUKS disk encryption, host-based firewall (iptables, nftables, UFW), SSH hardening, user and account management, service minimisation, update and patch management, system logging.

**Why it matters:** Linux powers the vast majority of internet-facing servers, cloud infrastructure, and container hosts. A poorly hardened Linux system is a routine target — automated scanners find weak SSH configurations, unpatched services, and default credentials within minutes of exposure to the internet. Hardening directly reduces the probability and impact of compromise.

**Where these concepts apply:** Every Linux server deployment — web servers (Apache, Nginx), database servers (PostgreSQL, MySQL), container hosts (Docker, Kubernetes nodes), cloud VMs (AWS EC2, GCP Compute Engine), and on-premises infrastructure.

---

## Big Picture Overview

Linux hardening addresses threats at every layer of the system stack:

```
┌──────────────────────────────────────────────┐
│  Physical Layer: Physical access = root access│
│  → GRUB password, BIOS/UEFI password          │
├──────────────────────────────────────────────┤
│  Storage Layer: Stolen disk = data exposure   │
│  → LUKS full-disk encryption                 │
├──────────────────────────────────────────────┤
│  Network Layer: Exposed ports = attack surface│
│  → Host firewall, disable unused services    │
├──────────────────────────────────────────────┤
│  Authentication Layer: Weak auth = compromise │
│  → SSH hardening, MFA, strong passwords      │
├──────────────────────────────────────────────┤
│  Account Layer: Overprivileged users = risk   │
│  → Principle of least privilege, sudo, no root│
├──────────────────────────────────────────────┤
│  Software Layer: Unpatched software = vulns   │
│  → Regular updates, minimal installed packages│
├──────────────────────────────────────────────┤
│  Monitoring Layer: No logs = no detection     │
│  → System logs, audit logs, log centralisation│
└──────────────────────────────────────────────┘
```

---

## Core Concepts

---

### Concept 1 — Physical Security and GRUB Hardening

**The Core Principle:**
"Boot access = root access." If an attacker can reach the physical keyboard of a Linux system, they can use GRUB (Grand Unified Bootloader) to interrupt the boot process and reset the root password without knowing the current one. Physical security is therefore a prerequisite for all software security controls.

**GRUB Password:**
Adding a GRUB password forces an attacker to provide a password before accessing advanced boot options, including single-user mode and kernel parameters that allow root password resets.

```bash
# Generate a PBKDF2-hashed GRUB password
grub2-mkpasswd-pbkdf2
# Output: grub.pbkdf2.sha512.10000.HASH...

# Add to GRUB configuration (Fedora/RHEL: /etc/grub.d/40_custom)
# (Ubuntu: /etc/grub.d/40_custom)
set superusers="admin"
password_pbkdf2 admin grub.pbkdf2.sha512.10000.HASH...
```

**Limitations:**
- GRUB passwords only make sense for systems with physical console access — cloud VMs typically cannot use this control
- BIOS/UEFI passwords provide an earlier boot-level protection
- If the attacker can remove the disk, the GRUB password is bypassed — disk encryption is the next control layer

---

### Concept 2 — LUKS Disk Encryption

**What Is It?**
LUKS (Linux Unified Key Setup) is the standard disk encryption specification for Linux. It encrypts entire partitions so that the data is unreadable without the correct passphrase, even if the physical disk is removed.

**Why It Exists:**
Physical access to an unencrypted disk is equivalent to full data access — boot the disk in any other machine and all data is readable. LUKS makes the disk useless to anyone without the decryption key.

**LUKS Disk Layout:**
```
LUKS phdr (Partition Header)
  └── UUID, cipher, cipher mode, key length, master key checksum

Key Material Slots (KM1–KM8)
  └── Up to 8 different user passphrases, each encrypting a copy of the master key

Bulk Data
  └── Actual encrypted filesystem data (encrypted with master key)
```

**How LUKS Works:**
- The master key encrypts the bulk data using the chosen cipher (e.g., AES-256-XTS)
- The user's passphrase is used to derive an encryption key via PBKDF2 (salted, iterated)
- This derived key encrypts the master key and stores it in a key material slot
- On decryption: passphrase → PBKDF2 → derived key → decrypt master key → decrypt data

**LUKS Encryption Formula:**
```
enc_data = encrypt(cipher_name, cipher_mode, key, plaintext, length)
key = PBKDF2(password, salt, iteration_count, key_length)
```

**Setting Up LUKS from the CLI:**
```bash
# 1. Install cryptsetup
apt install cryptsetup   # Debian/Ubuntu
dnf install cryptsetup-luks  # Fedora

# 2. Identify the target partition
fdisk -l
lsblk

# 3. Format the partition with LUKS
cryptsetup -y -v luksFormat /dev/sdb1

# 4. Open the encrypted partition
cryptsetup luksOpen /dev/sdb1 EDCdrive

# 5. Verify mapping
ls -l /dev/mapper/EDCdrive
cryptsetup -v status EDCdrive

# 6. Overwrite existing data with zeros (secure wipe)
dd if=/dev/zero of=/dev/mapper/EDCdrive

# 7. Create filesystem on the mapped device
mkfs.ext4 /dev/mapper/EDCdrive -L "Secure Drive"

# 8. Mount and use
mount /dev/mapper/EDCdrive /media/secure-drive

# View LUKS header information
cryptsetup luksDump /dev/sdb1
```

**Real-World Usage:**
- Laptop full-disk encryption: all enterprise laptops should have LUKS or equivalent (BitLocker on Windows)
- Server data partitions: database volumes containing PII or financial data
- USB/removable media: any device leaving the office should be encrypted

---

### Concept 3 — Linux Host-Based Firewalls

**The Linux Firewall Stack:**
```
User Space:
  UFW / firewalld (high-level CLI/GUI)
       ↓
  iptables / nftables (lower-level rules)
       ↓
Kernel Space:
  netfilter (packet filtering hooks in the kernel)
```

**Why Host-Based Firewalls Matter:**
Network-based firewalls control traffic between network segments but cannot control traffic arriving at a specific host from within its segment. A compromised server in the same VLAN could attack an unprotected host. Host firewalls enforce controls at the host level regardless of network position.

**iptables:**

iptables uses three default chains:
- **INPUT:** Packets destined for the local system
- **OUTPUT:** Packets leaving the local system
- **FORWARD:** Packets being routed through the system

```bash
# Allow inbound SSH (port 22)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow outbound SSH responses
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

# Block all other inbound traffic
iptables -A INPUT -j DROP

# Block all other outbound traffic
iptables -A OUTPUT -j DROP

# Clear all rules
iptables -F
```

**Flag breakdown:**
- `-A INPUT` — append to INPUT chain
- `-p tcp` — TCP protocol
- `--dport 22` — destination port 22
- `-j ACCEPT` — action: accept (jump to ACCEPT target)

**nftables (modern replacement for iptables):**
```bash
# Create a table
nft add table fwfilter

# Add input and output chains
nft add chain fwfilter fwinput { type filter hook input priority 0 \; }
nft add chain fwfilter fwoutput { type filter hook output priority 0 \; }

# Allow SSH inbound
nft add rule fwfilter fwinput tcp dport 22 accept

# Allow SSH responses outbound
nft add rule fwfilter fwoutput tcp sport 22 accept

# Verify configuration
nft list table fwfilter
```

**UFW (Uncomplicated Firewall — recommended for most use cases):**
```bash
# Allow SSH
ufw allow 22/tcp

# Check status
ufw status

# Enable UFW
ufw enable

# Default deny all inbound
ufw default deny incoming

# Default allow all outbound
ufw default allow outgoing
```

**Firewall Policy Approaches:**

| Approach | Principle | Use Case |
|----------|----------|---------|
| **Default deny + explicit allow** | Block everything; explicitly permit required traffic | Production servers, high-security |
| **Default allow + explicit deny** | Allow everything; explicitly block known bad traffic | Development, low-security environments |

Default deny is always preferred for production systems — it minimises the attack surface and requires intentional action to open ports.

**SELinux and AppArmor:**
iptables cannot restrict traffic per process — it only works by port. SELinux and AppArmor provide mandatory access controls that can restrict which specific binary can bind to which port:
- **AppArmor:** Profile-based; available on Debian/Ubuntu; restricts `/usr/sbin/apache2` to ports 80/443
- **SELinux:** Label-based; available on RHEL/Fedora; more granular but more complex to manage

---

### Concept 4 — SSH Hardening

**SSH Attack Vectors:**
1. Password sniffing (cleartext protocols — use SSH, not Telnet)
2. Password guessing and brute force (weak passwords, no lockout)
3. Exploitation of the SSH service itself (CVE-based attacks)

**Key SSH Configuration (`/etc/ssh/sshd_config`):**

```bash
# Disable root login - force use of non-root accounts
PermitRootLogin no

# Disable password authentication - force public key only
PasswordAuthentication no

# Enable public key authentication
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Limit login grace time
LoginGraceTime 30

# Limit authentication attempts
MaxAuthTries 3

# Only allow specific users
AllowUsers adminuser

# Use specific SSH protocol version
Protocol 2
```

**Setting Up Public Key Authentication:**
```bash
# Generate RSA key pair (on client)
ssh-keygen -t rsa -b 4096

# Copy public key to server
ssh-copy-id username@server

# The public key is added to ~/.ssh/authorized_keys on the server
# Private key stays on the client
```

**Why Public Key Authentication Is Stronger:**
- No password to phish or brute-force
- The private key never leaves the client machine
- Even if the server is compromised, the private key is not exposed
- Can be combined with passphrase protection on the private key

**Why `PermitRootLogin no` Matters:**
Attackers commonly target the `root` account because it exists on every Linux system (known username). Disabling root login forces attackers to first discover a valid non-root username before attempting to escalate privilege.

---

### Concept 5 — User and Account Management

**Use sudo, Not Root:**
The root account has unlimited power — a mistake as root can corrupt the system. Using `sudo` limits blast radius: only explicitly permitted commands run with elevated privileges, and all sudo commands are logged.

```bash
# Add user to sudo group (Debian/Ubuntu)
usermod -aG sudo username

# Add user to wheel group (RHEL/Fedora)
usermod -aG wheel username
```

**Disable the Root Account:**
```bash
# Change root shell to nologin (prevents interactive login)
# Edit /etc/passwd:
# Before: root:x:0:0:root:/root:/bin/bash
# After:  root:x:0:0:root:/root:/sbin/nologin
```

**Disable Unused Accounts:**
```bash
# Disable a user account by setting shell to nologin
# Edit /etc/passwd:
# michael:x:1000:1000:Michael:/home/michael:/usr/bin/fish
# → michael:x:1000:1000:Michael:/home/michael:/sbin/nologin

# Also disable service accounts that should never log in
# www-data, mongo, nginx → all should have /sbin/nologin
```

**Password Policy — libpwquality:**

Configuration file: `/etc/security/pwquality.conf` (RHEL/Fedora) or `/etc/pam.d/common-password` (Debian/Ubuntu)

```
# Require 5 characters different from old password
difok = 5

# Minimum password length
minlen = 14

# Require at least 3 character classes (uppercase, lowercase, digits, symbols)
minclass = 3

# Banned words list
badwords = company companyname

# Prompt 3 times before returning error
retry = 3
```

---

### Concept 6 — Service and Package Minimisation

**Principle:** Every installed package and running service is a potential attack vector. Eliminate what is not needed.

**Disable Unnecessary Services:**
```bash
# Check running services
systemctl list-units --type=service --state=running

# Disable a service
systemctl disable servicename
systemctl stop servicename

# Check what is listening on network ports
ss -tlnp
netstat -tlnp
```

**Block Unneeded Network Ports:**
Even if a service is disabled, firewall rules blocking its ports add defence-in-depth. If an attacker re-enables or starts the service, the firewall prevents network access.

**Avoid Legacy Protocols:**

| Insecure Protocol | Secure Replacement |
|------------------|-------------------|
| Telnet (port 23) | SSH (port 22) |
| FTP (port 21) | SFTP or FTPS |
| HTTP (port 80) | HTTPS (port 443) |
| SMTP without TLS | SMTPS / STARTTLS |
| SNMPv1/v2 | SNMPv3 |
| rsh, rlogin | SSH |

**Remove Identification Strings:**
Services that report version numbers in banners give attackers a direct CVE lookup. Configure web servers, SSH, and other services to suppress or minimise version disclosure:

```bash
# Apache: ServerSignature Off, ServerTokens Prod
# Nginx: server_tokens off
# SSH: disable banner or customise it
```

---

### Concept 7 — Updates and Patch Management

**Why It Matters — Dirty COW Example:**
CVE-2016-5195 (Dirty COW) was a privilege escalation vulnerability present in all Linux kernels from 2.6.22 onwards. It allowed an unprivileged local user to gain root access by exploiting a race condition. Without kernel updates, every Linux system running an affected kernel version is compromisable by any local user.

**Update Commands:**
```bash
# Debian/Ubuntu
apt update          # Refresh package lists
apt upgrade         # Install available updates

# RHEL/Fedora (modern)
dnf update          # RHEL 8+, Fedora

# RHEL (legacy)
yum update          # RHEL 7 and earlier
```

**LTS (Long-Term Support) Distributions:**

| Distribution | LTS Duration | Recommendation |
|-------------|-------------|----------------|
| Ubuntu LTS (e.g., 22.04) | 5 years free + 5 years ESM | Use LTS versions for servers |
| RHEL 8/9 | 12 years (5 full + 5 maintenance + 2 extended) | Enterprise standard |
| Debian Stable | ~3 years | Conservative; high stability |

**Never run end-of-life systems in production.** Ubuntu 14.04 (EOL April 2019) receives no free security updates — every new vulnerability published after that date remains permanently unpatched without a paid ESM subscription.

**Automatic Updates:**
For stability-focused distributions, automatic security updates are safe and recommended. Use `unattended-upgrades` on Ubuntu/Debian:

```bash
apt install unattended-upgrades
dpkg-reconfigure unattended-upgrades
```

---

### Concept 8 — System Logging

**Key Log Files:**

| Log File | Contents | Use Case |
|---------|---------|---------|
| `/var/log/messages` | General system messages | General troubleshooting |
| `/var/log/auth.log` | Authentication attempts (Debian) | Failed logins, sudo usage |
| `/var/log/secure` | Authentication attempts (RHEL) | Failed logins, sudo usage |
| `/var/log/utmp` | Currently logged-in users | Active session monitoring |
| `/var/log/wtmp` | Historical login/logout records | Account activity audit |
| `/var/log/kern.log` | Kernel messages | Hardware issues, kernel errors |
| `/var/log/boot.log` | Boot process messages | Boot failure diagnosis |

**Useful Log Commands:**
```bash
# View last 12 lines of a log file
tail -n 12 /var/log/boot.log

# Search for specific keyword in log
grep FAILED /var/log/auth.log

# View recent authentication failures
ausearch --message USER_LOGIN --success no --interpret

# Count failed root logins
ausearch -m USER_LOGIN -sv no -i | grep ct=root | wc -l

# Audit summary report
aureport --summary
aureport --failed
```

**Log Centralisation:**
Individual host logs should be forwarded to a central syslog server or SIEM. This prevents attackers from covering tracks by deleting local log files and enables correlation across multiple systems.

---

## Security Engineer Perspective

### Hardening Priority Order

1. Physical security first — if physical access is not controlled, all other controls can be bypassed
2. LUKS encryption — protects data at rest if physical access is compromised
3. SSH hardening — usually the primary remote attack vector
4. Firewall + service minimisation — reduce network-visible attack surface
5. User account controls — prevent privilege escalation
6. Package and kernel updates — patch known vulnerabilities
7. Logging and monitoring — detect what gets through

### Offensive Perspective

| Attack | What It Exploits | Hardening Control |
|--------|----------------|------------------|
| GRUB root password reset | Physical access | GRUB password, physical security |
| Disk theft / cold boot | Unencrypted storage | LUKS |
| SSH brute force | Password auth, no lockout | Public key only, `MaxAuthTries 3` |
| SSH as root | PermitRootLogin yes | `PermitRootLogin no` |
| Service exploitation | Unnecessary running services | Minimal services, firewall |
| Dirty COW and similar | Unpatched kernel | Regular kernel updates |
| Privilege escalation via SUID | World-writable SUID binaries | Audit SUID files, remove unnecessary |
| Log tampering | Local-only log storage | Centralised logging |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **AppArmor** | Linux MAC framework using profiles to restrict process behaviour |
| **GRUB** | Grand Unified Bootloader — the bootloader on most Linux systems |
| **iptables** | User-space command-line tool for managing netfilter packet filter rules |
| **LUKS** | Linux Unified Key Setup — standard Linux disk encryption |
| **netfilter** | Linux kernel subsystem providing packet filtering, NAT, and related functionality |
| **nftables** | Modern replacement for iptables — improved performance and scalability |
| **PBKDF2** | Password-Based Key Derivation Function 2 — iterative key derivation for secure password storage |
| **SELinux** | Security-Enhanced Linux — label-based mandatory access control framework |
| **SSH** | Secure Shell — encrypted remote access protocol |
| **sudo** | Super User Do — executes commands with elevated privileges, with logging |
| **UFW** | Uncomplicated Firewall — simplified front-end to iptables |
| **unattended-upgrades** | Debian/Ubuntu package for automatic security update installation |
| **sshd_config** | OpenSSH server configuration file at `/etc/ssh/sshd_config` |
| **libpwquality** | Library providing password quality controls |

---

## Exam and Interview Revision

### Must Remember

- Boot access = root access — physical security is the foundation
- LUKS uses PBKDF2 to derive encryption key from passphrase; supports AES, multiple key slots (up to 8 users)
- Linux firewall stack: UFW/firewalld → iptables/nftables → netfilter (kernel)
- Default deny is always preferred for production servers
- SSH hardening essentials: `PermitRootLogin no`, `PasswordAuthentication no`, `PubkeyAuthentication yes`
- Never allow root login over SSH — attackers know root always exists
- Public key auth: private key on client, public key on server (`~/.ssh/authorized_keys`)
- Disable accounts by setting shell to `/sbin/nologin` in `/etc/passwd`
- Service accounts (www-data, nginx, mongo) should never have a login shell
- Every installed package = potential attack vector; minimize to required packages only
- Legacy protocols in clear text (Telnet, FTP, rsh) must be replaced with encrypted alternatives
- Ubuntu LTS = 5 years free updates; RHEL 8/9 = 12 years total
- Dirty COW (CVE-2016-5195) = kernel privilege escalation via copy-on-write race condition
- Centralise logs to prevent attackers from deleting evidence

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is LUKS and why is it important? | Linux disk encryption standard. Protects data at rest — stolen disk is useless without passphrase. Uses PBKDF2 + AES. Supports up to 8 key slots. |
| How does SSH public key auth work? | Client has private key; server has public key in `authorized_keys`. Client proves possession of private key via cryptographic challenge. No password transmitted. |
| What is the difference between iptables and nftables? | iptables is older; nftables is newer with better performance and a cleaner syntax. Both are front-ends to netfilter. |
| Why should root login be disabled over SSH? | Root exists on every system — known username makes brute force easier. Root compromise = full system compromise immediately. Force non-root + sudo instead. |
| What is the principle of minimal installed packages? | Every package is a potential attack vector. Install only what is needed; uninstall/disable everything else to minimise the attack surface. |
| How do you check what ports are listening on Linux? | `ss -tlnp` or `netstat -tlnp` — shows listening TCP ports and the process using each port. |
