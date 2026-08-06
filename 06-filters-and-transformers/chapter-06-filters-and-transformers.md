# Chapter 6 — Filters & Transformers

**Depends on:** Chapter 2 — Cortex Work Plans, Chapter 3 — Playbook Task Types, Chapter 4 — Inputs, Outputs & Context, Chapter 5 — Indicator Extraction

```
Platform
   ↓
Detection
   ↓
Indicator Extraction
   ↓
Context
   ↓
Filters & Transformers   ← YOU ARE HERE
   ↓
Conditional Logic
   ↓
Response
```

---

## Overview

Not every piece of data collected during an investigation is useful. A single detection may produce dozens — or even hundreds — of indicators, logs, processes, users, IP addresses, registry keys, and files. Before Cortex can make intelligent decisions, this data must be filtered, transformed, and normalized into a format that playbooks can consume.

Filters and Transformers act as Cortex's data processing engine. Rather than collecting more information, they **refine** existing information, allowing downstream tasks to operate on clean, meaningful, and actionable data.

Without Filters and Transformers, every Conditional Task would need to process raw, inconsistent data — resulting in slower playbooks, unnecessary API calls, and poor automation decisions.

## Learning Objectives

- What Filters are
- What Transformers are
- Why Cortex separates filtering from transformation
- Data normalization
- JSON manipulation
- Context manipulation
- Chaining Filters and Transformers
- Production data pipelines
- Performance optimization
- Best practices

## Data Processing Pipeline

```
 Detection
     │
     ▼
 Raw Context
     │
     ▼
   Filters
     │
     ▼
Relevant Data
     │
     ▼
 Transformers
     │
     ▼
Normalized Context
     │
     ▼
Conditional Logic
     │
     ▼
 Automation
```

---

## What Is a Filter?

A Filter removes unwanted information. It answers one question: **what data should continue through the playbook?**

Filters reduce noise. They never create new information.

### Purpose of Filters

Filters exist to reduce processing. Instead of enriching everything, Cortex processes only the information that matters.

### Example

Your Windows Script Host detection (Chapters 4–5) produces:

```
Host             WIN-01
User             alice
Process          wscript.exe
Parent           explorer.exe
Destination IP   185.100.87.14
Source IP        10.0.1.24
Timestamp        13:42
```

The playbook only needs:

```
Destination IP
Process
Host
```

Everything else is ignored:

```
Raw Context
     │
     ▼
   Filter
     │
     ▼
Destination IP, Process, Host
```

### Production Example

Suppose the detection extracts 250 indicators.

```
250 Indicators
      │
      ▼
    Filter
      │
      ▼
External IPs Only
      │
      ▼
 12 Indicators
```

Instead of **250** threat intelligence lookups, Cortex performs **12**.

**Result:** faster execution, lower API usage, lower cost, less noise.

### Common Filters

- External IPs only
- Public domains only
- High-severity alerts
- Executable files only
- Windows endpoints only
- Critical assets only
- New indicators only

---

## What Is a Transformer?

A Transformer changes data into another format. Unlike Filters, **Transformers do not remove data — they reshape it.**

### Purpose of Transformers

Transformers prepare data for the next task. Examples:

- Convert JSON
- Convert timestamps
- Extract filenames
- Parse command lines
- Normalize hashes
- Convert arrays
- Change case
- Join values
- Split values

### Architecture

```
 Context
    │
    ▼
Transformer
    │
    ▼
Modified Context
```

### Example 1 — Extract Filename

Raw Context:
```json
{ "command_line": "wscript.exe invoice.js" }
```

Transformer: *Extract filename*

Output:
```json
{ "filename": "invoice.js" }
```

### Example 2 — Rename for Compatibility

Raw Context:
```json
{ "destination_ip": "185.100.87.14" }
```

Transformer output:
```json
{ "indicator": "185.100.87.14" }
```

Now every downstream task expects `indicator` instead of `destination_ip` — this is a common transformer use case when chaining a playbook into a generic enrichment sub-playbook that was written to expect a generic `indicator` field, not a detection-specific one.

## Filters vs. Transformers

| Filters | Transformers |
|---|---|
| Remove data | Change data |
| Reduce processing | Normalize processing |
| Reduce API calls | Improve compatibility |
| Remove noise | Prepare data |

## Engineering Pipeline

```
  Detection
      │
      ▼
Indicator Extraction
      │
      ▼
   Context
      │
      ▼
   Filter
      │
      ▼
Relevant Indicators
      │
      ▼
 Transformer
      │
      ▼
Normalized Indicators
      │
      ▼
Threat Intelligence
      │
      ▼
Conditional Task
      │
      ▼
  Response
```

---

## Production Example — Windows Script Host Detection

**Detection:** `wscript.exe invoice.js`

**Context:**
```json
{
  "host": "WIN-01",
  "user": "alice",
  "process": "wscript.exe",
  "command_line": "wscript.exe invoice.js",
  "destination_ip": "185.100.87.14",
  "source_ip": "10.0.0.5"
}
```

**Filter — Keep:** External IP, Process, Host. **Remove:** Internal IP.

**Transformer — Extract:** `invoice.js` (filename). **Normalize:** `185.100.87.14` → `indicator`.

```
Threat Intelligence
      │
      ▼
  Conditional
      │
      ▼
Continue Investigation
```

## Production Example — Ransomware Detection

Your ransomware Sigma rule detects:

```
Encrypted Files
      │
      ▼
  PowerShell
      │
      ▼
    SHA256
      │
      ▼
      Host
      │
      ▼
      User
      │
      ▼
Registry Changes
      │
      ▼
Network Connections
```

**Filter — Keep:** SHA256, External IP, Process, Host. **Ignore:** Temporary files, known backup process, trusted internal IPs.

**Transformer — Normalize:** Hashes.

```
Threat Intelligence
      │
      ▼
   Conditional
      │
      ▼
 High Confidence?
      │
      ▼
   Response
```

This significantly reduces unnecessary enrichment while focusing on the indicators that matter most.

---

## Performance Benefits

**Without Filters:**
```
400 Indicators
      │
      ▼
400 API Calls
```

**With Filters:**
```
400 Indicators
      │
      ▼
    Filter
      │
      ▼
 18 Indicators
      │
      ▼
 18 API Calls
```

The savings become substantial in enterprise environments processing millions of alerts.

## Engineering Insight — Order Matters

Filters should execute **before** expensive operations.

**Good order:**
```
Extract → Filter → Transform → Threat Intelligence → Conditional
```

**Poor order:**
```
Extract → Threat Intelligence → Filter
```

The second approach wastes API calls and increases latency — you've already paid the cost of enriching data you were about to discard.

## Best Practices

- Filter as early as possible.
- Transform only the required data.
- Normalize indicator formats.
- Avoid duplicate transformations.
- Keep transformations simple and reusable.
- Document every custom transformation.

## Common Mistakes

- Enriching before filtering.
- Filtering after API calls.
- Chaining unnecessary transformers.
- Transforming data multiple times.
- Using inconsistent Context names.

## Key Takeaways

- Filters decide what data survives. Transformers decide what surviving data looks like.
- Filters reduce workload. Transformers improve consistency.
- Together they create clean, reusable Context for automation.
- Efficient Filters and Transformers reduce API usage, improve playbook performance, and make downstream decision logic more reliable.
- Separating filtering from transformation keeps playbooks modular, efficient, and easier to maintain — each responsibility can be reasoned about, tested, and reused independently.

---

## Chapter Bridge

From this point onward, every Cortex playbook should be viewed as a data pipeline. The quality of your automation is determined not just by the detections you write, but by how effectively you filter, normalize, and prepare data before making decisions. This mindset is what distinguishes a detection engineer from someone who simply chains tasks together.
