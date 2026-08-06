# Chapter 8 — Automation Features & Playbook Automation

**Depends on:** Chapter 2 — Cortex Work Plans, Chapter 3 — Playbook Task Types, Chapter 4 — Inputs, Outputs & Context, Chapter 5 — Indicator Extraction, Chapter 6 — Filters & Transformers, Chapter 7 — Lists

```
Platform
   ↓
Detection
   ↓
Playbook
   ↓
Automation Rules
   ↓
Jobs
   ↓
Quick Actions
   ↓
Response
```

---

## Overview

Automation is the engine that allows Cortex to investigate and respond without requiring constant analyst intervention. Rather than manually starting every playbook, Cortex can automatically execute workflows based on predefined conditions, scheduled intervals, or changes in external data sources.

Automation consists of four major components:

- **Automation Rules**
- **Jobs**
- **Playbooks**
- **Quick Actions** (introduced here, covered in full in Chapter 11)

Each component solves a different operational problem while working together as one automation framework.

## Learning Objectives

- Automation Rules
- Trigger Types
- Issue-based automation
- Manual triggers
- Sub-playbooks
- Jobs
- Time-triggered Jobs
- Feed-triggered Jobs
- Playbook Catalog
- Org Playbooks
- Playbook adoption
- Integration instances
- Automation architecture
- Production workflows
- Best practices

## Cortex Automation Architecture

```
   SIEM
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
   Tasks
     │
     ▼
  Context
     │
     ▼
 Decision
     │
     ▼
 Response
```

Automation decides **when** a playbook should execute.

## Automation Components

```
Automation
    │
    ├── Rules
    ├── Jobs
    ├── Playbooks
    └── Quick Actions
```

| Component | Purpose |
|---|---|
| Playbook | Defines the workflow |
| Automation Rule | Starts the playbook based on an issue |
| Job | Runs the playbook on a schedule or feed update |
| Quick Action | Executes a single action quickly |

---

## Automation Rules

Automation Rules are **reactive**. They wait for an Issue.

```
Issue Created
     │
     ▼
 Rule Matches
     │
     ▼
 Run Playbook
```

**Examples:** Malware, privilege escalation, phishing, suspicious PowerShell, impossible travel.

### Trigger Types

**1. Issue-Based**
```
Issue → Rule → Playbook
```
Most common. Runs immediately after a matching Issue is created.

**2. Manual**
```
Analyst → Run Playbook
```
Useful for testing, one-time investigations, re-running automation.

**3. Sub-Playbook**
```
Parent Playbook → Sub-playbook
```
Allows reusable logic. Example: every playbook uses the same Threat Intelligence enrichment. Instead of duplicating it, create one Sub-playbook and call it from every parent playbook that needs it.

---

## Jobs

Unlike Rules, Jobs are **proactive**. They don't wait for Issues.

### Time-Based Jobs

```
   08:00
     │
     ▼
Run Threat Hunt
```

**Examples:** Weekly compliance scan, daily IOC enrichment, weekly Active Directory sync.

### Feed-Based Jobs

```
Threat Feed
     │
     ▼
  Updated
     │
     ▼
    Job
     │
     ▼
 Playbook
```

Exactly the Jobs → Lists pattern established in Chapter 7.

### Production Example 1 — Feed Sync

```
Threat Intelligence Feed
      (AlienVault OTX)
        │
        ▼
     New IOC
        │
        ▼
    Feed Job
        │
        ▼
Update Malicious IP List
        │
        ▼
Tomorrow's Investigation
```

No analyst needed.

### Production Example 2 — Ransomware Detection

```
 Sigma Detection
        │
        ▼
   Issue Created
        │
        ▼
Automation Rule
        │
        ▼
Ransomware Playbook
        │
        ▼
  Extract Hash
        │
        ▼
Read Ransomware List
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
 Data Collection
        │
        ▼
  SOC Approval
        │
        ▼
Endpoint Isolation
        │
        ▼
Memory Collection
        │
        ▼
 Remediation
```

Notice: **the Rule started everything.** No analyst clicked Run — the same ransomware playbook from earlier chapters, now triggered automatically the instant the Issue is created.

### Jobs + Lists

One of Cortex's most powerful designs:

```
Threat Feed
     │
     ▼
    Job
     │
     ▼
 List Updated
     │
     ▼
  Playbook
     │
     ▼
  Read List
     │
     ▼
 Decision
```

The playbook never contacts the feed directly.

---

## Playbook Catalog

Cortex includes many prebuilt playbooks — AWS, Azure, Active Directory, Office 365, Slack, Jira, VirusTotal, ServiceNow, and more. These can be adopted into your organization rather than built from scratch.

### Adopt vs. Duplicate

Out-of-the-box playbooks cannot be edited directly.

```
Marketplace
     │
     ▼
   Adopt
     │
     ▼
Org Playbook
     │
     ▼
 Duplicate
     │
     ▼
 Customize
```

This preserves the original template while allowing customization — the same "don't edit the vendor's copy, adapt your own" discipline you'd apply to any third-party template or library.

### Integration Instances

Many playbooks require third-party access — AWS, Slack, ServiceNow, Jira, VirusTotal. Instead of leaving the editor, Automation Features allow integration instances to be created directly from the playbook:

```
 Playbook
     │
     ▼
Missing Integration
     │
     ▼
Create Instance
     │
     ▼
  API Keys
     │
     ▼
   Ready
```

No context switching required.

## Engineering Insight — Integrations Are Not Automation

Integrations are not automation. They simply allow playbooks to communicate with external systems.

**Automation decides when to execute. Integrations decide where to execute actions.**

This distinction matters when troubleshooting: if a playbook isn't triggering, the problem is likely in the Automation Rule or Job. If a playbook triggers but a task fails, the problem is likely in the Integration Instance.

---

## Automation Workflow

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
 Conditional
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
 Response
```

## Full Enterprise Workflow

```
Threat Feed
     │
     ▼
  Feed Job
     │
     ▼
Update IOC Lists
──────────────────────
 SIEM Alert
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
 Read Lists
     │
     ▼
Threat Intel
     │
     ▼
Conditional Logic
     │
     ▼
Business Approval
     │
     ▼
Endpoint Isolation
     │
     ▼
Collect Memory
     │
     ▼
Notify Slack
     │
     ▼
Create Jira Ticket
     │
     ▼
Resolve Case
```

Notice how every previous chapter appears in one workflow: Work Plans (Ch.2) structure it, Task Types (Ch.3) build it, Context (Ch.4) carries data through it, Indicator Extraction (Ch.5) and Filters/Transformers (Ch.6) prepare the data, Lists (Ch.7) supply fast local knowledge, and this chapter is what starts it running without a human clicking anything.

---

## Best Practices

- Use Rules for reactive investigations.
- Use Jobs for recurring work.
- Keep playbooks modular.
- Reuse Sub-playbooks.
- Adopt before modifying.
- Test automation in non-production first.
- Minimize destructive automatic actions.

## Common Mistakes

- Using Jobs instead of Rules (or vice versa) — reactive and proactive automation solve different problems and aren't interchangeable.
- Duplicating logic instead of Sub-playbooks.
- Editing production/out-of-the-box playbooks directly instead of adopting and duplicating.
- Automating high-risk actions without approval.
- Forgetting required integrations, causing a playbook to trigger correctly but fail mid-execution.

## Key Takeaways

- Automation Rules react to Issues. Jobs proactively execute recurring work — the two are event-driven vs. schedule/feed-driven, and are suited to different problems, not interchangeable.
- Playbooks define the workflow; Rules and Jobs decide when it runs.
- Integration Instances connect Cortex to external platforms — they're not automation themselves, just the "where."
- Automation Features eliminate unnecessary context switching (creating integration instances inline in the playbook editor).
- Mature SOCs combine Rules, Jobs, Lists, and Playbooks to create scalable, repeatable investigations.

---

## Chapter Bridge

This chapter marks the point where the handbook becomes a complete SOC engineering reference. Every concept covered so far — Context, Lists, Filters, Work Plans, and Playbooks — comes together here. From this point onward, the remaining chapters focus on what happens *after* automation starts: Quick Actions, Issues, Cases, Response Actions, Endpoint Response, and Advanced Response. By the end of the handbook, the full lifecycle will be documented — Detection → Automation → Investigation → Response → Remediation — using real engineering examples rather than generic vendor documentation.
