# The RFC Process

*Status: Draft · Instrument owner: Steering Council · Template: [`../templates/rfc-template.md`](../templates/rfc-template.md)*

An **RFC (Request for Comments)** is how Sankalpa makes a *substantial* design decision — one that spans multiple subsystems, introduces a new concept, or changes a public contract in a way a single ADR cannot capture. The process is adapted from the Rust and Kubernetes (KEP) traditions.

## When an RFC is required

An RFC is REQUIRED for any change that:

- Adds, removes, or redefines a core concept (a Resource kind, an IR construct, a Manager, a lifecycle phase).
- Changes the AOS IR, the compiler pipeline, or the Kernel API in an observable way.
- Alters a security invariant or trust boundary.
- Introduces a new mandatory review gate or governance rule.
- Would otherwise require several coordinated ADRs.

An RFC is NOT required for bug fixes to prose, clarifications that preserve meaning, or purely additive research documents.

## Lifecycle

```
Draft ──► Proposed ──► [Review] ──► FCP(accept) ──► Accepted ──► Reflected in spec/ ──► Final
   │           │                        │
   │           └──► FCP(postpone) ──► Deferred
   └──────────────► Withdrawn / Rejected (recorded, never deleted)
```

| Status | Meaning |
|--------|---------|
| **Draft** | Author is still writing. Not yet seeking consensus. |
| **Proposed** | Complete per template; open for community review. |
| **Accepted** | Consensus reached; disposition is *accept*. Becomes normative once reflected in `spec/`. |
| **Final** | Accepted **and** fully reflected into the specification. |
| **Deferred** | Good idea, wrong time. May be revived. |
| **Rejected** | Will not be pursued; rationale recorded. |
| **Withdrawn** | Author retracted it. |
| **Superseded** | Replaced by a later RFC (linked). |

## Steps

1. **Socialize.** Open a Discussion describing the problem. Gauge appetite before investing in a long document.
2. **Draft.** Copy the template; fill *every* mandatory section (the 15-section shape from [`../CONTRIBUTING.md`](../CONTRIBUTING.md)). Keep status `Draft`.
3. **Reserve a number.** A maintainer assigns the next free `NNNN` when you open the PR.
4. **Propose.** Move to `Proposed`; request review from the owning Domain Lead and relevant Reviewers.
5. **Review.** Address feedback in-thread. Update the document; do not squash away the reasoning.
6. **Final Comment Period.** The Domain Lead calls a **10-working-day FCP** with a disposition. Blocking objections must cite concrete technical harm.
7. **Accept.** With Domain Lead approval, ≥2 Reviewer approvals, and no unresolved blocking objection, the RFC becomes `Accepted` and merges. *(While the owning domain is unstaffed this is unsatisfiable; acceptance follows the [interim review process](interim-review.md) instead — bootstrap authority plus a cooling-off, an adversarial self-review, a recorded adversarial pass, a provenance stamp, and a ledger entry; high-stakes changes to a security invariant, the AOS IR, or the Kernel API additionally require a named outside human or are deferred (RFC-0012 §4.8). The step is replaced, not skipped. This substitution is **provisional** until RFC-0012 is accepted under its own §4.2–4.5, RFC-0012 §4.7.)*
8. **Reflect.** Follow-up PR(s) update `spec/`, `GLOSSARY.md`, diagrams, and spawn any implementing ADRs. On completion the RFC becomes `Final`.

## Numbering & location

- File: `rfcs/NNNN-short-slug.md`, zero-padded to four digits.
- `rfcs/0000-rfc-process.md` is the meta-RFC describing this process itself.

## Amending an accepted RFC

Small corrections: editorial PR. Semantic changes: a **new** RFC that supersedes the old one. Accepted RFCs are historical records and are not rewritten.
