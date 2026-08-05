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
| 2 | Cortex Work Plans | ⏳ Pending |
| 3 | Playbook Task Types (SCDS) | ⏳ Pending |
| 4 | Inputs, Outputs & Context | ⏳ Pending |
| 5 | Indicator Extraction | ⏳ Pending |
| 6 | Filters and Transformers | ⏳ Pending |
| 7 | Lists | ⏳ Pending |
| 8 | Jobs | ⏳ Pending |
| 9 | Automation Features | ⏳ Pending |
| 10 | Playbook Automation | ⏳ Pending |
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
