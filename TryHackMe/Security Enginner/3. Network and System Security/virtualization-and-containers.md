# Virtualization and Containers

## Executive Summary

Virtualization and containerization are the foundational technologies of modern cloud infrastructure, DevOps, and enterprise IT. Understanding how hypervisors, containers, Docker, and Kubernetes work — and how they are attacked and secured — is essential for security engineers operating in any modern environment. This room provides a thorough technical grounding in the virtualization stack from Type 1/Type 2 hypervisors through Docker container management to Kubernetes orchestration.

**Major concepts covered:** Virtualization concepts, Type 1 and Type 2 hypervisors, container engines, Docker (images, Dockerfiles, running containers), Kubernetes (K8s) orchestration features, and security implications throughout.

**Why it matters:** Modern infrastructure is virtualised and containerised. Misconfigured Docker daemons, privileged containers, and Kubernetes RBAC misconfigurations are consistently exploited in real-world attacks. Security engineers must understand the technology to audit it, harden it, and respond to incidents within it.

---

## Big Picture Overview

The evolution of compute infrastructure:

```
Physical Servers (1:1 OS per machine)
  → Expensive, underutilised, slow to provision
         ↓
Type 1 Hypervisors (multiple VMs per physical host)
  → Better utilisation; isolated VMs; fast provisioning
         ↓
Type 2 Hypervisors (VMs on top of existing OS)
  → Developer workstations, lab environments
         ↓
Containers (lightweight process isolation, shared kernel)
  → Faster startup; smaller footprint; ideal for microservices
         ↓
Kubernetes (orchestration of containers at scale)
  → Automated deployment, scaling, healing, rolling updates
```

Each layer solves scale and efficiency problems while introducing new security attack surfaces.

---

## Core Concepts

---

### Concept 1 — Virtualization Fundamentals

**What Is Virtualization?**
Virtualization abstracts physical hardware resources (CPU, memory, storage, network) into logical resources that can be allocated to virtual machines (VMs). A single physical machine can host multiple independent virtual machines, each running its own OS and applications.

**Why Organizations Need It:**

| Business Driver | Problem Solved |
|---------------|---------------|
| **Decrease expenses** | Fewer physical servers needed; reduced hardware, power, cooling costs |
| **Scale** | Allocate/deallocate VM resources dynamically based on workload |
| **Efficiency** | One physical server running at 80% utilisation instead of 8 servers at 10% |
| **Isolation** | Development, testing, and production environments isolated from each other |
| **Disaster recovery** | VMs can be snapshot, cloned, and migrated |

**Virtualization Structure:**
```
Physical Hardware (CPU, RAM, Storage, Network)
         ↓
Hypervisor (abstraction layer)
  → Allocates physical resources to virtual machines
  → Manages isolation between VMs
         ↓
Guest OS 1 | Guest OS 2 | Guest OS 3
App A       App B        App C
```

**Terminology:**
- **Host OS:** The operating system running directly on the physical hardware (for Type 2)
- **Guest OS:** The operating system running inside a virtual machine
- **Hypervisor:** The software creating and managing the abstraction layer

---

### Concept 2 — Type 1 Hypervisors (Bare Metal)

**What Is It?**
Type 1 hypervisors (bare metal hypervisors) run directly on physical hardware without an underlying host OS. The hypervisor itself is the operating system — it is often headless with a web-based management portal.

**Architecture:**
```
Physical Hardware
         ↓
Type 1 Hypervisor (IS the OS)
         ↓
VM1 | VM2 | VM3 | VM4
```

**Design Goals:** Scale, performance, resource efficiency. Designed to run hundreds of VMs. All physical resources dedicated to VMs — nothing "wasted" on a host OS.

**Examples:**
- VMware ESXi
- Proxmox VE (open-source)
- VMware vSphere
- Xen
- KVM (Kernel-based Virtual Machine, built into Linux kernel)

**Management:**
Typically managed via a remote web interface or CLI tool (VMware vCenter, Proxmox web UI). The hypervisor itself may have no physical display output.

**Security Considerations:**
- The hypervisor is the single point of failure for all hosted VMs — hypervisor vulnerabilities affect all guests
- VM escape attacks target hypervisor vulnerabilities to break out of a VM into the hypervisor layer
- Hypervisor management interfaces must be isolated on a dedicated management network
- Enable secure boot on the hypervisor itself to prevent bootkit attacks

---

### Concept 3 — Type 2 Hypervisors (Hosted)

**What Is It?**
Type 2 hypervisors run as applications on top of a pre-existing host OS. The hypervisor sits between the host OS and the VMs.

**Architecture:**
```
Physical Hardware
         ↓
Host OS (Windows, macOS, Linux)
         ↓
Type 2 Hypervisor Application
         ↓
VM1 | VM2
```

**Design Goals:** End-user convenience, developer environments, lab setups. Not designed for large-scale production deployments.

**Examples:**
- VMware Workstation (Windows/Linux)
- VMware Fusion (macOS)
- VirtualBox (open-source, cross-platform)
- Parallels (macOS)
- QEMU (open-source, flexible)

**Security Considerations:**
- A compromise of the host OS affects all VMs running on it
- Less performance isolation than Type 1 — host OS scheduling affects VM performance
- Type 2 is appropriate for isolated development/testing; never for production workloads

**Type 1 vs Type 2 Comparison:**

| Characteristic | Type 1 | Type 2 |
|---------------|--------|--------|
| Runs on | Bare hardware | Host OS |
| Performance | High | Lower (host OS overhead) |
| Scale | High (enterprise) | Low (personal/dev) |
| Management | Web UI / headless | Application GUI |
| Examples | ESXi, KVM, Proxmox | VirtualBox, VMware Workstation |
| Use case | Production data centre | Developer workstations, labs |

---

### Concept 4 — Containers

**What Are Containers?**
Containers are a lightweight form of virtualisation that share the host OS kernel while providing isolated user space (filesystem, processes, network namespace). Unlike VMs, containers do not run a separate OS — they run processes in isolated namespaces on the host kernel.

**VM vs Container Comparison:**

| Characteristic | Virtual Machine | Container |
|---------------|----------------|----------|
| Isolation | Hardware-level (own kernel) | Process-level (shared kernel) |
| OS overhead | Full guest OS | None (uses host kernel) |
| Startup time | Minutes | Seconds or less |
| Image size | GB range | MB range |
| Security isolation | Stronger | Weaker (shared kernel) |
| Use case | Full workloads, legacy apps | Microservices, web servers |

**Why Containers Emerged:**
Microservice architectures decompose applications into dozens of small, independently deployable services. Running each microservice in a separate VM wastes resources — a 200MB application shouldn't require a 20GB VM. Containers provide isolation with minimal overhead.

**Container Namespaces and cgroups (Linux):**
- **Namespaces:** Provide isolation for PID (processes), network, filesystem, IPC, UTS (hostname), user namespaces
- **cgroups (Control Groups):** Limit CPU, memory, and I/O resources available to a container

These Linux kernel features are what containers are actually built on — Docker and Kubernetes are abstractions over these primitives.

---

### Concept 5 — Docker

**What Is It?**
Docker is the dominant container platform. It provides tools to build container images, distribute them via Docker Hub, and run them as containers.

**Core Docker Concepts:**

| Concept | Description |
|---------|------------|
| **Dockerfile** | Blueprint for building a container image — defines base image and setup commands |
| **Image** | Read-only template built from a Dockerfile — stored in layers |
| **Container** | Running instance of an image — ephemeral by default |
| **Docker Hub** | Container image registry — equivalent to GitHub for images |
| **Docker Engine** | The daemon running containers on the host |

**Dockerfile Example:**
```dockerfile
# Base image
FROM python:3.11-alpine

# Set working directory
WORKDIR /app

# Copy application files
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# Expose port
EXPOSE 5000

# Run the application
CMD ["python", "app.py"]
```

**Docker Commands:**
```bash
# Pull an image from Docker Hub
docker pull cryillic/thm_example_app

# Run a container
docker run cryillic/thm_example_app

# Run with port exposure and detached mode
docker run -p 5000:5000 -d cryillic/thm_example_app
# -p 5000:5000: map host port 5000 to container port 5000
# -d: detached (background)

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs <container_id>

# Stop a container
docker stop <container_id>

# Execute a command inside a running container
docker exec -it <container_id> /bin/sh

# Build an image from a Dockerfile
docker build -t myapp:1.0 .
```

**Port Exposure (`-p` flag):**
Containers are isolated by default — no host ports are exposed. The `-p host_port:container_port` flag creates a mapping from a host port to a container port, making the container service accessible from outside the host.

**Docker Security Concerns:**

| Risk | Description | Mitigation |
|------|-------------|-----------|
| **Root containers** | Containers running as root — if compromised, root access inside container | Run as non-root user (`USER` in Dockerfile) |
| **Exposed Docker socket** | `/var/run/docker.sock` mounted in container = container escape to host | Never mount Docker socket in containers |
| **Privileged containers** | `--privileged` flag removes all namespace isolation | Only use privileged mode if absolutely required |
| **Outdated base images** | Vulnerable OS packages in base image | Use minimal base images (Alpine, distroless); scan with Trivy |
| **Secrets in images** | Credentials hardcoded in Dockerfile or image layers | Use Docker secrets, environment variables, or secrets manager |
| **Container-to-container attacks** | Default Docker network allows all containers to communicate | Use custom networks; isolate sensitive containers |

---

### Concept 6 — Kubernetes (K8s)

**What Is It?**
Kubernetes is a container orchestration platform that automates the deployment, scaling, management, and healing of containerised applications across a cluster of nodes.

**Why K8s Exists:**
Running containers with Docker works for single hosts. Running hundreds of containers across dozens of hosts — managing which host each runs on, restarting failed containers, rolling out updates without downtime, scaling based on traffic — requires orchestration. Kubernetes solves this.

**Key K8s Capabilities:**

| Capability | Description |
|-----------|------------|
| **Horizontal scaling** | Add more container instances (pods) as load increases — not more CPU/RAM |
| **Extensibility** | Dynamically modify cluster configuration without affecting running workloads |
| **Self-healing** | Automatically restart, replace, reschedule, or kill unhealthy containers |
| **Automated rollouts and rollbacks** | Progressively deploy new versions; automatically roll back on failure |
| **Service discovery and load balancing** | Route traffic to containers automatically |
| **Secrets and configuration management** | Manage sensitive configuration data separately from container images |

**K8s Architecture:**

```
Control Plane (Master)
  ├── API Server — accepts all K8s API requests
  ├── etcd — distributed key-value store for cluster state
  ├── Scheduler — decides which node runs each pod
  └── Controller Manager — maintains desired state

Worker Nodes
  ├── kubelet — agent ensuring containers are running
  ├── kube-proxy — network rules for pod communication
  └── Container Runtime (Docker, containerd, CRI-O)
         └── Pods (groups of containers)
```

**Key K8s Objects:**

| Object | Description |
|--------|------------|
| **Pod** | Smallest deployable unit — one or more containers sharing network and storage |
| **Deployment** | Manages replicated pods; handles updates and rollbacks |
| **Service** | Stable network endpoint for a group of pods — load balances traffic |
| **Namespace** | Virtual cluster within K8s — provides isolation and resource scoping |
| **ConfigMap** | Non-sensitive configuration data |
| **Secret** | Sensitive data (passwords, tokens) — base64-encoded (not encrypted by default) |

**kubectl Commands:**
```bash
# View cluster nodes
kubectl get nodes

# List pods in all namespaces
kubectl get pods --all-namespaces

# View pod details
kubectl describe pod <pod_name>

# View pod logs
kubectl logs <pod_name>

# Execute command in pod
kubectl exec -it <pod_name> -- /bin/sh

# Scale a deployment
kubectl scale deployment <name> --replicas=5

# Apply a configuration
kubectl apply -f deployment.yaml
```

**K8s Security Concerns:**

| Risk | Description | Mitigation |
|------|-------------|-----------|
| **Exposed API server** | K8s API without authentication = full cluster control | Enable RBAC; restrict API server access |
| **Over-permissive RBAC** | Service accounts with cluster-admin access | Principle of least privilege in RBAC |
| **Exposed K8s dashboard** | Default dashboard without auth = cluster access | Disable or restrict dashboard; require authentication |
| **Secrets in plaintext** | K8s Secrets are base64-encoded (not encrypted) | Enable etcd encryption at rest |
| **Privileged pods** | Pod running with root, host network, or host PID | Use Pod Security Standards/Admission Controllers |
| **Container escape** | Vulnerability in container runtime or kernel namespace | Regular runtime and OS updates; use seccomp/AppArmor profiles |
| **Supply chain** | Malicious container images in public registries | Use private registry; scan images before use |

---

## Architecture and Relationships

### Container Security Layers

```
Hardware Security
  (Secure boot, TPM, physical access controls)
         ↓
Hypervisor Security (if VMs host K8s nodes)
  (VM isolation, hypervisor hardening)
         ↓
Host OS Security
  (Kernel hardening, AppArmor/seccomp, regular updates)
         ↓
Container Runtime Security
  (Docker daemon access control, rootless containers)
         ↓
Container Image Security
  (Vulnerability scanning, minimal base images, no root)
         ↓
Kubernetes RBAC and Admission Control
  (Who can deploy what; pod security policies)
         ↓
Application Security
  (Secure coding, secrets management, network policies)
```

### Docker Attack Path

```
Misconfigured Docker:
  Docker socket mounted in container: /var/run/docker.sock
         ↓
  Attacker runs: docker run -v /:/mnt --rm -it alpine chroot /mnt sh
         ↓
  Full host filesystem access — container escape
         ↓
  Complete host compromise
```

This is why mounting the Docker socket inside a container is a critical misconfiguration.

---

## Security Engineer Perspective

### Hardening Checklist

**Docker:**
- Never run containers as root — add `USER appuser` to Dockerfile
- Never mount `/var/run/docker.sock` in containers
- Never use `--privileged` unless required with documented justification
- Use minimal base images (Alpine, distroless) — smaller attack surface
- Scan images with Trivy or Snyk before deployment
- Use read-only filesystems where possible: `--read-only`
- Set resource limits: `--memory 512m --cpus 1`

**Kubernetes:**
- Enable RBAC — never use `cluster-admin` for application service accounts
- Disable the Kubernetes dashboard or restrict access to admin IPs only
- Enable etcd encryption at rest for Secrets
- Use Network Policies to restrict pod-to-pod communication
- Use Pod Security Admission to prevent privileged pods in non-privileged namespaces
- Audit RBAC permissions regularly — least privilege
- Enable audit logging for the API server

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **cgroups** | Linux kernel control groups — limit CPU, memory, I/O for processes/containers |
| **Container** | Lightweight process isolation using Linux namespaces and cgroups |
| **Docker Hub** | Public container image registry |
| **Dockerfile** | Text file defining how to build a Docker container image |
| **etcd** | Distributed key-value store used by Kubernetes for cluster state |
| **Horizontal scaling** | Adding more instances of a service vs adding more resources to one instance |
| **Hypervisor** | Software creating an abstraction layer between hardware and VMs |
| **Image (Docker)** | Read-only template of a container filesystem and configuration |
| **K8s / Kubernetes** | Container orchestration platform for automated deployment and scaling |
| **kubectl** | Command-line tool for interacting with Kubernetes clusters |
| **Namespace (K8s)** | Virtual cluster providing resource isolation within a Kubernetes cluster |
| **Namespace (Linux)** | Kernel feature isolating process views of system resources |
| **Pod** | Smallest deployable K8s unit — one or more containers sharing network/storage |
| **RBAC** | Role-Based Access Control — controls who can do what in a Kubernetes cluster |
| **Self-healing** | K8s feature automatically restarting or rescheduling failed containers |
| **Type 1 Hypervisor** | Bare metal hypervisor running directly on hardware |
| **Type 2 Hypervisor** | Hosted hypervisor running as an application on top of a host OS |
| **VM escape** | Attack breaking out of a VM to access the hypervisor or other VMs |

---

## Exam and Interview Revision

### Must Remember

- Type 1 hypervisors run on bare metal — directly on hardware (ESXi, KVM, Proxmox)
- Type 2 hypervisors run on a host OS — as applications (VirtualBox, VMware Workstation)
- Containers share the host kernel — faster and lighter than VMs, less isolation
- Docker image = blueprint; container = running instance of an image
- `-p host:container` exposes container ports; `-d` runs detached; `-v` mounts volumes
- Mounting Docker socket in container = container escape risk — never do it
- K8s smallest unit = Pod; Deployment manages replica sets; Service exposes pods
- K8s Secrets are base64-encoded — NOT encrypted; enable etcd encryption at rest
- K8s RBAC — principle of least privilege; never cluster-admin for application accounts
- `kubectl get pods`, `kubectl describe`, `kubectl logs`, `kubectl exec` are key diagnostic commands
- Self-healing in K8s: automatically restarts, replaces, reschedules unhealthy containers
- Horizontal scaling = more instances; vertical scaling = more CPU/RAM per instance

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What is the difference between a VM and a container? | VM has its own OS kernel — full isolation. Container shares host kernel — lighter and faster but weaker isolation. VMs are better for workload isolation; containers for microservices and scale. |
| What is the difference between Type 1 and Type 2 hypervisors? | Type 1 runs on bare metal (is the OS). Type 2 runs on a host OS. Type 1 is for production scale; Type 2 for personal/dev environments. |
| What makes mounting the Docker socket dangerous? | `/var/run/docker.sock` is the Docker daemon API socket. Mounting it in a container allows the container to control the Docker daemon on the host — equivalent to root host access. Classic container escape technique. |
| What is a Kubernetes Pod? | The smallest deployable unit in K8s. Contains one or more containers that share a network namespace and storage volumes. Pods are ephemeral — they are created, killed, and replaced by the scheduler. |
| How does Kubernetes self-healing work? | K8s monitors container health via liveness probes. If a container fails its health check, K8s automatically restarts it. If a node fails, K8s reschedules the pods on healthy nodes. |
| What is the risk with Kubernetes Secrets? | K8s Secrets are only base64-encoded — not encrypted. Anyone with access to etcd can decode them. Mitigation: enable etcd encryption at rest; use an external secrets manager (HashiCorp Vault, AWS Secrets Manager). |
