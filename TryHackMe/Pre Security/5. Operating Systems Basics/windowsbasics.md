# Windows Basics

## Overview

Windows is the most widely used desktop operating system and a dominant presence in corporate environments. This room covers the core components of the Windows interface, user account types, and the built-in security tools every Windows system ships with. For security professionals, understanding the Windows environment is essential — it's the most common target for malware, phishing, and post-exploitation activity.

---

## Topics Covered

- Windows authentication and account types
- Windows Security dashboard
- Windows Defender Firewall
- Core Windows interface components and tools

---

## Key Concepts

### Authentication and Account Types

Before accessing the Windows desktop, a user must authenticate — proving their identity to the system. Authentication can use a password, PIN, biometric (fingerprint/face), or other method.

Each account is assigned a permission level that determines what the user can access and modify:

| Account Type | Description |
|---|---|
| Guest | Temporary access with minimal permissions; cannot change system settings |
| Standard | Everyday use — run applications, change personal settings; no system-wide changes |
| Administrator | Full control — install software, modify system configuration, manage other users |

The principle of least privilege applies here: users should only have the permissions they need. Administrator accounts should not be used for day-to-day tasks.

---

### Windows Security

Windows Security is the central dashboard for managing Windows' built-in protection features. It is divided into four main areas:

| Section | Function |
|---|---|
| Virus & Threat Protection | Real-time malware detection and removal using Windows Defender Antivirus |
| Firewall & Network Protection | Controls inbound and outbound network traffic rules |
| App & Browser Control | Protects against unsafe apps, files, and malicious websites |
| Device Security | Hardware-based security features (e.g., Secure Boot, TPM) |

---

### Windows Defender Firewall

Windows Defender Firewall is the built-in host-based firewall. It monitors network connections and applies rules to allow or deny traffic. It operates across three network profiles:

| Profile | When It Applies |
|---|---|
| Domain | Connected to an organisation's Active Directory domain network |
| Private | Trusted networks — home or lab environments |
| Public | Untrusted networks — public Wi-Fi, coffee shops, airports |

Each profile can have different rules, allowing stricter controls on untrusted networks.

---

### Core Windows Interface Components

| Component | Description |
|---|---|
| Desktop | The main workspace — files, folders, and shortcuts live here |
| Taskbar | Control strip at the bottom — access to open apps, system tray, notifications |
| Start Menu | Primary launcher for applications, settings, and power options |
| Search | Quick access to applications, settings, and files |
| File Explorer | Built-in file manager for browsing and organising files and folders |
| Windows Update | Keeps the OS, native apps, and security definitions up to date |
| Microsoft Store | Official application marketplace for installing trusted apps |
| Windows Settings | Centralised configuration for system, devices, personalisation, and security |
| Control Panel | Legacy management interface — still used for some advanced configuration |
| Task Manager | Real-time monitoring of running processes, CPU/RAM usage, and services |

---

## Important Terminology

| Term | Definition |
|---|---|
| Authentication | The process of verifying a user's identity before granting access |
| Administrator | A Windows account with full system privileges |
| Standard User | A Windows account with limited, everyday-use permissions |
| Windows Defender | Microsoft's built-in antivirus and security suite |
| Firewall | A system that filters network traffic based on defined rules |
| Network Profile | A classification of a network connection (Domain, Private, Public) that determines firewall rules |
| Task Manager | Windows tool for monitoring and managing running processes |
| Windows Update | The built-in update mechanism for OS and security patches |

---

## Real-World Relevance

- Administrator accounts are a primary target in post-exploitation — attackers aim to escalate from standard user to administrator or SYSTEM
- Windows Defender has improved significantly and is now a legitimate first line of defence — attackers use obfuscation and living-off-the-land techniques to evade it
- Firewall rules are commonly misconfigured in corporate environments — overly permissive rules allow lateral movement
- Task Manager reveals running processes — useful for detecting malware, but sophisticated malware hides from it (rootkits)
- Windows Update is a critical security control — unpatched systems are a primary attack vector (e.g., EternalBlue exploited unpatched SMB)
- The Domain network profile is relevant in Active Directory environments — most corporate Windows machines operate in this profile

---

## Key Learnings

- Windows has three account types: Guest, Standard, and Administrator — least privilege applies
- Windows Security centralises antivirus, firewall, app control, and hardware security in one dashboard
- Windows Defender Firewall applies different rules based on the network profile (Domain, Private, Public)
- Core tools like Task Manager, File Explorer, and Windows Update are essential for both administration and security monitoring

---

## Additional Notes

- The SYSTEM account is the highest privilege level in Windows — above Administrator; many services run as SYSTEM
- UAC (User Account Control) prompts users before allowing actions that require elevated privileges — a basic but important security control
- PowerShell is the modern CLI for Windows administration and is heavily used in both legitimate administration and offensive security (living-off-the-land)
- Event Viewer logs system, security, and application events — a key tool for incident response on Windows

---

## Conclusion

Windows is the dominant target in enterprise security. Understanding its account model, built-in security tools, and core interface components is the starting point for both defending Windows environments and understanding how attackers operate within them.
