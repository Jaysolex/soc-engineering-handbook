# Chapter 13 — Response Actions

**Depends on:** Chapter 10 — Working with Issues, Chapter 11 — Issue Prioritization, Chapter 12 — Working with Cases

```
Detection
    ↓
  Issue
    ↓
   Case
    ↓
Investigation
    ↓
Response Actions   ← YOU ARE HERE
    ↓
Containment
    ↓
Remediation
    ↓
Recovery
```

---

## Overview

Response Actions are the mechanisms Cortex provides to investigate, contain, remediate, and manage security incidents directly from the management console.

Rather than requiring analysts to remotely access endpoints or use multiple security products, Cortex allows response actions to be initiated centrally and delivered directly to managed endpoints.

Response Actions range from simple operations — such as retrieving files or running malware scans — to high-impact containment measures like endpoint isolation, file destruction, and agent upgrades.

Every Response Action is tracked within the **Action Center**, providing visibility into execution status, results, failures, and follow-up actions.

## Learning Objectives

- Response Action categories
- Response Action targets
- Server-initiated actions
- WebSocket communication
- Action Center
- Tracking actions
- Action lifecycle
- Action status
- Follow-up actions
- Investigation actions
- Maintenance actions
- Production workflows
- Best practices

## What Are Response Actions?

Response Actions are commands sent from Cortex to managed endpoints. These commands allow analysts to investigate or contain threats without physically accessing the affected systems.

Think of Response Actions as remote operational commands executed by the Cortex XDR agent.

---

## Three Categories of Response Actions

```
Response Actions
       │
       ├── Instant Response
       ├── Investigation
       └── Maintenance
```

Each category has a different operational objective.

### 1. Instant Response

These actions immediately contain or stop malicious activity.

**Examples:** Isolate Endpoint, Terminate Process, Quarantine File, Destroy File, Block Hash, Allow Hash, External Dynamic List (EDL).

**Purpose:** Stop the attack.

### 2. Investigation

These actions collect additional evidence.

**Examples:** Live Terminal, Retrieve Files, Retrieve Memory, Support File Retrieval, Malware Scan, File Search.

**Purpose:** Gather evidence before remediation.

### 3. Maintenance

Administrative actions used to manage Cortex itself.

**Examples:** Agent Upgrade, Agent Uninstall, Import Hash Exceptions, Run Endpoint Scripts (see Chapter 17).

**Purpose:** Maintain the endpoint environment.

---

## Response Action Targets

Not every Response Action affects the same object.

```
Response Actions
       │
       ├── Processes
       ├── Files
       ├── Endpoints
       └── Networks
```

**Processes** — Kill Process, Suspend Process. Only the selected process is affected.

**Files** — Quarantine, Destroy, Retrieve. Only the selected file changes.

**Endpoints** — Isolate Endpoint, Malware Scan, Live Terminal. The entire endpoint is affected.

**Networks** — External Dynamic List:
```
EDL
 │
 ▼
Firewall
 │
 ▼
Block IP / Block Domain
```
These actions affect network communication rather than endpoint processes.

---

## Server-Initiated Actions

An important Cortex concept: Response Actions originate from the Cortex server.

```
 Analyst
    │
    ▼
Cortex Server
    │
    ▼
 WebSocket
    │
    ▼
Endpoint Agent
```

Unlike traditional polling, Cortex pushes actions to agents using **WebSocket**.

### Why WebSocket?

WebSocket provides faster delivery, persistent communication, near real-time execution, and better Live Terminal performance. If WebSocket becomes unavailable, Cortex can fall back to HTTP.

---

## Action Center

The Action Center is the operational dashboard for every Response Action. It allows analysts to launch actions, track execution, review results, perform follow-up actions, access the Script Library (Chapter 17), manage block lists, and review completed actions.

Think of it as the command center for endpoint response.

### Action Center Workflow

```
 Analyst
    │
    ▼
Action Center
    │
    ▼
New Action
    │
    ▼
 Endpoints
    │
    ▼
Execution
    │
    ▼
  Status
    │
    ▼
Completed
```

## Common Response Actions

| Action | Purpose |
|---|---|
| Isolate Endpoint | Network containment |
| Retrieve File | Evidence collection |
| Malware Scan | Endpoint investigation |
| Destroy File | Remove malware |
| File Search | Locate files |
| Memory Collection | Capture RAM |
| Run Endpoint Script | Execute Python remotely |
| Agent Upgrade | Upgrade Cortex Agent |

## Action Lifecycle

```
 Create
    │
    ▼
 Queued
    │
    ▼
Delivered
    │
    ▼
Executing
    │
    ▼
Completed
    │
    ▼
 Tracked
```

Nothing happens silently. Every action is logged.

## Action Status

Analysts monitor execution through the Action Center.

**Common:** Pending → In Progress → Completed Successfully

**Alternatives:** Expired, Canceled, Failed, Partial Success

These statuses help analysts determine whether additional action is required.

## Follow-Up Actions

Many Response Actions support follow-up operations:

- After **Quarantine** → Restore File
- After **Isolation** → Cancel Isolation
- After **Retrieve File** → Download Evidence

Response is therefore reversible when appropriate.

---

## Production Example 1 — Windows Script Host Detection

Sigma rule detects `wscript.exe` → Malicious Connection.

**Investigation:**
```
Issue → Case → Threat Intelligence → High Confidence
```

**Response Actions:**
```
Retrieve Script
      │
      ▼
Retrieve Memory
      │
      ▼
Malware Scan
      │
      ▼
Live Terminal
      │
      ▼
Containment Decision
```

Evidence is collected before remediation.

## Production Example 2 — Ransomware Detection

Ransomware detection triggers.

```
Encryption
    │
    ▼
   Case
    │
    ▼
Threat Intelligence
    │
    ▼
Critical Asset?
    │
   YES
    │
    ▼
SOC Approval
    │
    ▼
Response Actions
```

**Actions:**
```
Memory Collection
       │
       ▼
Retrieve Files
       │
       ▼
Destroy Malware
       │
       ▼
Block Hash
       │
       ▼
Notify IR Team
```

Notice: containment follows investigation — not the other way around.

---

## Engineering Insight

One of the biggest mistakes junior analysts make is responding too quickly. Good analysts follow this sequence:

```
Detect → Investigate → Collect Evidence → Contain → Remediate
```

**Not:**
```
Detect → Delete Everything
```

Evidence is often lost if destructive actions occur first.

## Best Practices

- Collect evidence before destructive actions whenever feasible.
- Track every Response Action in the Action Center.
- Validate endpoint connectivity before launching actions.
- Use Live Terminal for deeper investigations.
- Review action results before closing Cases.
- Document all manual response decisions.

## Common Mistakes

- Isolating endpoints before collecting evidence.
- Ignoring failed Response Actions.
- Assuming actions completed without checking status.
- Performing destructive actions on critical assets without approval.
- Forgetting to cancel temporary containment measures.

## Key Takeaways

- Response Actions allow analysts to investigate and contain threats remotely.
- Cortex groups actions into Instant Response, Investigation, and Maintenance categories.
- Actions are delivered to agents using WebSocket for near real-time execution, with HTTP fallback.
- The Action Center is the central management interface for all Response Actions and tracks every action through its lifecycle.
- Mature SOC analysts prioritize evidence collection before destructive containment actions.

---

## Chapter Bridge — Response Actions as the Hands of Cortex

```
 Detection
     │
     ▼
Investigation
     │
     ▼
Analyst Decision
     │
     ▼
Response Action
     │
     ▼
 Endpoint
     │
     ▼
Containment
```

Detection identifies the problem. Investigation explains the problem. Response Actions actually change the environment. Because they directly impact production systems, they should always be executed with an understanding of both technical risk and business impact — the same principle Chapter 14 (Endpoint Response) applies specifically to the endpoint-targeted subset of these actions.
