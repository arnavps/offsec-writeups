# Summit

**Platform:** TryHackMe  
**Room:** Summit  
**Category:** Incident Response / Purple Team / Threat Hunting  
**Difficulty:** Medium  

---

## Overview

**Summit** is an interactive, purple-team-style challenge room designed to test your understanding of the **Pyramid of Pain**. You take on the role of a SOC analyst for **PicoSecure** who is responding to active attacks simulated by an external penetration tester named **Sphinx**.

The challenge is iterative: Sphinx executes malware samples, and you must analyze them in a virtual **Malware Sandbox** to extract indicators. You then configure defensive tools (Hash Manager, Firewall, DNS Filter, and Sigma Rule Builder) to block or detect the threat. As you block their attacks, Sphinx is forced to ascend the Pyramid of Pain—retooling and changing their parameters—making it progressively harder and more expensive for them to succeed.

---

## Walkthrough & Progression

```
                   [   TTPs (Sample 6)   ] -> Final Flag
                           |
                [   Tools/Beacons (Sample 5)   ] -> Flag 5
                           |
             [   Host/Registry (Sample 4)   ] -> Flag 4
                           |
             [   Domain/DNS (Sample 3)   ] -> Flag 3
                           |
             [   IP/Firewall (Sample 2)   ] -> Flag 2
                           |
             [   File Hash (Sample 1)   ] -> Flag 1
```

### Task 2 — Sample 1: Hash Values (Trivial Level)
* **Goal:** Block the threat using its file hash.
* **Steps:**
  1. Download the first sample (`sample1.exe`) and submit it to the **Malware Sandbox**.
  2. Analyze the report to find the SHA256/MD5 hash.
  3. Go to the **Manage Hashes** tool and paste the hash to block it.
  4. Sphinx attempts to execute the malware again and fails.
* **Flag Received:** `THM{f3cbf08151a11a6a331db9c6cf5f4fe4}`

### Task 3 — Sample 2: IP Addresses (Easy Level)
* **Goal:** Block the threat using C2 network connection logs.
* **Steps:**
  1. Sphinx recompiles the malware to bypass the hash block list. Submit `sample2.exe` to the sandbox.
  2. The sandbox report reveals outbound network connections to IP address `154.35.10.113`.
  3. Open the **Firewall Rule Manager** and create an **Egress Deny rule** for destination IP `154.35.10.113`.
  4. Re-run the simulation.
* **Flag Received:** `THM{2ff48a3421a938b388418be273f4806d}`

### Task 4 — Sample 3: Domain Names (Simple Level)
* **Goal:** Block the threat using DNS filters.
* **Steps:**
  1. Sphinx rotates C2 IPs dynamically. Submit `sample3.exe` to the sandbox.
  2. The sandbox report shows DNS queries resolving the domain `emudyn.bresonicz.info`.
  3. Go to the **DNS Filter** and block the domain `emudyn.bresonicz.info`.
  4. Re-run the simulation.
* **Flag Received:** `THM{4eca9e2f61a19ecd5df34c788e7dce16}`

### Task 5 — Sample 4: Host Artifacts (Annoying Level)
* **Goal:** Detect local system/registry modifications via Sigma rules.
* **Steps:**
  1. Sphinx changes their network infrastructure. Submit `sample4.exe` to the sandbox.
  2. The sandbox report shows no external connection but indicates the malware modifies the registry key `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection` setting `DisableRealtimeMonitoring` to `1` (disabling Windows Defender).
  3. Open the **Sigma Rule Builder**. Set:
     - **Log Source:** Sysmon Event Logs
     - **Event Type:** Registry Modification (Event ID 13)
     - **Target Key:** `SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`
     - **Target Name:** `DisableRealtimeMonitoring`
     - **Target Value:** `1`
  4. Save the rule. The SOC system detects and stops the execution.
* **Flag Received:** `THM{546875732069732074686520666f75727468}`

### Task 6 — Sample 5: Network Artifacts / Beacons (Challenging Level)
* **Goal:** Identify beaconing behavior via Egress port logging.
* **Steps:**
  1. Submit `sample5.exe` to the sandbox.
  2. The sandbox logs show regular egress beaconing traffic to port `9889`.
  3. Open the **Sigma Rule Builder** to create a Sysmon Network Connection detection:
     - **Log Source:** Sysmon Event Logs
     - **Event Type:** Network Connection (Event ID 3)
     - **Destination Port:** `9889`
  4. Activate the rule. Egress traffic to the C2 port is immediately flagged and terminated.
* **Flag Received:** `THM{46b21c4410e47dc5729ceadef0fc722e}`

### Task 7 — Sample 6: TTPs (Tough Level)
* **Goal:** Disrupt the adversary's playbook of living-off-the-land commands.
* **Steps:**
  1. Submit `sample6.exe` to the sandbox.
  2. Analysis reveals the malware leverages a command-line script executing the `wevtutil.exe` utility to clear Windows Event Logs (Security/Sysmon) to cover its tracks (specifically running `wevtutil.exe cl Security` or `wevtutil cl Microsoft-Windows-Sysmon/Operational`).
  3. Create a behavioral **Sigma rule** for Sysmon Process Creation (Event ID 1):
     - **Log Source:** Sysmon Event Logs
     - **Event Type:** Process Creation (Event ID 1)
     - **Command Line Contains:** `wevtutil` AND `cl`
  4. Deploy the rule. This stops the attacker's TTP and blocks the intrusion completely.
* **Flag Received (Sphinx Evicted):** `THM{c8951b2ad24bbcbac60c16cf2c83d92c}`

---

## Key Learnings

- **Purple Team Emulation:** Interactive simulations provide practical experience in defensive adaptation. Security teams must learn to adapt as fast as threat actors.
- **Value of Behavioral Detections:** Static blocks (MD5, IPs) were bypassed instantly by Sphinx. However, once the behavioral rules (registry keys, ports, command-line arguments) were deployed, the adversary was forced to completely retool, which eventually led to their eviction.
- **Sigma Rules in Action:** Sigma rules provide a vendor-agnostic way to describe log detection patterns, allowing analysts to write rules once and translate them into their target SIEM (Splunk, Elastic, Sentinel) instantly.
