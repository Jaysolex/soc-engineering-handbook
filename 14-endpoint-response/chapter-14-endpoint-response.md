# Chapter 14 — Endpoint Response

**Depends on:** Chapter 13 — Response Actions

```
Detection
    ↓
  Issue
    ↓
   Case
    ↓
Response Actions
    ↓
Endpoint Response   ← YOU ARE HERE
    ↓
Containment
    ↓
Remediation
    ↓
Recovery
```

---

## Overview

Endpoint Response provides analysts with the ability to remotely investigate, contain, and manage endpoints directly from Cortex without requiring physical access or remote desktop sessions.

Once a threat has been confirmed, analysts can isolate endpoints, remotely inspect running processes, browse the filesystem, execute commands, run Python scripts, retrieve forensic evidence, and perform containment actions — all from within the Cortex management console.

Unlike standard Response Actions (Chapter 13), which execute a single command, Endpoint Response provides an **interactive investigation environment** where analysts can actively examine and respond to compromised systems.

## Learning Objectives

- Endpoint Isolation
- Canceling Isolation
- Allowed Process Lists
- Live Terminal
- Task Manager
- File Explorer
- Command Line
- Python Console
- Endpoint Capabilities
- Remote Investigations
- Production response workflows
- Best practices

## Endpoint Response Architecture

```
 Analyst
    │
    ▼
 Cortex
    │
    ▼
Live Terminal
    │
    ▼
Endpoint Agent
    │
    ▼
 Endpoint
```

Everything occurs through the Cortex Agent.

## Why Endpoint Response Exists

Security analysts often need to verify alerts, investigate suspicious processes, collect evidence, search for malware, execute commands, and perform containment.

Without Endpoint Response, analysts would need RDP, SSH, VPN, or other remote desktop tools — each with its own access model, logging, and attack surface. Cortex centralizes these capabilities into one platform.

---

## Endpoint Isolation

Endpoint Isolation disconnects an endpoint from the network while maintaining communication with Cortex. This prevents attackers from moving laterally, communicating with command-and-control servers, exfiltrating data, or accessing additional internal systems — while the Cortex agent remains connected so analysts can continue their investigation.

### Isolation Workflow

```
 Endpoint
    │
    ▼
 Isolate
    │
    ▼
Network Blocked
    │
    ▼
Agent Still Connected
    │
    ▼
SOC Investigation Continues
```

This distinction is critical: **isolation blocks the endpoint's normal network communication — not the Cortex management channel.**

### Allowed Processes During Isolation

Sometimes a server cannot be completely disconnected. Cortex allows administrators to define Allowed Processes that can continue communicating with approved IP addresses even while the endpoint remains isolated.

**Example:**
```
sqlservr.exe    → Allowed → Backup Server
Backup Agent    → Allowed → Backup Network
```

This prevents business-critical services from failing during containment — a direct application of the "business context matters" principle from Chapters 3 and 11.

### Cancel Endpoint Isolation

Isolation is reversible.

```
 Endpoint
    │
    ▼
 Isolated
    │
    ▼
Threat Removed
    │
    ▼
Cancel Isolation
    │
    ▼
Network Restored
```

Analysts can cancel isolation through the Action Center, Cytool (`isolate stop`), or Live Terminal.

---

## Live Terminal

Live Terminal establishes a secure, interactive session with the endpoint. Unlike simple Response Actions, Live Terminal allows analysts to actively investigate the system in real time.

Think of it as a secure remote shell managed entirely by Cortex.

### Live Terminal Components

```
Live Terminal
      │
      ├── Task Manager
      ├── File Explorer
      ├── Command Line
      └── Python
```

Each interface provides different investigative capabilities.

### Task Manager

Allows analysts to inspect and control running processes: view process hierarchy, terminate process, suspend process, resume process, view WildFire verdict, open hash information.

```
explorer.exe
     │
     ▼
 wscript.exe
     │
     ▼
powershell.exe
     │
     ▼
encryptor.exe
```

Analysts can immediately identify suspicious processes and terminate them if necessary.

### File Explorer

Enables remote management of the endpoint filesystem: browse folders, retrieve files, rename files, move files, delete files.

```
C:\Users\Alice\Downloads
         │
         ▼
     invoice.js
         │
         ▼
  Retrieve Evidence
```

No RDP session is required.

### Command Line

Allows analysts to execute operating system commands remotely.

**Windows:** `tasklist`, `netstat -ano`, `whoami`
**Linux:** `ps aux`, `netstat`, `ls -la`

This provides rapid investigation without interactive desktop access.

### Python Console

One of Cortex's most powerful capabilities — Python scripts execute directly on the endpoint, even if Python is not installed locally.

**Examples:** Search registry keys, collect forensic artifacts, enumerate users, parse event logs, search for persistence, gather system information.

This is the same investigative surface Chapter 17's Scripts Library formalizes into named, versioned, reusable scripts (`check_file_signature`, `read_registry_value`, etc.) — Live Terminal's Python Console is where an analyst runs Python **ad hoc**; the Scripts Library is where that same logic graduates into a repeatable, auditable action once it's proven useful more than once.

---

## Disable Endpoint Capabilities

Cortex allows administrators to permanently disable certain endpoint capabilities — Live Terminal, File Retrieval, Script Execution.

Once disabled, they cannot simply be re-enabled. The Cortex Agent must be **uninstalled and reinstalled**. This protects sensitive systems from unauthorized remote access.

---

## Production Example 1 — Windows Script Host Detection

Sigma rule detects `wscript.exe` → External Connection → **High Confidence**.

**SOC workflow:**
```
   Issue
     │
     ▼
    Case
     │
     ▼
Threat Intelligence
     │
     ▼
Endpoint Isolation
     │
     ▼
Live Terminal
     │
     ▼
 Task Manager
     │
     ▼
Terminate wscript.exe
     │
     ▼
 File Explorer
     │
     ▼
Retrieve invoice.js
     │
     ▼
 Command Line
     │
     ▼
   netstat
     │
     ▼
   Python
     │
     ▼
Collect Registry Keys
     │
     ▼
 Resolve Case
```

Notice that analysts investigate before deleting evidence.

## Production Example 2 — Ransomware Detection

```
Encryption Activity
        │
        ▼
       Case
        │
        ▼
 Critical Server?
        │
       YES
        │
        ▼
  SOC Approval
        │
        ▼
Endpoint Isolation
        │
        ▼
  Live Terminal
        │
   ┌────┴────┬───────────┐
   ▼         ▼           ▼
Task Mgr  File Explorer  Command Line
   │         │           │
Terminate  Retrieve    vssadmin
encryptor  encrypted   list shadows
.exe       sample
        │
        ▼
     Python
        │
        ▼
Collect persistence artifacts
        │
        ▼
Memory Collection
        │
        ▼
   Remediation
        │
        ▼
 Cancel Isolation
        │
        ▼
  Resolve Case
```

## Live Investigation Workflow

```
Detection
    │
    ▼
   Case
    │
    ▼
Threat Intel
    │
    ▼
Endpoint Isolation
    │
    ▼
Live Terminal
    │
    ▼
Task Manager
    │
    ▼
File Explorer
    │
    ▼
Command Line
    │
    ▼
  Python
    │
    ▼
Evidence Collected
    │
    ▼
Containment
    │
    ▼
 Recovery
```

This represents a real-world endpoint investigation.

---

## Engineering Insight

Isolation does not end the investigation. It creates a safe environment for investigation.

Good analysts think:
```
Contain → Investigate → Collect Evidence → Remediate
```

**Not:**
```
Contain → Delete Everything
```

Evidence is often the most valuable asset during incident response.

## Best Practices

- Isolate endpoints only after understanding business impact.
- Collect forensic evidence before remediation.
- Use Live Terminal instead of RDP whenever possible.
- Review the process tree before terminating processes.
- Retrieve suspicious files before deleting them.
- Cancel isolation after remediation is complete.

## Common Mistakes

- Isolating domain controllers without approval.
- Killing processes before understanding their role.
- Deleting files before retrieval.
- Forgetting to restore isolated endpoints.
- Disabling endpoint capabilities unnecessarily.

## Key Takeaways

- Endpoint Response provides interactive remote investigation capabilities — Live Terminal is not just another Response Action, it's an investigation environment.
- Endpoint Isolation blocks normal network communication while maintaining the Cortex management channel.
- Live Terminal includes Task Manager, File Explorer, Command Line, and Python.
- Analysts should collect evidence before remediation whenever possible.
- Endpoint capabilities can be permanently disabled and require agent reinstallation to restore.
- Mature SOC teams use Endpoint Response to safely investigate and contain threats while preserving forensic evidence.

---

## Chapter Bridge — The Operating Room of Cortex

```
 Detection
     │
     ▼
Investigation
     │
     ▼
Endpoint Isolation
     │
     ▼
Live Terminal
     │
 ┌───┼──────────────┐
 ▼   ▼               ▼
Task File         Command
Mgr Explorer        Line
     │
     ▼
   Python
     │
     ▼
Evidence Collected
     │
     ▼
 Remediation
     │
     ▼
  Recovery
```

This is where analysts transition from observing an attack to actively controlling the compromised endpoint. Live Terminal provides the tools to investigate safely, validate findings, preserve evidence, and execute containment with confidence — and where its ad hoc Python work outgrows one-off use, Chapter 17's Scripts Library and Remote Script Executions are where that logic becomes a permanent, reusable part of the SOC's toolkit.
