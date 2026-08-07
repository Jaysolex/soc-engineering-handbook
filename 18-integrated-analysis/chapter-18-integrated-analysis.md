# Chapter 18 — Integrated Analysis

**Depends on:** Chapter 10 — Working with Issues, Chapter 12 — Working with Cases, Chapter 13 — Response Actions, Chapter 14 — Endpoint Response

```
   Logs
     ↓
  Issues
     ↓
Log Stitching
     ↓
Enhanced Issue
     ↓
Analytics Engine
     ↓
Behavior Analysis
     ↓
   Case
     ↓
Investigation
```

---

## Overview

Integrated Analysis is the intelligence layer of Cortex XDR.

Individual alerts (Issues) often contain only enough information to notify analysts that something suspicious occurred. Separately, logs contain detailed telemetry about system activity but lack the context to determine whether the activity is malicious.

Integrated Analysis combines both. Through **Log Stitching**, Cortex correlates Issues with related logs to create **Stitched Issues** containing complete attack context. The **Cortex Analytics Engine (CAE)** then applies machine learning and behavioral analytics to identify suspicious activity that traditional signature-based detections might miss.

The result is a complete attack narrative rather than isolated alerts.

## Learning Objectives

- Issues vs. Logs
- Log Stitching
- Stitched Issues
- Enhanced Endpoint Data (EED)
- Causality Chains
- Timeline View
- Cortex Analytics Engine (CAE)
- Machine Learning Baselines
- MITRE ATT&CK Coverage
- Analytics Detectors
- Behavioral Analytics
- Production workflows

## Why Integrated Analysis Exists

Suppose Cortex receives: `powershell.exe launched`. That alone tells us very little.

Now imagine Cortex also knows: which parent process launched PowerShell, which file it modified, which registry keys changed, which IP it contacted, which user executed it, and which DNS queries occurred afterward.

Now we understand the entire attack. Integrated Analysis combines these pieces automatically.

---

## Issues vs. Logs

This is the first concept to understand.

### Issues

Issues are alerts, generated immediately whenever Cortex detects suspicious behavior.

```
Behavioral Threat
       │
       ▼
 High Severity
       │
       ▼
 Issue Created
```

Issues are lightweight. They answer: *"Something suspicious happened."*

### Logs

Logs contain detailed telemetry: process creation, file writes, registry modifications, network connections, authentication events, firewall traffic.

Logs answer: *"Exactly what happened?"*

| Issues | Logs |
|---|---|
| Alerts | Telemetry |
| Small | Detailed |
| Immediate | Uploaded periodically |
| Detection focused | Investigation focused |

### Why Issues Alone Are Not Enough

Your Sigma rule detects `powershell.exe`. Issue: **PowerShell Execution**. Was it malicious? Unknown.

Now add logs:

```
PowerShell
    │
    ▼
Downloaded payload
    │
    ▼
Created Registry Key
    │
    ▼
Spawned cmd.exe
    │
    ▼
Connected to C2
    │
    ▼
Encrypted Files
```

Now we understand the attack.

---

## Log Stitching

One of Cortex's most important capabilities. Log Stitching means **correlating Issues with related logs to reconstruct the attack.**

```
   Issue
     +
   Logs
     │
     ▼
Log Stitching
     │
     ▼
Enhanced Issue
```

## Stitched Issues

After stitching, the Issue becomes much richer. Instead of `PowerShell Detected`, you now see:

```
Explorer → PowerShell → cmd → rundll32 → Network → Registry → File Changes
```

This is a **Stitched Issue** — the same concept introduced in Chapter 10, now with the mechanism behind it explained.

### What Stitched Issues Provide

**Causality Chain** — shows *who started what*:
```
explorer.exe → powershell.exe → cmd.exe → encryptor.exe
```

**Timeline** — shows *when everything happened*:
```
09:01   PowerShell
09:02   Registry Modified
09:03   DNS Query
09:04   Network Connection
09:05   File Encryption
```

---

## Enhanced Endpoint Data (EED)

How does Cortex get all this information? Through **Enhanced Endpoint Data (EED)**.

The Cortex XDR Pro agent continuously collects process, file, registry, and network activity. It compresses and uploads this data approximately every **five minutes**.

### What Does EED Collect?

| Category | Examples |
|---|---|
| Process Operations | Process creation, child process creation, process termination |
| File Operations | File reads, file writes, file deletion |
| Network Operations | Connections, listening ports, failed connections, source/destination IPs |
| Registry Operations | Registry creation, modification, deletion |

## Extended Threat Hunting (XTH)

Organizations requiring deeper visibility can enable the **Extended Threat Hunting (XTH)** add-on. Additional data includes Windows Event Logs, RPC Calls, System Calls, additional file activity, and additional macOS/Linux telemetry — this provides richer hunting capabilities.

## Identifying Stitched Issues

Inside the Issues table, look for causality-related fields such as **CGO Name**, **CGO Signer**, and **CID (Causality ID)**. If these fields contain values, the Issue has been stitched.

### Debug Issue

Analysts can verify stitching:

```
Alt + Right Click
       │
       ▼
 Debug Issue
       │
       ▼
    JSON
```

Two important fields:
- `matching_status` — shows whether stitching succeeded
- `agent_data_collection_status` — shows whether EED collection was enabled

---

## Cortex Analytics Engine (CAE)

After stitching, the Analytics Engine begins analyzing behavior. Think of CAE as Cortex's AI engine — instead of looking for known malware, it looks for **unusual behavior**.

### Behavioral Analytics

CAE first learns normal activity. This process is called **Profiling** or **Baseline Creation**.

```
User Alice
   │
Usually logs in
   │
8 AM, Toronto, Monday–Friday
```

This becomes the baseline.

### Detecting Anomalies

Now suppose: Alice logs in at 3 AM, from Russia, and downloads 20 GB.

```
   Normal
     │
     ▼
Current Activity
     │
     ▼
 Different
     │
     ▼
Issue Generated
```

### Machine Learning

CAE uses **unsupervised machine learning** — it does not require manually labeled attack data. Instead, it learns normal behavior automatically.

---

## MITRE ATT&CK Coverage

CAE detects behaviors across the MITRE ATT&CK framework: Execution, Persistence, Discovery, Lateral Movement, Command & Control, Exfiltration.

### How Analytics Detects Each Stage

| Stage | What It Looks For |
|---|---|
| Discovery | Port scanning, increased connections, failed connections |
| Lateral Movement | SMB usage, RDP, SSH, remote code execution |
| Command & Control | Beaconing, DNS anomalies, periodic connections, random DNS lookups |
| Exfiltration | Large outbound transfers, unusual upload volume, unexpected destinations |

## Analytics Detectors

The Analytics Engine is composed of specialized **Detectors**. Each detector monitors one attack behavior, builds its own baseline, and generates one type of Issue.

```
 Detector
    │
    ▼
DNS Beaconing
    │
    ▼
 Baseline
    │
    ▼
 Anomaly
    │
    ▼
  Issue
```

Each detector continuously updates its baseline as the environment changes.

## Enabling Analytics Engine

**Requirements:** minimum of 30 endpoints, approximately 2 weeks of data, at least 5 days of cloud audit logs (where applicable).

Until sufficient data exists, Analytics cannot build reliable baselines — this is the same reason any anomaly-detection system needs a training period before it's trustworthy.

---

## Production Example 1 — Windows Script Host Detection

Sigma rule detects `wscript.exe`. Issue: **Windows Script Host**.

Log Stitching adds:
```
Explorer → wscript.exe → PowerShell → HTTP Download → Registry Persistence → DNS Query → Network Connection
```

CAE detects:
```
Beaconing Pattern → Command & Control → Issue Raised
```

Now analysts have both the detection and the full attack context.

## Production Example 2 — Ransomware Detection

Sigma rule detects `encryptor.exe`. Issue: **Ransomware Activity**.

EED provides:
```
File Encryption → Shadow Copy Delete → Registry Changes → SMB Activity → DNS Queries → Outbound Traffic
```

Analytics Engine identifies:
```
Discovery → Lateral Movement → Exfiltration Attempt
```

Case now contains the complete ransomware attack chain.

---

## Engineering Insight

Think of Cortex as solving a puzzle.

```
   Issue                     Logs                    Log Stitching
     │                         │                            │
     ▼                         ▼                            ▼
One Puzzle Piece    Thousands of Puzzle Pieces      Completed Picture
```

Without stitching, analysts investigate alerts. With stitching, analysts investigate attacks.

## Best Practices

- Enable Enhanced Endpoint Data.
- Verify stitching status when investigations appear incomplete.
- Use Timeline before remediation.
- Review Causality Chains to identify root processes.
- Enable Analytics Engine only after sufficient training data exists.

## Common Mistakes

- Assuming Issues contain the full attack.
- Disabling EED collection.
- Ignoring Timeline View.
- Investigating child processes before identifying the CGO.
- Expecting machine learning to work without enough historical data.

## Key Takeaways

- Issues provide alerts; logs provide detailed telemetry — Log Stitching is the process of correlating the two to create Stitched Issues.
- Enhanced Endpoint Data supplies the telemetry used for stitching, uploaded roughly every five minutes.
- Causality Chains identify attack origins; Timeline View reconstructs attack progression.
- Cortex Analytics Engine applies unsupervised machine learning to detect behavioral anomalies without needing labeled attack data.
- Analytics Detectors monitor specific attack techniques mapped to MITRE ATT&CK, each maintaining its own baseline.
- Integrated Analysis transforms isolated alerts into complete attack stories.

---

## Senior SOC Analyst Perspective

Integrated Analysis is what separates a modern XDR platform from a traditional EDR. An EDR may tell you *"PowerShell executed."* Cortex Integrated Analysis tells you who launched PowerShell, what it touched, where it connected, how it persisted, what MITRE tactics were used, and whether the behavior deviates from the organization's normal baseline. This allows analysts to investigate the entire attack lifecycle instead of chasing disconnected alerts.

## Chapter Bridge

Everything in this chapter — Log Stitching, EED, Causality Chains, CAE — describes what Cortex does *automatically* with the underlying log data. The next chapter picks up the other half of that story: how an analyst queries that same underlying log data directly, by hand, when the automated stitching and analytics raise a question they don't fully answer.
