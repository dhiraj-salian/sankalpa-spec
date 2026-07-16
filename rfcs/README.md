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
| [0005](0005-secret-materialization-stability-and-determinism.md) | Secret materialization stability and the secret carve-out to the determinism guarantee | Final |
| [0006](0006-vault-base-version-stamping-for-lost-update-safe-sync.md) | Vault base-version stamping for lost-update-safe bidirectional Knowledge synchronization | Final |
| [0007](0007-scheduling-admission-liveness-priority-deadlines-and-terminals.md) | Scheduling admission liveness — priority, deadlines, starvation-freedom, and the pending-work terminal | Accepted |
| [0008](0008-grant-reauthorization-on-package-upgrade.md) | Capability-grant re-authorization on package upgrade — binding grants to verified version identity | Final |
| [0009](0009-channel-identity-binding-and-assurance.md) | Channel identity binding and assurance — authenticating messaging-channel senders and gating cross-channel session continuity | Final |
| [0010](0010-runtime-observability-egress-verification.md) | Runtime observability egress verification — enforcing secret-freedom where the only plaintext secret exists | Accepted |
| [0011](0011-semantics-preservation-enforcement-for-compiler-passes.md) | Semantics-preservation enforcement for compiler passes — giving IR-P10 a mechanism | Accepted |

> RFC-0002–0004 are the Phase 2 hardening set, **Final 2026-07-15** — accepted (FCP accept disposition, called and concluded the same day) and reflected into `spec/` and the Glossary in the same change. Solo-maintainer repo: the 10-working-day FCP window was shortened and the ≥2-Reviewer gate waived, both recorded in each RFC header for auditability. 0002 and 0003 were accepted together so 0003 does not precede its 0002 dependency; 0004 shares a replay boundary with 0002 (§4.5) and adds the `RemediationTask` core kind. Reflection touched Books 01–06 and 10 plus the Glossary and CHANGELOG.
>
> RFC-0005–0011 are the **second Phase 2 hardening batch**, **Accepted 2026-07-16** — FCP (accept disposition) called and concluded the same day. Solo-maintainer repo: the 10-working-day FCP window was shortened and the ≥2-Reviewer gate waived, both recorded in each RFC header for auditability. All 21 open design questions were resolved before proposing, and a review pass fixed two defects it found (0005 conflated rotation with revocation, which would have let a memoized value shield a revoked grant against Book 11 §03 §5's "even past any caching"; 0007 claimed deadline-less work could be held indefinitely, contradicting its own shed budget and Book 03 §13 §2). Per process §8 these become **normative only on reflection into `spec/`** — status advances to Final once each §12 lands. Authored against the 0002–0004-Final spec. Each pins one concrete gap in a normative guarantee: **0005** — secret values are unrecordable (P7) resolved bindings, so intra-execution stability and the secret carve-out to the determinism guarantee are unspecified (amends the reflected Book 01 §05 / Book 06 §03 guarantee; extends re-execution replay). **0006** (Knowledge) — vault↔graph optimistic concurrency has no baseline for offline direct vault edits. **0007** (Kernel) — scheduling fairness/deadline/shed promises are unbacked and `Pending` can stall silently. **0008** (Packages) — capability grants are not re-authorized on upgrade (upgrade-laundering). **0009** (Interfaces) — messaging-channel sender identity is mapped, not authenticated. **0010** (Security/Observability) — the "hard" log secret-freedom guarantee rests on "nothing secret on the path," which is false for the untrusted runtime that holds the only plaintext secret and emits the Events the ledger is built from. **0011** (Compiler/IR) — IR-P10 semantics preservation cites a "semantics-preservation check" that is defined nowhere; the checks that exist are single-artifact well-formedness predicates that never compare a pass's output to its input. 0005 touches 0004's `CompensationFailed` condition; 0007 shares its no-silent-stall spine; 0010 enforces the secret-freedom that 0002's ledger asserts and builds on 0005's stable materialized set; 0011 repairs the backstop that 0003's determinization gates depend on. 0006/0008/0009 are independent, first findings in their domains.
>
> **Pattern across the batch.** Five of the seven are the same shape: a guarantee stated absolutely, resting on a mechanism that is unspecified (0007, 0011), inapplicable at the one point that matters (0010), or contradicted elsewhere in the spec (0009, and 0011's "trust is never extended to a pass's claim"). Where the honest answer is that the property is undecidable (0011) or the attacker is unbounded (0010), the drafts scope the claim rather than restate it — following the spec's own precedent of turning an undecidable question into a checkable annotation (Book 04 §Ch08 §2.6).

## Status legend

`Draft` → `Proposed` → `Accepted` → `Final` (also: `Deferred`, `Rejected`, `Withdrawn`, `Superseded`). See the process doc for transitions.
