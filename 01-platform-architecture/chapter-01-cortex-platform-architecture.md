# Chapter 1 — Cortex Platform Architecture

**Version:** 1.0 (Living Document)
**Author:** Solomon James
**Purpose:** Operational Engineering Reference for Cortex XDR/XSIAM Security Operations

---

## 1. Purpose

Cortex is not a Security Information and Event Management (SIEM), an Endpoint Detection and Response (EDR), or a Security Orchestration, Automation and Response (SOAR) platform in isolation. It is an operational security platform that combines endpoint telemetry, incident management, investigation, automation, orchestration, and response into a unified workflow.

Its purpose is to reduce the time between detection, investigation, decision, and response, while maintaining analyst control over business-critical actions.

Unlike traditional SOC workflows where analysts constantly switch between multiple consoles, Cortex attempts to centralize these capabilities into a single operational environment.

## 2. The Problem Cortex Solves

Traditional SOC operations often look like this:

```
Firewall Alert
      │
      ▼
  Open SIEM
      │
      ▼
  Open EDR
      │
      ▼
Open Active Directory
      │
      ▼
Open Threat Intelligence
      │
      ▼
Open Ticketing System
      │
      ▼
  Contact IT
      │
      ▼
 Contain Host
      │
      ▼
 Write Notes
      │
      ▼
 Close Ticket
```

Every investigation requires switching between tools, manually collecting evidence, and repeating the same investigative steps.

This results in:

- Long Mean Time to Respond (MTTR)
- Alert fatigue
- Inconsistent investigations
- Human error
- Slow containment
- Poor documentation
- Limited automation

Cortex changes the workflow:

```
Detection
    │
    ▼
  Issue
    │
    ▼
  Case
    │
    ▼
Playbook
    │
    ▼
Automation
    │
    ▼
Analyst Decision
    │
    ▼
Response
    │
    ▼
Closure
```

Everything is connected through a shared operational context.

## 3. Cortex's Position Inside a SOC

One of the most important concepts is understanding where Cortex sits within an enterprise security architecture.

```
                        USERS
                          │
                          ▼
                     Endpoints
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
Windows Logs        Linux Logs         Cloud Logs
      │                   │                   │
      └───────────────────┼───────────────────┘
                          │
                          ▼
                    SIEM Detection
                          │
                   Detection Rule Fires
                          │
                          ▼
                 Issue Created in Cortex
                          │
                          ▼
             Cortex Correlates Evidence
                          │
                          ▼
               Causality + Investigation
                          │
                          ▼
                    Playbook Executes
                          │
      ┌─────────────┬─────────────┬──────────────┐
      ▼              ▼             ▼
 Threat Intel      Lists      Integrations
      │              │             │
      └─────────────┼──────────────┘
                     ▼
              Decision Logic
                     │
                     ▼
              Analyst Review
                     │
                     ▼
            Response Actions
                     │
                     ▼
              Action Center
                     │
                     ▼
               Case Closure
```

## 4. Core Components

The Cortex ecosystem is composed of several interconnected building blocks.

### Issues

An Issue represents an individual security event or suspicious activity.

Examples:
- Malware detected
- Suspicious PowerShell
- Impossible travel
- Credential theft
- Ransomware behavior

**Think:** *"Something happened."*

### Cases

A Case groups related Issues together into a single investigation.

Instead of investigating twenty alerts individually, analysts investigate one Case containing all related evidence.

**Think:** *"Everything related to this attack."*

### Playbooks

Playbooks automate repetitive investigative and response actions.

A playbook is the orchestration engine of Cortex.

**Think:** *"What should happen automatically?"*

### Automation Rules

Automation Rules determine when a playbook starts.

Example:

```
If
    Severity = Critical
    AND
    Detection = Malware
↓
Run Malware Response Playbook
```

### Jobs

Jobs execute playbooks independently of incidents. They are proactive rather than reactive.

Examples:
- Nightly IOC synchronization
- Threat intelligence updates
- Daily vulnerability checks
- Asset synchronization

**Think:** Scheduled automation.

### Lists

Lists store organizational knowledge.

Examples:
- VIP users
- Critical servers
- Allowed software
- Blocked domains
- Internal IP ranges

Lists prevent repeated API lookups and ensure consistent automation decisions.

### Scripts

Scripts perform technical actions on endpoints.

Examples:
- Delete malware
- Collect logs
- Search registry
- Gather browser history
- Validate software versions

Scripts are reusable operational tools.

### Response Actions

Response Actions change the environment.

Examples:
- Isolate endpoint
- Kill process
- Quarantine file
- Run malware scan
- Retrieve memory

Response Actions affect production systems and therefore often require analyst judgment.

## 5. High-Level Data Flow

Understanding data flow is essential. Everything in Cortex builds on this pipeline.

```
   Logs
    │
    ▼
 Detection
    │
    ▼
  Issue
    │
    ▼
  Case
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
    │
    ▼
Action Center
    │
    ▼
 Closure
```

Notice that nothing exists independently. Each component feeds the next.

## 6. Operational Philosophy

Automation should never replace analyst judgment.

Automation should eliminate repetitive work while preserving human decision making where business impact exists.

Production decisions should always consider:
- Confidence level
- Asset criticality
- Business impact
- Potential blast radius
- Evidence preservation
- Recovery cost

Automation accelerates investigations. Analysts remain responsible for risk decisions.

## 7. Production Example

Imagine Defender detects ransomware. The workflow becomes:

```
Windows Endpoint
      │
      ▼
Ransomware Detected
      │
      ▼
Detection Rule Fires
      │
      ▼
 Issue Created
      │
      ▼
  Case Created
      │
      ▼
 Playbook Starts
      │
      ▼
Threat Intelligence
      │
      ▼
  Asset Lookup
      │
      ▼
Critical Server?
      │
     Yes
      │
      ▼
Pause Automation
      │
      ▼
Notify Senior SOC
      │
      ▼
 Analyst Reviews
      │
      ▼
Approve Isolation
      │
      ▼
Endpoint Isolated
      │
      ▼
Evidence Collected
      │
      ▼
 Remediation
      │
      ▼
 Case Resolved
```

Notice that the automation intentionally pauses before isolating a critical asset. Business impact determines the response.

## 8. Senior SOC Mindset

One of the biggest mindset shifts is moving from:

> "Can this be automated?"

to:

> "Should this be automated?"

Every automation introduces operational risk.

Experienced analysts balance:
- Detection confidence
- Business continuity
- Asset importance
- Threat severity
- Evidence requirements

Automation handles repetitive technical work. Humans make business decisions.

## 9. Engineering Principles

Throughout this handbook we will repeatedly apply the following principles:

1. Context is everything. Data without context creates noise.
2. Automate repetitive work, not judgment.
3. Evidence before remediation.
4. Protect business operations while protecting security.
5. Design for repeatability.
6. Everything should be observable, auditable, and reversible where possible.
7. Production automation must always consider risk and asset criticality.

## 10. Key Takeaways

- Cortex unifies investigation, automation, and response.
- Issues represent events; Cases represent investigations.
- Playbooks orchestrate workflows.
- Jobs automate recurring tasks.
- Lists provide persistent business knowledge.
- Scripts execute technical actions on endpoints.
- Response Actions modify production systems.
- Action Center records operational execution.
- Analyst judgment remains essential for high-risk decisions.

Every Cortex feature ultimately supports a single objective: **reduce Mean Time to Detect (MTTD), reduce Mean Time to Respond (MTTR), and improve security outcomes without sacrificing business continuity.**
