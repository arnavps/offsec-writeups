# Windows System Hardening

## Executive Summary

Windows hardening addresses the unique attack surface of Microsoft Windows systems — from the registry and services to UAC, Group Policy, browser security, and disk encryption. Windows is the dominant enterprise desktop OS and a primary target for ransomware, credential theft, and lateral movement. Systematic hardening reduces the number of exploitable footholds an attacker can use after initial access.

**Major concepts covered:** Windows services and registry, Event Viewer and telemetry, user account management and UAC, Group Policy and password policies, Windows Defender Firewall, SMB hardening, DNS and ARP protection, Remote Desktop hardening, BitLocker, Windows Sandbox, Secure Boot, and Microsoft Office hardening.

**Why it matters:** Most ransomware, APT lateral movement, and enterprise breaches involve Windows systems. UAC bypasses, credential harvesting from LSASS, SMB exploitation, and RDP brute force are all standard attack techniques that hardening directly mitigates.

**Where these concepts apply:** Every Windows workstation and server in an enterprise, Active Directory environments, corporate endpoints, remote worker machines.

---

## Big Picture Overview

Windows hardening is about reducing the attack surface of the Windows OS by disabling what is not needed, enforcing least privilege, configuring security policies, encrypting sensitive data, and monitoring for suspicious activity. Unlike Linux, Windows provides most hardening controls through graphical interfaces — Group Policy, Control Panel, Windows Security Centre — making them accessible but also sometimes easier to misconfigure or leave at defaults.

---

## Core Concepts

---

### Concept 1 — Windows Services

**What Is It?**
Windows Services are processes that run in the background without requiring user interaction. They handle critical OS functions (network connectivity, Windows Update, authentication) and also run third-party software. Services are managed by the Service Control Manager (SCM).

**Categories:**
- **Local services** — run under limited local service accounts
- **Network services** — run under network service accounts with specific network access
- **System services** — run under the SYSTEM account with full local privileges

**Security Relevance:**
- Unnecessary services expand the attack surface — disabled services cannot be exploited
- Many malware families persist by creating or hijacking services
- Services run under SYSTEM by default — overprivileged service accounts are a common lateral movement vector
- Services can be abused for privilege escalation via misconfigured service paths (unquoted service path attack)

**Access:** `services.msc` in Run dialog

**Hardening Actions:**
- Review all non-Microsoft services — identify and disable any not required for business function
- Ensure third-party services run under the minimum required account (not SYSTEM if avoidable)
- Audit service path configurations for unquoted paths with spaces

---

### Concept 2 — Windows Registry

**What Is It?**
The Windows Registry is a centralised hierarchical database storing configuration settings for Windows and installed applications. It contains system settings, user preferences, application data, and critically — autostart locations that control what runs at boot.

**Key Registry Hives:**

| Hive | Contents |
|------|---------|
| `HKEY_LOCAL_MACHINE (HKLM)` | System-wide settings, installed software, hardware, services |
| `HKEY_CURRENT_USER (HKCU)` | Settings for the currently logged-in user |
| `HKEY_CLASSES_ROOT (HKCR)` | File extension associations and COM object registrations |
| `HKEY_USERS (HKU)` | Profiles for all user accounts on the system |
| `HKEY_CURRENT_CONFIG (HKCC)` | Current hardware profile |

**Critical Registry Keys for Security:**

| Key | Purpose | Security Relevance |
|-----|---------|-------------------|
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` | Programs that run at startup for all users | Common malware persistence location |
| `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` | Programs that run at startup for current user | Common malware persistence location |
| `HKLM\SYSTEM\CurrentControlSet\Services` | Windows service definitions | Service hijacking, malicious service installation |
| `HKLM\SOFTWARE\Policies` | Group Policy applied settings | Security policy configuration |

**Hardening Actions:**
- Restrict registry editor access (`regedit.exe`) to administrators only
- Monitor Run keys for unauthorised entries
- Disable access to regedit via Group Policy for standard users: `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System: DisableRegistryTools = 1`

**Access:** `regedit` in Run dialog

---

### Concept 3 — Event Viewer and Logging

**What Is It?**
Event Viewer aggregates logs from all Windows components, applications, and security subsystems into a centralised database accessible at `C:\WINDOWS\system32\config\`.

**Event Log Categories:**

| Category | Contents | Security Use |
|----------|---------|-------------|
| **Application** | Application-level events | Application crashes, software errors |
| **System** | OS component events | Driver failures, service starts/stops |
| **Security** | Authentication, policy changes, privilege use | Login events, account changes, UAC prompts |

**Key Security Event IDs:**

| Event ID | Description |
|----------|------------|
| 4624 | Successful logon |
| 4625 | Failed logon (wrong username/password) |
| 4634 | Logoff completed |
| 4647 | User initiated logoff |
| 4720 | User account created |
| 4722 | User account enabled |
| 4723 | Password change attempt |
| 4728 | User added to privileged group |
| 4756 | Member added to universal security group |
| 4779 | Disconnect from remote session without logoff |

**Access:** `eventvwr` in Run dialog

**Hardening Actions:**
- Configure audit policies to capture logon/logoff, account management, privilege use, and object access
- Set log size large enough to retain events for the required retention period (PCI DSS: 1 year; HIPAA: 6 years)
- Forward security logs to a central SIEM
- Alert on: multiple 4625 events (brute force), 4720 (new accounts), 4728 (privilege escalation)

---

### Concept 4 — Windows Telemetry

**What Is It?**
Windows Telemetry is Microsoft's data collection system that sends crash reports, diagnostic data, and usage information to Microsoft. It runs through the DiagTrack service (`diagtrack.dll`) and stores data in `%ProgramData%\Microsoft\Diagnosis` before sending it every ~15 minutes.

**Security Relevance:**
- Telemetry data can contain sensitive information about installed software, error messages, and system configuration
- In regulated environments (HIPAA, GDPR), data sent to Microsoft must be accounted for in data processing agreements
- DiagTrack can be disabled via Services console: `Connected User Experiences and Telemetry` service → Disabled

---

### Concept 5 — User Account Control (UAC)

**What Is It?**
UAC (User Account Control) is a Windows security feature that enforces privilege boundaries. Standard operations run without elevation; operations requiring administrative privilege prompt for approval or credentials.

**Why It Exists:**
Pre-UAC (Windows XP), all users typically ran as administrators. Malware running as the user immediately had full administrator access. UAC creates a privilege barrier — even an admin account runs with standard privilege by default and must explicitly elevate.

**How It Works:**
- Users with admin accounts get two tokens: a standard token and an admin token
- Standard operations use the standard token (limited privilege)
- When elevation is required (installing software, modifying system settings), UAC prompts for consent or credentials

**UAC Levels:**

| Level | Description | Recommendation |
|-------|------------|----------------|
| Always Notify | Prompt for all changes — apps and user-initiated | Most secure — use this |
| Notify app changes (default) | Prompt only for app changes, not user-initiated changes | Default — acceptable |
| Notify app changes (no dim) | Same as above but without secure desktop | Less secure |
| Never Notify | UAC disabled | Never use |

**Access:** Control Panel → User Accounts → Change UAC Settings

**Hardening Actions:**
- Set UAC to "Always Notify"
- Never disable UAC on production systems
- Enforce that all users run as standard accounts for daily work; admin accounts reserved for administrative tasks

**UAC Bypass Attacks:**
UAC bypasses are a significant attack class. Common techniques include:
- **fodhelper.exe bypass:** Abuses `ms-settings` protocol handling to execute commands with elevated privilege
- **eventvwr.exe bypass:** Hijacks a registry key read by eventvwr during launch
- **DLL injection into auto-elevated processes:** Processes marked `autoElevate=true` in their manifest can be abused

**Mitigation:** Keep Windows updated; monitor for unexpected elevated processes; use Application Whitelisting (AppLocker) to prevent unauthorised binary execution.

---

### Concept 6 — Group Policy and Password Policies

**What Is It?**
Group Policy is Microsoft's mechanism for enforcing configuration settings across Windows systems, either locally (Local Group Policy) or domain-wide (via Active Directory Group Policy Objects).

**Access:** `gpedit.msc` (not available in Windows Home; Pro and Enterprise only)

**Password Policy Settings:**
`Security Settings → Account Policies → Password Policy`

| Setting | Recommended Value | Reason |
|---------|-----------------|--------|
| Enforce password history | 10–15 | Prevents password reuse |
| Minimum password length | 12–14 | Longer = harder to crack |
| Maximum password age | 0 (no expiry) or 90 days | NIST 800-63B recommends no forced rotation |
| Complexity requirements | Enabled | Requires uppercase, lowercase, digits, symbols |
| Store passwords using reversible encryption | Disabled | Never store plaintext passwords |

**Account Lockout Policy:**
`Security Settings → Account Policies → Account Lockout Policy`

- Account lockout threshold: 5–10 invalid attempts
- Lockout duration: 30–60 minutes (or manual unlock required)
- Reset counter after: 15 minutes

**This directly mitigates brute force and password spraying attacks.**

---

### Concept 7 — Windows Defender Firewall

**What Is It?**
Windows Defender Firewall is the built-in host firewall with three profiles:
- **Domain** — applied when connected to an Active Directory domain network
- **Private** — applied when connected to a trusted private network
- **Public** — applied when connected to untrusted public networks

**Access:** `WF.msc` in Run dialog

**Hardening Actions:**
- Private profile: Block all inbound connections by default
- Public profile: Block all inbound connections (most restrictive)
- Always configure "default deny" for inbound before adding exception rules
- Review and remove any unnecessary inbound rules

**Important:** Malware can create inbound firewall rules to allow C2 connections. Regularly audit firewall rules for unexpected entries.

---

### Concept 8 — SMB Protocol Hardening

**Why SMB Is a High-Risk Protocol:**
SMB (Server Message Block) has been exploited in major attacks:
- **EternalBlue (MS17-010):** Exploited SMBv1 → enabled WannaCry and NotPetya ransomware
- **Pass-the-Hash:** SMB authentication can be attacked using captured NTLM hashes
- **SMB relay attacks:** NTLM authentication challenge-response can be relayed to authenticate against other SMB services

**Hardening Actions:**
```powershell
# Disable SMBv1 (legacy, dangerous)
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# Verify SMB versions
Get-SmbServerConfiguration | Select EnableSMB1Protocol, EnableSMB2Protocol
```

SMBv1 should be disabled on all systems unless a specific legacy application requires it. SMBv2 and SMBv3 provide encryption and integrity features absent in v1.

---

### Concept 9 — DNS and ARP Hardening

**DNS Hosts File:**
The `hosts` file (`C:\Windows\System32\Drivers\etc\hosts`) is used for local hostname resolution. Malware commonly modifies this file to redirect traffic to attacker-controlled servers (DNS hijacking at the local level).

**Hardening Actions:**
- Monitor hosts file for unauthorised modifications
- Set read-only permissions on the hosts file for non-admin users
- Alert via SIEM on changes to the hosts file

**ARP Cache Poisoning:**
ARP offers no authentication. Attackers send crafted ARP replies to map their MAC address to the gateway IP, intercepting all traffic (layer-2 MITM).

```cmd
# View ARP cache
arp -a

# Indicator of ARP poisoning: two IPs mapped to the same MAC address

# Clear ARP cache
arp -d
```

**Mitigation:** Enable Dynamic ARP Inspection on managed switches; use static ARP entries for critical systems (gateway, DNS servers).

---

### Concept 10 — Remote Desktop Protocol (RDP) Hardening

**Why RDP Is High Risk:**
RDP (Remote Desktop Protocol) is a common initial access and lateral movement vector:
- **BlueKeep (CVE-2019-0708):** Pre-authentication RCE in RDP on Windows 7/Server 2008 — no credentials required
- **RDP brute force:** Automated tools scan internet ranges for port 3389 and brute force credentials
- **Pass-the-Hash/Pass-the-Ticket:** Can authenticate to RDP using captured credentials

**Hardening Actions:**
- Disable RDP entirely if not required: Settings → Remote Desktop → Disable
- If RDP is required: restrict to specific source IPs via firewall rules; place behind VPN; enable NLA (Network Level Authentication); enforce MFA
- Change RDP port from default 3389 (security through obscurity — not a primary control but reduces automated scanning noise)

---

### Concept 11 — Microsoft Office Hardening

**Why Office Is High Risk:**
Microsoft Office is the primary malware delivery vector globally — phishing emails with malicious Word/Excel/PowerPoint files are the most common initial access technique. Attack methods include:
- **Macros:** VBA macros execute arbitrary code when the document is opened and macros are enabled
- **Object Linking and Embedding (OLE):** Embed executable objects in documents
- **Flash/ActiveX:** Deprecated but historically exploited

**Hardening via Attack Surface Reduction (ASR) Rules:**
Microsoft's ASR rules block common Office-based attack techniques:
- Block Office apps from creating executable content
- Block all Office applications from creating child processes
- Block macros from running when content downloaded from the internet
- Block JavaScript/VBScript from launching downloaded executables

**The `office.bat` script referenced in the room applies ASR rules based on Microsoft's documented best practices.**

**Access to run the script:**
```
Right-click office.bat → Run as Administrator
```

---

### Concept 12 — BitLocker Encryption

**What Is It?**
BitLocker is Microsoft's full-disk encryption solution for Windows business editions. It encrypts the entire drive, protecting data at rest.

**Requirements:**
- TPM (Trusted Platform Module) chip 1.2 or later
- Windows Pro, Enterprise, or Education edition (not Home)
- UEFI firmware (preferred) or BIOS with TPM support

**Access:** Start → Control Panel → System and Security → BitLocker Drive Encryption

**Critical:** Store the BitLocker recovery key securely — preferably in Azure AD, or in a secure offline location. If the TPM is cleared or the system fails, the recovery key is the only way to access the encrypted data.

---

### Concept 13 — Windows Sandbox

**What Is It?**
Windows Sandbox is an isolated, temporary Windows environment for running untrusted applications. When the Sandbox is closed, everything inside is permanently deleted — files, software, and state.

**Why It Exists:**
Opening suspicious email attachments or downloaded files in the host OS can compromise the system. Sandbox provides a safe analysis environment with complete isolation.

**Requirements:** Virtualisation must be enabled in BIOS/UEFI; Windows 10/11 Pro or Enterprise

**Enable:**
```
Windows Features → Windows Sandbox → OK → Restart
```

**Use Cases:**
- Opening suspicious email attachments before deciding to trust them
- Testing unknown software
- Security analysts examining potential malware samples (for deeper analysis, use a dedicated VM)

---

### Concept 14 — Secure Boot

**What Is It?**
Secure Boot is a UEFI firmware feature that verifies the digital signature of bootloaders and OS kernels before execution. It prevents unauthorised software (bootkits, rootkits) from loading at system startup.

**How It Works:**
1. UEFI stores a database of trusted certificate authorities and known-bad software hashes
2. During boot, every piece of boot software is verified against this database
3. If the signature is not trusted → boot is halted

**Windows 11 Requirement:** Secure Boot is a mandatory requirement for Windows 11 installation.

**Key Point:** Secure Boot is transparent when everything is normal. It requires no configuration for standard operation — it works silently in the background. Disabling it (which some users do for Linux dual-boot) removes protection against bootloader-level attacks.

---

## Security Engineer Perspective

### Windows Hardening Checklist

| Area | Control | Command/Path |
|------|---------|-------------|
| UAC | Set to Always Notify | Control Panel → User Accounts |
| Accounts | Use standard accounts daily; admin only for admin tasks | Control Panel → User Accounts |
| Firewall | Block all inbound on Public/Private by default | WF.msc |
| SMBv1 | Disabled | PowerShell: `Disable-WindowsOptionalFeature` |
| RDP | Disabled if not required | Settings → Remote Desktop |
| Auto-updates | Enabled — automatic | Settings → Update & Security |
| BitLocker | Enabled on all drives | Control Panel → BitLocker |
| AppLocker | Enabled — block unapproved executables | gpedit.msc → AppLocker |
| Password policy | Minimum 12 chars, complexity, lockout after 5 attempts | gpedit.msc → Account Policies |
| Hosts file | Monitor for modifications | C:\Windows\System32\Drivers\etc\hosts |
| Office macros | Disable for internet-downloaded content | ASR rules via Group Policy |

### Detection Opportunities via Event Viewer

| Suspicious Pattern | Event IDs | Indicates |
|-------------------|----------|---------|
| Multiple 4625 in short window | 4625 | Brute force / password spray |
| 4720 outside change window | 4720 | Unauthorised account creation |
| 4728 for unexpected user | 4728 | Privilege escalation |
| 4624 at unusual hour | 4624 | Potentially compromised credential |
| New service created | 7045 | Malware service installation |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **AppLocker** | Windows feature restricting which executables, scripts, and installers can run |
| **ASR** | Attack Surface Reduction — Microsoft rules blocking common malware techniques |
| **BitLocker** | Windows full-disk encryption for Pro/Enterprise editions |
| **EternalBlue** | NSA-developed SMBv1 exploit (MS17-010) — used by WannaCry and NotPetya |
| **Event Viewer** | Windows application for viewing system, application, and security event logs |
| **Group Policy** | Microsoft mechanism for enforcing configuration settings across Windows systems |
| **gpedit.msc** | Local Group Policy Editor — access point for Group Policy configuration |
| **NLA** | Network Level Authentication — RDP pre-authentication via NTLM/Kerberos |
| **RDP** | Remote Desktop Protocol — Microsoft remote access protocol on port 3389 |
| **Registry** | Centralised hierarchical database storing Windows and application configuration |
| **Secure Boot** | UEFI feature verifying bootloader signatures to prevent bootkit attacks |
| **SMB** | Server Message Block — Windows file sharing and authentication protocol |
| **Telemetry** | Windows data collection sending diagnostic/usage data to Microsoft |
| **TPM** | Trusted Platform Module — hardware security chip used by BitLocker |
| **UAC** | User Account Control — privilege barrier preventing unauthorised elevation |
| **Windows Defender Firewall** | Built-in Windows host-based firewall |
| **Windows Sandbox** | Isolated temporary environment for running untrusted applications |

---

## Exam and Interview Revision

### Must Remember

- UAC should be set to **Always Notify** — never disable
- Always use standard accounts for daily work; reserve admin accounts for admin tasks
- Principle of Least Privilege: users only need access to what their role requires
- SMBv1 is dangerous and must be disabled — it enabled WannaCry/NotPetya via EternalBlue
- RDP on port 3389 is heavily scanned and attacked — disable if not needed; protect with VPN+MFA if required
- Key security event IDs: **4624** (logon success), **4625** (logon failure), **4720** (account created), **4728** (added to privileged group)
- BitLocker requires TPM; store recovery key securely outside the encrypted system
- Secure Boot prevents bootloader/rootkit attacks — runs silently; do not disable
- Windows Sandbox is isolated — everything deleted when closed; ideal for testing untrusted files
- Registry Run keys (`HKLM/HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`) are primary malware persistence locations
- Office macros in downloaded documents are the most common initial access vector — restrict via ASR rules

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is UAC and why is it important? | Privilege barrier between standard and admin operations. Prevents malware from immediately gaining admin rights. Set to Always Notify. Bypasses exist — keep Windows updated. |
| What event ID indicates failed logons? | 4625. Multiple 4625 events in short succession = brute force or password spray attack. |
| Why is SMBv1 dangerous? | It was exploited by EternalBlue (MS17-010) — enabled WannaCry and NotPetya ransomware. Has no encryption. Disable with `Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol`. |
| What does BitLocker require? | TPM chip, Windows Pro/Enterprise edition, UEFI preferred. Encrypts entire drive. Recovery key must be stored securely. |
| What is Secure Boot? | UEFI firmware feature that verifies digital signatures of bootloaders before execution. Prevents bootkits and rootkits at the firmware level. |
| How would you detect a brute force attack against a Windows system? | Monitor Event Viewer Security logs for multiple 4625 (failed logon) events in a short window, especially from the same source IP or targeting the same account. |
