# Chapter 4 — Inputs, Outputs & Context

**Depends on:** Chapter 2 — Cortex Work Plans, Chapter 3 — Playbook Task Types

```
Platform
   ↓
Detection
   ↓
Playbook
   ↓
Tasks
   ↓
Inputs / Outputs / Context   ← YOU ARE HERE
   ↓
Automation
   ↓
Response
   ↓
Closure
```

---

## Overview

Every Cortex playbook is powered by **Context**.

Context is Cortex's working memory. It stores everything discovered during an investigation so every downstream task can reuse previous results without repeating work.

Without Context, every task would need to independently query the SIEM, EDR, Threat Intelligence, Active Directory, and every other integration — making playbooks slow, expensive, and difficult to maintain. With Context, a value looked up once by Task 1 is simply *there*, ready to read, for Task 40.

This chapter answers one question: **how does Cortex remember everything while a playbook is running?**

## Learning Objectives

- Understand Context
- Understand Inputs
- Understand Outputs
- Understand Task Inputs
- Understand Task Outputs
- Understand Extend Context
- Understand Context Keys
- Understand the Task Cheatsheet
- Understand data movement inside Cortex
- Understand why Context is Cortex's working memory
- Design playbooks that reuse data efficiently
- Apply Context in production automation

---

## The Big Picture

```
   Alert
     │
     ▼
Playbook Starts
     │
     ▼
   Task 1
     │
     ▼
Context Updated
     │
     ▼
   Task 2
     │
     ▼
Context Updated
     │
     ▼
   Task 3
     │
     ▼
Context Updated
     │
     ▼
 Case Closed
```

Every task contributes new knowledge. Nothing should be queried twice if it already exists in Context.

## Engineering Insight

A common beginner mistake is thinking of a task in isolation:

```
Task
  │
  ▼
Output
  │
  ▼
Finished
```

An experienced Cortex engineer thinks one step further:

```
Task
  │
  ▼
Output
  │
  ▼
Context
  │
  ▼
Reusable by Every Future Task
```

That one extra arrow — output flows *into Context*, not just to the next task — is what makes large playbooks scalable instead of a chain of redundant lookups.

---

## Running Example: Windows Script Host with Network Connection

Every section below uses the same Sigma detection so you can watch one real investigation accumulate Context from the first task to the last.

```yaml
title: Windows Script Host with Network Connection
```

This detection is a good teaching example precisely because it's information-rich: a single alert hands you a process, a parent process, a command line, a user, a host, a destination IP, and a timestamp — seven pieces of raw material that the rest of the playbook will enrich, branch on, and eventually act on.

### Detection Fires — Initial Context

```json
{
  "hostname": "WIN-01",
  "user": "alice",
  "process": "wscript.exe",
  "parent": "explorer.exe",
  "command_line": "wscript.exe invoice.js",
  "destination_ip": "185.100.x.x"
}
```

This is Context at time zero — before any playbook task has run. Everything from here forward is additive.

### Task 1 — Threat Intelligence Lookup

```
   Input: destination_ip
        │
        ▼
Threat Intelligence Lookup   (Standard Task)
        │
        ▼
  Output: reputation, confidence
```

Context becomes:

```json
{
  "hostname": "WIN-01",
  "user": "alice",
  "process": "wscript.exe",
  "parent": "explorer.exe",
  "command_line": "wscript.exe invoice.js",
  "destination_ip": "185.100.x.x",
  "reputation": "Malicious",
  "confidence": 95
}
```

Notice the task only needed one field (`destination_ip`) as input — everything else in Context was simply along for the ride, available if a later task needs it.

### Task 2 — Active Directory Lookup

```
   Input: user
        │
        ▼
Active Directory Lookup   (Standard Task)
        │
        ▼
  Output: department, manager, vip_status
```

Context expands — nothing from Task 1 is overwritten:

```json
{
  "hostname": "WIN-01",
  "user": "alice",
  "process": "wscript.exe",
  "parent": "explorer.exe",
  "command_line": "wscript.exe invoice.js",
  "destination_ip": "185.100.x.x",
  "reputation": "Malicious",
  "confidence": 95,
  "department": "Finance",
  "vip": false,
  "manager": "John"
}
```

This is the core mechanic of Context: it only ever **grows**. Each task adds its own fields; it doesn't need to know or care what fields already exist.

### Task 3 — Conditional: Confidence > 90?

```
   Input: confidence
        │
        ▼
Conditional Task   (Confidence > 90?)
        │
       YES
        │
        ▼
    Branch A
```

No new Context field is written here — a Conditional Task *reads* Context to decide, it doesn't add to it. This is the SCDS "C" from Chapter 3 in action: choosing the path, not doing the work.

### Task 4 — Endpoint Lookup

```
   Input: hostname
        │
        ▼
Endpoint Lookup   (Standard Task)
        │
        ▼
  Output: critical_asset
```

Context now includes:

```json
{
  "critical_asset": true
}
```

(merged into the same growing object as everything above).

### Task 5 — Conditional → Data Collection

```
Conditional Task   (Critical Asset?)
        │
       YES
        │
        ▼
Data Collection Task
"Request IT Approval"
```

This is why Context exists: **every decision reuses previously collected data.** By the time the playbook asks IT for approval, it already knows the user, department, manager, VIP status, threat confidence, and asset criticality — none of it re-queried, all of it just *read* from Context and shown to the human making the call.

---

## Context Growth

```
Detection
    │
    ▼
 Context
    │
    ▼
   Host
    │
    ▼
   User
    │
    ▼
    IP
    │
    ▼
Threat Intel
    │
    ▼
Active Directory
    │
    ▼
Asset Criticality
    │
    ▼
Business Approval
    │
    ▼
Final Decision
```

Context becomes richer after every task — this diagram is just the running example above, generalized.

## Relationship Between Inputs and Outputs

```
Output (Task 1)
      │
      ▼
Input (Task 2)
      │
      ▼
Output (Task 2)
      │
      ▼
Input (Task 3)
      │
      ▼
Output (Task 3)
```

Outputs never disappear. They become Context, and Context is what every later task's input is drawn from.

---

## Extend Context

"Extend Context" is how new variables that didn't exist in the original alert get created *during* playbook execution — a task computes something new and writes it in.

Example, continuing the running example after Task 5's approval step resolves:

```json
{
  "risk_score": 98,
  "asset_criticality": "Critical",
  "business_owner": "Finance"
}
```

None of these three fields existed in the original detection. `risk_score` might be a task that combines `confidence` and `critical_asset` into one derived number; `business_owner` might come from a Lists lookup (Chapter 7). Either way, once written, they're just Context — available to every task after them exactly like `hostname` or `user` were from the start.

## Context Keys

Good naming is what keeps a large playbook maintainable six months later, by you or by someone else. Cortex Context Keys typically follow an `Object.Field` convention:

| Context Key | Meaning |
|---|---|
| `File.Hash` | SHA256 (or similar) of a file artifact |
| `Endpoint.Hostname` | The host involved in the investigation |
| `User.Email` | The identity involved |
| `Threat.Reputation` | Threat intel verdict |
| `Risk.Score` | A computed risk value |

Compare that to the alternative — a task author who doesn't follow this convention and instead invents `host1`, `theuser`, `rep`, `score2` — and you can see why naming discipline matters: at ten tasks it's an inconvenience, at a hundred tasks it's the difference between a maintainable automation library and one nobody dares touch.

## Task Cheatsheet

Cortex's Task Cheatsheet exists to solve exactly the problem above. Instead of an engineer needing to remember (or dig up) the raw underlying path to a value —

```
incident.artifacts[0].hash.sha256
```

— they select the equivalent field from the cheatsheet by its readable name. This isn't just convenience: it also means every playbook in the SOC's library ends up referencing the same field the same way, instead of five playbooks each hand-rolling a slightly different path to the same underlying value.

---

## Production Example — Full Walkthrough

Putting the whole running example together, start to finish:

```
Detection
    │
    ▼
Threat Intel
    │
    ▼
AD Lookup
    │
    ▼
Asset Lookup
    │
    ▼
Conditional
    │
    ▼
IT Approval
    │
    ▼
Endpoint Isolation
    │
    ▼
Collect Memory
    │
    ▼
Case Closed
```

| Step | Input (from Context) | Output (written to Context) |
|---|---|---|
| Detection fires | — | `hostname`, `user`, `process`, `parent`, `command_line`, `destination_ip` |
| Threat Intel | `destination_ip` | `reputation`, `confidence` |
| AD Lookup | `user` | `department`, `manager`, `vip` |
| Asset Lookup | `hostname` | `critical_asset` |
| Conditional | `confidence`, `critical_asset` | — (branch decision only) |
| IT Approval (Data Collection) | all of the above, shown to IT | `approval_status` |
| Endpoint Isolation | `hostname`, `approval_status` | `isolation_status` |
| Collect Memory | `hostname` | `memory_artifact_id` |
| Case Closed | — | `resolution` |

Every row shows the same pattern: **Input → Output → Context Update.** By the last row, the case record holds a complete, auditable trail of exactly what was known at each decision point — without a single field having been looked up twice.

## Engineering Insight

Never call an API twice if Context already contains the answer. Good playbooks minimize external lookups — not because API calls are expensive in isolation, but because every redundant lookup is a redundant point of failure, a redundant few seconds of latency, and (across thousands of alerts a month) a real, measurable drag on MTTR.

Good Context design makes automation fast. It's the difference between a playbook that gets slower as it grows more thorough, and one that gets *smarter* as it grows more thorough.

## Key Takeaways

- Context is Cortex's working memory — it persists everything discovered for the life of the investigation.
- Inputs come from Context. Outputs are written back into Context.
- Every downstream task can consume any previous task's outputs, not just the immediately preceding one.
- Extend Context is how new, computed variables (like `risk_score`) come into existence during execution.
- Consistent Context Key naming (`Object.Field`) is what keeps large playbook libraries maintainable.
- The Task Cheatsheet exists so engineers reference fields by readable name instead of raw underlying paths.
- A mature SOC engineer thinks in terms of data flow, not just task execution — the question is never just "what does this task do," it's "what does this task add to what every future task will know."
