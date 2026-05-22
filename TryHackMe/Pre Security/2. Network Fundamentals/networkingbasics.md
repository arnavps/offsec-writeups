# Networking Basics

## Overview

This room covers the foundational concepts of how devices communicate on networks — from how the internet is structured down to how individual devices identify themselves. Understanding these basics is essential before diving into any area of cybersecurity, whether offensive or defensive.

---

## Topics Covered

- What a network is and how the internet is structured
- IP addresses (IPv4 and IPv6)
- MAC addresses and spoofing
- ICMP and the `ping` utility

---

## Key Concepts

### What is a Network?

A network is a collection of devices connected together to share data and resources. The internet itself is not a single network — it is a massive collection of smaller private networks all interconnected through public networks.

- **Private network** — a local network (e.g., home or office)
- **Public network** — the internet, connecting private networks together

### IP Addresses

An IP (Internet Protocol) address is a numerical label assigned to a device on a network, used to identify and locate it. IP addresses are divided into four octets (e.g., `192.168.1.100`) and are calculated through IP addressing and subnetting.

Key points:
- IP addresses can change between devices but cannot be active on two devices simultaneously within the same network
- **Private IP** — identifies a device within a local network
- **Public IP** — identifies a device on the internet

**IPv4 vs IPv6:**

| Version | Address Space | Example |
|---|---|---|
| IPv4 | ~4.3 billion addresses | `192.168.1.1` |
| IPv6 | 2^128 (~340 trillion+) | `2001:0db8:85a3::8a2e:0370:7334` |

IPv6 was introduced to solve IPv4 address exhaustion and also brings improved routing efficiency.

### MAC Addresses

Every network-enabled device has a Network Interface Card (NIC) with a hardcoded MAC (Media Access Control) address assigned at the factory. It is a 12-character hexadecimal value split into pairs separated by colons.

Example: `a4:c3:f0:85:ac:2d`

- First 6 characters — identify the manufacturer
- Last 6 characters — unique identifier for the device

**MAC Spoofing:**
MAC addresses can be faked, a technique called spoofing. An attacker can impersonate another device's MAC address to bypass security controls that rely on MAC-based trust — for example, a firewall configured to allow traffic only from a specific MAC address.

### ICMP and Ping

ICMP (Internet Control Message Protocol) is used to diagnose network connectivity. The `ping` tool uses ICMP echo requests and echo replies to test whether a host is reachable and measure round-trip latency.

---

## Important Terminology

| Term | Definition |
|---|---|
| Private Network | A local network not directly exposed to the internet |
| Public Network | The internet — connects private networks globally |
| IP Address | Numerical identifier for a device on a network |
| IPv4 | 32-bit IP addressing scheme, ~4.3 billion addresses |
| IPv6 | 128-bit IP addressing scheme, effectively unlimited addresses |
| MAC Address | Hardware-level identifier assigned to a NIC at manufacture |
| MAC Spoofing | Faking a MAC address to impersonate another device |
| ICMP | Internet Control Message Protocol — used for diagnostics |
| Ping | Tool that uses ICMP to test connectivity and measure latency |

---

## Practical Examples / Demonstrations

### Ping a host

```bash
ping 192.168.1.1
```

```bash
ping tryhackme.com
```

Ping sends ICMP echo packets to the target and reports back response time and packet loss.

---

## Workflow / Process

### How Ping Works

```
Device A  --[ICMP Echo Request]-->  Device B
Device A  <--[ICMP Echo Reply]---   Device B
```

Round-trip time is measured and reported for each packet.

---

## Real-World Relevance

- IP and MAC addresses are fundamental to nearly every network attack and defence technique
- MAC spoofing is used in real attacks to bypass network access controls and firewalls
- `ping` is one of the first tools used during network reconnaissance to check if a host is alive
- Understanding public vs private IP addressing is essential for firewall rules, NAT, and network segmentation

---

## Key Learnings

- The internet is a network of networks — private networks connected via public infrastructure
- Devices have two identifiers: IP address (logical, changeable) and MAC address (physical, persistent)
- IPv6 exists to solve IPv4 exhaustion and is increasingly common
- MAC addresses can be spoofed, making MAC-based security controls unreliable on their own
- `ping` uses ICMP and is a basic but essential connectivity diagnostic tool

---

## Additional Notes

- MAC spoofing is trivially easy on most operating systems and should never be the sole security control
- ICMP is sometimes blocked by firewalls, meaning a non-response to ping doesn't always mean a host is down
- Private IP ranges: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`

---

## Conclusion

Networking basics form the bedrock of all cybersecurity knowledge. Understanding how devices are identified — both logically via IP and physically via MAC — and how tools like `ping` work at the protocol level gives you the foundation needed to understand attacks, defences, and everything in between.
