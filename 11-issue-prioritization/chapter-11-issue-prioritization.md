# Chapter 11 — Issue Prioritization

**Depends on:** Chapter 10 — Working with Issues

```
Detections
    ↓
  Issues
    ↓
Issue Prioritization   ← YOU ARE HERE
    │
    ├── Starring Rules
    ├── Featured Fields
    ├── Severity
    ├── Tags
    └── Status
    ↓
  Cases
    ↓
Analyst Investigation
    ↓
Response
```

---

## Overview

Enterprise Security Operations Centers (SOCs) receive thousands of alerts every day. Not every alert represents a real threat, and not every threat requires the same level of attention.

**Issue Prioritization** allows Cortex to identify the most important Issues so analysts can focus on high-risk assets, users, and attacks first. Rather than relying only on severity, Cortex provides multiple prioritization mechanisms:

- Starred Issues
- Featured Fields
- Severity
- Tags
- Status
- Automation Rules

These mechanisms work together to reduce alert fatigue and ensure analysts spend their time investigating the most critical events.

## Learning Objectives

- Why Issue Prioritization is necessary
- Starred Issues
- Starring Rules
- Featured Fields
- Featured Hosts, Featured Users, Featured IP Addresses
- Tags
- Status
- Severity
- Priority workflow
- Production examples
- Best practices

## Why Prioritization Matters

```
   Today
     │
     ▼
12,000 Issues
     │
     ▼
  Analyst
```

Investigating every alert equally is impossible. Instead:

```
12,000 Issues
      │
      ▼
Prioritization
      │
      ▼
   Critical
      │
      ▼
    High
      │
      ▼
   Medium
      │
      ▼
     Low
      │
      ▼
   Analyst
```

The goal is to surface the events that matter most.

## Issue Prioritization Components

```
   Issue
     │
     ▼
  Severity
     │
     ▼
  Starred?
     │
     ▼
Featured Asset?
     │
     ▼
    Tags
     │
     ▼
   Status
     │
     ▼
  Priority
```

Every component contributes additional context to help analysts decide what to investigate first.

---

## Starred Issues

A Starred Issue is an Issue that has been explicitly marked as high priority. When an Issue is starred:

- It becomes easier to locate in the Issues table.
- The associated Case is also prioritized.
- Analysts can filter and sort by starred status.

Starring does not change the detection. **It changes its operational priority.**

## Starring Rules

Rather than starring Issues manually, Cortex allows organizations to create Starring Rules — a Starring Rule automatically stars Issues that match predefined criteria.

```
Severity = Critical
     AND
Category = Malware
        │
        ▼
Automatically Star Issue
```

This ensures that high-risk events are highlighted immediately.

### Rule Evaluation

```
   Issue
     │
     ▼
Matches Rule?
     │
    YES
     │
     ▼
 Star Issue
     │
     ▼
Prioritize Case
```

**Remember:** Issue Exclusion Rules take precedence over Starring Rules. If an Issue matches both, the exclusion wins.

---

## Featured Fields

Featured Fields provide another layer of prioritization. Instead of starring every Issue individually, analysts can define important organizational values. Supported Featured Fields include:

- Hosts
- Users
- IP Addresses

When an Issue contains one of these values, Cortex highlights it automatically.

### Featured Hosts

```
Featured Hosts
   DC01
   DC02
   ERP01
   PAYROLL01
```

Now imagine:
```
PowerShell Detection
        │
        ▼
       Host
        │
        ▼
       DC01
```

Cortex immediately flags the Issue because the affected host belongs to the Featured Host list.

### Featured Users

```
Featured Users
    CEO
    CFO
    CISO
SOC Manager
```

Now:
```
Impossible Travel
        │
        ▼
       User
        │
        ▼
       CEO
```

Even if the severity is Medium, the analyst immediately recognizes the event as high priority because it affects a high-value user.

### Featured IP Addresses

```
Featured IPs
Payment Gateway
VPN Concentrator
Domain Controller
```

**Detection:**
```
Network Scan
      │
      ▼
Destination
      │
      ▼
Payment Gateway
```

The Issue becomes significantly more important because it targets a critical infrastructure component.

---

## Production Example 1 — Windows Script Host Detection

Sigma rule detects `wscript.exe` → Network Connection → **Host: DC01**

```
Check Featured Hosts
        │
        ▼
      Found?
        │
       YES
        │
        ▼
   Star Issue
        │
        ▼
Increase Priority
        │
        ▼
  Notify SOC
```

Even if the detection is only High severity, the analyst immediately sees it because it affects a critical server.

## Production Example 2 — Ransomware Detection

**Detection:** Suspicious File Encryption → **Host: PAYROLL01**

```
  Featured Host?
        │
       YES
        │
        ▼
   Star Issue
        │
        ▼
Automation Rule
        │
        ▼
  Create Case
        │
        ▼
Notify Incident Commander
        │
        ▼
 SOC Investigation
```

This ensures ransomware affecting payroll receives immediate attention.

---

## Severity

Severity represents the estimated technical impact of an Issue.

**Typical values:** Informational, Low, Medium, High, Critical.

Severity helps analysts understand how serious an event appears from a technical perspective. **However, severity alone should never determine investigation priority — business context also matters**, as the examples above demonstrate.

## Tags

Tags provide additional metadata. Examples: `PCI`, `Finance`, `Production`, `Cloud`, `Ransomware`, `VIP User`.

Tags make searching, filtering, and reporting significantly easier.

## Status

Issues progress through different operational states:

```
   New
    │
    ▼
In Progress
    │
    ▼
 Resolved
```

Status tracks investigation progress rather than technical risk.

---

## Putting It All Together

```
Detection
    │
    ▼
  Issue
    │
    ▼
 Severity
    │
    ▼
Featured Host?
    │
    ▼
Star Rule?
    │
    ▼
VIP User?
    │
    ▼
  Tags
    │
    ▼
 Status
    │
    ▼
Priority Queue
```

Every layer contributes to the analyst's decision.

## Enterprise Workflow

```
Ransomware Detection
        │
        ▼
      Issue
        │
        ▼
Severity = High
        │
        ▼
Featured Host = YES
        │
        ▼
VIP User = NO
        │
        ▼
Star Rule = YES
        │
        ▼
Automation Rule
        │
        ▼
  Create Case
        │
        ▼
SOC Notification
        │
        ▼
Investigation Begins
```

Notice: priority isn't determined by severity alone. Business importance changes everything.

## Engineering Insight

One lesson from mature SOCs: **technical severity and business priority are not always the same.**

```
   High                    Medium
     │                        │
     ▼                        ▼
Developer Laptop     Domain Controller
```

Most organizations investigate the Domain Controller first — this is the exact same "technical certainty vs. business uncertainty" distinction from Chapter 3, now applied to triage ordering instead of automation gating.

---

## Best Practices

- Use Starring Rules instead of manual starring whenever possible.
- Maintain Featured Field lists regularly.
- Prioritize critical assets and VIP users.
- Combine severity with business context.
- Use Tags consistently across investigations.
- Review prioritization rules periodically.

## Common Mistakes

- Treating severity as the only priority metric.
- Forgetting to maintain Featured Field lists.
- Overusing Starring Rules (if everything is starred, nothing is).
- Ignoring business-critical assets.
- Using inconsistent Tags.

## Key Takeaways

- Issue Prioritization helps analysts focus on the most important events, not just the most numerous ones.
- Starring Rules automatically prioritize matching Issues; Issue Exclusion Rules take precedence over them.
- Featured Fields highlight critical organizational assets (Hosts, Users, IPs) — Starring prioritizes specific Issues, while Featured Fields prioritize Issues based on business context.
- Severity represents technical impact, not business impact.
- Tags improve organization and reporting.
- Mature SOCs combine technical severity with business context when deciding investigation priority.

---

## Chapter Bridge — The SOC Triage System

```
Thousands of Issues
        │
        ▼
Technical Severity
        │
        ▼
Business Context
        │
        ▼
 Starred Rules
        │
        ▼
Featured Fields
        │
        ▼
Analyst Queue
        │
        ▼
Investigation
```

This is how enterprise SOCs reduce alert fatigue. Rather than asking "Which alert is most severe?", they ask "Which alert represents the greatest business risk?" Cortex's prioritization features are designed to answer that question automatically — and once an Issue clears this triage, it typically becomes (or joins) a Case, the subject of Chapter 12.
