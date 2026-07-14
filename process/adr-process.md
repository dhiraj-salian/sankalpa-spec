# The ADR Process

*Status: Draft · Instrument owner: owning Domain Lead · Template: [`../templates/adr-template.md`](../templates/adr-template.md)*

An **ADR (Architecture Decision Record)** captures a *single* architecturally significant decision, the context that forced it, and its consequences. ADRs are short, immutable, and append-only. The format follows Michael Nygard's original ADR convention.

## What is "architecturally significant"?

A decision is significant — and thus deserves an ADR — if it is **costly to reverse** or **constrains future work**: choosing a data model, a protocol, a boundary, a dependency, a naming convention with wide reach, or a trade-off between competing qualities (e.g., consistency vs. availability).

## Lifecycle

```
Proposed ──► Accepted ──► (later) Superseded by ADR-NNNN
                     └──► Deprecated
        └──► Rejected
```

ADRs are **immutable once Accepted.** To change a decision you write a *new* ADR that supersedes the old one; you then set the old ADR's status to `Superseded by ADR-NNNN` (a link is the only permitted edit).

## Steps

1. Copy [`../templates/adr-template.md`](../templates/adr-template.md).
2. State the decision in one sentence at the top. If you cannot, it is probably an RFC.
3. Fill Context, Decision, Consequences (positive **and** negative), and Alternatives Considered.
4. Open a PR; the owning Domain Lead reviews. A short FCP (5 working days) applies for cross-cutting ADRs.
5. On acceptance, the ADR is merged with status `Accepted` and an assigned number.

## Relationship to RFCs

An RFC decides a *design*; the individual decisions inside it are recorded as ADRs so they remain discoverable without reading the whole RFC. An ADR that grows to need multiple coordinated decisions should be promoted to an RFC.

## Numbering & location

- File: `adrs/NNNN-short-slug.md`.
- `adrs/0000-record-architecture-decisions.md` records the decision to use ADRs at all.
