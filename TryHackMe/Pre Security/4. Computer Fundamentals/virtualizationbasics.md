# Virtualization Basics

## Overview

Virtualization changed how computing infrastructure is built and managed. Instead of one physical server per application, virtualization allows multiple isolated environments to share the same hardware. For cybersecurity, this is especially relevant — virtual machines and containers are the standard environment for malware analysis, penetration testing labs, and cloud infrastructure.

---

## Topics Covered

- Why virtualization exists
- Hypervisors (Type 1 and Type 2)
- Virtual Machines (VMs)
- Containers
- Docker
- Key benefits of virtualization

---

## Key Concepts

### The Problem Virtualization Solved

Before virtualization, the standard was one physical server per application. This created several problems:

- **High cost** — hardware, electricity, cooling, and data center space for each server
- **Low utilisation** — most servers ran at 5–20% capacity, wasting resources
- **Slow deployment** — provisioning new physical servers took days or weeks
- **Poor scalability** — adding capacity meant buying more hardware

Virtualization introduced the ability to run multiple isolated environments on a single physical machine, solving all of the above.

---

### Hypervisors

A hypervisor is the software layer that makes virtualization possible. It sits between the physical hardware and the virtual machines, managing resource allocation and isolation.

**What a hypervisor does:**
- Divides a physical machine into multiple virtual ones
- Allocates CPU, RAM, and storage to each VM
- Keeps VMs isolated from each other
- Manages VM lifecycle: create, start, stop, pause, clone, delete

**Two types:**

| Type | Runs On | Use Case | Examples |
|---|---|---|---|
| Type 1 (Bare-metal) | Directly on physical hardware | Production servers, data centers | VMware ESXi, Microsoft Hyper-V, Proxmox |
| Type 2 (Hosted) | On top of an existing OS | Learning, testing, home labs | VirtualBox, VMware Workstation |

Type 1 is faster and more efficient — preferred for production. Type 2 is easier to set up — preferred for personal use and labs.

**Use case guidance:**

| Use Case | Best Hypervisor Type |
|---|---|
| Production server | Type 1 |
| Database server | Type 1 |
| Data center | Type 1 |
| Malware analysis | Type 2 |
| Software testing | Type 2 |
| Running Kali Linux | Type 2 |

---

### Virtual Machines (VMs)

A VM is a complete virtual computer created and managed by a hypervisor. Despite being virtual, it behaves exactly like a physical machine:

- Has its own virtual CPU, RAM, storage, and network interface
- Runs a full operating system (Windows, Linux, etc.)
- Is fully isolated from other VMs — a crash or compromise in one VM doesn't affect others

**Security use cases:**
- Run Kali Linux on a Windows host without a second machine
- Test potentially malicious files in an isolated environment without risking the host

**Important note on malware analysis in VMs:**
When testing malicious files, care must be taken to prevent the malware from escaping the VM and infecting the host. Best practices include using a different OS for host and guest, disabling shared folders, and isolating the VM's network.

---

### Containers

A container is a lightweight, isolated environment for running a single application and its dependencies. Unlike VMs, containers do not include a full OS — they share the host system's kernel.

**Key characteristics:**
- Package the application and all its dependencies (libraries, tools, runtime versions)
- Share the host OS kernel — start almost instantly and use far fewer resources than VMs
- Isolated from each other — a misbehaving container doesn't affect others
- Portable — run consistently on any machine with a compatible kernel

**Limitation:** Containers must match the host system type. A Windows container cannot run on a Linux host (and vice versa) without additional compatibility layers.

**Docker** is the most widely used container platform. It simplifies building, deploying, and running containers through a straightforward CLI and image-based workflow.

---

### VMs vs Containers

| | Virtual Machine | Container |
|---|---|---|
| Includes full OS | Yes | No (shares host kernel) |
| Startup time | Minutes | Seconds |
| Resource usage | Higher | Much lower |
| Isolation level | Strong (separate kernel) | Process-level isolation |
| Best for | Full OS environments, malware analysis | Application deployment, microservices |

---

## Important Terminology

| Term | Definition |
|---|---|
| Virtualization | Running multiple isolated environments on a single physical machine |
| Hypervisor | Software that creates and manages virtual machines |
| Type 1 Hypervisor | Runs directly on hardware — used in production environments |
| Type 2 Hypervisor | Runs on top of an OS — used for testing and home labs |
| Virtual Machine (VM) | A complete virtual computer with its own OS, isolated from others |
| Container | A lightweight isolated environment sharing the host OS kernel |
| Container Image | A pre-built template used to create and deploy containers |
| Docker | Open-source platform for building and running containers |
| Kernel | The core of an OS that manages hardware resources |

---

## Real-World Relevance

- VMs are the standard environment for malware analysis and reverse engineering — isolation prevents the malware from affecting the analyst's machine
- Containers (Docker, Kubernetes) power most modern cloud infrastructure — understanding them is essential for cloud security and DevSecOps
- Container escape vulnerabilities allow an attacker to break out of a container and access the host system — a critical class of vulnerability in containerised environments
- Snapshots (VM state saves) allow analysts to revert to a clean state after testing malware, making repeated analysis efficient
- Misconfigured Docker sockets (`/var/run/docker.sock`) exposed to containers are a common privilege escalation vector

---

## Key Learnings

- Virtualization allows multiple isolated environments to share one physical machine
- Hypervisors manage VMs — Type 1 for production, Type 2 for labs and testing
- VMs provide strong isolation with a full OS; containers are lightweight and share the host kernel
- Docker is the standard tool for container deployment
- Both VMs and containers are central to modern security workflows — from malware analysis to cloud infrastructure

---

## Additional Notes

- Popular Type 2 hypervisors for security work: VirtualBox (free), VMware Workstation (paid)
- Kali Linux is available as a pre-built VM image from the official Kali website
- Container images are pulled from registries like Docker Hub — always verify image sources to avoid supply chain risks
- Kubernetes (K8s) is the standard orchestration platform for managing containers at scale

---

## Conclusion

Virtualization is foundational to modern IT and cybersecurity. Whether it's spinning up a Kali Linux VM for a pentest, isolating malware in a sandbox, or understanding the container infrastructure that runs cloud applications, virtualization concepts appear constantly in security work. Knowing the difference between VMs and containers — and when to use each — is a practical skill from day one.
