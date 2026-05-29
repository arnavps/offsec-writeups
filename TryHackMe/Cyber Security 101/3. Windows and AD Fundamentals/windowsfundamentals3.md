# Windows Fundamentals 3

## Overview

This room covers the security-focused features built into Windows — Windows Update, Windows Security (Defender), the firewall, BitLocker, and Volume Shadow Copy Service. These are the native defences that protect a Windows system before any third-party security tooling is introduced. Understanding them is essential for both hardening Windows environments and knowing what attackers try to disable or bypass.

---

## Topics Covered

- Windows Update and Patch Tuesday
- Windows Security dashboard
- Windows Defender Antivirus — scan types, settings, exclusions
- Windows Defender Firewall — profiles
- BitLocker Drive Encryption
- Volume Shadow Copy Service (VSS)

---

## Key Concepts

### Windows Update

Windows Update delivers security patches, feature updates, and fixes for Windows and Microsoft products (including Defender). Updates are typically released on the second Tuesday of each month — known as **Patch Tuesday**.

Critical or urgent patches can be pushed outside of this schedule at any time.

```cmd
control /name Microsoft.WindowsUpdate
```

---

### Windows Security Dashboard

Windows Security is the central hub for all built-in protection features. Status is indicated by colour:

| Colour | Meaning |
|---|---|
| Green | Device is sufficiently protected |
| Yellow | A safety recommendation needs review |
| Red | Immediate attention required |

**Protection areas:**
- Virus & threat protection
- Firewall & network protection
- App & browser control
- Device security

---

### Windows Defender Antivirus

**Scan types:**

| Scan | Description |
|---|---|
| Quick scan | Checks common threat locations — fast |
| Full scan | Scans all files and running programs — can take over an hour |
| Custom scan | User-defined files and locations |

**Threat history:**
- Last scan results
- Quarantined threats — isolated, prevented from running, periodically removed
- Allowed threats — items flagged but manually permitted to run (use with caution)

**Key settings:**

| Setting | Description |
|---|---|
| Real-time protection | Continuously monitors for and blocks malware |
| Cloud-delivered protection | Uses Microsoft's cloud for faster, up-to-date threat detection |
| Automatic sample submission | Sends suspicious files to Microsoft for analysis |
| Controlled folder access | Blocks unauthorised apps from modifying protected folders — key ransomware defence |
| Exclusions | Files/folders excluded from scanning — reduces false positives but introduces risk |

**Ransomware protection:**
Controlled folder access must be enabled (requires real-time protection to be on). Only approved applications can modify files in protected folders.

**On-demand scan:**
Right-click any file or folder → "Scan with Microsoft Defender"

---

### Windows Defender Firewall

The firewall controls what network traffic is allowed in and out of the system based on rules applied per port and application.

**Three network profiles:**

| Profile | When Applied |
|---|---|
| Domain | Connected to a network with an Active Directory domain controller |
| Private | Trusted networks — home or office |
| Public | Untrusted networks — public Wi-Fi, airports, coffee shops |

Each profile can have different inbound and outbound rules. The public profile is the most restrictive by default.

---

### BitLocker Drive Encryption

BitLocker encrypts entire drives to protect data if a device is lost, stolen, or decommissioned. It integrates with the OS and is transparent to the user once unlocked.

**TPM (Trusted Platform Module):**
BitLocker provides the strongest protection when used with a TPM chip (version 1.2 or later). The TPM verifies that the system hasn't been tampered with while offline before releasing the encryption key.

Without TPM, BitLocker can still be used with a USB startup key or PIN.

---

### Volume Shadow Copy Service (VSS)

VSS creates point-in-time snapshots (shadow copies) of data on a drive. These snapshots are stored in the `System Volume Information` folder on each protected drive.

When System Protection is enabled, you can:
- Create restore points
- Perform system restores
- Configure restore settings
- Delete restore points

**Security relevance:**
Ransomware routinely targets and deletes VSS shadow copies to prevent recovery. If shadow copies are deleted and there is no offline or off-site backup, recovery from ransomware becomes impossible.

Common ransomware command to delete shadow copies:
```cmd
vssadmin delete shadows /all /quiet
```

This is a well-known indicator of ransomware activity in Windows event logs.

---

## Important Terminology

| Term | Definition |
|---|---|
| Patch Tuesday | The second Tuesday of each month — Microsoft's standard patch release day |
| Windows Defender | Microsoft's built-in antivirus and security suite |
| Real-time protection | Continuous monitoring that blocks threats as they appear |
| Controlled folder access | Feature that prevents unauthorised apps from modifying protected folders |
| Exclusion | A file or folder exempted from antivirus scanning |
| Firewall profile | A set of firewall rules applied based on network type (Domain, Private, Public) |
| BitLocker | Windows full-disk encryption feature |
| TPM | Trusted Platform Module — hardware chip that stores cryptographic keys |
| VSS | Volume Shadow Copy Service — creates point-in-time snapshots of drive data |
| Shadow copy | A snapshot of a volume at a specific point in time |
| Restore point | A saved system state that can be used to roll back changes |

---

## Practical Examples / Demonstrations

### Open Windows Update

```cmd
control /name Microsoft.WindowsUpdate
```

### Force GPO sync (also triggers update checks in domain environments)

```cmd
gpupdate /force
```

### Check VSS shadow copies

```cmd
vssadmin list shadows
```

### Delete all shadow copies (ransomware indicator — for awareness only)

```cmd
vssadmin delete shadows /all /quiet
```

---

## Workflow / Process

### How BitLocker Protects a Drive

```
System boots
        |
TPM verifies system integrity (no tampering detected)
        |
BitLocker releases encryption key
        |
Drive decrypts transparently — user accesses data normally

If TPM check fails (e.g., boot files modified):
        |
BitLocker enters recovery mode — requires recovery key
```

---

## Real-World Relevance

- Patch Tuesday is a critical date for both defenders (apply patches) and attackers (analyse newly disclosed CVEs for unpatched systems)
- Defender exclusions are abused by attackers — adding a malicious file's directory to the exclusion list prevents detection
- Controlled folder access directly mitigates ransomware — enabling it is a high-value defensive control
- VSS deletion (`vssadmin delete shadows`) is one of the most reliable indicators of ransomware in Windows event logs — monitoring for this command is a standard detection rule
- BitLocker protects data at rest — critical for laptops and devices that could be physically stolen
- The firewall's Domain profile is relevant in Active Directory environments — understanding it is important for lateral movement and network segmentation analysis
- Disabling real-time protection is a common attacker step after gaining access — monitoring for Defender state changes is a useful detection

---

## Key Learnings

- Windows Update delivers patches on Patch Tuesday — critical patches can arrive at any time
- Windows Security centralises antivirus, firewall, app control, and hardware security
- Defender's controlled folder access is the primary built-in ransomware defence
- Exclusions reduce false positives but create blind spots — they should be minimal and audited
- The firewall applies different rules based on network profile — Domain, Private, Public
- BitLocker encrypts drives and uses TPM for integrity verification
- VSS shadow copies enable recovery — ransomware deletes them; offline backups are the only reliable fallback

---

## Additional Notes

- Windows Security Center events are logged — Defender state changes (disabled, exclusions added) appear in the Windows event log
- BitLocker recovery keys should be stored in Active Directory or Azure AD for enterprise environments
- `wbadmin` is the Windows command-line backup tool — an alternative to VSS for backup management
- Microsoft Defender for Endpoint (MDE) is the enterprise version of Defender with EDR capabilities — significantly more powerful than the consumer version

---

## Conclusion

Windows Fundamentals 3 covers the native security layer of Windows. Update management, antivirus configuration, firewall profiles, disk encryption, and shadow copies are all controls that defenders rely on and attackers actively target or disable. Understanding how each works — and where each can be bypassed or abused — is essential knowledge for anyone working in Windows security.
