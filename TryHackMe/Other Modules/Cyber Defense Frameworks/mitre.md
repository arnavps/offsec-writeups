# MITRE

**Platform:** TryHackMe  
**Room:** MITRE  
**Category:** Threat Intelligence / Cyber Defense Frameworks  
**Difficulty:** Easy  

---

## Overview

The **MITRE Corporation** is a US non-profit organization that manages federally funded research and development centers. In the cybersecurity domain, MITRE has provided some of the most critical and widely adopted defensive and offensive frameworks in the world.

This room explores MITRE's diverse ecosystem of resources:
1. **MITRE ATT&CK®:** Adversarial Tactics, Techniques, and Common Knowledge. A globally-accessible matrix mapping attacker behaviors.
2. **MITRE CAR (Cyber Analytics Repository):** A repository of actionable detection analytics (queries and pseudocode) mapped to ATT&CK techniques.
3. **MITRE D3FEND:** A knowledge graph outlining defensive countermeasures that directly mitigate specific offensive techniques in ATT&CK.
4. **MITRE Engage:** A framework for adversary engagement, active defense, and cyber deception.
5. **MITRE Adversary Emulation Plans:** Playbooks that security teams can use to simulate specific real-world threat actors (like APT3 or APT29) to test security controls.

---

## The MITRE Framework Ecosystem

```
             [ ATT&CK ]
       (Adversary Behaviors)
          /            \
         v              v
     [ D3FEND ]      [ CAR ]
  (Countermeasures) (Detection Queries)
         \              /
          v            v
     [ Adversary Emulation Plans ]
          (Testing Controls)
```

---

## Room Questions & Answers

### Task 2 — MITRE ATT&CK
* **Question:** Who else uses the ATT&CK Matrix besides blue teamers?  
  * **Answer:** `Red Teamers`

### Task 3 — ATT&CK Matrix Tasks
* **Question:** What is the ID for the "Phishing" technique?  
  * **Answer:** `T1566`
* **Question:** What Tactic does "Hide Artifacts" belong to?  
  * **Answer:** `Defense Evasion`
* **Question:** What ID is associated with "Create Account"?  
  * **Answer:** `T1136`

### Task 4 — ATT&CK Matrix: Groups and Software
* **Question:** Mustang Panda is a China-based cyber espionage actor. What is their base country?  
  * **Answer:** `China`
* **Question:** Under the Software section for Mustang Panda, what software is listed for Access Token Manipulation?  
  * **Answer:** `Cobalt Strike`

### Task 5 — Cyber Analytics Repository (CAR)
* **Question:** CAR-2020-09-001 (Scheduled Task - File Access) monitors file creation in specific Windows directories. What are the two primary directories (legacy and current) mentioned for scheduled task files? (Format: Directory1 and Directory2)  
  * **Answer:** `C:\Windows\Tasks and C:\Windows\System32\Tasks`

### Task 6 — MITRE D3FEND
* **Question:** What is the first MITRE ATT&CK technique listed in the ATT&CK Lookup dropdown?  
  * **Answer:** `Data Obfuscation`
* **Question:** In D3FEND Inferred Relationships, what does the ATT&CK technique from the previous question produce?  
  * **Answer:** `Outbound Internet Network Traffic`

### Task 7 — ATT&CK® Emulation Plans
* **Question:** In Phase 1 for the APT3 Emulation Plan, what is listed first?  
  * **Answer:** `C2 Setup`

### Task 8 — ATT&CK and Threat Intelligence (Aviation Sector Scenario)
* **Question:** What is a group that targets your sector who has been in operation since at least 2013?  
  * **Answer:** `APT33`
* **Question:** As your organization is moving to the cloud, is there anything attributed to this group that you should focus on?  
  * **Answer:** `Cloud Accounts`
* **Question:** Under the Software section for the technique from the previous question, what tool is associated with this technique?  
  * **Answer:** `Ruler`
* **Question:** Under the Mitigation section for this technique, what mitigation suggests SMS as an alternative method?  
  * **Answer:** `Multi-factor Authentication`
* **Question:** What mitigation strategy focuses on the removal of inactive or unused accounts?  
  * **Answer:** `User Account Management`
* **Question:** What Detection Strategy ID would you implement to detect abused or compromised cloud accounts?  
  * **Answer:** `DET0546`

---

## Key Learnings

- **Bridging Offense and Defense:** While ATT&CK catalogues attacker behaviors, D3FEND and CAR show defenders how to actively mitigate and detect those behaviors.
- **Actionable Detection Engineering:** CAR translates theoretical ATT&CK techniques into concrete queries (Splunk EQL, Sysmon) that can be directly deployed in a SIEM.
- **Active Deception (Engage):** Rather than just defending passively, Engage provides a matrix for setting up honey tokens, decoy servers, and active engagement policies to study and exhaust threat actors.
- **Empirical Validation:** Emulation Plans (like those for APT3/APT29) are critical for moving security testing from compliance-based checkmarks to real-world behavioral testing.
