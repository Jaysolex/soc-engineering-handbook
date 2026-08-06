# Chapter 7 — Lists

**Depends on:** Chapter 2 — Cortex Work Plans, Chapter 3 — Playbook Task Types, Chapter 4 — Inputs, Outputs & Context, Chapter 5 — Indicator Extraction, Chapter 6 — Filters & Transformers

```
Platform
   ↓
Detection
   ↓
Indicators
   ↓
Context
   ↓
Filters
   ↓
Lists   ← YOU ARE HERE
   ↓
Decision
   ↓
Automation
   ↓
Response
```

---

## Overview

Lists are Cortex's **persistent knowledge database**.

Unlike Context (Chapter 4), which exists only while a playbook is running, Lists persist across every playbook, investigation, job, and automation rule. They allow Cortex to remember information *between* investigations.

Instead of repeatedly querying external systems or rebuilding the same data during every investigation, Cortex stores reusable information inside Lists for fast lookup and decision-making. Lists dramatically improve automation speed, consistency, and reliability.

## Learning Objectives

- What Lists are
- Why Lists exist
- Static vs. Dynamic Lists
- List lifecycle
- List structure
- Reading Lists
- Writing to Lists
- CRUD operations and CLI commands
- Lists vs. Context
- Lists vs. Threat Intelligence
- Production use cases
- Best practices

---

## What Is a List?

A List is a persistent collection of structured information stored inside Cortex.

Unlike Context, which disappears when the playbook ends, Lists remain available until someone changes or deletes them. Think of a List as Cortex's long-term memory.

## Context vs. Lists

| Context | Lists |
|---|---|
| Temporary | Persistent |
| Lives inside one playbook | Shared across Cortex |
| Deleted when playbook finishes | Remains available indefinitely |
| Investigation memory | Organizational memory |

```
Context
   │
   ▼
Temporary Memory
   │
   ▼
Investigation Ends
   │
   ▼
  Deleted
```

```
Lists
   │
   ▼
Persistent Memory
   │
   ▼
Tomorrow → Next Week → Next Month
   │
   ▼
Still Available
```

## Why Lists Exist

Without Lists, every investigation must query Active Directory, Threat Intelligence, Asset Inventory, VIP users, and Critical Assets — **again, and again, and again.**

Lists eliminate repetitive lookups.

## Common Production Lists

| List | Purpose |
|---|---|
| Critical Assets | Production servers |
| VIP Users | Executives |
| Allowed Applications | Approved software |
| Known Good Hashes | Trusted files |
| Blocked Hashes | Malicious files |
| Approved Domains | Business domains |
| Internal IPs | Corporate networks |
| Emergency Contacts | SOC escalation |

## List Architecture

```
 Playbook
     │
     ▼
 Read List
     │
     ▼
 Decision
     │
     ▼
Update List
     │
     ▼
Future Investigations
```

Lists become smarter over time — each investigation that updates a List makes the next investigation that reads it faster or more informed.

## List Lifecycle

```
 Create
    │
    ▼
Populate
    │
    ▼
  Read
    │
    ▼
 Update
    │
    ▼
 Reuse
    │
    ▼
Archive
```

Lists evolve continuously.

---

## Static Lists

Static Lists change only when an administrator edits them.

**Examples:**

*VIP Users:* CEO, CFO, CISO
*Critical Servers:* DC01, SQL01, Backup01

These rarely change.

## Dynamic Lists

Dynamic Lists update automatically — usually through Jobs (Chapter 8).

```
Threat Feed
     │
     ▼
Job Runs Every Hour
     │
     ▼
 Update List
     │
     ▼
Latest Malicious IPs
```

This is exactly why Jobs exist: Jobs keep Lists fresh.

## Relationship Between Jobs and Lists

```
Threat Feed
     │
     ▼
    Job
     │
     ▼
Update List
     │
     ▼
Playbook Reads List
     │
     ▼
 Decision
```

The playbook never contacts the threat feed directly. It simply reads the updated List. This reduces internet lookups, API usage, and execution time.

---

## CRUD Operations and CLI Commands

Every List supports the same four operations any persistent data store supports — **Create, Read, Update, Delete** — and Cortex exposes each as a dedicated CLI command a playbook task can call:

| Operation | CLI Command | Purpose |
|---|---|---|
| Create | `createList` | Create a new, empty List |
| Read | `getList` | Retrieve the current contents of a List |
| Update (add) | `addToList` | Add one or more entries to an existing List |
| Update (remove) | `removeFromList` | Remove one or more entries from a List |
| Update (replace) | `setList` | Overwrite a List's entire contents |

**Example — a Job maintaining a dynamic malicious-IP List:**

```
Threat Feed Pulled
        │
        ▼
getList("Known_Malicious_IPs")
        │
        ▼
Diff against new feed data
        │
        ▼
addToList("Known_Malicious_IPs", [new IPs])
        │
        ▼
removeFromList("Known_Malicious_IPs", [expired IPs])
```

**Example — a playbook reading a List during investigation:**

```
getList("Critical_Assets")
        │
        ▼
Host in list?
        │
       YES
        │
        ▼
Require Analyst Approval
```

Knowing these five commands by name is what separates "I know Lists exist" from "I can build a Job that maintains one" — every dynamic List in this handbook (Chapter 8's threat-intel sync, Chapter 13's Critical Assets checks) is these same five operations underneath.

---

## Production Example 1 — Windows Script Host Detection

**Detection:** `wscript.exe` → `185.100.87.14`

```
Extract IP
     │
     ▼
Read Known Malicious IP List
     │
     ▼
   Found?
     │
    YES
     │
     ▼
High Confidence
     │
     ▼
Threat Intelligence
     │
     ▼
Analyst Review
```

If the IP already exists in the List, Cortex immediately increases confidence without waiting for another external lookup.

## Production Example 2 — Ransomware Sigma Rule

```
Sigma Rule
     │
     ▼
Suspicious File Encryption
     │
     ▼
Extract SHA256
     │
     ▼
Read Known Ransomware Hash List
     │
     ▼
   Match?
     │
    YES
     │
     ▼
Critical Asset?
     │
    YES
     │
     ▼
Data Collection — Contact IT
     │
     ▼
Approved Maintenance?
     │
     NO
     │
     ▼
Isolate Endpoint
     │
     ▼
Collect Memory
     │
     ▼
Run Remediation
```

If the hash is already stored in the ransomware List, the playbook can immediately treat the detection as high confidence.

## Production Example 3 — Critical Assets

One of the most common enterprise Lists.

**Critical Assets List:** DC01, DC02, ERP01, PAYROLL01

**Detection:** PowerShell on Host `DC01`

```
Read Critical Asset List
        │
        ▼
   Host Found?
        │
   ┌────┴────┐
  YES         NO
   │           │
Require    Automatic
Analyst    Response
Approval
```

Notice the response changes based on the List — the same PowerShell detection produces a different playbook path depending purely on whether the host is in this one List.

---

## Reading Lists

Playbooks constantly query Lists.

```
Current Host
     │
     ▼
Search List
     │
     ▼
  Found?
     │
    YES
     │
     ▼
 Continue
```

Reading Lists is extremely fast because the data is already stored inside Cortex — no external round-trip required.

## Updating Lists

Lists can be updated manually, through Jobs, through Playbooks, or through Automation.

```
Threat Intel
     │
     ▼
  New IOC
     │
     ▼
    Job
     │
     ▼
Update List
```

Tomorrow's investigations immediately benefit.

## Lists vs. Threat Intelligence

Many beginners confuse these.

**Threat Intelligence answers:** *Is this malicious?*
**Lists answer:** *What does my organization already know?*

**Threat Intelligence:** `185.100.87.14` → Known Malware

**List:** `185.100.87.14` → Seen Yesterday → Already Investigated → Known Campaign

Threat Intelligence provides global knowledge. Lists provide organizational knowledge.

## Engineering Insight

A mature SOC minimizes external dependencies.

**Instead of:**
```
Detection → Threat Intel API → Threat Intel API → Threat Intel API
```

**Use:**
```
Threat Feed → Job → Update List → Playbook Reads List
```

Much faster. Much cheaper. Much more reliable.

---

## Best Practices

- Keep Lists focused on one purpose.
- Separate static and dynamic Lists.
- Use Jobs to maintain dynamic Lists.
- Document every List.
- Remove stale entries regularly.
- Avoid duplicate Lists.

## Common Mistakes

- Using Context instead of Lists for persistent data.
- Storing unrelated data in one List.
- Never cleaning old entries.
- Calling external APIs instead of reading Lists.
- Forgetting that Lists represent organizational knowledge, not just a data dump.

## Key Takeaways

- Lists are Cortex's persistent memory.
- Context is temporary; Lists are permanent.
- Jobs keep dynamic Lists updated.
- Playbooks read Lists to make fast decisions.
- CRUD on Lists is five commands: `createList`, `getList`, `addToList`, `removeFromList`, `setList`.
- Lists reduce API calls and improve automation performance — they let playbooks make fast decisions using previously collected intelligence instead of re-querying external providers every run.
- Mature SOCs treat Lists as reusable organizational knowledge shared across every investigation.

---

## Chapter Bridge — Jobs and Lists Together

```
Threat Intelligence Feed
        │
        ▼
      Job Runs
        │
        ▼
   Update Cortex List
        │
        ▼
 SIEM Alert → Playbook
        │
        ▼
 Read Local List (Fast)
        │
        ▼
 Conditional Decision
        │
        ▼
 Analyst Review or Automated Response
```

This is a core SOC engineering pattern. Rather than every playbook reaching out to external threat intelligence providers, Jobs continuously synchronize intelligence into Lists, allowing playbooks to make rapid, consistent decisions using locally available data. That architecture — the subject of Chapter 8 — is one of the key reasons Cortex automations scale efficiently in enterprise environments.
