# Inside a Computer

## Overview

Before you can secure a system, you need to understand what it's made of. This room covers the core hardware components of a computer — what each one does, how they connect, and what happens from the moment you press the power button to a fully booted OS. This is the foundation for understanding attack surfaces at the hardware and firmware level.

---

## Topics Covered

- Core hardware components: CPU, RAM, storage, motherboard, GPU, NIC, PSU
- Input/Output devices
- The boot process (POST → UEFI → Bootloader → OS)

---

## Key Concepts

### Core Hardware Components

**CPU (Central Processing Unit)**
The processor — executes instructions. Modern CPUs have multiple cores that handle instructions in parallel, significantly increasing throughput. The CPU connects to the motherboard via a CPU socket.

**RAM (Random Access Memory)**
Short-term, volatile memory. Holds data the CPU needs quick access to during active tasks. When power is lost, all contents are wiped. Modern RAM uses DDR4/DDR5 technology for high-speed access.

**Storage (HDD / SSD)**
Long-term, non-volatile memory. Data persists after power loss.

| Type | Technology | Speed | Cost |
|---|---|---|---|
| HDD | Spinning magnetic platters | Slower | Cheaper per GB |
| SSD | Flash memory chips (no moving parts) | Much faster | More expensive per GB |

Both connect via SATA cables or PCIe slots.

**Motherboard**
The central circuit board that physically connects and allows communication between all components. Houses the CPU socket, RAM slots, PCIe expansion slots, and I/O ports. Every other component plugs into or through it.

**Network Adapter (NIC)**
Allows the computer to communicate over a network. Available in wired (Ethernet) and wireless (Wi-Fi) variants. Often integrated into the motherboard but can also be added as a PCIe expansion card.

**PSU (Power Supply Unit)**
Converts AC power from the wall outlet into DC power for system components. Distributes power via connectors to the motherboard, drives, and other components. If components demand more power than the PSU can supply, the system will fail or become unstable.

**Graphics Card (GPU)**
Processes and renders visual output to a display. Connects via PCIe slots on the motherboard. Dedicated GPUs are used for gaming, video editing, and compute-heavy tasks like machine learning.

**Input / Output (I/O) Devices**
- Input: keyboard, mouse, microphone, scanner, webcam
- Output: monitor, speakers, printer
- Common connectors: USB, HDMI, DisplayPort

---

### The Boot Process

What happens between pressing the power button and seeing the desktop:

**Step 1 — Power Button**
The power button sends a signal to the PSU to begin supplying power to all components.

**Step 2 — Firmware Initialisation (UEFI/BIOS)**
The CPU starts executing firmware stored on a chip on the motherboard. This firmware is called UEFI (Unified Extensible Firmware Interface) — the modern replacement for BIOS. It initialises and configures hardware components so they're ready to use.

**Step 3 — POST (Power-On Self Test)**
UEFI runs a diagnostic routine that checks whether all required hardware components are present, correctly configured, and functional. If a critical component fails, the system halts and may emit beep codes or display an error.

**Step 4 — Select Boot Device**
UEFI checks a prioritised list of boot devices (e.g., SSD, USB drive, network) to find one containing a bootloader.

**Step 5 — Bootloader**
The bootloader on the selected device is executed. It loads the Operating System from storage into RAM and hands control of the hardware over to the OS.

---

## Important Terminology

| Term | Definition |
|---|---|
| CPU | Central Processing Unit — executes program instructions |
| RAM | Random Access Memory — volatile short-term memory |
| HDD | Hard Disk Drive — magnetic storage with moving parts |
| SSD | Solid State Drive — flash-based storage, no moving parts |
| Motherboard | Main circuit board connecting all components |
| NIC | Network Interface Card — enables network communication |
| PSU | Power Supply Unit — converts and distributes power |
| GPU | Graphics Processing Unit — renders visual output |
| UEFI | Unified Extensible Firmware Interface — modern firmware replacing BIOS |
| BIOS | Basic Input/Output System — legacy firmware, largely replaced by UEFI |
| POST | Power-On Self Test — hardware diagnostic run at startup |
| Bootloader | Software that loads the OS from storage into RAM |

---

## Workflow / Process

### Boot Sequence

```
Power button pressed
        |
PSU supplies power to components
        |
CPU executes UEFI firmware
        |
POST — hardware diagnostic check
        |
UEFI scans boot device priority list
        |
Bootloader found and executed
        |
OS loaded from storage into RAM
        |
UEFI hands control to the OS
        |
Desktop / login screen appears
```

---

## Real-World Relevance

- UEFI/BIOS is a target for firmware-level malware (bootkits) — malicious code that persists below the OS and survives reinstallation
- Secure Boot (a UEFI feature) prevents unauthorised bootloaders from running — disabling it is sometimes required for dual-booting or running custom OS images
- RAM forensics is a key technique in incident response — volatile memory contains running processes, encryption keys, credentials, and network connections that disappear on shutdown
- Physical access to a machine allows an attacker to boot from a USB device and bypass OS-level security entirely — UEFI boot order and password protection mitigate this
- SSDs complicate forensic data recovery compared to HDDs due to TRIM and wear-levelling behaviour

---

## Key Learnings

- Every component has a specific role — CPU processes, RAM holds active data, storage persists data, the motherboard connects everything
- HDDs are cheaper and larger; SSDs are faster with no moving parts
- UEFI replaces BIOS and manages the boot process from power-on to OS handoff
- POST verifies hardware health at every startup
- The boot sequence is a critical security boundary — controlling what runs at boot is fundamental to system integrity

---

## Additional Notes

- UEFI settings are accessible at startup (usually via F2, F10, or DEL) and control boot order, Secure Boot, and hardware configuration
- Cold boot attacks exploit the fact that RAM contents don't disappear instantly after power loss — an attacker can freeze RAM and read its contents
- NVMe SSDs connect via PCIe and are significantly faster than SATA SSDs

---

## Conclusion

Understanding computer hardware isn't just for sysadmins — it directly informs how attacks work at the lowest levels. From firmware exploits to RAM forensics to physical access attacks, the hardware layer is where some of the most impactful and persistent threats operate.
