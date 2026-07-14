# Contributing to `sankalpa-spec`

*Status: Draft · Read [`GOVERNANCE.md`](GOVERNANCE.md) first.*

Thank you for helping design Sankalpa. This repository is a **specification**, so "contributing" here means contributing *ideas, designs, and words* — not (yet) implementation. Precision and reasoning matter more than volume.

## What kind of change are you making?

| You want to… | Do this |
|--------------|---------|
| Fix a typo / broken link / formatting | Open a PR labeled `editorial`. One maintainer can merge. |
| Clarify existing normative text without changing meaning | PR labeled `clarification` + owning Domain Lead review. |
| Change how a subsystem works | Open an **RFC** ([`templates/rfc-template.md`](templates/rfc-template.md)). |
| Record a single decision + rationale | Open an **ADR** ([`templates/adr-template.md`](templates/adr-template.md)). |
| Define/evolve a public extension interface | Open an **AEP** ([`templates/aep-template.md`](templates/aep-template.md)). |
| Contribute prior-art or pattern research | Add to [`research/`](research/README.md). Non-normative; lighter review. |
| Propose a diagram | Add source to [`diagrams/src/`](diagrams/) and reference it from the spec. |

**Do not** open a PR that edits normative Book text without a linked Accepted (or in-FCP) RFC/ADR. Such PRs will be closed with a pointer here.

## The mandatory shape of a design contribution

Every RFC and every substantial ADR must answer, in order (this mirrors the project's Output Expectations):

1. Executive Summary
2. Problem Statement
3. Alternatives Considered
4. Proposed Design
5. Tradeoffs
6. API Changes
7. Resource Changes
8. Event Changes
9. Security Impact
10. Performance Impact
11. Testing Strategy
12. Documentation Changes
13. Migration Strategy
14. Risks
15. Future Improvements

Sections that genuinely do not apply are marked **"N/A — because …"**. Blank or "TODO" sections block acceptance.

## Quality bar

- **No placeholder content.** If a section is not ready, keep the artifact in `Draft` rather than merging a stub.
- **Cite prior art.** If Linux/LLVM/Kubernetes/Temporal/Git/Postgres solved a related problem, say how, and how we differ and why.
- **Define your terms.** New capitalized terms must be added to [`GLOSSARY.md`](GLOSSARY.md) in the same PR.
- **Prefer diagrams for anything with more than three moving parts.** Sequence diagrams for protocols; component diagrams for structure.
- **Write for two readers:** the newcomer who needs the *why*, and the implementer who needs the *exactly what*.

## Workflow

1. **Discuss first.** Open a `Discussion` or issue to socialize the idea before writing a long RFC.
2. **Draft** from the template in a branch; number it using the next free integer (maintainers reserve numbers at PR time).
3. **Open a PR.** CI checks: link integrity, glossary coverage, template completeness, spelling, diagram render.
4. **Review** per the [review gates](process/review-gates.md).
5. **FCP → Accepted.** A maintainer transitions status and merges. Follow-up PRs reflect the decision into [`spec/`](spec/README.md).

## Style

- Normative keywords **MUST / MUST NOT / SHOULD / SHOULD NOT / MAY** follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and are used *only* in normative sections.
- One sentence per line in normative text (keeps diffs reviewable).
- American English; Oxford comma; ISO-8601 dates.
- File names: `kebab-case.md`. Numbered artifacts: `NNNN-short-slug.md`.

## Developer Certificate of Origin

All commits must be signed off (`git commit -s`), certifying the [DCO](https://developercertificate.org/). By contributing you agree your contribution is licensed under the repository's licenses (see [`LICENSE`](LICENSE)).
