# Unified Kill Chain

**Platform:** TryHackMe  
**Room:** Unified Kill Chain  
**Category:** Threat Intelligence / Cyber Defense Frameworks  
**Difficulty:** Easy  

---

## Overview

The **Unified Kill Chain (UKC)** is a comprehensive cyber security framework released in 2017 by Paul Pols. It integrates and extends existing models (like the Lockheed Martin Cyber Kill Chain and the MITRE ATT&CK framework) to better represent modern, complex, and non-linear cyber attacks.

While the traditional Cyber Kill Chain focuses heavily on the initial entry of a threat, the UKC addresses the reality of persistent internal threats, lateral movement, and multi-phased actions across internal endpoints and cloud services. The UKC defines **18 distinct phases** organized into three core tactical areas: **In**, **Through**, and **Out**.

---

## The 18 Attack Phases of the UKC

```
                      [   I N   ]
Reconnaissance -> Weaponization -> Delivery -> Social Engineering -> Exploitation -> Persistence -> Defense Evasion
                                      |
                                      v
                                [ THROUGH ]
Credential Access -> Discovery -> Privilege Escalation -> Execution -> Pivoting -> Lateral Movement
                                      |
                                      v
                                [   O U T   ]
    Collection -> Exfiltration -> Command & Control (C2) -> Objectives
```

### 1. In (Initial Access & Foothold)
- **Reconnaissance:** Discovering target info.
- **Weaponization:** Preparing infrastructure/payload.
- **Delivery:** Transporting payload to target.
- **Social Engineering:** Manipulating human behavior to trigger execution.
- **Exploitation:** Exploiting software/hardware vulnerabilities.
- **Persistence:** Ensuring continued access (backdoors, run keys).
- **Defense Evasion:** Evading detection mechanisms.

### 2. Through (Internal Propagation)
- **Credential Access:** Dumping credentials (mimikatz, token theft).
- **Discovery:** Mapping internal network/domain structure.
- **Privilege Escalation:** Gaining administrative/system privileges.
- **Execution:** Running tools/malware locally.
- **Pivoting:** Using a compromised system as a bridge to other segments.
- **Lateral Movement:** Accessing and controlling other internal hosts.

### 3. Out (Action & Mission Success)
- **Collection:** Gathering sensitive database contents or files.
- **Exfiltration:** Transferring data out of the target environment.
- **Command & Control (C2):** Maintaining command sessions (beacons).
- **Objectives:** Executing the final goal (e.g., encryption, destruction, espionage).

---

## Room Questions & Answers

### Task 2 — Traditional Kill Chains
* **Question:** Where does the term "Kill Chain" originate from?  
  * **Answer:** `military`

### Task 3 — The Unified Kill Chain
* **Question:** What is the technical term for a piece of software/hardware in IT?  
  * **Answer:** `asset`
* **Question:** In what year was the Unified Kill Chain framework released?  
  * **Answer:** `2017`
* **Question:** According to the Unified Kill Chain, how many phases are there?  
  * **Answer:** `18`

### Task 4 — Phase: In, Through, Out
* **Question:** What is the name of the attack phase where an attacker employs techniques to evade detection?  
  * **Answer:** `Defense Evasion`
* **Question:** What is the name of the attack phase where an attacker removes data from a network?  
  * **Answer:** `Exfiltration`
* **Question:** What is the name of the attack phase where an attacker achieves their objectives?  
  * **Answer:** `Objectives`

### Task 5 — Practical Scenario (Match the Phase)
* **Scenario mapping of activities:**
  - Setting up Command & Control (C2) server infrastructure -> `Weaponization`
  - Exploiting a vulnerability on a system -> `Exploitation`
  - Leaving behind a malicious service to regain access -> `Persistence`
  - Moving from one system to another -> `Pivoting`
  - Seeking to gain higher-level account access following failed admin login attempts -> `Privilege Escalation`
  - Running Mimikatz to extract passwords -> `Credential Dumping` (or `Credential Access`)
  - Impersonating an employee to request a password reset -> `Social Engineering`
* **Question:** What is the flag?  
  * **Answer:** `THM{UKC_SCENARIO}`

---

## Key Learnings

- **Granularity:** The UKC provides a much more detailed and realistic model of modern threat actors (especially APTs) than traditional 7-stage chains.
- **Internal Defenses (Through):** By explicitly detailing phases like lateral movement, privilege escalation, and pivoting, the UKC guides defenders to build robust internal segmentation and zero-trust controls, rather than relying solely on perimeter defenses.
- **Unified Defense Mapping:** Combining the structural timeline of the Cyber Kill Chain with the detailed behavior catalog of MITRE ATT&CK makes the UKC a powerful framework for threat hunting and mapping active detections.
