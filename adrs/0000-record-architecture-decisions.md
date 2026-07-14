# ADR-0000: Record architecture decisions

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **Date** | 2026-07-14 |
| **Deciders** | Founding maintainers |
| **Domain / Book** | Governance / Book 00 |
| **Related** | ADR-0001 |

## Context

Sankalpa is designed for a 10+ year horizon and will be built by a rotating community of contributors. Decisions made today — a boundary, a data model, a trade-off — will constrain work for years, and their *rationale* is the first thing lost to time. Without a durable record, future contributors re-litigate settled questions, or worse, silently violate the reasoning behind a design because they never knew it existed. Long-lived projects (the Linux kernel's mailing-list archives, LLVM's design docs, the K8s KEP history) all preserve decision context; the lightweight, standardized form for this is Michael Nygard's Architecture Decision Record.

## Decision

We will record every architecturally significant decision as an **ADR** — a short, immutable Markdown file in `adrs/NNNN-*.md` following [`../templates/adr-template.md`](../templates/adr-template.md). ADRs are immutable once Accepted; a decision is changed by writing a new ADR that supersedes the old one.

## Consequences

**Positive**
- Rationale is preserved next to the decision, discoverable forever.
- New contributors can reconstruct the "why" without archaeology.
- Superseding, not editing, preserves an honest history of how thinking evolved.

**Negative / costs**
- Discipline required: contributors must write the record, not just make the change.
- Some overhead deciding "is this significant enough for an ADR?" (mitigated by the guidance in the ADR process).

**Neutral / follow-ups**
- ADRs complement RFCs (which decide whole designs) and AEPs (which decide public interfaces). See [`../process/README.md`](../process/README.md).

## Alternatives Considered

- **No formal record (rely on commit messages / chat).** Rejected: rationale scatters and rots; unsearchable at the granularity of *decisions*.
- **Only RFCs.** Rejected: RFCs are heavyweight; many worthwhile decisions are single choices that would never justify an RFC and would go unrecorded.
- **A wiki.** Rejected: mutable, drifts from the versioned spec, and loses the "immutable history" property.

## Compliance / Invariants touched

Establishes the "Everything is documented" and "The record outlives the decision" principles (Book 01, [`../GOVERNANCE.md`](../GOVERNANCE.md)).
