# Tcpdump: The Basics

## Overview

Tcpdump is a command-line packet capture and analysis tool for Unix-like systems. Written in C/C++ and built on the libpcap library, it is extremely stable, fast, and available on virtually every Linux system. It is the go-to tool for capturing network traffic in environments without a GUI, and its output can be saved as PCAP files for analysis in Wireshark.

---

## Topics Covered

- Basic tcpdump options: interface, file I/O, packet count, verbosity
- Filtering by host, port, and protocol
- Logical operators
- Output formatting: `-q`, `-e`, `-A`, `-xx`, `-X`
- Advanced filtering: packet length, TCP flags, binary operations

---

## Key Concepts

### Basic Options

| Command | Explanation |
|---|---|
| `tcpdump -i INTERFACE` | Capture on a specific interface |
| `tcpdump -i any` | Capture on all interfaces |
| `tcpdump -w FILE` | Write captured packets to a file (.pcap) |
| `tcpdump -r FILE` | Read packets from a file |
| `tcpdump -c COUNT` | Capture a specific number of packets |
| `tcpdump -n` | Don't resolve IP addresses |
| `tcpdump -nn` | Don't resolve IP addresses or port numbers |
| `tcpdump -v` | Verbose output |
| `tcpdump -vv` | More verbose |
| `tcpdump -vvv` | Maximum verbosity |

**Examples:**

```bash
tcpdump -i eth0 -c 50 -v
tcpdump -i wlo1 -w data.pcap
tcpdump -i any -nn
tcpdump -r traffic.pcap -c 5 -n
```

---

### Filtering

**By host:**

```bash
tcpdump host 192.168.1.1
tcpdump src host 192.168.1.1
tcpdump dst host 192.168.1.1
tcpdump host example.com
```

**By port:**

```bash
tcpdump port 53
tcpdump src port 1234
tcpdump dst port 80
```

**By protocol:**

```bash
tcpdump icmp
tcpdump tcp
tcpdump udp
tcpdump arp
```

**Logical operators:**

```bash
tcpdump host 1.1.1.1 and tcp
tcpdump udp or icmp
tcpdump not tcp
```

**Combined examples:**

```bash
tcpdump -i any tcp port 22                                    # SSH traffic
tcpdump -i wlo1 udp port 123                                  # NTP traffic
tcpdump -i eth0 host example.com and tcp port 443 -w https.pcap
```

---

### Output Formatting

| Option | Description |
|---|---|
| `-q` | Quick/brief output — timestamp, IPs, ports only |
| `-e` | Include MAC addresses (link-layer header) |
| `-A` | Display packet data as ASCII |
| `-xx` | Display packet data in hexadecimal |
| `-X` | Display packet data in both hex and ASCII |

```bash
tcpdump -r traffic.pcap arp -e          # Show MAC addresses for ARP
tcpdump -r traffic.pcap -A              # ASCII output
tcpdump -r traffic.pcap -X             # Hex + ASCII
```

---

### Advanced Filtering

**By packet length:**

```bash
tcpdump greater 1000        # Packets >= 1000 bytes
tcpdump less 100            # Packets <= 100 bytes
```

**TCP flags using `tcp[tcpflags]`:**

```bash
tcpdump "tcp[tcpflags] == tcp-syn"                    # Only SYN flag set
tcpdump "tcp[tcpflags] & tcp-syn != 0"                # At least SYN set
tcpdump "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"      # SYN or ACK set
```

Available TCP flag names: `tcp-syn`, `tcp-ack`, `tcp-fin`, `tcp-rst`, `tcp-push`

**Header byte filtering syntax:**

```
proto[expr:size]
```

- `proto` — protocol (arp, ether, icmp, ip, tcp, udp)
- `expr` — byte offset (0 = first byte)
- `size` — number of bytes (default: 1)

---

### Counting Results

Pipe output through `wc -l` to count matching packets:

```bash
tcpdump -r traffic.pcap icmp -n | wc -l
tcpdump -r traffic.pcap 'tcp[tcpflags] == tcp-rst' | wc -l
```

---

## Practical Examples / Demonstrations

### Lab walkthrough

**How many ICMP packets in traffic.pcap?**

```bash
sudo tcpdump -r traffic.pcap icmp -n | wc -l
# Answer: 26
```

**What IP asked for the MAC of 192.168.124.137?**

```bash
sudo tcpdump -r traffic.pcap arp and host 192.168.124.137
# Answer: 192.168.124.148
```

**What hostname appears in the first DNS query?**

```bash
sudo tcpdump -r traffic.pcap port 53 -A
# Answer: mirrors.rockylinux.org
```

**How many packets have only the TCP RST flag set?**

```bash
sudo tcpdump -r traffic.pcap 'tcp[tcpflags] == tcp-rst' | wc -l
# Answer: 57
```

**What IP sent packets larger than 15000 bytes?**

```bash
sudo tcpdump -r traffic.pcap 'greater 15000' -n
# Answer: 185.117.80.53
```

**What is the MAC address of the host that sent an ARP request?**

```bash
sudo tcpdump -r traffic.pcap arp -e
# Answer: 52:54:00:7c:d3:5b
```

---

## Important Terminology

| Term | Definition |
|---|---|
| libpcap | C library for packet capture — the foundation of tcpdump and Wireshark |
| PCAP | Packet capture file format |
| BPF | Berkeley Packet Filter — the filter syntax used by tcpdump |
| TTL | Time to Live — IP field limiting packet lifetime |
| TCP flags | Control bits in the TCP header: SYN, ACK, FIN, RST, PSH |
| `-n` | Suppress DNS resolution of IP addresses |
| `-nn` | Suppress both DNS and port name resolution |

---

## Real-World Relevance

- Tcpdump is the standard tool for capturing traffic on remote Linux servers where Wireshark is unavailable
- Saved PCAP files can be transferred to a local machine and opened in Wireshark for detailed analysis
- TCP flag filtering (`tcp-syn`, `tcp-rst`) is used to detect port scans, SYN floods, and connection resets
- ARP filtering with `-e` reveals MAC addresses — useful for identifying devices on a network
- Packet length filtering helps identify large data transfers that may indicate exfiltration
- `-nn` is essential in time-sensitive captures to avoid DNS lookup delays

---

## Key Learnings

- `-i` specifies the interface; `-w` writes to file; `-r` reads from file; `-c` limits packet count
- `-n` and `-nn` suppress DNS and port resolution — important for speed and accuracy
- Filters can be combined with `and`, `or`, and `not`
- TCP flags can be filtered using `tcp[tcpflags]` with named flag constants
- Pipe to `wc -l` to count matching packets
- `-e` shows MAC addresses; `-A` shows ASCII; `-X` shows hex + ASCII

---

## Conclusion

Tcpdump is the command-line equivalent of Wireshark — lightweight, fast, and available everywhere. Its filtering capabilities, combined with the ability to save captures for later analysis, make it an essential tool for network monitoring, incident response, and protocol analysis in environments where a GUI is not available.
