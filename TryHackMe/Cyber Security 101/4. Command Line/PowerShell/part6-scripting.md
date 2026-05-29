# PowerShell — Part 6: Scripting

## Overview

PowerShell scripting is the ability to write and execute sequences of commands stored in a `.ps1` file to automate tasks. Scripting is a force multiplier in cybersecurity — it enables automation of repetitive tasks, complex data processing, and remote execution across multiple machines. This part introduces the concept of scripting and covers `Invoke-Command` for remote execution.

---

## Topics Covered

- What scripting is and why it matters
- PowerShell scripting in cybersecurity contexts
- Remote execution with `Invoke-Command`

---

## Key Concepts

### What is Scripting?

A script is a text file containing a sequence of commands that the shell executes automatically. Instead of typing commands one by one, you write them once in a script and run the script whenever needed.

PowerShell scripts use the `.ps1` extension.

**Benefits:**
- Automates repetitive tasks
- Reduces human error
- Enables complex multi-step operations
- Can be scheduled, triggered, or executed remotely

---

### PowerShell Scripting in Cybersecurity

**Blue team / Defensive:**
- Automate log analysis and anomaly detection
- Extract Indicators of Compromise (IOCs) from event logs
- Scan systems for signs of intrusion
- Enforce security policies and configurations
- Automate incident response playbooks

**Red team / Offensive:**
- Automate system enumeration
- Execute remote commands across multiple targets
- Craft obfuscated scripts to bypass defences
- Simulate attack techniques for testing

**System administration:**
- Automate integrity checks and configuration management
- Monitor system health and respond to incidents
- Manage users, services, and software at scale

---

### Remote Execution — `Invoke-Command`

`Invoke-Command` executes commands on remote systems over PowerShell remoting (WinRM). It is one of the most powerful cmdlets for both administration and offensive use.

**Execute a cmdlet on a remote machine:**

```powershell
Invoke-Command -ComputerName RemotePC -ScriptBlock { Get-Service }
```

**Execute a local script on a remote machine:**

```powershell
Invoke-Command -ComputerName RemotePC -FilePath C:\scripts\audit.ps1
```

**Execute on multiple machines simultaneously:**

```powershell
Invoke-Command -ComputerName PC1, PC2, PC3 -ScriptBlock { Get-Process }
```

**With credentials:**

```powershell
$cred = Get-Credential
Invoke-Command -ComputerName RemotePC -Credential $cred -ScriptBlock { whoami }
```

The `-ScriptBlock { ... }` parameter accepts any PowerShell commands — no scripting knowledge required to use it effectively.

---

## Important Terminology

| Term | Definition |
|---|---|
| Script | A `.ps1` file containing a sequence of PowerShell commands |
| `Invoke-Command` | Executes commands on local or remote systems |
| `-ScriptBlock` | A block of PowerShell code passed to `Invoke-Command` for execution |
| `-ComputerName` | Specifies the target remote machine(s) |
| `-FilePath` | Specifies a local script file to execute on the remote machine |
| WinRM | Windows Remote Management — the protocol used by `Invoke-Command` |
| IOC | Indicator of Compromise — evidence of malicious activity |
| Obfuscation | Techniques used to disguise script content to evade detection |

---

## Practical Examples / Demonstrations

```powershell
# Execute Get-Service on a remote machine
Invoke-Command -ComputerName RoyalFortune -ScriptBlock { Get-Service }

# Run a local script on a remote machine
Invoke-Command -ComputerName Server01 -FilePath C:\scripts\collect_info.ps1

# Execute on multiple machines
Invoke-Command -ComputerName PC1, PC2, PC3 -ScriptBlock { Get-Process | Select-Object Name, CPU }

# Get system info from a remote machine
Invoke-Command -ComputerName RemoteHost -ScriptBlock { Get-ComputerInfo | Select-Object CsName, OsName }
```

---

## Real-World Relevance

- `Invoke-Command` is used by attackers for lateral movement — executing commands on remote machines after compromising credentials
- PowerShell remoting (WinRM) is enabled by default in many enterprise environments — making `Invoke-Command` a reliable lateral movement vector
- Defenders monitor PowerShell Script Block Logging and Module Logging to detect malicious scripts
- AMSI (Antimalware Scan Interface) inspects PowerShell scripts before execution — attackers use obfuscation to bypass it
- PowerShell Constrained Language Mode (CLM) restricts script capabilities — a defensive hardening measure
- Incident responders use `Invoke-Command` to collect forensic data from multiple machines simultaneously without needing to log into each one

---

## Key Learnings

- PowerShell scripting automates tasks that would otherwise require manual command entry
- Scripts are `.ps1` files — sequences of cmdlets executed in order
- `Invoke-Command` executes commands on remote machines using `-ScriptBlock` or `-FilePath`
- `-ComputerName` accepts multiple targets — enabling simultaneous execution across many machines
- PowerShell scripting is relevant to all cybersecurity roles — blue team, red team, and administration

---

## Additional Notes

- PowerShell execution policy controls whether scripts can run — `Get-ExecutionPolicy` shows the current setting; `Set-ExecutionPolicy Bypass` is commonly used to override it (requires admin)
- Script Block Logging (enabled via GPO) records all executed PowerShell code — a critical detection source
- `Invoke-Expression` (IEX) is frequently used in malicious one-liners to execute downloaded scripts — a common red flag in PowerShell logs
- PowerShell 5.0+ includes enhanced logging features — always enable them in enterprise environments

---

## Conclusion

PowerShell scripting transforms individual cmdlets into automated workflows. `Invoke-Command` extends this capability to remote systems, making PowerShell the primary tool for both large-scale administration and remote offensive operations. Understanding what PowerShell can do — and how it's logged and detected — is essential knowledge for both sides of the security equation.
