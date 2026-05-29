# PowerShell — Part 4: System and Network Information

## Overview

PowerShell provides a rich set of cmdlets for gathering detailed system and network information — going well beyond what traditional CMD tools like `systeminfo` and `ipconfig` offer. These cmdlets are essential for system administration, security auditing, and post-exploitation enumeration.

---

## Topics Covered

- System information with `Get-ComputerInfo`
- Local user enumeration with `Get-LocalUser`
- Network configuration with `Get-NetIPConfiguration` and `Get-NetIPAddress`

---

## Key Concepts

### System Information — `Get-ComputerInfo`

Retrieves comprehensive system information in a single command — OS details, hardware specs, BIOS information, domain membership, and more. Significantly more detailed than the traditional `systeminfo` command.

```powershell
Get-ComputerInfo
Get-ComputerInfo | Select-Object CsName, OsName, OsVersion, BiosVersion
```

---

### Local User Accounts — `Get-LocalUser`

Lists all local user accounts on the system, including their status (enabled/disabled) and description.

```powershell
Get-LocalUser
Get-LocalUser | Select-Object Name, Enabled, Description
```

Useful for identifying hidden or unexpected accounts — a common persistence technique involves creating a low-profile local user.

---

### Network Configuration

**`Get-NetIPConfiguration`** — detailed network interface information including IP addresses, DNS servers, and default gateway. More structured than `ipconfig`.

```powershell
Get-NetIPConfiguration
Get-NetIPConfiguration | Select-Object InterfaceAlias, IPv4Address, DNSServer
```

**`Get-NetIPAddress`** — shows all IP addresses configured on the system, including inactive ones.

```powershell
Get-NetIPAddress
Get-NetIPAddress | Where-Object -Property AddressFamily -eq "IPv4"
```

---

## Important Terminology

| Term | Definition |
|---|---|
| `Get-ComputerInfo` | Retrieves comprehensive system hardware and OS information |
| `Get-LocalUser` | Lists local user accounts and their status |
| `Get-NetIPConfiguration` | Shows network interface configuration including IP, DNS, and gateway |
| `Get-NetIPAddress` | Lists all IP addresses configured on the system |
| `Enabled` | Property of a user account indicating whether it is active |
| `AddressFamily` | Specifies IPv4 or IPv6 |

---

## Practical Examples / Demonstrations

```powershell
# Full system info
Get-ComputerInfo

# Key system details only
Get-ComputerInfo | Select-Object CsName, OsName, OsVersion

# List all local users
Get-LocalUser

# Find enabled users only
Get-LocalUser | Where-Object -Property Enabled -eq $true

# Network configuration
Get-NetIPConfiguration

# IPv4 addresses only
Get-NetIPAddress | Where-Object -Property AddressFamily -eq "IPv4"
```

### Challenge walkthrough

```powershell
# Find a hidden local user
Get-LocalUser

# Navigate to their home directory
Set-Location C:\Users\p1r4t3

# List contents
Get-ChildItem

# Navigate to a subdirectory
Set-Location hidden-treasure-chest

# Read the file
Get-Content big-treasure.txt
```

---

## Real-World Relevance

- `Get-LocalUser` is a standard post-exploitation enumeration command — reveals all local accounts including hidden ones created for persistence
- `Get-ComputerInfo` provides OS version and patch level — used to identify applicable exploits
- `Get-NetIPConfiguration` reveals the internal network layout — used to plan lateral movement
- `Get-NetIPAddress` shows all configured IPs including VPN and virtual adapter addresses — useful for understanding the full network footprint of a compromised machine
- Filtering `Get-LocalUser` for enabled accounts with unusual descriptions is a practical technique for finding attacker-created accounts

---

## Key Learnings

- `Get-ComputerInfo` is far more detailed than `systeminfo` — covers hardware, OS, BIOS, and domain info
- `Get-LocalUser` lists all local accounts including disabled and hidden ones
- `Get-NetIPConfiguration` provides structured network interface data including DNS and gateway
- `Get-NetIPAddress` shows all IP addresses including inactive ones
- All of these cmdlets integrate with the pipeline — output can be filtered and sorted using `Where-Object` and `Select-Object`

---

## Conclusion

PowerShell's system and network information cmdlets provide a comprehensive view of a machine's configuration in a structured, filterable format. For security professionals, these cmdlets are the starting point for both system hardening audits and post-exploitation enumeration — revealing users, network layout, and system details that inform the next steps of an engagement.
