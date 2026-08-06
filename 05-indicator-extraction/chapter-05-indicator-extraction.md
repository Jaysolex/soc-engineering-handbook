# Chapter 5 — Indicator Extraction

**Depends on:** Chapter 2 — Cortex Work Plans, Chapter 3 — Playbook Task Types, Chapter 4 — Inputs, Outputs & Context

```
Platform
   ↓
Detection
   ↓
Context
   ↓
Indicator Extraction   ← YOU ARE HERE
   ↓
Threat Intelligence
   ↓
Decision
   ↓
Response
```

---

## Overview

Security investigations begin with evidence. That evidence exists as **Indicators of Compromise (IOCs)** — IP addresses, domains, URLs, file hashes, email addresses, usernames, file names, registry keys, process names.

**Indicator Extraction** is the process by which Cortex automatically discovers these values from alerts, logs, endpoint telemetry, and playbook outputs, then converts them into structured indicators that can be enriched, analyzed, and acted upon.

Without Indicator Extraction, automation would have nothing meaningful to investigate — Context (Chapter 4) can only hold and pass along values that were identified as indicators in the first place.

## Learning Objectives

- What an Indicator is
- Why Indicator Extraction exists
- Indicator lifecycle
- Inline Extraction
- Out-of-Band Extraction
- No Extraction
- Indicator enrichment
- Threat Intelligence integration
- Indicator Context
- Production investigation workflows
- SOC engineering best practices

---

## What Is an Indicator?

An Indicator is a piece of observable evidence that may be associated with malicious activity.

| Indicator Type | Example |
|---|---|
| IP Address | `185.100.87.14` |
| Domain | `evil-update.com` |
| URL | `hxxps://evil-update.com/payload` |
| SHA256 | `3f6e8f2b...` |
| File Name | `invoice.js` |
| Process | `wscript.exe` |
| User | `alice` |
| Host | `WIN-01` |

Indicators become the primary objects used throughout a Cortex investigation — everything downstream (enrichment, correlation, decisions, response) operates on indicators, not on the raw alert text they came from.

## Why Indicator Extraction Exists

A detection often contains multiple artifacts bundled into one alert. Take the running example from Chapter 4:

```yaml
title: Windows Script Host with Network Connection
```

The alert contains:

```
Process:          wscript.exe
Command Line:      wscript.exe invoice.js
User:              alice
Hostname:          WIN-01
Destination IP:    185.100.87.14
```

Instead of treating this as one block of text, Cortex extracts each artifact into its own indicator. Those indicators can then be enriched **independently** — the destination IP goes to threat intelligence, the user goes to Active Directory, the file name might go to a hash-reputation check — each on its own track, in parallel, rather than as one undifferentiated blob nothing can act on.

## Indicator Extraction Architecture

```
 Detection
     │
     ▼
 Raw Alert
     │
     ▼
Indicator Extraction
     │
     ├── IP
     ├── Domain
     ├── Hash
     ├── Process
     ├── User
     └── Host
     │
     ▼
Threat Intelligence
     │
     ▼
  Playbook
```

---

## Extraction Methods

### 1. Inline Extraction

Indicators are extracted immediately while the playbook is running.

```
Detection
    │
    ▼
Extract IP
    │
    ▼
Threat Intel Lookup
    │
    ▼
Continue Playbook
```

**Use when:**
- Immediate enrichment is required.
- Decisions depend on IOC reputation.
- Real-time response is needed.

### 2. Out-of-Band Extraction

Extraction occurs separately from playbook execution.

```
Detection
    │
    ▼
Case Created
    │
    ▼
Background Extraction
    │
    ▼
Indicators Stored
    │
    ▼
Future Playbooks Consume Them
```

**Useful when:**
- Processing large datasets.
- Heavy enrichment operations.
- Batch intelligence updates.

### 3. No Extraction

Sometimes indicators are intentionally ignored.

**Examples:**
- Internal IP addresses.
- Known trusted domains.
- Approved software paths.

No enrichment is necessary — and, per the Engineering Insight later in this chapter, deliberately skipping extraction on low-value/trusted values is what keeps enrichment fast and cheap for the indicators that actually matter.

## Indicator Lifecycle

```
Detection
    │
    ▼
Extract Indicator
    │
    ▼
 Normalize
    │
    ▼
   Enrich
    │
    ▼
Store in Context
    │
    ▼
 Decision
    │
    ▼
 Response
    │
    ▼
Case Closed
```

---

## Production Example — Windows Script Host Detection

```
Detection
    │
    ▼
wscript.exe
    │
    ▼
Network Connection
    │
    ▼
185.100.87.14
```

**Indicator Extraction:**

```
Host              →  WIN-01
User              →  alice
Process           →  wscript.exe
File              →  invoice.js
Destination IP    →  185.100.87.14
```

**Threat Intelligence:**

```
185.100.87.14
      │
      ▼
  AbuseIPDB
      │
      ▼
  Malicious
Confidence 97%
```

**Context becomes:**

```json
{
  "host": "WIN-01",
  "user": "alice",
  "process": "wscript.exe",
  "file": "invoice.js",
  "destination_ip": "185.100.87.14",
  "reputation": "Malicious",
  "confidence": 97
}
```

The playbook now has enough evidence — five extracted indicators plus one enrichment result — to continue making decisions using the Conditional/Data Collection pattern from Chapter 3.

## Production Example — Ransomware Detection

```
Sigma Rule Fires
        │
        ▼
Suspicious Encryption Activity
        │
        ▼
    Extract
        │
   ┌────┼────┬──────────┬─────────┬──────────────┐
   ▼    ▼    ▼          ▼         ▼              ▼
 Host  User SHA256  File Path  Process   Command Line
        │
        ▼
Threat Intelligence
        │
        ▼
   Hash Known?
        │
       YES
        │
        ▼
Malware Family Identified
        │
        ▼
    Conditional
   Critical Asset?
        │
       YES
        │
        ▼
Request IT Approval
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

This example demonstrates how multiple indicators — host, user, hash, file path, process, command line — are extracted and independently enriched before the playbook decides how to respond. Notice the shape is identical to Chapter 3's production playbook design (Threat Intelligence → Risk/Confidence → Asset Criticality → Conditional → human approval on the sensitive branch) — Indicator Extraction is simply what supplies the raw material that shape operates on.

---

## Indicator Enrichment

Extraction identifies the indicator. **Enrichment gives it meaning.**

```
    IP
     │
     ▼
185.100.87.14
     │
     ▼
Threat Intel
     │
     ▼
Known C2 Server
Confidence 98%
```

**Without enrichment:** `185.100.87.14` — just a string.

**With enrichment:**
```
Known Ransomware Infrastructure
Last Seen: Yesterday
Confidence: 98%
Family: BlackCat
```

The extracted indicator and the enriched indicator are the same object at two different points in the lifecycle — extraction gets you the value, enrichment gets you the *significance* of the value.

## Indicator Relationships

Indicators rarely exist alone.

```
   User
     │
     ▼
   Host
     │
     ▼
  Process
     │
     ▼
   File
     │
     ▼
   Hash
     │
     ▼
Destination IP
     │
     ▼
   Domain
```

The more relationships Cortex discovers between extracted indicators, the higher the investigation confidence — a malicious IP alone is a weak signal; a malicious IP *reached by an unsigned wscript.exe child process spawned from a phishing attachment on a finance workstation* is a strong one. Relationships are what turn a pile of indicators into a coherent story.

## Engineering Insight — Don't Enrich Everything

Do not enrich every extracted indicator.

```
500 Indicators
      │
      ▼
500 API Calls
```

That's the naive approach, and it doesn't scale. Instead:

```
  Extract
     │
     ▼
   Filter
     │
     ▼
Only High-Value Indicators
     │
     ▼
Threat Intelligence
```

This reduces:
- API usage
- Execution time
- False positives (low-value indicators enriched at scale tend to generate noisy, low-confidence matches)

## Best Practices

- Extract indicators as early as possible.
- Normalize before enrichment (consistent casing, defanged vs. live URL format, IPv4 vs. IPv6 representation, etc.).
- Avoid enriching trusted assets — internal IPs, approved domains, known-good file paths.
- Reuse enriched indicators from Context (Chapter 4) rather than re-querying.
- Prioritize external-facing indicators (destination IPs, domains, URLs, hashes) over purely internal artifacts.
- Filter before making expensive API calls.

## Common Mistakes

- Enriching every IOC without filtering.
- Repeating enrichment for the same indicator instead of reading it from Context.
- Ignoring indicator relationships — treating each IOC as independent evidence rather than part of one chain.
- Treating raw strings as indicators without normalization.
- Performing threat intelligence lookups *before* extraction — enriching against unstructured alert text instead of a clean, extracted value.

## Key Takeaways

- Indicators are structured evidence extracted from alerts.
- Extraction converts raw detections into actionable security objects.
- Enrichment adds intelligence to indicators.
- Context stores enriched indicators for reuse (Chapter 4).
- Indicator Extraction enables automated investigation, correlation, and response — automation cannot enrich or make decisions on raw, unstructured alert text.
- A mature SOC engineer treats indicators as reusable investigation objects, not just values inside an alert.

---

## Chapter Bridge

From this chapter onward, the handbook shifts from understanding individual Cortex features to engineering efficient investigations. Every later chapter — Filters & Transformers, Lists, Jobs, Automation, SOAR — assumes indicators have already been extracted and normalized. This chapter is the bridge between detection and intelligent automation.
