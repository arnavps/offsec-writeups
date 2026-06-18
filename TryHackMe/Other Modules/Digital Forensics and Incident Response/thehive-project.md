# TheHive Project

## Overview

TheHive is a scalable, open-source Security Incident Response Platform designed for SOCs, CSIRTs, and CERTs. It provides a collaborative environment where multiple analysts can work simultaneously on the same security incident, tracking cases, tasks, observables, and IOCs in real time. This room covers TheHive's core functions, features, permission model, and how to work with cases and observables.

---

## Topics Covered

- What TheHive is and who uses it
- Three core functions: Collaborate, Elaborate, Act
- Key features and integrations
- Permission model
- Case and task management
- Observable management and enrichment
- MISP and Cortex integrations

---

## Key Concepts

### What is TheHive?

TheHive is a web-based incident response platform that centralises the investigation and management of security incidents. It is designed for teams that need to coordinate across multiple analysts during active investigations.

**Primary users:**
- SOC (Security Operations Center) analysts
- CSIRT (Computer Security Incident Response Team) members
- CERT (Computer Emergency Response Team) members

---

### Three Core Functions

| Function | Description |
|----------|-------------|
| **Collaborate** | Multiple analysts from one organisation work on the same case simultaneously. Live stream capabilities keep all team members updated in real time |
| **Elaborate** | Investigations are structured as cases with associated tasks, templates, progress notes, evidence attachments, and tags |
| **Act** | Analysts add observables to cases, tag IOCs, flag previously seen indicators, and feed threat intelligence workflows |

---

### Key Features

| Feature | Description |
|---------|-------------|
| **Case/Task Management** | Investigations are organised as cases. Cases break down into tasks. Templates allow repeatable investigation structures. Analysts can attach evidence, add tags, and record progress |
| **Alert Triage** | Alerts can be imported from SIEMs, email reports, and other security sources. Analysts review and decide whether to escalate to a full investigation |
| **Observable Enrichment with Cortex** | Cortex is an analysis and active response engine that enriches observables through correlation analysis and pattern development |
| **Active Response** | Responders allow analysts to take active containment and communication actions directly from TheHive |
| **Custom Dashboards** | Case statistics, task metrics, observable counts, and KPIs can be compiled and displayed on configurable dashboards |
| **Built-in MISP Integration** | MISP is a threat intelligence sharing platform. TheHive can create cases from MISP events, import IOCs from MISP, and export indicators back to MISP communities |

---

### Notable Integrations

| Integration | Purpose |
|-------------|---------|
| **Cortex** | Observable analysis and active response — enriches indicators and executes response actions |
| **MISP** | Threat intelligence sharing — import/export IOCs and create cases from threat intelligence events |
| **DigitalShadows2TH** | Alert feeder from DigitalShadows threat intelligence |
| **ZeroFox2TH** | Alert feeder from ZeroFox threat intelligence |

These integrations allow alerts from external intelligence sources to be automatically imported and transformed into TheHive cases using pre-defined incident response templates.

---

### Permission Model

TheHive uses a role-based permission system. Key permissions:

| Permission | Functions | Notes |
|------------|-----------|-------|
| `manageOrganisation` | Create and update organisations | Global — admin org only |
| `manageConfig` | Update configuration | Global — admin org only |
| `manageProfile` | Create, update, delete profiles | Global — admin org only |
| `manageTag` | Create, update, delete tags | Global — admin org only |
| `manageCustomField` | Create, update, delete custom fields | Global — admin org only |
| `manageCase` | Create, update, delete cases | Standard analyst permission |
| `manageObservable` | Create, update, delete observables | Standard analyst permission |
| `manageAlert` | Create, update, import alerts | Standard analyst permission |
| `manageUser` | Create, update, delete users | Admin permission |
| `manageCaseTemplate` | Create, update, delete case templates | Admin/senior analyst |
| `manageTask` | Create, update, delete tasks | Standard analyst permission |
| `manageShare` | Share cases, tasks, observables with other organisations | Cross-org collaboration |
| `manageAnalyse` | Execute analyses | Requires Cortex connector |
| `manageAction` | Execute actions | Requires Cortex connector |
| `manageAnalyserTemplate` | Create, update, delete analyser templates | Requires Cortex connector |

---

### Working with Observables

Observables are the indicators associated with a case — IP addresses, domains, file hashes, email addresses, URLs, etc.

**Adding an observable from the Observables tab:**

| Field | Description | Examples |
|-------|-------------|---------|
| Type | The observable data type | IP address, Hash, Domain, URL, Email |
| Value | The observable value | An IP address, a hash string, a domain name |
| One observable per line | Create one observable per line in the value field | Multiple IPs listed |
| One single multiline observable | Create one observable regardless of line count | Long URLs |
| TLP | Traffic Light Protocol — defines how the information can be shared | WHITE, GREEN, AMBER, RED |
| Is IOC | Flag if this observable is a confirmed Indicator of Compromise | Check for known malicious IP |
| Has been sighted | Whether this observable has been seen in your environment | Seen in firewall logs |
| Ignore for similarity | Do not correlate this observable with similar ones in other cases | Exclude common infrastructure IPs |
| Tags | Labels for context | `Malware IP`, `MITRE Tactic`, `Phishing` |
| Description | Free-text description of the observable | |

---

### TLP (Traffic Light Protocol)

| Level | Sharing Permission |
|-------|-------------------|
| WHITE | Unlimited — public sharing |
| GREEN | Community — share within the security community |
| AMBER | Limited — share within the organisation and trusted partners |
| RED | Restricted — do not share outside named recipients |

---

## Workflow / Process

```
Security alert received (from SIEM, email, threat intelligence feed)
        |
        v
Alert imported into TheHive
        |
        v
Analyst reviews alert triage — escalate or dismiss
        |
        v
Case created (from template or scratch)
        |
        v
Tasks created and assigned to team members
        |
        v
Analysts collaborate in real time — recording progress, attaching evidence
        |
        v
Observables added to the case (IPs, hashes, domains, URLs)
        |
        v
Cortex enriches observables — correlation, reputation lookups, pattern analysis
        |
        v
IOCs flagged, tagged, and exported to MISP for community sharing
        |
        v
Active response executed via Responders if needed
        |
        v
Case closed with full documentation
        |
        v
KPIs and metrics available on custom dashboards
```

---

## Real-World Relevance

- TheHive is used by enterprise SOCs and national CERTs as their primary case management platform for incident response coordination
- The MISP integration is critical for organisations that participate in threat intelligence sharing communities — IOCs identified during an investigation can be shared back to the community immediately
- Cortex enrichment enables analysts to automatically look up the reputation and context of observables without leaving the platform — reducing the manual pivot time during triage
- Alert triage directly from TheHive means analysts can process SIEM alerts without switching between multiple tools
- The live stream collaboration feature is particularly valuable during major incidents where multiple analysts need to work simultaneously without overwriting each other's work

---

## Key Learnings

- TheHive is an open-source collaborative incident response platform for SOCs, CSIRTs, and CERTs
- Three core functions: Collaborate (real-time), Elaborate (cases/tasks/evidence), Act (observables/IOCs/response)
- Cases contain tasks; tasks contain notes, attachments, and progress tracking
- Observables are typed indicators linked to a case — enriched by Cortex
- TLP defines how observable information can be shared beyond the investigation
- MISP integration enables bidirectional IOC sharing with threat intelligence communities

---

## Conclusion

TheHive provides the structural and collaborative backbone for incident response operations. By centralising case management, observable tracking, and threat intelligence sharing in one platform — and integrating with Cortex for enrichment and MISP for community intelligence — it gives analyst teams the coordination and context they need to investigate and respond to incidents efficiently. Its open-source nature makes it accessible to organisations of all sizes.
