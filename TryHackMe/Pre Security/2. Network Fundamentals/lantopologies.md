# LAN Topologies

## Overview

A network topology describes the physical or logical arrangement of devices in a network. Choosing the right topology affects performance, cost, scalability, and fault tolerance. This room covers the three core topologies and introduces the key hardware — switches and routers — that make modern networks function, along with subnetting, ARP, and DHCP.

---

## Topics Covered

- Star, Bus, and Ring topologies
- Switches vs Hubs
- Routers and routing
- Subnetting
- ARP (Address Resolution Protocol)
- DHCP (Dynamic Host Configuration Protocol)

---

## Key Concepts

### Network Topologies

#### Star Topology

Devices connect individually to a central device (switch or hub). This is the most common topology in modern networks.

| Advantages | Disadvantages |
|---|---|
| Highly scalable — easy to add devices | More expensive due to cabling and dedicated hardware |
| Fault in one device doesn't affect others | Central device failure brings down the whole network |
| Easier to manage and isolate faults | More maintenance as the network grows |

#### Bus Topology

All devices connect to a single shared cable called the backbone cable.

| Advantages | Disadvantages |
|---|---|
| Simple and cheap to set up | Single point of failure — backbone cable break = full outage |
| Minimal hardware required | Prone to bottlenecks under heavy traffic |
| | Difficult to troubleshoot — all traffic shares one path |

#### Ring Topology

Devices connect directly to each other in a closed loop. Data travels in one direction around the ring until it reaches its destination.

| Advantages | Disadvantages |
|---|---|
| Easy to troubleshoot — single data direction | A single break (cable or device) takes down the entire network |
| Less prone to bottlenecks than bus | Inefficient — data may pass through many devices before reaching its target |
| Less hardware dependency than star | |

---

### Switches

A switch is a dedicated networking device that connects multiple devices within a network. Unlike a hub (which broadcasts all traffic to every port), a switch is intelligent — it tracks which device is connected to which port and forwards packets only to the intended destination.

- Operates at Layer 2 (Data Link) of the OSI model
- Reduces unnecessary network traffic
- Common port counts: 4, 8, 16, 24, 32, 64
- Switches and routers can be interconnected to add redundancy — if one path fails, another can be used

### Routers

A router connects separate networks and directs data between them. It determines the best path for data to travel across networks — a process called routing.

- Operates at Layer 3 (Network) of the OSI model
- Creates paths between networks for data delivery
- Typically provides an admin interface for configuring rules (port forwarding, firewalling, etc.)

---

### Subnetting

Subnetting divides a network into smaller logical segments. Network administrators use it to organise, manage, and secure different parts of a network. The division is represented by a subnet mask.

| Address Type | Purpose | Example |
|---|---|---|
| Network Address | Identifies the network itself | `192.168.1.0` |
| Host Address | Identifies a specific device on the subnet | `192.168.1.100` |
| Default Gateway | The device that routes traffic to other networks | `192.168.1.254` |

The default gateway is typically assigned the first (`.1`) or last (`.254`) host address in a subnet.

---

### ARP (Address Resolution Protocol)

ARP maps IP addresses to MAC addresses. Since devices communicate at the hardware level using MAC addresses, ARP is needed to resolve a known IP address to its corresponding MAC address before communication can occur.

**How ARP works:**

1. Device A wants to communicate with `192.168.1.50` but doesn't know its MAC address
2. Device A broadcasts an **ARP Request** to the entire network: "Who has `192.168.1.50`?"
3. The device with that IP responds with an **ARP Reply** containing its MAC address
4. Device A stores this mapping in its **ARP cache** for future use

---

### DHCP (Dynamic Host Configuration Protocol)

DHCP automates IP address assignment. When a device joins a network without a manually configured IP, it goes through the following process:

| Step | Message | Description |
|---|---|---|
| 1 | DHCP Discover | Device broadcasts to find any available DHCP server |
| 2 | DHCP Offer | DHCP server responds with an available IP address |
| 3 | DHCP Request | Device confirms it wants the offered IP |
| 4 | DHCP ACK | Server acknowledges and the device begins using the IP |

---

## Important Terminology

| Term | Definition |
|---|---|
| Topology | The arrangement of devices in a network |
| Backbone Cable | The single shared cable in a bus topology |
| Switch | Layer 2 device that forwards traffic only to the intended port |
| Hub | Broadcasts all traffic to every connected port — largely obsolete |
| Router | Layer 3 device that connects and routes traffic between networks |
| Routing | The process of finding a path for data to travel between networks |
| Subnetting | Dividing a network into smaller logical segments |
| Subnet Mask | A number that defines the size of a subnet |
| Default Gateway | The device that handles traffic destined for other networks |
| ARP | Protocol that resolves IP addresses to MAC addresses |
| ARP Cache | A local table storing IP-to-MAC mappings |
| DHCP | Protocol that automatically assigns IP addresses to devices |

---

## Workflow / Process

### ARP Resolution Flow

```
Device A  --[ARP Request: Who has 192.168.1.50?]--> (broadcast)
Device B  --[ARP Reply: I have it, my MAC is XX:XX:XX:XX:XX:XX]--> Device A
Device A stores mapping in ARP cache
```

### DHCP Flow

```
Client  --[DHCP Discover]--> (broadcast)
Server  --[DHCP Offer]-----> Client
Client  --[DHCP Request]---> Server
Server  --[DHCP ACK]-------> Client
Client now has a valid IP address
```

---

## Real-World Relevance

- Star topology is the standard in enterprise and home networks — understanding it is essential for network mapping during recon
- ARP is the basis for ARP spoofing/poisoning attacks, where an attacker sends fake ARP replies to redirect traffic through their machine (man-in-the-middle)
- DHCP starvation and rogue DHCP server attacks exploit the DHCP process to disrupt networks or intercept traffic
- Subnetting knowledge is critical for understanding network segmentation and firewall rules

---

## Key Learnings

- Star topology is the most common and scalable; bus and ring are largely legacy
- Switches are smarter than hubs — they forward traffic only to the correct port
- Routers connect networks; switches connect devices within a network
- Subnetting organises networks into logical segments with defined address ranges
- ARP resolves IP addresses to MAC addresses and is vulnerable to spoofing
- DHCP automates IP assignment through a four-step discover/offer/request/acknowledge process

---

## Additional Notes

- ARP spoofing is a foundational technique in man-in-the-middle attacks — tools like `arpspoof` and `ettercap` automate this
- DHCP leases are temporary; devices must renew them periodically
- VLANs (Virtual LANs) can be used on managed switches to logically segment a network without physical separation — covered in later rooms

---

## Conclusion

LAN topologies define how devices are physically and logically connected. Combined with an understanding of how switches, routers, ARP, and DHCP operate, this knowledge forms the basis for understanding both how networks function and how they can be attacked or defended.
