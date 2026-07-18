# Interim-Acceptance Ledger

*Status: Living document · Defined by: [RFC-0012](../rfcs/0012-interim-review-process.md) §4.5.*

> Every artifact accepted under the [interim review process](interim-review.md) is recorded here as a **standing re-review obligation**. An entry is `pending` until a **seated Reviewer or Domain Lead** (never the founding maintainer acting under bootstrap authority) re-reviews it; it then becomes `cleared` with a date and reviewer. **A Book with any `pending` ledger entry touching it cannot pass the Phase 2 v1.0 gate** ([`ROADMAP.md`](../ROADMAP.md): two independent reviewers per Book). This ledger is the debt record of the bootstrap period — visible, so it can be paid.

## How to read the status

- **`pending`** — accepted under interim rules; owes an independent re-review before v1.0.
- **`cleared`** — a seated reviewer has re-reviewed it; records who and when.
- **`superseded`** — replaced by a later artifact (linked); the successor carries any remaining obligation.

## Grandfathered class — accepted before RFC-0012

These were accepted under *ad-hoc* pre-RFC-0012 interim practice (solo authorship, prose waiver, no compensating-control package). They are logged as a class rather than retrofitted with self-reviews (RFC-0012 §13). All are `pending` and owe the same pre-v1.0 re-review.

| Artifact(s) | Domain / Books | Accepted | Status |
|-------------|----------------|----------|--------|
| RFCs 0001–0011 (Final) | IR, Compiler, Runtimes, Security, Knowledge, Interfaces, Packages | 2026-07 (various) | `pending` |
| ADRs 0000–0002 | Governance, Kernel | 2026-07 | `pending` |
| `spec/` Books 00–15 (Draft-complete) | all | 2026-07 | `pending` |
| Phase 0 research (18 studies) + [audit](../research/README.md#provenance-and-its-limits) | research (non-normative) | 2026-07 | `pending` |

*Note on the research: it is non-normative, so its ledger obligation is lighter — it does not gate a Book's v1.0 — but it is recorded here for completeness and because its own provenance note already flags it for a second read.*

## Entries under RFC-0012

| # | Artifact | Domain | Accepted | Authority | Adversarial pass | Re-review |
|---|----------|--------|----------|-----------|------------------|-----------|
| 1 | [RFC-0012](../rfcs/0012-interim-review-process.md) — interim review process | Governance | *pending cooling-off* | founding maintainer (bootstrap) | AI adversarial pass (recorded in PR) | `pending` |

*(RFC-0012 is itself the first artifact processed under the rules it defines. It is `Proposed`, not yet `Accepted`: its acceptance awaits the §4.2 cooling-off. The row is seeded so the ledger is live from day one.)*
