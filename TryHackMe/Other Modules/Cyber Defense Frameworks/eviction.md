# Eviction

**Platform:** TryHackMe  
**Room:** Eviction  
**Category:** Incident Response / Threat Intelligence / MITRE ATT&CK  
**Difficulty:** Easy-Medium  

---

## Overview

The **Eviction** room is a scenario-based cybersecurity challenge where you play the role of **Sunny**, a SOC analyst at **E-corp**. E-corp is a manufacturer of rare-earth metals for government and commercial clients, making them a prime target for advanced threat actors.

Sunny receives a intelligence report warning that the Russian state-sponsored threat group **APT28 (Fancy Bear)** might be targeting organizations similar to E-corp. To proactively defend the enterprise, Sunny uses the **MITRE ATT&CK® Navigator** to visualize, map, and analyze APT28's Tactics, Techniques, and Procedures (TTPs). By mapping the threat intelligence report, Sunny establishes a baseline to monitor, hunt, and block the threat actor.

---

## Technical Scenario & MITRE ATT&CK Mapping

Using the MITRE ATT&CK Navigator, Sunny isolates the exact TTP profile of APT28:

```
[ Recon & Initial Access ] ---> Spearphishing Link / Compromised Email Accounts
                                      |
                                      v
[ Execution ] ----------------> User Execution (Malicious File & Malicious Link)
                                      |
                                      v
[ Scripting Interpreters ] ----> PowerShell & Windows Command Shell
                                      |
                                      v
[ Persistence & Proxying ] ----> Registry Run Keys & Rundll32 Proxy Execution
                                      |
                                      v
[ Lateral & Exfiltration ] ----> SMB/Admin Shares, SharePoint Data Theft, Multi-hop Proxies
```

---

## Room Questions & Answers

### Task 1 — Understand the Adversary & Map the TTPs
* **Question 1:** Какую технику использует APT для проведения разведки и получения первоначального доступа?  
  *(What is a technique used by the APT to both perform recon and gain initial access?)*  
  * **Answer:** `Spearphishing link`
* **Question 2:** Санни определил, что APT могла продвинуться вперед от фазы разведки. Какие учетные записи APT могла скомпрометировать при разработке ресурсов?  
  *(Sunny determined that the APT could have advanced from the recon phase. Which accounts might the APT compromise while developing resources?)*  
  * **Answer:** `Email accounts`
* **Question 3:** E-corp обнаружила, что APT могла получить начальный доступ, используя социальную инженерию, чтобы заставить пользователя выполнить код для злоумышленника. Санни хочет определить, была ли APT также успешной в выполнении. На какие два метода выполнения пользователя следует обратить внимание Санни?  
  *(E-corp discovered that the APT could have gained initial access using social engineering to execute code. What two techniques of user execution should Sunny look out for?)*  
  * **Answer:** `Malicious file and malicious link`
* **Question 4:** Если описанная выше методика оказалась успешной, какие интерпретаторы сценариев следует искать Санни, чтобы определить успешное выполнение?  
  *(If the above technique was successful, which scripting interpreters should Sunny search for to identify successful execution?)*  
  * **Answer:** `Powershell and Windows Command shell`
* **Question 5:** При просмотре интерпретаторов скриптов, определенных в Q4, Санни обнаружила несколько запутанных скриптов, которые изменили реестр. Предполагая, что эти изменения предназначены для поддержания персистентности, какие ключи реестра должен отслеживать Санни, чтобы отслеживать эти изменения?  
  *(While looking at the scripting interpreters, which registry keys should Sunny observe to track changes for persistence?)*  
  * **Answer:** `Registry run keys`
* **Question 6:** Санни определил, что APT выполняет системные двоичные файлы, чтобы обойти защиту. Выполнение какого системного двоичного файла Санни должен тщательно проверить на предмет выполнения через прокси?  
  *(Sunny determined that the APT executes system binaries to bypass defenses. Which system binary's execution should Sunny scrutinize for proxy execution?)*  
  * **Answer:** `Rundll32`
* **Question 7:** Санни идентифицировал tcpdump на одном из скомпрометированных хостов. Если предположить, что это было размещено там субъектом угрозы, какой метод APT может использовать здесь для обнаружения?  
  *(Sunny identified tcpdump on one of the compromised hosts. Assuming this was placed there by the threat actor, what technique of APT can be used here for discovery?)*  
  * **Answer:** `Network sniffing`
* **Question 8:** Похоже, что APT достигла бокового перемещения, эксплуатируя удаленные сервисы. Какие удаленные сервисы должен наблюдать Санни, чтобы идентифицировать следы активности APT?  
  *(It seems that the APT achieved lateral movement by exploiting remote services. Which remote services should Sunny observe to identify traces of APT activity?)*  
  * **Answer:** `SMB/Windows Admin shares`
* **Question 9:** Похоже, что основной целью APT была кража интеллектуальной собственности из информационных репозиториев E-corp. Какой информационный репозиторий может быть вероятной целью APT?  
  *(It seems that the main goal of the APT was to steal IP from information repositories. Which information repository might be a likely target?)*  
  * **Answer:** `Sharepoint`
* **Question 10:** Хотя APT собрала данные, она не смогла подключиться к C2 для эксфильтрации данных. Чтобы помешать любым попыткам сделать это, какие типы прокси-серверов может использовать APT?  
  *(Although the APT gathered data, they failed to connect to C2. To prevent any attempts, which types of proxy servers might the APT use?)*  
  * **Answer:** `external proxy and multi-hop proxy`

---

## Key Learnings

- **Adversary Modeling:** The MITRE ATT&CK Navigator is an invaluable tool for mapping and visualizing threat intelligence reports. It allows defenders to see a threat actor's entire playbook at a glance.
- **Rundll32 Abuse (Lolbins):** System binaries like `rundll32.exe` are built-in and trusted, but threat actors frequently abuse them to run DLL payloads under trusted processes. Monitoring executing command-lines for `rundll32.exe` is essential for Evasion detection.
- **Defensive Focus areas:** Profiling APT28 shows that they rely on user execution (malicious attachments/links) to bypass perimeter controls. Implementing strict endpoint protection policies on scripting hosts (PowerShell, Command Shell) and monitoring registry run keys are low-friction, high-impact detections.
