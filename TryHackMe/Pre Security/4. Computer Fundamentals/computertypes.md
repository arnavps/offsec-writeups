# Computer Types

## Overview

Not all computers look like desktops or laptops. Computing hardware spans a wide range of form factors, each designed for a specific purpose. Understanding the different types of computers — and their security implications — is relevant for both offensive and defensive security work, especially as IoT and embedded devices become increasingly common attack surfaces.

---

## Topics Covered

- Traditional computer types: laptop, desktop, workstation, server
- Mobile and specialised devices: smartphones, tablets
- IoT devices and embedded computers
- The difference between IoT and embedded systems

---

## Key Concepts

### Traditional Computers

| Type | Screen & Keyboard | Primary Purpose |
|---|---|---|
| Laptop | Yes | Portable everyday computing |
| Desktop | Yes | Sustained performance at a fixed location |
| Workstation | Yes | Precision and reliability for professional/technical tasks |
| Server | No | Providing services to many users over a network |

Servers are designed to run continuously, handle many simultaneous connections, and prioritise reliability over user interaction. They typically have no display and are managed remotely.

---

### Mobile and Specialised Devices

| Type | Description | Examples |
|---|---|---|
| Smartphone | Pocket-sized computer optimised for battery life and connectivity | iPhone, Android devices |
| Tablet | Touch-first device with a larger screen than a smartphone | iPad, drawing tablets |

---

### IoT Devices vs Embedded Computers

Both are often small and single-purpose, but the key distinction is network connectivity.

| Type | Description | Examples |
|---|---|---|
| IoT Device | Network-connected device with a single purpose — reports data or receives commands over a network | Smart thermostat, smart doorbell, fitness tracker |
| Embedded Computer | A computer built into another device to control it — may have no network connection at all | Coffee maker controller, automatic door sensor, lamp dimmer chip |

**The key difference:** IoT devices are connected to a network by design. Embedded computers may operate entirely in isolation, performing their function inside a machine without any external communication.

---

## Important Terminology

| Term | Definition |
|---|---|
| Server | A computer dedicated to providing services to other devices over a network |
| Workstation | A high-performance computer used for professional or technical tasks |
| IoT | Internet of Things — network-connected devices with a specific function |
| Embedded Computer | A computer integrated into a larger device to control a specific function |

---

## Real-World Relevance

- Servers are high-value targets — they host services, store data, and are often internet-facing
- IoT devices are a major and growing attack surface: they frequently run outdated firmware, use default credentials, and lack security controls
- Embedded systems in industrial environments (ICS/SCADA) are critical infrastructure targets — compromising them can have physical consequences
- Smartphones are targeted for credential theft, spyware, and as pivot points into corporate networks via MDM (Mobile Device Management) misconfigurations
- Workstations in corporate environments are common initial access targets via phishing — they often have access to internal network resources

---

## Key Learnings

- Computers come in many form factors, each with different security considerations
- Servers have no direct user interface and are managed remotely — misconfigurations are harder to spot
- IoT devices connect to networks and are frequently insecure by default
- Embedded computers may be invisible from a network perspective but can still be compromised via physical access or supply chain attacks
- Understanding what type of system you're targeting or defending shapes the approach entirely

---

## Additional Notes

- The line between IoT and embedded is blurring — many modern embedded systems now include network connectivity
- IoT security is a dedicated field; common issues include default credentials, unencrypted communications, no update mechanism, and exposed management interfaces
- Shodan is a search engine that indexes internet-connected devices — it's commonly used to find exposed IoT devices and servers

---

## Conclusion

The diversity of computing hardware means the attack surface extends far beyond traditional PCs and servers. IoT devices, embedded systems, and mobile devices each introduce unique security challenges. Knowing what type of system you're dealing with is the first step in understanding how to approach it.
