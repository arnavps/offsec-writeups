# IDS Fundamentals

## Overview

A firewall controls what traffic can enter or leave a network, but it cannot detect what happens after a connection is permitted. An Intrusion Detection System (IDS) fills this gap — it monitors network traffic and host activity for signs of malicious behaviour, regardless of whether it passed through the firewall. This room covers what an IDS is, how it differs from a firewall, the two deployment modes (HIDS and NIDS), the three detection methods (signature, anomaly, hybrid), and an introduction to Snort — the most widely used open-source IDS.

---

## Topics Covered

- What an IDS is and why it is needed alongside a firewall
- IDS deployment modes: HIDS and NIDS
- IDS detection methods: signature-based, anomaly-based, hybrid
- Snort and its three operating modes
- When to use each Snort mode

---

## Key Concepts

### What is an IDS and Why is it Needed?

A firewall is a gatekeeping control — it inspects traffic at the perimeter and either allows or denies it. But if an attacker successfully enters the network through a legitimate-looking connection (exploiting an allowed port, using valid credentials, etc.), the firewall offers no further visibility.

An IDS sits inside the network and acts as continuous surveillance. It passively monitors all traffic and host activity, compares it against known attack signatures or behavioural baselines, and generates alerts when something suspicious is detected.

**Key distinction:** An IDS **detects and alerts** — it does not block or respond. That is the role of an IPS (Intrusion Prevention System). An IDS is a monitoring and notification tool.

---

### IDS Deployment Modes

#### Host Intrusion Detection System (HIDS)

- Installed **individually on each host** (endpoint, server, etc.)
- Monitors activity specific to that host: file changes, process execution, log entries, user activity
- Provides deep, granular visibility into what is happening on that particular machine
- **Limitation:** Resource-intensive; difficult to manage at scale across large numbers of hosts
- **Best for:** Critical servers or endpoints where deep host-level visibility is required

#### Network Intrusion Detection System (NIDS)

- Deployed at the **network level** — monitors traffic flowing across the network, not just individual hosts
- Provides a centralised view of all suspicious activity detected across the entire network
- Does not require installation on individual hosts
- **Limitation:** Cannot see encrypted traffic contents without SSL/TLS inspection; cannot observe host-internal activity that doesn't generate network traffic
- **Best for:** Organisation-wide threat monitoring and centralised visibility

---

### IDS Detection Methods

#### Signature-Based Detection

- Maintains a **database of known attack signatures** — unique patterns associated with specific attacks
- Compares traffic or activity against signatures in the database
- **Advantage:** Fast, accurate, and low false positive rate for known threats
- **Limitation:** Cannot detect **zero-day attacks** — threats with no prior signature. If a pattern is not in the database, it will not be detected

Zero-day attacks are particularly dangerous in this context because they are new exploits with no recorded history — signature-based IDS is blind to them by definition.

#### Anomaly-Based Detection

- First establishes a **baseline** of normal network or system behaviour
- Flags any activity that deviates significantly from this baseline as potentially malicious
- **Advantage:** Can detect zero-day attacks and novel threats that have no known signature
- **Limitation:** Higher false positive rate — legitimate but unusual behaviour (a large backup job, a software update, a first-time login from a new device) may be flagged. False positives can be reduced through fine-tuning (manually defining expected normal behaviour)

#### Hybrid Detection

- Combines signature-based and anomaly-based methods
- Uses signature matching for known threats (fast, accurate)
- Falls back to anomaly detection for unknown or novel threats
- **Best of both approaches** — broadest coverage with reasonable false positive rates
- Increasingly common in modern security products

#### Detection Method Comparison

| Method | Detects Known Threats | Detects Zero-Days | False Positive Rate | Processing Overhead |
|--------|----------------------|-------------------|---------------------|---------------------|
| Signature-Based | Yes | No | Low | Low |
| Anomaly-Based | Yes | Yes | Higher | Higher |
| Hybrid | Yes | Yes | Moderate | Moderate |

---

### Snort

Snort is the most widely used open-source network IDS/IPS. It is signature-based and operates primarily as a NIDS, though it can function as a full IPS (Snort IPS mode) when configured to block as well as detect.

Snort has three operating modes:

#### 1. Packet Sniffer Mode

- Reads and displays network packets in real time without performing any analysis or detection
- Does not use Snort's IDS capabilities — purely a traffic display tool
- Useful for: network monitoring, diagnosing traffic issues, verifying traffic flow

**Use case:** The network team observes performance degradation and needs to inspect raw traffic to diagnose the cause. Snort's packet sniffer mode provides this without running detection rules.

#### 2. Packet Logging Mode

- Captures and logs network traffic to a PCAP file for later analysis
- Records all traffic and any detections from it
- The captured log file can be analysed offline using Snort or other tools (Wireshark, etc.)

**Use case:** The security team needs to perform forensic investigation of a past network attack. Snort's packet logging mode provides the traffic capture required for root cause analysis.

#### 3. NIDS Mode (Network Intrusion Detection System Mode)

- Snort's primary operating mode
- Monitors network traffic in real time and applies configured **rule files** (signatures) to detect known attack patterns
- Generates an alert when traffic matches a rule
- Rules are written in Snort's own rule syntax and stored in rule files

**Use case:** The security team needs proactive, continuous monitoring of the network for known attack patterns. Snort's NIDS mode provides this with real-time alerting.

#### Snort Mode Summary

| Mode | Detection | Logging | Real-Time Alerts | Use Case |
|------|-----------|---------|-----------------|---------|
| Packet Sniffer | No | No | No | Traffic visibility and network diagnosis |
| Packet Logging | Yes | Yes (PCAP) | No | Forensic capture for post-incident analysis |
| NIDS Mode | Yes | Optional | Yes | Continuous threat detection and alerting |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| IDS | Intrusion Detection System — monitors and alerts on suspicious activity |
| IPS | Intrusion Prevention System — monitors, alerts, and actively blocks threats |
| HIDS | Host-based IDS — installed on individual hosts for host-level monitoring |
| NIDS | Network-based IDS — monitors network traffic across the whole network |
| Signature | A unique pattern associated with a specific attack or threat |
| Baseline | The established profile of normal behaviour for a network or system |
| Zero-day | A vulnerability or attack with no prior signature — unknown to the defender until exploited |
| Snort | The most popular open-source NIDS/IPS, uses signature-based detection |
| PCAP | Packet capture file format — stores raw network traffic for offline analysis |
| False positive | An alert triggered by benign activity incorrectly identified as malicious |

---

## Workflow / Process

```
Network traffic passes through the monitored network segment
        |
        v
IDS engine inspects each packet/flow

Signature-Based:
  Compare against signature database
  Match found? --> Generate alert
  No match?    --> Traffic passes without alert

Anomaly-Based:
  Compare against established baseline
  Deviation detected? --> Generate alert
  Behaviour within baseline? --> No alert

        |
        v
Alert generated and sent to SOC / security dashboard
        |
        v
Analyst triages the alert
  False positive? --> Close and tune rule if needed
  True positive?  --> Escalate as incident
```

---

## Real-World Relevance

- IDS is a standard component in enterprise security architecture — often deployed at network choke points (Internet gateway, DMZ, core switch span port) for visibility across all traffic
- Signature-based IDS like Snort is effective and widely deployed, but the increasing prevalence of zero-day exploits and fileless attacks is driving adoption of anomaly-based and ML-powered detection
- Snort rules are written by the community and Cisco Talos (Snort's maintainer) — understanding how to read and write Snort rules is a useful skill for detection engineering
- IDS/IPS is distinct from SIEM — IDS monitors raw traffic; SIEM correlates log data from multiple sources. In practice, IDS alert data is often forwarded into a SIEM for correlation
- Suricata is a strong open-source alternative to Snort with multi-threading support and compatible rule syntax
- From an offensive perspective, IDS evasion techniques (fragmentation, encoding, timing attacks, encrypted channels) are specifically designed to avoid triggering IDS signatures

---

## Key Learnings

- An IDS monitors and alerts — it does not block. An IPS adds active blocking capability
- HIDS provides deep host-level visibility; NIDS provides centralised network-wide visibility
- Signature-based detection is fast and accurate for known threats but blind to zero-days
- Anomaly-based detection can catch novel threats but generates more false positives without tuning
- Hybrid detection combines both approaches for broader coverage
- Snort has three modes: packet sniffer (visibility), packet logging (forensic capture), NIDS mode (real-time detection)

---

## Additional Notes

- Snort rules follow a specific syntax: `action protocol src_ip src_port direction dst_ip dst_port (options)` — understanding this format is necessary for writing or reviewing custom rules
- IDS placement matters — a NIDS placed outside the firewall sees all traffic including blocked attempts; placed inside, it sees only permitted traffic. Both positions provide different value
- High traffic volumes can overwhelm an IDS if it lacks the processing capacity — tuning the rule set to focus on high-priority signatures helps manage this
- Encrypted traffic (HTTPS, TLS) limits what a traditional IDS can inspect — NGFWs with SSL/TLS decryption or network-based anomaly detection are needed to handle encrypted sessions

---

## Conclusion

An IDS complements the firewall by providing visibility into what happens after traffic is permitted into the network. Understanding the difference between HIDS and NIDS, and between signature-based, anomaly-based, and hybrid detection, provides the conceptual foundation for deploying and operating intrusion detection effectively. Snort is the practical implementation of these concepts — its three operating modes cover traffic inspection, forensic capture, and real-time threat detection, making it a versatile tool for network security monitoring.
