# Maintainers

*Status: Draft · Governed by [`GOVERNANCE.md`](GOVERNANCE.md).*

This file is the source of truth for who holds which role. It is updated by PR with Steering Council approval.

## Bootstrap authority (interim)

*Provisional — pending [RFC-0012](rfcs/0012-interim-review-process.md) reaching `Accepted` under its own §4.2–4.5 (RFC-0012 §4.7, "Adoption sequencing"). Staged in the same change.*

While the roles below are unfilled, the project runs under the **interim review process** ([RFC-0012](rfcs/0012-interim-review-process.md), [`GOVERNANCE.md`](GOVERNANCE.md) §7). Acceptance authority for unstaffed domains is held **temporarily** by the founding maintainer, exercised only through RFC-0012's compensating controls and logged in the [interim-acceptance ledger](process/interim-acceptance-ledger.md). This is not a Domain Lead seat; it lapses automatically as domains are staffed (RFC-0012 §4.6).

| Handle | Role | Scope | Ends when |
|--------|------|-------|-----------|
| Dhiraj Salian | Founding maintainer (bootstrap acceptance authority) | All domains with a vacant Domain Lead seat | Per domain: that domain seats a Domain Lead + ≥2 Reviewers. Project-wide: ≥3 Domain Leads seated. |

## Steering Council

| Name / Handle | Affiliation | Term ends |
|---------------|-------------|-----------|
| *(to be seated once the project has ≥3 active Domain Leads)* | — | — |

## Domain Leads

Each Book / subsystem has exactly one accountable Domain Lead.

| Domain | Book(s) | Lead |
|--------|---------|------|
| Foundations & Governance | 00, 01 | *vacant* |
| Resource Model (ARM) | 02 | *vacant* |
| Kernel & Controller Runtime | 03, 07 | *vacant* |
| AOS IR & Compiler | 04, 05 | *vacant* |
| Runtimes | 06 | *vacant* |
| Planners | 08 | *vacant* |
| Knowledge & Experience | 09, 10 | *vacant* |
| Security | 11 | *vacant* |
| Packages & Marketplace | 12 | *vacant* |
| Interfaces & Channels | 13 | *vacant* |
| Observability & Governance | 14 | *vacant* |

## Reviewers

| Handle | Areas |
|--------|-------|
| *(added as contributors earn trust)* | — |

> A reviewer seated **solely** under the founding maintainer's bootstrap authority, without endorsement from an already-independent party, is marked **(bootstrap-seated)** here. Per [RFC-0012 §4.5](rfcs/0012-interim-review-process.md), bootstrap-seated reviewers **alone** cannot clear an interim-acceptance-ledger entry or satisfy the v1.0 "two independent reviewers" gate — at least one clearing/signing reviewer must trace their seating to someone other than the bootstrap author. This is the standing guard against sock-puppet clearance of the bootstrap backlog.

## Agent reviewers (interim)

Per [RFC-0013](rfcs/0013-agent-reviewers-and-the-agent-review-quorum.md), an **independent agent review quorum** may clear ledger entries and satisfy the v1.0 two-reviewer gate while a domain is unstaffed by humans. To count, agent reviewers MUST be independent (RFC-0013 §4.2): **≥2** in distinct cold sessions, of **distinct model families** (or a disclosed third), never the authoring session, with model id/version, verbatim prompt, all runs, and findings recorded. High-stakes changes (security invariant / AOS IR / Kernel API) require a **≥3-agent, ≥2-family** panel (RFC-0013 §4.5). Agent reviewers are **not** a role in the tables above: they supply review, not accountability, and do **not** fill a Domain Lead or Steering Council seat or end the bootstrap period (RFC-0013 §4.7). An agent-cleared obligation is recorded `cleared (agent)` and remains upgradeable to `cleared (human)`.

## Emeritus

Former maintainers who have stepped back. Listed with gratitude; retain no merge rights.

## Becoming a maintainer

Sustained, high-quality contribution in an area, endorsed by the relevant Domain Lead and confirmed by the Steering Council. See [`GOVERNANCE.md`](GOVERNANCE.md) §2.
