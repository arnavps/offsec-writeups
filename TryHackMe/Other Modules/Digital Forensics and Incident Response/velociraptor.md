# Velociraptor

## Overview

Velociraptor is an advanced open-source endpoint monitoring, forensic collection, and response platform. It enables security teams to collect forensic artifacts from hundreds or thousands of endpoints simultaneously, run hunts across the fleet, and respond to threats in real time — all from a central server. Unlike point-in-time forensic tools like KAPE or Redline, Velociraptor provides continuous visibility and the ability to query live endpoint state at any time using its own query language (VQL).

---

## Topics Covered

- What Velociraptor is and how it differs from other DFIR tools
- Architecture: server and client components
- VQL (Velociraptor Query Language)
- Artifacts: pre-built and custom
- Hunts: fleet-wide collection
- Incident response workflows
- Notebook: collaborative investigation

---

## Key Concepts

### What is Velociraptor?

Velociraptor is an endpoint agent-based platform designed for:
- **Forensic collection** — gather artifacts from live endpoints on demand
- **Threat hunting** — run queries across the entire fleet simultaneously
- **Incident response** — contain, investigate, and remediate from a central interface
- **Continuous monitoring** — detect anomalies as they happen rather than after the fact

Unlike KAPE (which runs locally and produces files) or Redline (which performs a point-in-time collection), Velociraptor maintains a persistent agent on each endpoint and allows interactive querying at any time.

---

### Architecture

| Component | Description |
|-----------|-------------|
| **Server** | Central management server hosting the Velociraptor GUI and data store |
| **Client (Agent)** | Lightweight agent installed on each endpoint — executes VQL queries from the server |
| **GUI** | Web-based interface for managing clients, running hunts, and reviewing results |
| **VQL** | Velociraptor Query Language — SQL-like language for querying endpoint state |

The agent communicates with the server over HTTPS, making it compatible with standard network security controls.

---

### VQL (Velociraptor Query Language)

VQL is a purpose-built query language for interacting with endpoint data. It is modelled after SQL but extends it with functions and plugins that operate on live system state.

**Basic structure:**
```sql
SELECT column1, column2
FROM plugin()
WHERE condition
```

**Example — list running processes:**
```sql
SELECT Pid, Name, Exe
FROM pslist()
```

**Example — search for a file:**
```sql
SELECT FullPath, Size, Mtime
FROM glob(globs="C:/Windows/System32/*.exe")
```

VQL enables analysts to write custom artifact queries tailored to specific investigation needs.

---

### Artifacts

In Velociraptor, an **artifact** is a named, reusable VQL query (or set of queries) packaged with metadata — name, description, parameters, and sources.

Artifacts can:
- Collect specific forensic data (e.g. running processes, open network connections, registry keys)
- Perform detection (e.g. check for known malware indicators)
- Execute response actions (e.g. quarantine a file, kill a process)

**Artifact categories:**
- `Windows.*` — Windows-specific artifacts
- `Linux.*` — Linux artifacts
- `Generic.*` — Cross-platform artifacts
- `Server.*` — Server-side artifacts for managing the Velociraptor deployment

**Example artifacts:**
| Artifact | Purpose |
|----------|---------|
| `Windows.System.Pslist` | List running processes |
| `Windows.Network.Netstat` | Active network connections |
| `Windows.Forensics.Prefetch` | Parse Prefetch files |
| `Windows.EventLogs.Evtx` | Collect and parse Windows event logs |
| `Windows.Forensics.Usn` | Parse NTFS USN journal |
| `Generic.Forensic.SqliteHunter` | Hunt SQLite databases (browser history, etc.) |

---

### Hunts

A **Hunt** is a fleet-wide collection run that executes an artifact against all (or a subset of) connected endpoints simultaneously.

**Hunt workflow:**
1. Select an artifact to run
2. Configure parameters (date ranges, specific paths, keywords, etc.)
3. Launch the hunt — all matching clients execute the artifact
4. Review aggregated results in the Hunt Manager

Hunts are the primary mechanism for threat hunting at scale — running a single query across thousands of endpoints in minutes rather than days of manual investigation.

---

### Incident Response Workflow

Velociraptor supports live response directly from the GUI:

1. **Identify a suspicious endpoint** via monitoring or alert
2. **Collect targeted artifacts** — run specific VQL queries for the indicators of interest
3. **Review results in real time** — no waiting for a collection to complete and be transferred
4. **Respond** — quarantine the endpoint, kill a malicious process, delete a file, or collect a memory image
5. **Run fleet hunt** — if the threat may be on other endpoints, launch a hunt across the fleet

---

### Notebook

The **Notebook** is Velociraptor's collaborative investigation workspace. It allows analysts to:
- Write notes alongside VQL queries
- Run queries directly within the notebook
- Share investigation context with team members
- Document findings in a structured format

---

## Important Terminology

| Term | Meaning |
|------|---------|
| VQL | Velociraptor Query Language — SQL-like language for querying live endpoint state |
| Artifact | A named, reusable VQL collection/detection query |
| Hunt | A fleet-wide artifact execution across all or selected endpoints |
| Client | Velociraptor agent installed on an endpoint |
| Server | Central management server hosting the GUI and data store |
| Notebook | Collaborative investigation workspace within Velociraptor |
| Flow | A single artifact execution on a single client |
| Label | Tags applied to clients for grouping (e.g. by OS, criticality, department) |

---

## Comparison: Velociraptor vs Other DFIR Tools

| Feature | KAPE | Redline | Volatility | Velociraptor |
|---------|------|---------|-----------|-------------|
| Collection scope | Single system | Single system | Memory only | Fleet-wide |
| Real-time querying | No | No | No | Yes |
| Continuous monitoring | No | No | No | Yes |
| Requires agent | No | No | No | Yes |
| Response capability | No | No | No | Yes |
| Custom queries | Limited | No | No | Yes (VQL) |
| Scale | Manual per-host | Manual per-host | Post-analysis | Automated fleet |

---

## Real-World Relevance

- Velociraptor is used by enterprise security teams and MSSPs for both proactive threat hunting and reactive incident response at scale
- The ability to query endpoints in real time without waiting for a full forensic collection is a significant advantage during active incidents — analysts can answer questions in minutes rather than hours
- Fleet hunts are used after a threat intelligence report names a specific IoC — analysts can check whether it exists on any endpoint in the environment within minutes
- Velociraptor's built-in artifact library covers most common forensic needs out of the box — custom VQL artifacts extend this for organisation-specific requirements
- Its open-source nature makes it accessible to organisations that cannot afford commercial EDR or IR platforms

---

## Key Learnings

- Velociraptor maintains persistent agents on endpoints — enabling real-time querying and fleet-wide hunts
- VQL is the core interaction mechanism — SQL-like syntax with plugins for live system state
- Artifacts are reusable, parameterised VQL queries — the primary unit of work in Velociraptor
- Hunts distribute artifact execution across the entire fleet simultaneously
- Velociraptor supports both collection (forensics) and response (containment, remediation) from the same interface
- The Notebook enables collaborative, documented investigation within the platform

---

## Conclusion

Velociraptor represents the evolution of DFIR tooling from point-in-time collection to continuous, fleet-scale investigation and response. Its combination of a persistent agent, VQL-based querying, pre-built artifact library, and real-time response capability makes it uniquely powerful for both threat hunting and incident response. For organisations with many endpoints, Velociraptor's ability to answer forensic questions across the entire fleet in minutes rather than weeks fundamentally changes the economics of DFIR work.
