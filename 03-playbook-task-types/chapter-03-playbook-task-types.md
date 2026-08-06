# Chapter 3 — Playbook Task Types (SCDS)

**Depends on:** Chapter 1 — Cortex Platform Architecture, Chapter 2 — Cortex Work Plans

```
Platform
    ↓
Work Plan
    ↓
Playbook
    ↓
Task Types   ← YOU ARE HERE
    ↓
Inputs / Outputs
    ↓
Automation
```

---

## Overview

Playbook tasks are the fundamental building blocks of every Cortex playbook. Every investigation, automation, enrichment, decision, approval, and remediation activity is represented as one or more tasks.

A playbook is not a single automation. It is a collection of individual tasks that work together to investigate, enrich, make decisions, collect information, and respond to security events.

Understanding task types is essential because every Cortex automation — from a two-step enrichment playbook to a fifty-step ransomware response — is ultimately constructed from the same four components.

## Learning Objectives

After completing this chapter, you should understand:

- What playbook tasks are
- Why Cortex uses task-based workflows
- The four Cortex task types
- When each task should be used
- When each task should NOT be used
- How tasks exchange information
- How tasks execute
- Human versus automated tasks
- Production workflow design
- Risk-based automation
- Business approval workflows
- Best practices for playbook design

---

## Cortex Playbook Architecture

```
Playbook
   │
   ├── Standard Task
   │
   ├── Conditional Task
   │
   ├── Data Collection Task
   │
   └── Section Header
```

Everything inside a Cortex playbook is built using these four task types. There is no fifth primitive — every capability covered later in this handbook (enrichment, remediation, approvals, sub-playbooks) is an arrangement of these four building blocks.

## SCDS Memory Framework

**S** — Standard
**C** — Conditional
**D** — Data Collection
**S** — Section Header

```
Standard          →  Do the Work
Conditional       →  Choose the Path
Data Collection   →  Ask the Human
Section Header    →  Organize Everything
```

This framework should be remembered permanently — nearly every playbook you will ever build or debug is composed using it.

## Why Cortex Uses Four Task Types

Each task type has exactly one responsibility. Keeping these responsibilities separate — rather than one monolithic "do stuff" task type — is a deliberate engineering choice, and it's the same separation-of-concerns principle you'd apply designing any workflow engine.

| Task | Responsibility |
|---|---|
| Standard | Execute work |
| Conditional | Make decisions |
| Data Collection | Gather human input |
| Section Header | Organize workflow |

The payoff of this separation shows up the moment a playbook grows past a handful of steps: you can scan a playbook's task types alone and know, without reading any logic, which steps are pure automation (Standard), which steps branch (Conditional), which steps stop and wait on a person (Data Collection), and which steps are just visual scaffolding (Section Header). That scannability is what makes a large playbook maintainable by someone other than its original author.

---

## Standard Tasks

### Purpose

A Standard Task executes a single unit of work — running a script, calling an integration, performing an enrichment lookup, or invoking a response action. It is the "verb" of a playbook: the task type that actually *does* something rather than deciding or asking.

### Architecture

```
Input (Context)
      │
      ▼
Standard Task
(executes one action:
 script / integration / API call)
      │
      ▼
Output (written back to Context)
```

A Standard Task is stateless with respect to the playbook's control flow — it doesn't branch and it doesn't pause for a human. It takes input, does one thing, produces output, and hands control to the next task.

### Inputs

- Context values from prior tasks (e.g., an IP address extracted earlier)
- Static configuration values (e.g., a fixed API endpoint or threshold)
- List values (e.g., a critical-server list used for a lookup — see Chapter 7)

### Outputs

- Structured data written back into the playbook's context, available to every downstream task
- Optionally, a status/success indicator consumed by a following Conditional Task

### Examples

- Query an IP address against a threat intelligence source
- Run a script from the Scripts Library (Chapter 17) — e.g., `check_file_signature`
- Isolate an endpoint
- Send a Slack or email notification
- Enrich a hash against WildFire

### Automation

Standard Tasks are, by design, the fully-automated part of a playbook. There is no human-in-the-loop concept inside a Standard Task itself — if a step requires human judgment, that's a signal it should be a Data Collection or Conditional Task instead, not a Standard Task with a comment telling an analyst to "check before running."

### SOC Examples

- A phishing playbook's first several tasks (extract URLs, check reputation, sandbox the attachment) are all Standard Tasks — pure technical work, no decisions yet.
- A ransomware playbook's containment tasks (isolate endpoint, kill malicious process, collect memory image) are Standard Tasks executed after a Conditional Task has already determined containment is warranted.

### Production Workflow

Standard Tasks are where most of a playbook's *technical* risk lives, because they're the tasks that actually touch production systems. Two Standard Tasks that look similar in the playbook canvas — "look up a reputation score" versus "isolate an endpoint" — carry wildly different blast radii. Treat any Standard Task with a write/remediation side effect (see Chapter 17's read-vs-write split) as needing the same review rigor as the script or integration it invokes.

### Best Practices

- Keep each Standard Task doing exactly one thing — resist the urge to cram multiple actions into one script-backed task.
- Name Standard Tasks by what they *do*, not by their position in the playbook ("Check IP Reputation," not "Task 4").
- Document expected output shape so downstream Conditional Tasks can be written against a known contract.

### Common Mistakes

- Using a Standard Task to perform an action that actually requires a business decision (e.g., auto-isolating a server) without a preceding Conditional gate.
- Silent failure handling — not accounting for what happens downstream if a Standard Task's integration call fails or times out.

---

## Conditional Tasks

### Purpose

A Conditional Task is the decision engine of a playbook. It evaluates a condition against context data and branches the playbook down one of two or more paths. This is where a playbook stops being a straight-line script and becomes something that can respond differently to different situations.

### Decision Engine — IF/ELSE Logic

```
        Context Value
              │
              ▼
       Conditional Task
     (evaluate expression)
              │
      ┌───────┴───────┐
      ▼               ▼
    True            False
      │               │
 Path A Tasks    Path B Tasks
```

### Risk-Based Branching and Confidence Scoring

Conditional Tasks are how a playbook encodes risk tolerance directly into its logic rather than relying on an analyst to apply that judgment on every single run. A well-designed condition doesn't just check "is this malicious" — it checks confidence *and* business context together:

```
Confidence Score
        │
        ▼
High Confidence AND Non-Critical Asset?
        │
   ┌────┴────┐
  Yes        No
   │          │
Auto        Route to
Response    Analyst Review
```

### Business Logic

Conditional Tasks are also where business rules — not just technical signals — enter the playbook. "Is this asset on the Critical Servers list?" (Chapter 7) or "Is this user a VIP?" are Conditional checks that route technical automation through a business-awareness filter before anything disruptive happens.

### Examples

- `Malicious == True` → branch to remediation path vs. close-case path
- `Asset Criticality == Critical` → branch to human-approval path vs. auto-response path
- `Confidence Score > 90 AND Asset Criticality != Critical` → auto-isolate; otherwise route to analyst

### Production Examples

The Chapter 2 "Suspicious PowerShell on a Critical Server" Work Plan is, underneath, a chain of Conditional Tasks: *Critical Asset?* → *Known Administrator?* → *Approved?* → *Approval to Isolate?* Each diamond in that diagram is a Conditional Task in the underlying playbook. This is the clearest illustration in the handbook so far of Conditional Tasks doing exactly their one job: choosing the path, not doing the work.

### Best Practices

- Put the highest-risk branch condition first and make it explicit — don't bury a critical-asset check three conditions deep.
- Prefer combining confidence *and* business-impact signals in a single condition over chaining many shallow conditions — it's easier to audit one well-formed expression than five nested ones.
- Every Conditional Task should have a clearly named path for each branch (not "Yes/No" alone) so the playbook reads like a decision tree, not a coin flip.

---

## Data Collection Tasks

### Purpose

A Data Collection Task pauses playbook execution and asks a human for input — approval, a decision, or missing information the automation can't determine on its own. This is the task type that formally encodes "automation stops here; a person decides."

### Human Approval / Business Validation

```
Playbook Execution
        │
        ▼
Data Collection Task
(pause, notify analyst/IT/user)
        │
        ▼
  Human Responds
        │
        ▼
Playbook Resumes
(response is now context)
```

### IT Confirmation / User Interaction

Data Collection Tasks aren't limited to SOC analysts — they can route a question to IT ("was this software deployment approved?"), to an end user ("did you intend to run this script?"), or to a system owner ("is this server in a change window?"). The Chapter 2 phishing and ransomware Work Plans both use this pattern: "Contact User," "Contact IT," "Analyst Reviews Suggestions" are all Data Collection Tasks in playbook form.

### When to Interrupt Automation

This is the single most important design judgment in this chapter. A Data Collection Task should exist wherever a decision has genuine business uncertainty attached — not wherever an engineer feels nervous about automation in general. Concretely, insert a Data Collection Task when:

- The action has a meaningful blast radius (isolating a production server, not looking up a reputation score).
- Confidence is not high enough to act automatically.
- The correct action depends on information Cortex cannot see (a change window, a business justification, an ongoing approved project).

Do **not** insert a Data Collection Task purely to "be safe" on a low-risk, high-confidence, reversible action — that just reintroduces the manual bottleneck automation was built to remove.

### Approval Workflows

```
Automated Enrichment
        │
        ▼
Risk Determined High
        │
        ▼
Data Collection Task
"Approve endpoint isolation?"
        │
   ┌────┴────┐
  Yes         No
   │           │
Isolate    Continue
Endpoint   Monitoring
```

### Production Examples

- Ransomware playbook: "Analyst Reviews [Remediation] Suggestions" before remediation executes — evidence and technical containment are automated, but the remediation *action* itself waits on a human.
- Critical-server PowerShell playbook: "Contact IT... Approved?" — a Data Collection Task addressed to IT, not the SOC analyst, because the answer requires knowledge Cortex doesn't have.

### Best Practices

- Word the question precisely enough that a tired analyst at 3 a.m. can answer it without needing to re-derive context the playbook already gathered.
- Attach the relevant enrichment output to the Data Collection prompt — don't make the human re-fetch what automation already collected.
- Set a reasonable timeout/escalation path for unanswered Data Collection Tasks — a playbook that waits forever on a human is a playbook that can silently stall an active incident.

---

## Section Headers

### Purpose

A Section Header carries no execution logic — it exists purely to organize a playbook visually and structurally. As playbooks grow past a dozen or so tasks, an unstructured task list becomes as hard to navigate as an undocumented, unindented codebase.

### Organization / Readability

```
Playbook
 │
 ├── [Section] Enrichment
 │     ├── Standard: IP Reputation Lookup
 │     └── Standard: WildFire Sandbox
 │
 ├── [Section] Decision
 │     └── Conditional: Malicious?
 │
 ├── [Section] Human Review
 │     └── Data Collection: Analyst Approval
 │
 └── [Section] Response
       ├── Standard: Isolate Endpoint
       └── Standard: Collect Memory
```

### Large Playbooks / Documentation / Maintenance

Section Headers do for a playbook what headings do for this handbook — they let an engineer skim the shape of the automation before reading task-by-task, and they give you natural anchor points when documenting or reviewing a playbook with someone unfamiliar with it.

### Best Practices

- Group tasks by *phase* (Enrichment, Decision, Human Review, Response) rather than by task type — phase-based grouping is what actually helps someone reason about the investigation.
- Keep section names consistent across playbooks (e.g., always "Response," never sometimes "Remediation" and sometimes "Response") so engineers build pattern recognition across the whole Scripts/Playbook library.

---

## Designing Production Playbooks

Production automation should not blindly execute destructive actions. Every production playbook that ends in a disruptive response action should route through the same shape:

```
    Alert
      │
      ▼
Threat Intelligence
      │
      ▼
  Risk Score
      │
      ▼
Asset Criticality
      │
      ▼
Business Context
      │
      ▼
Conditional Decision
      │
      ▼
   Low Risk?
      │
  ┌───┴────┐
 Yes        No
  │          │
 Auto     Analyst
Response  Approval
```

Notice this is the SCDS framework applied end to end: Standard Tasks gather technical signal (Threat Intelligence, Risk Score), a Conditional Task fuses that signal with business context (Asset Criticality) into a single decision, and — only on the branch where confidence is insufficient or the asset is sensitive — a Data Collection Task hands control to a human. Section Headers hold the whole thing together so the next engineer who opens this playbook can see that shape in five seconds.

## The Engineering Principle

```
   Automation
       │
Handles Technical Certainty
───────────────────────────
    Analyst
       │
Handles Business Uncertainty
```

That sentence captures the essence of Cortex playbook design. The four task types exist to make that division explicit and enforceable in the playbook's structure itself — not as a policy written in a wiki somewhere, but as the literal shape of the automation: Standard and Conditional Tasks carry the technical-certainty work, Data Collection Tasks are the only place business-uncertainty work is allowed to live, and Section Headers keep the boundary between them legible as the playbook grows.

## Key Takeaways

- Every Cortex playbook is built from exactly four task types: Standard, Conditional, Data Collection, Section Header (SCDS).
- Standard Tasks do the work. Conditional Tasks choose the path. Data Collection Tasks ask the human. Section Headers organize everything.
- Insert Data Collection Tasks where business uncertainty exists — not as a blanket safety habit that reintroduces manual bottlenecks.
- Conditional Tasks should fuse technical confidence with business context (asset criticality, VIP status) rather than evaluating either in isolation.
- A well-designed production playbook is legible as SCDS at a glance: what's automated, what branches, and where it hands off to a human should be obvious without reading task-level logic.
- The task-type separation exists to enforce one principle: automation handles technical certainty; the analyst handles business uncertainty.
