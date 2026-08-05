# Remote Script Executions

**Module:** Advanced Response Actions
**Category:** Endpoint Response / Action Center
**Status:** Draft v1 — expand as production experience accumulates

---

## Purpose

Remote Script Executions exist to give a SOC engineer a **programmable response arm** on the endpoint — the ability to run arbitrary, vetted Python logic across one or many hosts without needing an agent update, a new detection rule, or a support ticket to the endpoint team.

The underlying engineering problem Cortex is solving: EDR platforms ship a finite set of built-in response actions (isolate host, kill process, quarantine file, etc.). Real investigations constantly need something *slightly different* — check a registry key, pull a specific log, validate a config value, collect a non-standard artifact. Building a native UI action for every possible investigative question doesn't scale. Instead, Cortex exposes a **script runtime** so the SOC can extend its own response capability without waiting on the vendor's release cycle.

This is the same design principle behind SOAR script actions and Sysinternals-style tooling: give the analyst a general-purpose execution primitive, then let the SOC build a library of specific capabilities on top of it.

## Problem It Solves

| Without Remote Script Execution | With Remote Script Execution |
|---|---|
| Analyst needs vendor to ship a new built-in action | Analyst/engineer writes and uploads a script same day |
| Live Terminal used for one-off, unscripted, unaudited commands | Scripted, versioned, repeatable action with structured I/O |
| No standard way to run the same custom check across 500 hosts | Single action fans out to any endpoint selection |
| Ad hoc PowerShell/Python run by hand, no execution record | Every run is logged in Action Center with status + return value |

## Architecture

```
Script Source (Python 3.7)
        │
        ▼
 Scripts Library  (Investigation & Response
                    → Response → Action Center
                    → Agent Scripts Library)
        │  (Cortex parses entry point + params from source)
        ▼
 Script Definition (JSON: entry point, parameters, supported OS)
        │
        ▼
   Run Script Action
        │
        ├── Input Mapping   (per-parameter values)
        ├── Endpoint Targeting (OS-compatible hosts)
        │
        ▼
   Agent Execution (on endpoint, sandboxed to script scope)
        │
        ▼
   Action Center — All Actions
        │  ACTION TYPE: Endpoint Script Execution
        │  STATUS: Pending / Completed Successfully / Failed
        ▼
   Additional Data (per-endpoint return value / execution results)
```

Two artifacts matter here and are easy to conflate:

- **Script source** — the actual Python you wrote or uploaded.
- **Script definition** — a JSON manifest Cortex derives from the source: entry point function name, its parameters (name, type), and supported OS. This is what makes a script "runnable" from the UI rather than just stored text.

## Workflow

1. **Author or obtain a script.** Either a Palo Alto-authored script from the pre-built library, or a custom Python 3.7 script.
2. **Upload via `+New Script`.** Cortex statically parses the source to identify the entry point function and its parameters — this is how the "Enter Input" step later knows what fields to render.
3. **Invoke Run.** Right-click → Run in the Scripts Library, or call it as a step from a playbook.
4. **Bind inputs.** Supply values for each parameter defined by the entry point signature (e.g., `ip: STRING`).
5. **Target endpoints.** Selection is filtered to hosts whose platform matches the script's declared Supported OS.
6. **Dispatch and track.** The action appears in Action Center as an `Endpoint Script Execution` with a per-host status.
7. **Retrieve output.** The entry point's return value becomes the structured output, visible per-endpoint under Additional Data.

## Inputs

- **Script source code** (Python 3.7) — supplied by the engineer or selected from the built-in library.
- **Entry point parameters** — derived automatically from the function signature Cortex parses (e.g., `def ping(ip: str)` → one STRING input named `ip`).
- **Target endpoint list** — selected manually, by group, or dynamically from playbook context (e.g., the host tied to the current alert).
- **Supported OS declaration** — set at upload time, used to filter valid targets and prevent execution attempts on incompatible platforms.

## Outputs

- **Return value of the entry point function** — the primary structured output, consumable by:
  - the analyst reviewing Additional Data in the UI
  - a downstream playbook task (e.g., feed the return value into a decision node)
  - a case field or war-room note for evidence tracking
- **Per-endpoint execution status** (Completed Successfully / Failed / Pending) — feeds SOAR error handling and retry logic.
- **Action Center audit record** — action type, initiator, timestamp, targets, and parameters, independent of the return value itself.

## Internal Logic

Cortex treats the uploaded script as a contract, not just a blob of code:

1. At upload, it statically inspects the source to extract the **entry point** — the function Cortex will actually invoke — along with parameter names/types and the return signature.
2. This produces the **script definition (JSON)**, decoupled from the source. The definition is what drives the "Choose step" UI (parameter form) and the endpoint compatibility filter — the UI never has to re-parse source code at runtime.
3. At execution, Cortex pushes the compiled/interpreted script to the targeted agents, invokes the entry point with the bound parameter values, and captures the function's return value as the result payload.
4. Because input/output is derived from a real function signature rather than free-text shell command, Cortex can validate types before dispatch (e.g., reject a non-IP string for an `IP (STRING)` parameter) and can render structured results instead of raw console output.

This is the key architectural difference from Live Terminal: Live Terminal is an **interactive, unstructured shell session** (good for exploratory work, hard to automate or audit at scale); Remote Script Execution is a **structured, parameterized, repeatable action** (good for known, repeatable investigative or remediation steps, and the only one of the two that a playbook can call cleanly).

## Operational Workflow

A Tier 2/3 analyst mid-investigation, needing a non-standard artifact:

1. Alert/case indicates a suspicious registry modification on a host.
2. No built-in action reads that specific registry key.
3. Analyst checks the Scripts Library for an existing "read registry key" script; if none exists, escalates to a detection/SOAR engineer to write one (or uses Live Terminal for the one-off case if speed matters more than repeatability).
4. Once a script exists, the analyst runs it against the affected host(s), enters the key path as input, targets the endpoint, and reviews the return value in Additional Data.
5. Findings get attached to the case as evidence; if the check is likely to recur, the script stays in the library for future use — this is how a SOC's custom action library organically grows.

## Production Workflow

Remote script execution is **code that runs on production endpoints with the same trust level as an EDR agent action** — it should be governed like a deployment, not like a UI click.

- **Approval requirement:** New or modified scripts should go through peer review before being published to the shared library, especially anything with write/remediation capability (not just read/collection).
- **Least-privilege targeting:** Default to running against a small, representative pilot set of endpoints before a fleet-wide run, particularly for anything unfamiliar or newly authored.
- **Change management:** Treat script uploads as a change — track who uploaded/modified a script and when (Action Center covers execution history; source control should cover script history, see Related Features).
- **Testing:** Validate against a non-production or lab endpoint first. Confirm behavior on every OS declared in Supported OS — a script tested only on Windows and marked "supports Linux/macOS" is a production incident waiting to happen.
- **Rollback:** For any script with side effects (registry writes, service changes, file modification), define and test the reverse action *before* running the forward action at scale.

## Senior SOC Decision Process

| Decision | Who / What decides |
|---|---|
| Should this check be scripted vs. run manually via Live Terminal? | Analyst judgment — frequency of need vs. one-off exploratory work |
| Should this script be added to the shared library? | Engineer judgment — is this reusable across future investigations? |
| Is this script read-only or does it have side effects? | Must be explicit before approval — changes review/approval bar |
| Should execution require dual approval before running fleet-wide? | SOC Lead / change management policy, based on blast radius |
| Is the target endpoint set correct for this investigation? | Analyst — over-broad targeting is a common and costly mistake |

Automation should own dispatch, targeting validation, and status tracking. Analysts and engineers own script authorship, scope decisions, and interpretation of return values in the context of a specific investigation.

## Common Use Cases

**Situation:** Case requires evidence collection from a config file not covered by any built-in action.
**Reason:** Investigative need outpaces the vendor's built-in action set.
**Expected Outcome:** Analyst runs an existing (or quickly-authored) read-only collection script, attaches return value as evidence.

**Situation:** A SOAR playbook needs to validate a host-side condition before deciding whether to isolate.
**Reason:** Automated response should not fire blind — needs a structured, machine-consumable check.
**Expected Outcome:** Playbook calls a script action as a decision-gate task; return value branches the playbook logic.

**Situation:** Recurring low-severity alert type requires the same manual check every time.
**Reason:** Analyst time is being spent on repeatable, scriptable work.
**Expected Outcome:** Engineer writes and publishes a reusable script; future occurrences resolve in minutes instead of manual investigation.

## Best Practices

- Keep scripts single-purpose and narrowly scoped — one script, one job — so review, testing, and reuse stay simple.
- Prefer read/collection scripts as the default; treat write/remediation scripts as a distinct, higher-scrutiny category.
- Version scripts outside Cortex (Git) as the source of truth; Cortex's library is the runtime deployment target, not the version control system.
- Name and document scripts (parameters, expected return shape, supported OS, side effects) well enough that another engineer can safely run them without reading the source first.
- Pilot on a small endpoint set before any fleet-wide execution.

## Common Mistakes

- **Treating scripted actions like Live Terminal commands** — running untested, ad hoc scripts against broad endpoint sets without a pilot phase. The structured/repeatable nature of this feature makes it easy to fan out an unvalidated action to thousands of hosts in one click.
- **Ignoring Supported OS accuracy** — declaring broader OS support than actually tested, leading to failures or, worse, unexpected behavior on an untested platform.
- **No return-value contract** — writing scripts whose return value isn't structured or documented, making the output unusable by playbooks or by another analyst later.
- **No source control** — editing scripts only inside the Cortex library UI, with no external versioning or review trail, so there's no way to diff, roll back, or audit changes over time.
- **Conflating read and write scripts** in the same review process — a config-read script and a registry-write script are not the same risk category and shouldn't get the same rubber-stamp approval.

## Automation Opportunities

**Where SOAR should use this:**
- As a decision-gate task inside a playbook (collect a value → branch logic on the result).
- As a standardized evidence-collection step attached automatically to specific alert types.
- As part of a validation step before a higher-impact automated action (e.g., confirm a process is actually malicious before an automated kill).

**Where automation should stop:**
- Any script with destructive or state-changing side effects should not be fully automated end-to-end without a human approval gate, unless the action has been proven low-risk and high-confidence over time.
- Automation should not silently expand endpoint targeting scope — target selection logic should be explicit and reviewed, not dynamically unbounded.

## Security Considerations

- **Permissions:** Only roles with sufficient privilege can run scripts on endpoints — this should map to a named RBAC role, not be granted broadly.
- **Access:** Uploading/editing scripts and running scripts are logically separate permissions and should be treated as separate roles where the platform allows it (author vs. operator).
- **Auditing:** Every execution is recorded in Action Center (action type, initiator, targets, status) — this record should feed SOC audit/compliance review, especially for any script with remediation capability.
- **Logging:** Return values may contain sensitive host data (file contents, registry values, credentials-adjacent artifacts) — treat script output with the same data-handling sensitivity as raw endpoint forensic data.
- **Least privilege:** Scripts should request/access only what's needed for their stated purpose; a "read one registry key" script that also has broad filesystem access is a scope violation.

## Related Features

- **Live Terminal** — the unscripted counterpart; used for exploratory, one-off interactive work where scripting isn't yet justified.
- **Playbooks / SOAR** — Remote Script Execution is a callable task type inside a playbook, making it the bridge between manual investigative scripts and automated response.
- **Action Center** — the execution/audit ledger for every script run, alongside every other response action.
- **Scripts Library (built-in)** — Palo Alto-authored scripts ship here; custom scripts live alongside them, distinguished by the Created By column.
- **Cases / Issues** — script return values become case evidence; script runs are often triggered from within a case investigation.
- **EDL / Threat Intelligence enrichment** — scripts can be a mechanism for custom enrichment lookups not natively supported by built-in TI integrations.

## Real Production Example

A SOC detects a correlation alert for a known persistence technique (suspicious scheduled task creation) on a finance-department workstation.

1. **Triage:** Analyst opens the case; the built-in action set can isolate the host or kill the process, but can't directly confirm whether the scheduled task's target binary is signed.
2. **Gap identified:** No built-in action checks binary signature status on an arbitrary file path.
3. **Response:** A previously-published "check file signature" script exists in the library (written by a detection engineer after a similar past incident).
4. **Execution:** Analyst runs the script, inputs the binary path pulled from the alert's raw event data, targets the single affected host.
5. **Result:** Return value confirms the binary is unsigned and located in a non-standard path — strong corroborating evidence of malicious persistence.
6. **Escalation:** Analyst attaches the script output to the case, escalates to isolate the host (separate, native response action) and opens a remediation task to remove the scheduled task.
7. **Engineering follow-up:** Because this pattern is likely to recur, the SOAR engineer adds this script as an automated evidence-collection step for this alert type going forward — turning a manual investigative step into a standard playbook task for future occurrences.

## Interview Notes

- Be able to explain the difference between Remote Script Execution and Live Terminal, and *why* each exists (structured/repeatable/automatable vs. interactive/exploratory).
- Understand that the "script definition" is a derived JSON artifact, not something authored directly — interviewers may probe whether you understand entry-point parsing.
- Be ready to discuss governance: why a scripting capability on an EDR platform is a security-sensitive feature in its own right (RBAC, auditing, blast radius).

## Exam Notes

- Scripts must be **Python 3.7**.
- Path: **Investigation & Response → Response → Action Center → Agent Scripts Library**.
- The **entry point** is the function Cortex calls; its parameters and return value define the runtime I/O.
- Script **definition** = JSON; script **source** = the actual code — know the distinction.
- Action Center shows **ACTION TYPE: Endpoint Script Execution**; status values include **Completed Successfully**.
- **Additional Data** is where the per-endpoint **Return Value** is surfaced.
- Endpoint targeting is filtered by the script's declared **Supported OS**.

## Key Takeaways

Remote Script Execution turns the EDR agent into a programmable response surface: any investigative or remediation action a SOC can express in Python becomes a first-class, auditable, targetable, playbook-callable action — closing the gap between "what the vendor built in" and "what this SOC actually needs to do." Its value is proportional to how disciplined the engineering practice around it is: version control, review, OS-tested claims, and a clear read-vs-write risk split are what separate a mature script library from a liability.

---
*Source material: Cortex "Advanced Response Actions — Remote Script Executions" training module. Content above is original engineering documentation derived from that training, not a reproduction of vendor text.*
