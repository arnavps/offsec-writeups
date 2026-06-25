# Pyramid of Pain

**Platform:** TryHackMe  
**Room:** Pyramid of Pain  
**Category:** Threat Intelligence / Cyber Defense Frameworks  
**Difficulty:** Easy  

---

## Overview

The **Pyramid of Pain** is a conceptual model developed by security professional David J. Bianco in 2013. It is designed to illustrate how much "pain" (in terms of time, effort, and cost) a defender causes an adversary when they detect and block different types of **Indicators of Compromise (IOCs)**.

Rather than looking at all indicators as equal, the model helps security analysts, threat hunters, and detection engineers prioritize behaviors over static indicators, creating resilient, long-lasting defenses.

---

## The Pyramid of Pain Hierarchy

The pyramid is structured in six layers. As you move from the bottom to the top, the indicators become progressively harder (more painful) for the attacker to change, but also more difficult for defenders to detect.

```
       /\
      /  \       TTPs (Tactics, Techniques & Procedures) - Tough
     /----\
    /      \     Tools - Challenging
   /--------\
  /          \   Host & Network Artifacts - Annoying
 /------------\
/              \ Domain Names / IP Addresses - Simple / Easy
/----------------\
/  Hash Values   \ Hash Values - Trivial
/----------------\
```

| Level | Indicator Type | Description | Evasion Pain for Attacker | Defense Strategy |
|---|---|---|---|---|
| **Tough** | **TTPs** | How the attacker operates (e.g., scripting, lateral movement, exfiltration behaviors). | **High** (Requires redesigning playbooks, training, and methodologies). | Behavioral analysis, process tree profiling, detection rules (Sigma/YARA). |
| **Challenging** | **Tools** | Software/utilities used by the attacker (e.g., Mimikatz, Cobalt Strike, Nmap). | **Medium-High** (Must develop, modify, or acquire alternative tools). | Signature and heuristic-based detection on tooling, endpoint monitoring. |
| **Annoying** | **Host/Network Artifacts** | Traces left on systems (registry keys, user agents, URI paths, dropped files, pcap activity). | **Medium** (Requires changing code configuration, compiling differences). | Registry monitoring, log analysis, User-Agent checking, pcap filtering. |
| **Simple** | **Domain Names** | C2 or staging addresses (e.g., malicious domains, punycode domains, shortened URLs). | **Medium-Low** (Must register new domains, use alternative hosting). | DNS filtering, sinkholing, reputation checking, URL previewing. |
| **Easy** | **IP Addresses** | Network identifiers (e.g., VPS addresses, proxy servers, VPNs, fast flux). | **Low** (Attacker can change IP via proxy, VPN, or new redirector in minutes). | Egress/Ingress firewall rules, IP blocklists. |
| **Trivial** | **Hash Values** | Cryptographic signatures of files (MD5, SHA-1, SHA-256). | **None** (Attacker changes one character or bit to generate a completely new hash). | Static blocklisting (VirusTotal, SIEM hash blocks). |

---

## Room Questions & Answers

### Task 2 — Hash Values (Trivial)
* **Question:** Analyze the report associated with the hash `b8ef959a9176aef07fdca8705254a163b50b49a17217a4ff0107487f59d4a35d` here. What is the filename of the sample?  
  * **Answer:** `Sales_Receipt 5606.xls`

### Task 3 — IP Address (Easy)
* **Question:** What is the first IP address the process 1632 initiates a connection with?  
  * **Answer:** `50.87.136.52`
* **Question:** What is the first domain name the process 1632 initiates a connection with?  
  * **Answer:** `craftingalegacy.com`

### Task 4 — Domain Names (Simple)
* **Question:** Go to this report on app.any.run and provide the first suspicious domain request you are seeing.  
  * **Answer:** `craftingalegacy.com`
* **Question:** What term refers to an address used to access websites?  
  * **Answer:** `Domain Name`
* **Question:** What type of attack uses Unicode characters in the domain name to imitate a known domain?  
  * **Answer:** `Punycode attack`
* **Question:** Provide the redirected website for the shortened URL using a preview: `https://tinyurl.com/bw7t8p4u`  
  * **Answer:** `https://tryhackme.com/`

### Task 5 — Host Artifacts (Annoying)
* **Question:** A process named `regidle.exe` makes a POST request to an IP address based in the United States (US) on port 8080. What is the IP address?  
  * **Answer:** `96.126.101.6`
* **Question:** The actor drops a malicious executable (EXE). What is the name of this executable?  
  * **Answer:** `o79927.exe`

### Task 6 — Network Artifacts (Annoying)
* **Question:** What browser uses the User-Agent string shown in the screenshot above?  
  * **Answer:** `Internet Explorer`
* **Question:** How many POST requests are in the screenshot from the pcap file?  
  * **Answer:** `6`

### Task 7 — Tools (Challenging)
* **Question:** Provide the method used to determine file similarity.  
  * **Answer:** `Fuzzy hashing`
* **Question:** Provide the alternative name for fuzzy hashes without the abbreviation.  
  * **Answer:** `context triggered piecewise hashes`

### Task 8 — TTPs (Tough)
* **Question:** Navigate to the ATT&CK Matrix webpage. How many techniques fall under the Exfiltration category?  
  * **Answer:** `9`
* **Question:** Chimera is a China-based hacking group that has been active since 2018. What is the name of the commercial, remote access tool they use for C2 beacons and data exfiltration?  
  * **Answer:** `Cobalt Strike`

---

## Key Learnings

- **Relative Efficacy of Detections:** Static indicators (like MD5 hashes and IPs) are trivial or easy for an attacker to change but are also cheap to implement as detections. However, their security shelf-life is extremely short.
- **Hunting for TTPs:** Effective security monitoring must shift from *indicators* to *behaviors*. Detecting process injection, token theft, or specific scripting executions forces the attacker to restructure their entire codebase and playbook, imposing maximum "pain" on their operations.
- **Advanced Hashing (SSDEEP):** Traditional hashes are fragile, but fuzzy hashes (Context Triggered Piecewise Hashes) allow analysts to detect variants of malware by checking similarity rather than exact matches.
