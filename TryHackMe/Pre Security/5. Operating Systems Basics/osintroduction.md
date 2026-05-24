# Operating Systems: Introduction

## Overview

An operating system is the core software layer that sits between hardware and applications. It manages every resource on the machine — CPU time, memory, storage, devices, and security — so that applications don't have to. Understanding what an OS does and how it's structured is foundational for understanding how systems are attacked and defended.

---

## Topics Covered

- What an operating system is and why it exists
- System privilege layers: kernel space vs user space
- Core OS responsibilities
- OS security functions
- OS interfaces: GUI vs CLI
- OS types and real-world examples

---

## Key Concepts

### What is an Operating System?

An OS is the invisible manager that coordinates all activity on a computer. Without it, every application would need to directly control the CPU, memory, storage, and devices — causing constant conflicts. The OS acts as the central organiser, handling all of this on behalf of applications.

- **Hardware** — the physical components (CPU, RAM, storage, devices)
- **Applications** — programs that request resources from the OS
- **Operating System** — the layer in between that manages and allocates everything

---

### System Privilege Layers

Modern operating systems enforce a strict separation between privileged and unprivileged code:

**Kernel Space**
The privileged core of the OS. The kernel runs here and has unrestricted access to the CPU, memory, storage, and all hardware. Code running in kernel space can do anything to the system.

**User Space**
Where all standard applications run. Applications here are deliberately prevented from accessing hardware directly. When an application needs to read a file, play audio, or open a network connection, it makes a **system call** — a request to the kernel to perform the action on its behalf.

This separation is a fundamental security boundary. Compromising kernel space means full control of the system.

---

### Core OS Responsibilities

| Responsibility | What the OS Does |
|---|---|
| Process Management | Creates, schedules, prioritises, and terminates processes; manages multitasking |
| Memory Management | Allocates RAM to processes, isolates process memory, uses virtual memory when RAM is low |
| File System Management | Organises files into directories, manages permissions, metadata, and paths |
| User Management | Handles user accounts, authentication, and access permissions |
| Device Management | Loads drivers and provides a hardware abstraction layer so apps can use devices uniformly |

---

### OS Security Functions

Before any third-party security tool is involved, the OS itself enforces baseline security:

- **Authentication** — verifies user identity via passwords, PINs, biometrics
- **Permissions** — controls what each user and application can read, write, or execute
- **Isolation** — keeps processes in separate protected memory spaces (kernel/user space separation)
- **System Protection** — prevents unauthorised modification of critical system files and settings

---

### OS Interfaces

**GUI (Graphical User Interface)**
Visual interaction — icons, windows, menus. Easier to use but less precise and slower for complex tasks. Most end users interact with the OS through a GUI.

**CLI (Command-Line Interface)**
Text-based interaction — commands typed directly. More precise, faster for advanced tasks, and essential for automation and scripting. Most security tools are CLI-based. Requires familiarity with command syntax.

---

### OS Types and Real-World Examples

| Type | Primary Use | Key Characteristics |
|---|---|---|
| Desktop | Personal computers, daily work, gaming | Rich GUI, runs many apps simultaneously |
| Server | Web hosting, databases, cloud back-end | Headless (no GUI), maximum uptime, multi-user, remote access |
| Mobile | Smartphones and tablets | Touch UI, power-efficient, app sandboxing |
| Embedded | Appliances, IoT, routers, cars | Minimal footprint, runs on limited hardware |
| Virtual/Cloud | VMs, containers, cloud instances | Lightweight, scalable, rapid deployment |

**Real-world OS families:**

| Category | OS | Examples/Versions |
|---|---|---|
| Desktop | Windows | Windows 10, Windows 11 |
| Desktop | macOS | Sonoma (14), Sequoia (15) |
| Desktop/Server | Linux | Ubuntu, Debian, Fedora, CentOS, Red Hat |
| Server | Windows Server | 2019, 2022, 2025 |
| Server | Unix | IBM AIX, Oracle Solaris |
| Mobile | Android | Android 14–16 |
| Mobile | iOS | iOS 17, 18 |
| Embedded/IoT | Embedded Linux | OpenWrt, Ubuntu Core, Yocto |
| Embedded/IoT | Real-Time OS | FreeRTOS, VxWorks, QNX |
| Cloud/VM | Cloud-optimised Linux | Ubuntu LTS, Amazon Linux, Rocky Linux |
| Container | Container-optimised | Alpine Linux, Bottlerocket, Flatcar Linux |

---

## Important Terminology

| Term | Definition |
|---|---|
| Operating System | Core software managing hardware, applications, and system resources |
| Kernel | The privileged core of the OS with direct hardware access |
| Kernel Space | The protected memory region where the kernel runs |
| User Space | The restricted environment where applications run |
| System Call | A request from user space to the kernel to perform a privileged operation |
| Process | A running instance of a program |
| Virtual Memory | Disk space used as an extension of RAM when physical memory is full |
| GUI | Graphical User Interface — visual, icon-based interaction |
| CLI | Command-Line Interface — text-based, command-driven interaction |
| Driver | Software that allows the OS to communicate with a hardware device |

---

## Real-World Relevance

- Kernel exploits are among the most powerful attacks — gaining kernel space access means full system compromise
- Privilege escalation attacks move from user space to kernel space or from a low-privilege user to an administrator/root
- Understanding OS types is essential for targeting the right attack surface — a server OS has different exposure than a desktop or embedded system
- Embedded and IoT OSes often lack security features present in desktop/server OSes, making them easier targets
- The CLI is the primary interface for security tools, remote administration, and post-exploitation activity

---

## Key Learnings

- The OS sits between hardware and applications, managing all resources and enforcing security
- Kernel space is privileged; user space is restricted — this separation is a core security boundary
- The OS handles process management, memory, file systems, users, and devices
- GUI is user-friendly; CLI is faster, more precise, and essential for security work
- Different OS types serve different purposes and have different security profiles

---

## Additional Notes

- Linux dominates server and cloud environments — most web infrastructure runs on Linux
- Windows dominates the desktop — most corporate endpoints run Windows, making it the primary target for malware
- Real-Time Operating Systems (RTOS) are used in safety-critical systems (aircraft, medical devices) where guaranteed response times are required
- Container-optimised OSes like Alpine Linux are stripped down to the minimum — smaller attack surface but also fewer built-in tools

---

## Conclusion

The operating system is the foundation everything else runs on. Understanding its structure — especially the kernel/user space separation and the core responsibilities it manages — is essential context for understanding how privilege escalation, kernel exploits, and OS-level attacks work.
