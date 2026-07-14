# Process

*Status: Draft · This directory defines how Sankalpa governs change. Start with [`../GOVERNANCE.md`](../GOVERNANCE.md).*

| Document | Defines |
|----------|---------|
| [`rfc-process.md`](rfc-process.md) | Lifecycle of a Request for Comments (substantial designs). |
| [`adr-process.md`](adr-process.md) | Lifecycle of an Architecture Decision Record (single decisions). |
| [`aep-process.md`](aep-process.md) | Lifecycle of an Architecture Extension Proposal (public interfaces). |
| [`review-gates.md`](review-gates.md) | The mandatory review gates every normative change must clear. |
| [`versioning-and-stability.md`](versioning-and-stability.md) | Status labels, SemVer for the spec, and stability guarantees. |

## Choosing an instrument

```
Is it an editorial/typo fix? ───────────────► plain PR (no artifact)
Does it define/change a PUBLIC extension API? ─► AEP
Is it ONE decision you can state in a page? ───► ADR
Is it a design with many decisions/subsystems? ► RFC  (which will spawn ADRs)
```

When in doubt, open an ADR; a maintainer will upgrade it to an RFC if its scope demands one.
