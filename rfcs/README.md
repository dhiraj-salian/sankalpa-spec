# Requests for Comments (RFCs)

*Process: [`../process/rfc-process.md`](../process/rfc-process.md) · Template: [`../templates/rfc-template.md`](../templates/rfc-template.md)*

RFCs are substantial design proposals — the primary vehicle for evolving Sankalpa's architecture. Each RFC follows the mandatory 15-section shape and passes the review gates before acceptance.

## Index

| # | Title | Status |
|---|-------|--------|
| [0000](0000-rfc-process.md) | The RFC process (meta) | Accepted |
| [0001](0001-aos-ir-as-sole-executable-representation.md) | AOS IR as the sole executable representation | Proposed |
| [0002](0002-replay-semantics-and-recorded-reasoning.md) | Replay semantics and the recorded-reasoning carve-out to the determinism guarantee | Final |
| [0003](0003-drift-detection-via-shadow-sampling.md) | Drift detection for determinized Capabilities via shadow sampling | Final |
| [0004](0004-compensation-failure-terminal-and-escalation.md) | Compensation-failure condition and escalation | Final |

> RFC-0002–0004 are the Phase 2 hardening set, **Final 2026-07-15** — accepted (FCP accept disposition, called and concluded the same day) and reflected into `spec/` and the Glossary in the same change. Solo-maintainer repo: the 10-working-day FCP window was shortened and the ≥2-Reviewer gate waived, both recorded in each RFC header for auditability. 0002 and 0003 were accepted together so 0003 does not precede its 0002 dependency; 0004 shares a replay boundary with 0002 (§4.5) and adds the `RemediationTask` core kind. Reflection touched Books 01–06 and 10 plus the Glossary and CHANGELOG.

## Status legend

`Draft` → `Proposed` → `Accepted` → `Final` (also: `Deferred`, `Rejected`, `Withdrawn`, `Superseded`). See the process doc for transitions.
