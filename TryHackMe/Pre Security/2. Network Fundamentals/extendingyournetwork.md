# Extending Your Network

## Overview

This room covers the technologies and devices used to extend networks beyond their local boundaries — including port forwarding, firewalls, VPNs, routers, and switches with VLAN support. These concepts are directly relevant to both network administration and security, as they define how traffic flows and how access is controlled.

---

## Topics Covered

- Port forwarding
- Firewalls (stateful vs stateless)
- VPN technologies (PPP, PPTP, IPSec)
- Routers and routing
- Switches and VLANs

---

## Key Concepts

### Port Forwarding

Port forwarding allows services running on a private network to be accessible from the internet. Without it, a web server on a private network is only reachable by devices on that same network.

A router is configured to forward incoming traffic on a specific port to a specific internal device. For example, forwarding external port 80 to an internal server at `192.168.1.10:80` makes that web server publicly accessible.

---

### Firewalls

A firewall is a network security device that controls what traffic is permitted to enter or leave a network based on defined rules.

**Two primary categories:**

| Category | Description | Strengths | Weaknesses |
|---|---|---|---|
| Stateful | Tracks the full state of a connection; makes decisions based on the entire session context | Smarter decisions; can block a device entirely if its connection behaviour is suspicious | Higher resource usage; more complex |
| Stateless | Evaluates each packet individually against a static ruleset | Lightweight; effective against high-volume attacks like DDoS | Only as good as its rules; can't detect patterns across packets |

Firewalls can be hardware devices, software applications, or both.

---

### VPNs (Virtual Private Networks)

A VPN creates an encrypted tunnel between two devices or networks over the internet, allowing secure communication as if both endpoints were on the same private network.

**VPN Technologies:**

| Technology | Description |
|---|---|
| PPP (Point-to-Point Protocol) | Provides authentication and encryption for VPN connections using a private key and public certificate. Not routable on its own — requires a tunnelling protocol like PPTP |
| PPTP (Point-to-Point Tunneling Protocol) | Encapsulates PPP traffic and allows it to travel across networks. Easy to set up and widely supported, but uses weak encryption |
| IPSec (Internet Protocol Security) | Encrypts data at the IP layer. More complex to configure than PPTP but provides strong encryption and is widely supported on modern devices |

---

### Routers

Routers connect separate networks and route traffic between them. They operate at **Layer 3 (Network)** of the OSI model.

- Determine the best path for data to travel between networks
- Typically provide an admin interface for configuring port forwarding, firewall rules, and routing tables
- Support both static routes (manually defined) and dynamic routing protocols

---

### Switches and VLANs

Switches operate at **Layer 2** and sometimes **Layer 3** of the OSI model. Beyond basic device connectivity, managed switches support VLANs.

**VLAN (Virtual Local Area Network):**
A VLAN logically segments a physical network into separate virtual networks. Devices on different VLANs cannot communicate directly with each other without going through a router, even if they're connected to the same physical switch.

Benefits:
- Network segmentation without additional physical hardware
- Improved security — limits lateral movement between network segments
- Devices can still share resources like internet access while being isolated from each other

---

## Important Terminology

| Term | Definition |
|---|---|
| Port Forwarding | Directing external traffic on a specific port to an internal device |
| Firewall | A device or software that filters network traffic based on rules |
| Stateful Firewall | Tracks full connection state to make filtering decisions |
| Stateless Firewall | Filters individual packets against a static ruleset |
| VPN | Encrypted tunnel enabling secure communication over public networks |
| PPP | Protocol providing authentication and encryption for VPN links |
| PPTP | Tunnelling protocol that carries PPP traffic across networks |
| IPSec | Strong encryption protocol operating at the IP layer |
| Router | Layer 3 device that connects and routes traffic between networks |
| Switch | Layer 2/3 device that connects devices within a network |
| VLAN | Logical network segmentation on a physical switch |

---

## Workflow / Process

### Port Forwarding Flow

```
Internet User  --[Request: PublicIP:80]-->  Router
Router         --[Forward to 192.168.1.10:80]-->  Internal Web Server
Internal Server  --[Response]-->  Router  --[Response]-->  Internet User
```

### VPN Tunnel Flow

```
Device A  --[Encrypted tunnel over public internet]-->  Device B
Both devices communicate as if on the same private network
```

---

## Real-World Relevance

- Port forwarding misconfigurations are a common attack surface — exposing internal services unintentionally
- Stateful firewalls are the standard in enterprise environments; understanding their logic is essential for firewall evasion techniques
- VPNs are used by attackers to anonymise traffic and by defenders to secure remote access
- VLAN hopping is an attack technique that exploits misconfigured VLANs to gain access to restricted network segments
- Understanding how routers and switches work is foundational for network reconnaissance and lateral movement

---

## Key Learnings

- Port forwarding exposes internal services to the internet — it must be configured carefully
- Stateful firewalls are smarter but heavier; stateless firewalls are lightweight but limited
- VPNs encrypt traffic over public networks — IPSec is the strongest of the three technologies covered
- Routers operate at Layer 3; switches at Layer 2 (and sometimes Layer 3)
- VLANs provide logical network segmentation and are a key tool for network security

---

## Additional Notes

- PPTP is considered insecure by modern standards due to known weaknesses in its encryption — IPSec or OpenVPN are preferred
- Firewalls can be bypassed using techniques like tunnelling traffic over allowed ports (e.g., HTTP/HTTPS)
- VLANs require a managed switch; unmanaged switches do not support VLAN configuration
- Double-tagging is a common VLAN hopping technique exploiting the 802.1Q protocol

---

## Conclusion

Extending a network beyond its local boundaries introduces both capability and risk. Port forwarding, firewalls, VPNs, and VLANs are the core tools used to control how traffic flows and who can access what. Understanding how each works — and where each can be misconfigured or exploited — is essential knowledge for both network defenders and penetration testers.
