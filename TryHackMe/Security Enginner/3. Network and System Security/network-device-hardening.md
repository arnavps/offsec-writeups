# Network Device Hardening

## Executive Summary

Network devices — routers, switches, VPNs, firewalls, and load balancers — are the backbone of enterprise connectivity. Unlike endpoint devices which are managed by endpoint security tools, network devices are frequently under-hardened: running default credentials, outdated firmware, and unnecessary services. Compromising a router or VPN server gives an attacker unparalleled visibility and control over all network traffic traversing it.

**Major concepts covered:** Endpoint vs network device distinctions, common attack vectors, general hardening techniques, secure protocols, VPN hardening (OpenVPN), router/switch hardening (OpenWrt), network monitoring tools, and enterprise-specific controls (port security, ARP spoofing prevention, DHCP protection).

**Why it matters:** A compromised router or switch is a persistent, network-level MITM attack point. Attackers with control of network infrastructure can intercept credentials, manipulate DNS, redirect traffic, and pivot to any reachable network segment — silently and persistently.

---

## Big Picture Overview

Network devices differ fundamentally from endpoint devices in their role, configuration model, and hardening requirements:

```
Endpoint Device (Workstation, Server)
  → Runs applications and services for users
  → Hardened via OS controls, AV, EDR, patch management
  → Typically has a full OS (Windows, Linux)

Network Device (Router, Switch, Firewall)
  → Forwards, filters, and manages traffic between network segments
  → Hardened via device-specific configuration, firmware updates
  → Often runs a stripped-down embedded OS (IOS, JunOS, OpenWrt)
  → No user-facing applications
```

---

## Core Concepts

---

### Concept 1 — Endpoint vs Network Devices

| Characteristic | Endpoint Device | Network Device |
|---------------|----------------|---------------|
| Primary function | Generate or consume data | Route, switch, and filter traffic |
| Examples | Laptops, servers, smartphones | Routers, switches, firewalls, VPN concentrators |
| Location | Network edge or internal | Network infrastructure |
| User interaction | Direct (keyboard, screen) | Remote management (SSH, web UI, serial) |
| Threat model | Malware, credential theft | Configuration exploitation, traffic interception |
| Management | OS-level tools, GPO, MDM | CLI (SSH/serial), web interface, SNMP |

---

### Concept 2 — Common Threats and Attack Vectors

| Threat | Description | Attack Vectors |
|--------|-------------|----------------|
| **Unauthorised access** | Gain control of the device | Default credentials, brute force, CVE exploitation, social engineering |
| **Denial of Service** | Disrupt network connectivity | Packet flooding, resource exhaustion, protocol manipulation |
| **Man-in-the-Middle** | Intercept traffic between devices | ARP spoofing, DNS spoofing, rogue access points |
| **Privilege escalation** | Gain admin access from limited access | Weak passwords, shared admin/user accounts, CVE exploitation |
| **Bandwidth theft** | Abuse device for traffic relay | Misconfigured port forwarding, DoS amplification |

---

### Concept 3 — General Hardening Techniques

#### Updating and Patching

Outdated firmware is the most common source of network device vulnerabilities. Unlike OS patch management, network device firmware must be tested before deployment — a failed router update can take down an entire network.

```bash
# OpenWrt firmware update
System → Software → Update firmware

# Cisco IOS example
# show version → identify current IOS version
# Check CCO for latest recommended release
# copy tftp flash → upload new firmware
```

**Best Practice:** Maintain a firmware inventory; subscribe to vendor security advisories; test updates on non-production devices first.

#### Disable Unnecessary Services and Ports

Every enabled service on a network device is a potential attack vector. Default configurations often enable services for convenience that are not required in a specific deployment.

**Services to disable if not needed:**
- HTTP management interface (use HTTPS only)
- SNMP v1/v2 (use SNMPv3 or disable)
- Telnet (use SSH only)
- FTP for firmware (use SCP/SFTP or HTTPS)
- CDP (Cisco Discovery Protocol) on untrusted interfaces
- Unused routing protocols (RIP, OSPF if not used)

#### Principle of Least Privilege (POLP)

Network device users should have only the access needed for their role:
- **Read-only access** for monitoring/NOC teams
- **Operator access** for routine configuration changes
- **Admin access** restricted to senior engineers

Never use admin accounts for routine monitoring tasks.

#### Strong Passwords and MFA

Default credentials on network devices are well-documented and widely known:
- Cisco routers: `admin/admin` or blank password
- OpenWrt: root with no password
- Many SOHO routers: `admin/admin`, `admin/password`

```bash
# OpenWrt: Change admin password
System → Administration → enter new password → Save

# Cisco IOS: Set enable secret (encrypted)
enable secret StrongPassword123!

# Set VTY line password for SSH access
line vty 0 4
 password VTYPassword
 login local
```

#### Multi-Factor Authentication (MFA)

For devices supporting RADIUS authentication, integrate with an MFA-capable RADIUS server (Cisco ISE, FreeRADIUS with Google Authenticator). This adds a second factor for administrative access to routers and switches.

#### Log Monitoring

```
Syslog → Forward all device logs to centralised syslog server
SNMP Traps → Send event notifications to management system
NetFlow → Export flow data for traffic analysis
Packet Capture → On-demand captures for incident analysis
```

#### Regular Backups

Before any change, back up the running configuration. Many network outages result from failed changes where the previous configuration was not saved.

```bash
# Cisco IOS: Save running config to TFTP
copy running-config tftp://192.168.1.100/router-backup.cfg

# OpenWrt: Backup via web interface
System → Backup / Flash Firmware → Generate Archive
```

---

### Concept 4 — Importance of Secure Protocols

**Replace insecure protocols with secure alternatives:**

| Insecure Protocol | Port | Secure Replacement | Port |
|------------------|------|-------------------|------|
| Telnet | 23 | SSH | 22 |
| HTTP management | 80 | HTTPS management | 443 |
| FTP | 21 | SFTP / SCP | 22 |
| SNMP v1/v2c | 161 | SNMPv3 | 161 |
| TFTP | 69 | HTTPS/SCP | — |
| rsh / rexec | 514 | SSH | 22 |

**Why insecure protocols are dangerous on network devices:**
- Telnet transmits admin credentials in plaintext — any on-path attacker captures them immediately
- HTTP management interfaces expose credentials to any network segment with routing access
- SNMP v1/v2c community strings are transmitted in cleartext and grant read/write access to device configuration

**Enabling SSH on OpenWrt:**
```
System → Administration → SSH Access
→ Select interface: LAN (never WAN unless VPN-protected)
→ Port: 22 (or custom port)
→ Save & Apply

# Optional: Add SSH public key for passwordless admin login
→ SSH-Keys section: paste public key
```

---

### Concept 5 — VPN Server Hardening (OpenVPN)

**Why VPN Security Matters:**
VPN servers are the gateway for remote access to the entire corporate network. A compromised VPN server allows an attacker to:
- Intercept all VPN tunnel traffic
- Impersonate any remote user
- Inject traffic into the corporate network without authentication

**OpenVPN Configuration File:** `/etc/openvpn/server/server.conf`

**Key Hardening Directives:**

#### Strong Encryption Cipher

```bash
# Use AES-256-CBC (strongest common cipher)
cipher AES-256-CBC

# Other acceptable options: AES-128-GCM, AES-256-GCM (faster on modern hardware)
# Avoid: DES, 3DES, RC4, Blowfish
```

#### Strong Authentication Algorithm

```bash
# Use SHA-256 for packet authentication (minimum)
auth SHA256

# Avoid: MD5, SHA1 (cryptographically broken)
# Better: SHA512 for high-security environments
```

#### Perfect Forward Secrecy (PFS)

PFS ensures that compromise of a session key does not compromise past or future sessions. Each VPN session uses a unique ephemeral key.

```bash
# Generate TLS pre-shared key for tls-crypt
sudo openvpn --genkey --secret my.key

# Enable PFS via tls-crypt directive
tls-crypt my.key

# Minimum TLS version (prevents downgrade attacks)
tls-version-min 1.2

# Combined hardened configuration example:
cipher AES-256-CBC
auth SHA256
tls-crypt my.key
tls-version-min 1.2
```

**Cipher Block Chaining (CBC) vs Galois/Counter Mode (GCM):**
- **CBC:** Older mode; requires HMAC for authentication separately
- **GCM:** Authenticated encryption — combines encryption and authentication; preferred for new deployments

#### Keep VPN Software Updated

```bash
# Update OpenVPN (requires internet connection)
sudo apt upgrade openvpn
```

#### Dedicated User Account

Create a dedicated service account with restricted permissions for the OpenVPN process — never run the VPN service as root in production.

#### Change Default Settings

Always change default ports, certificates, and any default values in the configuration. Automated scanners look for default OpenVPN configurations on UDP 1194.

---

### Concept 6 — Router and Switch Hardening (OpenWrt)

OpenWrt is an open-source Linux-based OS for network devices. The principles apply to all commercial routers and switches.

**Access:** `http://MACHINE_IP:8080` (web interface)

#### Initial Device Setup

Configure hostname, timezone, and logging before deployment:
```
System → System
→ Set hostname (identifies device in logs)
→ Set timezone (accurate timestamps in logs)
→ Enable logging at appropriate level (Debug or Info)
→ Configure NTP for time synchronisation
```

Accurate timestamps are critical for incident correlation. Log entries with wrong timestamps cannot be reliably correlated with events on other systems.

#### Change Default Credentials

```
System → Administration
→ Enter new strong password
→ Save
```

Never deploy a network device with default credentials in production.

#### Disable Unnecessary Startup Scripts

Malware targeting network devices adds persistence via startup scripts (cron jobs, init.d entries).

```
System → Startup
→ Review all startup entries
→ Disable/remove any unrecognised scripts
```

This is also a forensic indicator — new startup entries added after the initial deployment indicate potential compromise.

#### Traffic Rules and Port Forwarding

```
Network → Firewall → Traffic Rules
→ Review all rules
→ Remove any rules allowing unexpected external access

Network → Firewall → Port Forwards
→ Review all port forwarding rules
→ Threat actors add port forwarding rules to tunnel C2 traffic
```

Monitoring scheduled tasks and port forwarding rules for unauthorised modifications is a key indicator of compromise for network devices.

#### Monitor Real-Time Traffic

```
Status → Realtime Graph → Traffic
→ Unusual upload spikes may indicate data exfiltration
→ Unusual traffic to unknown external IPs
```

#### Securing Wi-Fi

```
Network → Wireless
→ WPA2 minimum; WPA3 preferred
→ Disable WPS (Wi-Fi Protected Setup — has known brute force vulnerabilities)
→ Disable SSID broadcast for sensitive networks (obscurity — not a primary control)
→ Use strong, unique PSK
→ Separate guest SSID on isolated VLAN
```

---

### Concept 7 — Enterprise Network Device Controls

#### Port Security (Switches)

**What It Does:** Limits the number of MAC addresses allowed on a switch port. If an unauthorised MAC address appears, the port can shut down, restrict, or send an alert.

**Prevents:**
- MAC flooding attacks (filling the switch's MAC address table to force it into hub mode)
- Unauthorised devices connecting to the network via a switched port

#### Preventing ARP Spoofing

- **Static ARP entries:** Manually configure ARP entries for critical hosts (gateway, DNS servers) — cannot be overridden by spoofed ARP replies
- **MAC address filtering:** Allow only known MAC addresses on specific ports
- **Dynamic ARP Inspection (DAI):** Layer-2 switch feature validating ARP against DHCP binding database (covered in Room 1)

#### Preventing Rogue DHCP Servers

Rogue DHCP servers assign attacker-controlled gateways and DNS servers to clients, enabling MITM:
- **DHCP Snooping:** Blocks DHCP responses on untrusted ports (covered in Room 1)
- **Static DHCP bindings:** Assign fixed IPs to known devices — unexpected DHCP requests stand out
- **Network mapping:** Continuously discover connected devices; alert on unknown devices

#### Enabling IPv6

IPv6 includes built-in IPsec support, providing:
- Authentication header (AH) for integrity and authenticity
- Encapsulating Security Payload (ESP) for confidentiality
- Protection against MITM, eavesdropping, and packet tampering

IPv4 networks are more vulnerable to MITM because ARP (used with IPv4) has no authentication. IPv6 uses NDP (Neighbour Discovery Protocol) with optional SEND (Secure Neighbour Discovery) providing cryptographic validation.

---

### Concept 8 — Network Monitoring Tools

| Tool | Type | Primary Use |
|------|------|------------|
| **Nagios** | Open-source | System, network, and infrastructure monitoring; real-time alerting |
| **SolarWinds NPM** | Commercial | Network performance monitoring; network mapping; customisable dashboards |
| **PRTG** | Commercial | All-in-one monitoring; real-time traffic analysis; customisable alerts |
| **Zabbix** | Open-source | Performance and availability monitoring; customisable dashboards and alerting |

**What to Monitor on Network Devices:**
- Interface utilisation (bandwidth)
- CPU and memory utilisation
- BGP/OSPF neighbour state changes
- Interface flapping (up/down events)
- Configuration change events
- Authentication failures
- New ARP entries from unexpected sources

---

## Architecture and Relationships

### Network Device Security Layers

```
Physical Access Control
  (Server room locks, cable management, console port locks)
         ↓
Firmware and Software Security
  (Latest firmware, disable unnecessary services, secure protocols)
         ↓
Authentication and Access Control
  (Strong passwords, MFA, least privilege, SSH keys)
         ↓
Traffic Security
  (Zone-pair firewalls, traffic rules, port security)
         ↓
Monitoring and Logging
  (Syslog, SNMP, NetFlow → SIEM)
         ↓
Backup and Recovery
  (Configuration backups, tested restore procedures)
```

---

## Security Engineer Perspective

### Network Device Hardening Checklist

| Control | Action | Priority |
|---------|--------|---------|
| Default credentials | Change immediately | Critical |
| Firmware | Update to latest stable | Critical |
| Telnet | Disable; use SSH only | Critical |
| HTTP management | Disable; use HTTPS only | Critical |
| SNMP v1/v2 | Disable or replace with v3 | High |
| Logging | Enable; forward to syslog/SIEM | High |
| Startup scripts | Audit and remove unnecessary | High |
| Port forwarding | Audit and remove unexpected | High |
| Wi-Fi | WPA3; disable WPS | High |
| Backups | Before every change | Medium |
| NTP | Configure accurate time sync | Medium |
| Port security | Enable on access ports | Medium |

### Indicators of Compromise on Network Devices

| Indicator | What It May Mean |
|----------|----------------|
| New port forwarding rules | Attacker creating C2 tunnel |
| New startup scripts | Persistence mechanism |
| Unexpected admin accounts | Backdoor account |
| Unusual outbound traffic | Data exfiltration |
| Configuration changes outside change windows | Unauthorised modification |
| DNS settings changed | DNS hijacking |
| New routing entries | Traffic redirection |

---

## Key Terms Glossary

| Term | Definition |
|------|-----------|
| **ARP Spoofing** | Sending forged ARP replies to map an attacker's MAC to a legitimate IP address |
| **CBC** | Cipher Block Chaining — symmetric encryption mode; requires separate authentication |
| **CDP** | Cisco Discovery Protocol — layer-2 neighbour discovery; disable on untrusted interfaces |
| **DHCPv4 Snooping** | Switch feature blocking rogue DHCP servers on untrusted ports |
| **GCM** | Galois/Counter Mode — authenticated encryption combining encryption and integrity |
| **gMSA** | Group Managed Service Account — auto-rotating service account password |
| **NDP** | Neighbour Discovery Protocol — IPv6 equivalent of ARP |
| **NetFlow** | Protocol for collecting and analysing network traffic flow data |
| **OpenVPN** | Open-source VPN implementation using SSL/TLS |
| **OpenWrt** | Open-source Linux-based OS for embedded network devices |
| **PFS** | Perfect Forward Secrecy — ensures session key compromise does not affect other sessions |
| **POLP** | Principle of Least Privilege |
| **Port Security** | Switch feature limiting MAC addresses per port to prevent MAC flooding |
| **SNMP** | Simple Network Management Protocol — used for device monitoring and management |
| **Syslog** | Standard protocol for sending log messages to a central logging server |
| **WPA3** | Wi-Fi Protected Access 3 — current Wi-Fi security standard |
| **WPS** | Wi-Fi Protected Setup — convenience feature with known brute-force vulnerabilities |

---

## Exam and Interview Revision

### Must Remember

- Network devices are high-value targets — control infrastructure, not just endpoints
- Default credentials are the easiest attack vector — change before deployment
- Telnet transmits passwords in plaintext — always replace with SSH
- HTTP management = credentials visible on the wire — replace with HTTPS
- SNMP v1/v2c community strings are cleartext — upgrade to SNMPv3 or disable
- VPN hardening: cipher AES-256-CBC/GCM, auth SHA256, tls-crypt for PFS, tls-version-min 1.2
- PFS ensures that compromise of one session key does not compromise past/future sessions
- Monitor startup scripts and port forwarding rules — these are common attacker persistence points
- Port security limits MAC addresses per switch port — prevents MAC flooding
- DHCP snooping prevents rogue DHCP servers; DAI prevents ARP spoofing (both covered in Room 1)
- Log everything to a centralised syslog — local logs can be deleted by an attacker

### Common Interview Questions

| Question | Answer Points |
|----------|--------------|
| What makes network devices different from endpoints for hardening? | Network devices route and filter traffic — compromise gives network-wide visibility. They run embedded OSes with device-specific CLI. No user applications. Managed via SSH/web UI/SNMP. |
| What is Perfect Forward Secrecy and why does it matter for VPNs? | PFS generates unique session keys for each VPN session. If one session key is compromised, past and future sessions remain secure. Implemented via `tls-crypt` in OpenVPN. |
| What should you do immediately when deploying a new network device? | Change default credentials, update firmware, disable unnecessary services, enable logging, configure SSH (disable Telnet), restrict management access to specific IPs. |
| How do attackers persist on network devices? | Adding startup scripts (cron jobs, init.d), creating new admin accounts, adding port forwarding rules for C2 tunnels. Monitor these locations for changes. |
| What is the difference between SNMPv1/v2 and SNMPv3? | SNMPv1/v2 use plaintext community strings — no authentication, no encryption. SNMPv3 adds authentication (MD5/SHA) and optional encryption (DES/AES). |
