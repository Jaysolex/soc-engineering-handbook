# Chapter 9 — Quick Actions

**Depends on:** Chapter 2 — Cortex Work Plans, Chapter 3 — Playbook Task Types, Chapter 4 — Inputs, Outputs & Context, Chapter 5 — Indicator Extraction, Chapter 6 — Filters & Transformers, Chapter 7 — Lists, Chapter 8 — Automation Features & Playbook Automation

```
 Detection
     ↓
   Issue
     ↓
 Automation
     ↓
 Playbook
     ↓
Quick Actions   ← YOU ARE HERE
     ↓
 Response
     ↓
Case Update
```

---

## Overview

Quick Actions are lightweight, single-purpose automations designed to perform common Security Operations Center (SOC) tasks without executing an entire playbook.

Unlike playbooks, which orchestrate multiple tasks and decision branches, Quick Actions execute **one focused operation immediately**. They enable analysts to respond rapidly to common situations while maintaining consistency across investigations.

Quick Actions can be:

- Executed manually by an analyst
- Triggered automatically by an Automation Rule
- Embedded as tasks inside a Playbook

Because of their speed and simplicity, Quick Actions are ideal for repetitive operational tasks that do not require complex workflow logic.

## Learning Objectives

- What Quick Actions are
- Why they exist
- Manual execution
- Automation Rule execution
- Playbook execution
- Quick Action lifecycle
- Integration requirements
- Production workflows
- SOC use cases
- Best practices

## What Is a Quick Action?

A Quick Action performs one action only. Think of it as a reusable command.

**Examples:** Send Slack message, send email, update issue severity, change issue status, add tag, isolate endpoint, quarantine file, run script, create ticket.

Instead of building an entire playbook for these common operations, Cortex provides Quick Actions that can be executed almost instantly.

## Playbook vs. Quick Action

| Playbook | Quick Action |
|---|---|
| Multiple tasks | One task |
| Complex logic | Simple operation |
| Conditions | No workflow logic |
| Multiple decisions | Immediate execution |
| Investigation workflow | Single response |

## Quick Action Architecture

```
   Issue
     │
     ▼
Run Quick Action
     │
     ▼
Single Action
     │
     ▼
Issue Updated
```

---

## Three Ways to Run Quick Actions

### 1. Manual Execution

The analyst manually launches the Quick Action — the most common usage.

```
   Issue
     │
     ▼
Right Click
     │
     ▼
Run Automation
     │
     ▼
Quick Action
     │
     ▼
 Completed
```

### 2. Automation Rule

A Quick Action runs automatically when an Issue matches predefined conditions. No analyst intervention required.

```
   Issue
     │
     ▼
Automation Rule
     │
     ▼
Quick Action
     │
     ▼
 Completed
```

### 3. Inside a Playbook

Quick Actions can be embedded as Standard Tasks (Chapter 3) within a larger playbook, combining lightweight actions with complex automation workflows.

```
 Playbook
     │
     ▼
Threat Intelligence
     │
     ▼
Conditional
     │
     ▼
Quick Action
     │
     ▼
Continue Playbook
```

## Common Quick Actions

| Quick Action | Purpose |
|---|---|
| Send Email | Notify stakeholders |
| Send Slack Message | Notify SOC |
| Update Severity | Escalate or downgrade |
| Change Status | New → In Progress → Resolved |
| Quarantine File | Contain malware |
| Isolate Endpoint | Network containment |
| Create Ticket | ServiceNow/Jira integration |
| Add Tags | Organize investigations |

---

## Manual Execution Example

An analyst receives a phishing alert:

```
   Issue
     │
     ▼
Right Click
     │
     ▼
Run Automation
     │
     ▼
Send Issue Summary Email
     │
     ▼
SOC Manager Receives Notification
```

The analyst does not need to open the playbook editor for this.

## Automation Rule Example

Critical ransomware detection:

```
Severity = Critical
        │
        ▼
Automation Rule
        │
        ▼
  Quick Action
        │
        ▼
  Notify Slack
        │
        ▼
  SOC Channel
```

Every critical ransomware alert immediately appears in Slack.

## Playbook Example

Your ransomware playbook:

```
Sigma Detection
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
   Quick Action
        │
        ▼
   Send Email
        │
        ▼
Continue Investigation
        │
        ▼
  IT Approval
        │
        ▼
Isolate Endpoint
```

Notice that the Quick Action performs one notification before the playbook continues — it doesn't replace the investigation, it accelerates one piece of it.

---

## Production Example 1 — Windows Script Host Detection

Sigma rule detects `wscript.exe` → external connection.

Playbook determines: **Confidence 95%**

```
  Quick Action
        │
        ▼
Send Slack Message
        │
        ▼
SOC Alerts Channel
```

Message:
```
High-confidence WScript execution detected.
Host: WIN-01
User: alice
Confidence: 95%
```

The analyst is notified immediately, while the rest of the investigation (Threat Intel, AD Lookup, Asset Lookup from Chapter 4) continues in parallel.

## Production Example 2 — Ransomware Detection

```
Suspicious Encryption
        │
        ▼
     Playbook
        │
        ▼
   Known Hash?
        │
       YES
        │
        ▼
 Critical Server?
        │
       YES
        │
        ▼
   Quick Action
        │
        ▼
Notify Incident Commander
        │
        ▼
Continue Playbook
```

The notification is immediate while the playbook continues gathering evidence — this is the same pattern as Example 1: the Quick Action buys communication speed without blocking the rest of the investigation.

---

## Integration Requirements

Many Quick Actions require integrations.

| Action | Integration |
|---|---|
| Slack | Slack API |
| Email | SMTP / Exchange |
| ServiceNow | ServiceNow Instance |
| Jira | Jira Instance |
| Teams | Microsoft Teams |

Automation Features (Chapter 8) allow these integrations to be configured directly within the interface.

## Engineering Insight

A Quick Action does not replace a playbook. Instead, it complements a playbook by handling fast, repetitive tasks.

Good engineers ask: **"Can this be solved with one action?"**

If yes, use a Quick Action. If no, build a playbook.

## Quick Action Lifecycle

```
   Issue
     │
     ▼
Select Quick Action
     │
     ▼
Validate Inputs
     │
     ▼
Run Integration
     │
     ▼
 Complete
     │
     ▼
Log Execution
```

Every execution is recorded for auditing.

## Production Workflow

```
 Detection
     │
     ▼
   Issue
     │
     ▼
Automation Rule
     │
     ▼
  Playbook
     │
     ▼
Threat Intelligence
     │
     ▼
 Critical?
     │
    YES
     │
     ▼
Quick Action
     │
     ▼
Send Slack Alert
     │
     ▼
Continue Investigation
     │
     ▼
Collect Memory
     │
     ▼
Endpoint Isolation
     │
     ▼
Resolve Case
```

The Quick Action accelerates communication while the playbook continues executing.

---

## Best Practices

- Use Quick Actions for single, repeatable tasks.
- Keep them simple.
- Reuse them across multiple playbooks.
- Configure integrations before production.
- Log all executions.
- Avoid embedding complex logic in Quick Actions.

## Common Mistakes

- Using a Quick Action instead of a playbook when the task actually needs branching or multiple steps.
- Building multiple actions into one Quick Action, eroding the "one action" contract.
- Forgetting integration credentials.
- Executing destructive Quick Actions without approval (e.g., isolate endpoint fired directly from an Automation Rule with no Conditional gate — see Chapter 3's SCDS framework).
- Duplicating Quick Actions unnecessarily instead of reusing one across playbooks.

## Key Takeaways

- Quick Actions perform one task.
- They can run manually, automatically (via Automation Rule), or inside playbooks.
- They improve analyst efficiency for repetitive operations.
- They rely on integrations to communicate with external systems.
- They complement, rather than replace, playbooks — the right question is "can this be solved with one action?"
- Mature SOC teams use Quick Actions to accelerate common operational tasks while reserving Playbooks for complex investigation workflows.

---

## Chapter Bridge — Three Layers of Automation

```
  Quick Action
       │
One Immediate Action
──────────────────────
   Playbook
       │
Multiple Coordinated Tasks
──────────────────────
  Work Plan
       │
Entire Investigation Lifecycle
```

This mental model makes it easy to choose the right tool:

- **Quick Action** → one action.
- **Playbook** → one workflow.
- **Work Plan** (Chapter 2) → one investigation.
