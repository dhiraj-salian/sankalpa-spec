# Book 00 · Chapter 05 — Relationship to Implementation

*Nature: **Normative** (restates the binding spec-first discipline). · Reflects: ADR-0001.*

> This chapter restates, for readers of the specification, the relationship between this document and any implementation of it: **the specification is authoritative; an implementation realizes it.** This is ADR-0001 (specification-first development) stated from the reader's vantage.

## 1. The specification is authoritative

`sankalpa-spec` **is the product** until it reaches a stable v1.0 (ADR-0001, ROADMAP). The normative text here defines *what Sankalpa is*; an implementation is *one realization* of it. Where an implementation and the specification disagree on required behavior, the specification governs, and the disagreement is a **bug** — in the implementation, or (if the spec is wrong) in the specification, resolved through the process (§3).

This inverts the common "docs describe the code" relationship: here, the code realizes the spec. Rationale: conceptual integrity is preserved by designing the whole before building the parts (Book 01 §Ch02 §6, ADR-0001).

## 2. Implementation does not precede architecture

No normative behavior exists in a conforming implementation that is not first specified here (ADR-0001, P12). Implementation phases (ROADMAP Phase 3+) begin only after the specification stabilizes (Phase 2 gate). An implementation MUST NOT introduce required behavior the specification does not define; if it needs to, the specification is changed first (via RFC/ADR), then the implementation follows. This is what keeps many possible implementations — and third-party extensions — targeting one coherent, authoritative contract.

## 3. Discrepancies are resolved through the process

When implementation reveals a specification gap or error (as it will — implementation is a rigorous test of a design):
- The discrepancy is raised as a **defect** (`.github` issue templates) or, if it requires a design change, an **RFC/ADR** (`process/`).
- The specification is corrected through the normal governance and review gates (`GOVERNANCE`, `process/review-gates`), and the CHANGELOG and version reflect it (`process/versioning-and-stability`).
- The implementation is then aligned to the corrected specification.

Discrepancies are thus not resolved by the code silently winning, but by the *process* deciding whether the spec or the implementation was wrong, and recording the decision. This preserves the specification's authority while letting implementation feed back into it (the Phase 2+ loop, ROADMAP).

## 4. What this means for a reader

- **Implementers:** treat this specification as the contract. Where it says MUST, implement it; where it is silent or unclear, do not invent required behavior — raise it (§3). Pass the conformance suites (Book 00 §Ch02 §4) to claim conformance.
- **Extenders:** build against the AEP interfaces (Books 06/08/12), which are the stable contracts for third parties. Your extension conforms to a *version* (Book 00 §Ch02 §2).
- **Reviewers/contributors:** a change to *how Sankalpa works* is a change to *this specification* first (GOVERNANCE §1), and to code second.

## 5. Invariants (normative summary)

1. The specification is authoritative; an implementation is one realization of it; on disagreement about required behavior, the specification governs and the disagreement is a bug.
2. No required behavior exists in a conforming implementation that the specification does not first define; implementation follows architecture, never precedes it (ADR-0001, P12).
3. Discrepancies revealed by implementation are resolved through the governance process (defect/RFC/ADR + review gates + versioning), which decides whether spec or implementation was wrong and records it — the code does not silently win.
4. Implementers treat MUST as binding and raise gaps rather than inventing behavior; extenders build against versioned AEP interfaces; contributors change the specification first.
