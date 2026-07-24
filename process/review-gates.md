# Review Gates

*Status: Draft*

Every normative change clears a set of **gates** proportional to its impact. A gate is a question a designated reviewer must answer affirmatively (or justify N/A) before acceptance. Gates exist so that non-functional qualities are designed in, not discovered later.

## Impact tiers

| Tier | Example | Gates that apply |
|------|---------|------------------|
| **T0 — Editorial** | typo, link fix, formatting | Documentation only (trivially). |
| **T1 — Clarification** | reword without changing meaning | Documentation + Architecture (sanity). |
| **T2 — Local design** | one subsystem, no public API change | Architecture, Testability, Documentation, Backward-Compat. |
| **T3 — Cross-cutting / public API** | new IR op, Kernel API change, new AEP | **All** gates below. |

The author proposes a tier; the owning Domain Lead confirms it.

## The gates

Each maps to a template in [`../templates/`](../templates/).

1. **Architecture gate.** Does the change preserve conceptual integrity? Does it respect the layered flow (Intent → … → Knowledge), the "everything is a Resource" rule, and the Kernel-mediated communication rule? Owner: Domain Lead. Artifact: the RFC/ADR itself.
2. **Security gate.** Does it uphold every invariant in [`../SECURITY.md`](../SECURITY.md) and Book 11? New trust boundaries, secret flows, or capabilities require a filled [security-review-template](../templates/security-review-template.md). Owner: Security Domain Lead.
3. **Performance gate.** What are the latency, throughput, memory, and scaling implications? For T3, a filled [performance-review-template](../templates/performance-review-template.md) with a cost model. Owner: relevant Domain Lead.
4. **Testability gate.** Can the behavior be verified deterministically? A [testing-plan-template](../templates/testing-plan-template.md) describing conformance/property/integration tests. Owner: Reviewer.
5. **Documentation gate.** Are `spec/`, `GLOSSARY.md`, and affected diagrams updated? No new capitalized term without a glossary entry. Owner: Reviewer.
6. **Backward-compatibility gate.** Does it break any existing Resource, IR, API, or Package? If so, is the break justified and versioned? Owner: Domain Lead.
7. **Migration gate.** If anything breaks or changes shape, is there a migration path and (eventually) tooling? Owner: Domain Lead.

## Passing a gate

A gate passes when its owner approves, or when the author documents **"N/A — because …"** and no reviewer disputes it during FCP. Unaddressed gates are blocking objections by default.

**During the bootstrap period** (a domain with no seated gate owners, [RFC-0012](../rfcs/0012-interim-review-process.md)): the gates are unchanged in *what* they ask, but the owner who would answer them does not exist. The author answers each gate **adversarially** in the [interim self-review](../templates/interim-self-review-template.md), and an [independent adversarial pass](interim-review.md#2-the-interim-acceptance-checklist) probes those answers. Where the gate owners do not exist, the gate is *cleared for v1.0* by an independent re-review — a seated human or an **agent review quorum** ([RFC-0013](../rfcs/0013-agent-reviewers-and-the-agent-review-quorum.md)), recorded with its provenance (`cleared (human)` / `cleared (agent)`). The **tier itself** is proposed by the author and recorded in the PR (the confirming Domain Lead being vacant); the adversarial pass may challenge it, and a mis-tiering that hides a normative change behind an "editorial" label is a blocking objection (see [`interim-review.md`](interim-review.md) §1). T3 changes still require the filled security/performance/testing artifacts — those are not waived, only differently signed off — and a T3 change to a **security invariant, the AOS IR, or the Kernel API** additionally requires a **named outside human** and a longer cooling-off, or is deferred (RFC-0012 §4.8). Every such acceptance is ledgered for real re-review before v1.0. This is explicitly weaker than the named owners and is recorded as such.

## Automation

CI enforces the mechanical subset: template completeness, glossary coverage, link integrity, diagram rendering, and RFC/ADR cross-reference validity. Human gates are recorded as PR review approvals from the designated owners.
