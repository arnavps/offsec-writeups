# PowerShell — Part 1: Introduction

## Overview

PowerShell is Microsoft's task automation and configuration management framework, combining a command-line shell with a scripting language built on the .NET framework. Unlike CMD, PowerShell is object-oriented — commands return structured objects rather than plain text, enabling far more powerful data manipulation. Originally Windows-only, PowerShell is now cross-platform (Windows, macOS, Linux).

---

## Topics Covered

- What PowerShell is and why it was created
- Objects vs plain text output
- Cmdlets and the Verb-Noun naming convention
- Core discovery cmdlets: `Get-Command`, `Get-Help`, `Get-Alias`
- Finding and installing modules from online repositories

---

## Key Concepts

### Why PowerShell Exists

CMD and batch scripts were insufficient for managing complex enterprise Windows environments. Microsoft needed a tool that could interact with modern Windows APIs, handle structured data, and support sophisticated automation. PowerShell was built to fill that gap.

---

### Objects vs Plain Text

Traditional shells output plain text — to extract information, you have to parse strings. PowerShell cmdlets return **objects** with properties and methods. This means you can directly access specific attributes without text parsing.

Example: a file object has properties like `Name`, `Length`, `LastWriteTime` and methods like `CopyTo()`, `Delete()`.

This object-based approach is what makes PowerShell significantly more powerful than CMD for data manipulation and automation.

---

### Cmdlets

PowerShell commands are called **cmdlets** (pronounced "command-lets"). They follow a consistent `Verb-Noun` naming convention:

- `Get-Content` — retrieves the content of a file
- `Set-Location` — changes the current working directory
- `Remove-Item` — deletes a file or directory

The verb describes the action; the noun specifies what it acts on. This makes cmdlets intuitive to discover and remember.

---

### Core Discovery Cmdlets

**`Get-Command`** — lists all available cmdlets, functions, aliases, and scripts in the current session.

```powershell
Get-Command
Get-Command -Name Remove*                    # Filter by name pattern
Get-Command -CommandType Function            # Filter by type
```

**`Get-Help`** — provides detailed documentation for any cmdlet, including parameters and examples.

```powershell
Get-Help Get-Content
Get-Help New-LocalUser -Examples             # Show usage examples
```

**`Get-Alias`** — lists all aliases (shortcuts for cmdlets). Many traditional CMD commands are aliases in PowerShell.

```powershell
Get-Alias
# dir  → Get-ChildItem
# cd   → Set-Location
# cat  → Get-Content
```

---

### Finding and Installing Modules

PowerShell's functionality can be extended by downloading modules from online repositories like the PowerShell Gallery.

```powershell
Find-Module -Name *network*          # Search for modules by partial name
Install-Module -Name <ModuleName>    # Download and install a module
```

Note: these commands require internet access.

---

## Important Terminology

| Term | Definition |
|---|---|
| Cmdlet | A PowerShell command following the Verb-Noun naming convention |
| Object | A structured data unit with properties and methods |
| Property | A characteristic of an object (e.g., file name, size) |
| Method | An action an object can perform (e.g., copy, delete) |
| Alias | A shortcut name for a cmdlet (e.g., `dir` for `Get-ChildItem`) |
| Module | A collection of cmdlets packaged together |
| PowerShell Gallery | Microsoft's online repository for PowerShell modules |
| .NET Framework | The runtime PowerShell is built on |

---

## Practical Examples / Demonstrations

```powershell
# List all commands starting with Remove
Get-Command -Name Remove*

# Get help with examples for New-LocalUser
Get-Help New-LocalUser -Examples

# Find the cmdlet aliased as echo
Get-Alias echo
# → Write-Output

# List all available aliases
Get-Alias
```

---

## Real-World Relevance

- PowerShell is the primary tool for Windows administration and automation in enterprise environments
- Red teamers use PowerShell for enumeration, lateral movement, and payload execution — its deep OS integration makes it a powerful offensive tool
- Blue teamers use PowerShell for log analysis, threat hunting, and incident response automation
- PowerShell logging (Script Block Logging, Module Logging, Transcription) is a critical detection source — understanding what PowerShell can do helps defenders know what to monitor
- `Invoke-Command` enables remote execution across multiple machines — used by both administrators and attackers

---

## Key Learnings

- PowerShell returns objects, not plain text — this enables direct property access without string parsing
- Cmdlets follow `Verb-Noun` naming — making them discoverable and consistent
- `Get-Command`, `Get-Help`, and `Get-Alias` are the three essential discovery cmdlets
- Modules extend PowerShell's capabilities and can be installed from the PowerShell Gallery
- PowerShell is cross-platform — available on Windows, macOS, and Linux

---

## Conclusion

PowerShell is the modern Windows command-line environment. Its object-oriented design, consistent naming convention, and deep integration with the OS make it significantly more powerful than CMD for both administration and security work. The discovery cmdlets — `Get-Command`, `Get-Help`, `Get-Alias` — are the starting point for learning any new PowerShell capability.
