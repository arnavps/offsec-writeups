# Windows Fundamentals 2

## Overview

This room covers the built-in Windows administration and diagnostic tools — MSConfig, UAC, Task Scheduler, Event Viewer, Computer Management, System Information, Resource Monitor, and the Registry. These tools are used daily by administrators and are equally important for security professionals conducting investigations, privilege escalation, or system hardening.

---

## Topics Covered

- System Configuration (MSConfig)
- User Account Control (UAC) levels
- Task Scheduler
- Event Viewer
- Computer Management (compmgmt)
- System Information (msinfo32)
- Resource Monitor (resmon)
- Command Prompt (cmd)
- Registry Editor (regedt32)
- Quick-launch commands for each tool

---

## Key Concepts

### System Configuration — MSConfig

MSConfig (`msconfig`) is an advanced troubleshooting utility primarily used to diagnose startup issues. It has five tabs:

| Tab | Purpose |
|---|---|
| General | Choose startup mode: Normal, Diagnostic, or Selective |
| Boot | Configure boot options for the OS |
| Services | List all configured services (running or stopped) |
| Startup | Manage startup programs (redirects to Task Manager in modern Windows) |
| Tools | Launch various system utilities directly |

```cmd
msconfig
```

---

### User Account Control (UAC)

UAC controls how Windows alerts users when applications or system changes require elevated privileges. It has four levels:

| Level | Behaviour |
|---|---|
| Always notify | Notifies for all changes — apps and user-initiated. Desktop dims (Secure Desktop). Highest security. |
| Notify for apps (default) | Notifies only when apps try to make changes. User-initiated Windows settings changes are not prompted. |
| Notify without dimming | Same as above but the desktop does not dim. |
| Never notify | All notifications disabled. No warnings for any changes. Not recommended. |

```cmd
UserAccountControlSettings.exe
```

---

### Task Scheduler

Task Scheduler allows creating and managing automated tasks — running scripts, applications, or commands at specified times or events (login, logoff, schedule).

To view all scheduled tasks: open Task Scheduler → Task Scheduler Library.

```cmd
taskschd.msc
```

---

### Event Viewer

Event Viewer provides an audit trail of everything that has happened on the system — application events, security events, system events. Used for troubleshooting and incident investigation.

The interface has three panes:
1. Left — hierarchical tree of event log providers
2. Centre — overview and summary of events for the selected provider
3. Right — actions pane

```cmd
eventvwr.msc
```

---

### Computer Management — compmgmt

Computer Management consolidates several system tools into one interface, organised into three sections:

**System Tools**
- Task Scheduler — manage automated tasks
- Event Viewer — view system event logs
- Shared Folders — view shared folders and active sessions
- Device Manager — manage hardware devices

**Storage**
- Disk Management — partition management, drive letter assignment, extending/shrinking volumes

**Services & Applications**
- Services — start, stop, and configure Windows services
- WMI Control — configure Windows Management Instrumentation

```cmd
compmgmt.msc
```

**WMI (Windows Management Instrumentation)** allows scripting languages (PowerShell, VBScript) to manage Windows systems locally and remotely. The command-line interface is `wmic`.

---

### System Information — msinfo32

msinfo32 provides a comprehensive view of hardware, system components, and software environment. Useful for diagnosing issues and gathering system intelligence.

Three main sections:
- **System Summary** — OS version, processor, BIOS, RAM
- **Components** — hardware devices (display, input, network, storage)
- **Software Environment** — running tasks, startup programs, environment variables, network connections

```cmd
msinfo32.exe
```

---

### Resource Monitor — resmon

Resource Monitor displays real-time per-process and aggregate usage of CPU, memory, disk, and network. Also shows which processes hold specific file handles and modules.

Useful for identifying resource-heavy processes, investigating suspicious activity, and managing services from the UI.

```cmd
resmon.exe
```

---

### Command Prompt — cmd

The command prompt is the text-based interface for Windows. While the GUI handles most tasks, CMD is essential for scripting, remote administration, and security work.

```cmd
cmd
```

**Useful commands:**

```cmd
ipconfig              # Show network configuration
ipconfig /all         # Show detailed network configuration
control.exe           # Open Control Panel
```

---

### Registry Editor — regedt32

The Windows Registry is a centralised hierarchical database storing configuration for the OS, users, applications, and hardware. Windows references it continuously during operation.

```cmd
regedt32.exe
```

The registry is organised into hives (e.g., `HKEY_LOCAL_MACHINE`, `HKEY_CURRENT_USER`). Modifications here affect system behaviour directly — changes should be made carefully.

---

## Important Terminology

| Term | Definition |
|---|---|
| MSConfig | System Configuration utility for startup troubleshooting |
| UAC | User Account Control — prompts for elevation when privileged actions are attempted |
| Secure Desktop | A dimmed, isolated desktop shown during UAC prompts to prevent spoofing |
| Task Scheduler | Windows tool for automating tasks on a schedule or trigger |
| Event Viewer | Windows tool for viewing system, security, and application event logs |
| compmgmt | Computer Management — consolidated system administration console |
| WMI | Windows Management Instrumentation — API for managing Windows systems |
| msinfo32 | System Information tool — hardware and software overview |
| resmon | Resource Monitor — real-time process and resource usage |
| Registry | Central hierarchical database storing Windows configuration |
| WMIC | Command-line interface for WMI |

---

## Practical Examples / Demonstrations

### Quick-launch commands

```cmd
msconfig                                              # System Configuration
UserAccountControlSettings.exe                        # UAC Settings
compmgmt.msc                                          # Computer Management
taskschd.msc                                          # Task Scheduler
eventvwr.msc                                          # Event Viewer
msinfo32.exe                                          # System Information
resmon.exe                                            # Resource Monitor
regedt32.exe                                          # Registry Editor
control.exe                                           # Control Panel
C:\Windows\System32\control.exe /name Microsoft.Troubleshooting   # Troubleshooting
```

### Network configuration

```cmd
ipconfig
ipconfig /all
```

### Full path for ipconfig via MSConfig Tools tab

```cmd
C:\Windows\System32\cmd.exe /k %windir%\system32\ipconfig.exe
```

---

## Real-World Relevance

- Task Scheduler is a common persistence mechanism — attackers create scheduled tasks to re-establish access after reboot
- Event Viewer is the primary tool for Windows incident response — Security logs (Event ID 4624 for logon, 4625 for failed logon) are critical for detecting attacks
- The Registry is a primary target for persistence — malware frequently writes to `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` to execute on startup
- WMI is heavily abused for lateral movement and persistence in fileless attacks — WMI subscriptions can execute code without writing files to disk
- msinfo32 and resmon are used during post-exploitation to enumerate the system and identify running processes
- UAC bypass techniques are a standard part of Windows privilege escalation — understanding UAC levels helps assess the attack surface

---

## Key Learnings

- MSConfig diagnoses startup issues and controls what loads at boot
- UAC has four levels — "Always notify" is most secure; "Never notify" disables all protection
- Task Scheduler automates tasks and is a common persistence vector
- Event Viewer provides an audit trail — essential for incident investigation
- Computer Management consolidates disk, service, and event management in one place
- The Registry is the central configuration database — a critical target for both attackers and defenders
- Every major Windows tool has a quick-launch command — knowing them speeds up both administration and investigation

---

## Additional Notes

- Event IDs to know: 4624 (successful logon), 4625 (failed logon), 4688 (process creation), 4698 (scheduled task created)
- `wmic process list brief` lists running processes from the command line
- Registry hive `HKLM\SAM` stores local account password hashes — accessible only by SYSTEM
- Autoruns (Sysinternals) is a more comprehensive alternative to Task Scheduler and startup management for security investigations

---

## Conclusion

Windows Fundamentals 2 covers the administrative layer of Windows — the tools used to configure, monitor, and troubleshoot the system. For security professionals, these same tools are used to investigate incidents, identify persistence mechanisms, and understand system state. Knowing where to look and how to launch each tool quickly is a practical skill that applies directly to both blue team and red team work.
