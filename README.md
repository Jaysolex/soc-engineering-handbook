# Senior SOC Engineering Handbook

**Version:** 1.0 (Living Document)
**Author:** Solomon James
**Purpose:** Operational Engineering Reference for Cortex XDR/XSIAM Security Operations

An engineering-grade, production reference — not exam notes. Every chapter is written to be trustworthy enough to open cold during a real incident and make engineering decisions from.

## Chapter Format

Each chapter follows the standard operational template: Purpose, Problem It Solves, Architecture, Workflow, Inputs/Outputs, Internal Logic, Operational Workflow, Production Workflow, Senior SOC Decision Process, Common Use Cases, Best Practices, Common Mistakes, Automation Opportunities, Security Considerations, Related Features, Real Production Example, Example Scripts (where applicable), Key Takeaways — adapted per chapter as needed (e.g., Chapter 1 uses a 10-part architectural narrative since it establishes the platform model everything else depends on). No Interview/Exam Notes sections — this is a working production reference, not certification prep.

## Learning Order

Chapters build on each other in this exact order — later chapters assume earlier ones. Do not read out of order on a first pass.

| Ch. | Title | Status |
|---|---|---|
| 1 | [Cortex Platform Architecture](01-platform-architecture/chapter-01-cortex-platform-architecture.md) | ✅ Written |
| 2 | [Cortex Work Plans](02-work-plans/chapter-02-cortex-work-plans.md) | ✅ Written |
| 3 | [Playbook Task Types (SCDS)](03-playbook-task-types/chapter-03-playbook-task-types.md) | ✅ Written |
| 4 | [Inputs, Outputs & Context](04-inputs-outputs-context/chapter-04-inputs-outputs-context.md) | ✅ Written |
| 5 | [Indicator Extraction](05-indicator-extraction/chapter-05-indicator-extraction.md) | ✅ Written |
| 6 | [Filters and Transformers](06-filters-and-transformers/chapter-06-filters-and-transformers.md) | ✅ Written |
| 7 | [Lists](07-lists/chapter-07-lists.md) | ✅ Written |
| 8 | [Automation Features & Playbook Automation](08-automation-features-playbook-automation/chapter-08-automation-features-playbook-automation.md) — merges Jobs, Automation Rules, Playbook Catalog | ✅ Written |
| 9 | *(merged into Chapter 8)* | — |
| 10 | *(merged into Chapter 8)* | — |
| 11 | Quick Actions | ⏳ Pending |
| 12 | Issues | ⏳ Pending |
| 13 | Issue Prioritization | ⏳ Pending |
| 14 | Cases | ⏳ Pending |
| 15 | Response Actions | ⏳ Pending |
| 16 | Endpoint Response | ⏳ Pending |
| 17 | [Advanced Response Actions — Remote Script Executions](17-advanced-response-actions/chapter-17-remote-script-executions.md) | 🔶 Partial (Remote Script Executions written; Remediation Suggestions, versioning/release engineering still pending) |

## Engineering Principles (apply throughout)

1. Context is everything. Data without context creates noise.
2. Automate repetitive work, not judgment.
3. Evidence before remediation.
4. Protect business operations while protecting security.
5. Design for repeatability.
6. Everything should be observable, auditable, and reversible where possible.
7. Production automation must always consider risk and asset criticality.

## Status

Living document. Nothing here is final. Chapters are revised as production experience accumulates and as later chapters reveal gaps in earlier ones.
