# Senior SOC Engineering Handbook

**Version:** 1.0 (Living Document)
**Author:** Solomon James
**Purpose:** Operational Engineering Reference for Cortex XDR/XSIAM Security Operations

An engineering-grade, production reference — not exam notes. Every chapter is written to be trustworthy enough to open cold during a real incident and make engineering decisions from.

---

## Chapter Format

Each chapter follows the standard operational template:

Purpose · Problem It Solves · Architecture · Workflow · Inputs/Outputs · Internal Logic · Operational Workflow · Production Workflow · Senior SOC Decision Process · Common Use Cases · Best Practices · Common Mistakes · Automation Opportunities · Security Considerations · Related Features · Real Production Example · Example Scripts (where applicable) · Key Takeaways

Chapters adapt this template as needed — for example, Chapter 1 uses a 10-part architectural narrative since it establishes the platform model everything else depends on.

No Interview or Exam Notes sections. This is a working production reference, not certification prep.

---

## Learning Order

Chapters build on each other in this exact order — later chapters assume earlier ones. Do not read out of order on a first pass.

| Ch. | Title | Status |
|---|---|---|
| 1 | [Cortex Platform Architecture](01-platform-architecture/chapter-01-cortex-platform-architecture.md) | ✅ |
| 2 | [Cortex Work Plans](02-work-plans/chapter-02-cortex-work-plans.md) | ✅ |
| 3 | [Playbook Task Types (SCDS)](03-playbook-task-types/chapter-03-playbook-task-types.md) | ✅ |
| 4 | [Inputs, Outputs & Context](04-inputs-outputs-context/chapter-04-inputs-outputs-context.md) | ✅ |
| 5 | [Indicator Extraction](05-indicator-extraction/chapter-05-indicator-extraction.md) | ✅ |
| 6 | [Filters and Transformers](06-filters-and-transformers/chapter-06-filters-and-transformers.md) | ✅ |
| 7 | [Lists](07-lists/chapter-07-lists.md) | ✅ |
| 8 | [Automation Features & Playbook Automation](08-automation-features-playbook-automation/chapter-08-automation-features-playbook-automation.md) | ✅ |
| 9 | [Quick Actions](09-quick-actions/chapter-09-quick-actions.md) | ✅ |
| 10 | [Working with Issues](10-issues/chapter-10-issues.md) | ✅ |
| 11 | [Issue Prioritization](11-issue-prioritization/chapter-11-issue-prioritization.md) | ✅ |
| 12 | [Working with Cases](12-cases/chapter-12-cases.md) | ✅ |
| 13 | [Response Actions](13-response-actions/chapter-13-response-actions.md) | ✅ |
| 14 | [Endpoint Response](14-endpoint-response/chapter-14-endpoint-response.md) | ✅ |
| 15 | [Advanced Response Actions — Remote Script Executions](15-advanced-response-actions/chapter-15-remote-script-executions.md) | 🔶 |
| 16 | [NGFW Integration & External Dynamic Lists (EDL)](16-ngfw-integration-edl/chapter-16-ngfw-integration-edl.md) | ✅ |

**✅ Written · 🔶 Partial** — Chapter 15 covers Remote Script Executions in full; Remediation Suggestions and script versioning/release engineering are still to be added.

Chapter 8 consolidates what were originally three separate topics (Jobs, Automation Rules, Playbook Catalog) into one chapter.

---

## Engineering Principles

Applied throughout every chapter:

1. Context is everything. Data without context creates noise.
2. Automate repetitive work, not judgment.
3. Evidence before remediation.
4. Protect business operations while protecting security.
5. Design for repeatability.
6. Everything should be observable, auditable, and reversible where possible.
7. Production automation must always consider risk and asset criticality.

---

## Status

Living document. Nothing here is final. Chapters are revised as production experience accumulates and as later chapters reveal gaps in earlier ones.
