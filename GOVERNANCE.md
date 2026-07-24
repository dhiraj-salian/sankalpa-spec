# Governance

*Status: Draft · Applies to: the whole project · Owner: Steering Council*

This document defines how decisions are made in Sankalpa. It is deliberately modeled on the governance of long-lived open-source projects (Rust RFCs, Kubernetes SIGs/KEPs, Python PEPs, the IETF rough-consensus tradition) and adapted to a **specification-first** project.

## 1. Guiding principles

1. **Specification-first.** Architecture precedes implementation. A change to *how Sankalpa works* is a change to this repository first, and to code second. See [ADR-0001](adrs/0001-specification-first-development.md).
2. **Conceptual integrity over feature count.** A coherent system that does less beats an incoherent system that does more. Maintainers are stewards of coherence.
3. **Rough consensus, documented dissent.** We seek broad agreement, not unanimity. Dissent is recorded, not erased.
4. **Everything is reversible except where we say it is not.** Stability guarantees are explicit ([`process/versioning-and-stability.md`](process/versioning-and-stability.md)); everything else may evolve.
5. **The record outlives the decision.** ADRs and RFCs are append-only history. We supersede; we do not silently rewrite.

## 2. Roles

| Role | Who | Authority |
|------|-----|-----------|
| **Contributor** | Anyone who opens an issue, RFC, or PR. | Propose; comment; implement. |
| **Reviewer** | Contributors trusted in a specific area. | Approve changes within their area of expertise. |
| **Maintainer** | Long-term stewards of one or more Books/subsystems. | Merge; assign status transitions; shepherd RFCs. |
| **Domain Lead** | The accountable maintainer for a Book/subsystem (Kernel, Compiler, Security, …). | Final call within their domain; breaks review ties. |
| **Steering Council** | Small odd-numbered body of Domain Leads + elected members. | Cross-cutting decisions; conflict resolution; roadmap; amending this document. |

Roles are earned through sustained, high-quality contribution and are listed in [`MAINTAINERS.md`](MAINTAINERS.md). No role is permanent; inactivity leads to emeritus status.

## 3. Decision instruments

| Instrument | Use it for | Lifecycle |
|-----------|-----------|-----------|
| **ADR** (Architecture Decision Record) | A single decision and its rationale. Short, immutable. | [`process/adr-process.md`](process/adr-process.md) |
| **RFC** (Request for Comments) | A substantial design spanning multiple decisions or subsystems. | [`process/rfc-process.md`](process/rfc-process.md) |
| **AEP** (Architecture Extension Proposal) | Defining or evolving a public extension contract (planner/runtime/compiler/package/channel APIs). | [`process/aep-process.md`](process/aep-process.md) |
| **Spec change** | Editing normative Book text. Must reference an Accepted RFC or ADR. | via PR + review gates |
| **Editorial change** | Typos, formatting, non-normative clarifications. | Lightweight PR, single maintainer approval. |

**Rule of thumb:** *One decision → ADR. A design containing many decisions → RFC (which spawns ADRs). A promise to third parties about an interface → AEP.*

## 4. The decision lifecycle

```
Idea → Draft (RFC/ADR/AEP) → Proposed → Review + Final Comment Period → Accepted → Reflected in spec/
                                    │
                                    └── Rejected / Withdrawn / Deferred (recorded, not deleted)
```

- **Final Comment Period (FCP):** Once a Domain Lead believes consensus is near, they call a **10-working-day FCP** with a disposition (accept / reject / postpone). Silence during FCP is assent. A single Steering Council member may extend or escalate.
- **Consensus test:** Accepted requires (a) approval from the owning Domain Lead, (b) at least two Reviewer approvals, (c) no unresolved *blocking* objection. A blocking objection must state a concrete technical harm, not a preference. *(While a domain is unstaffed this test cannot be met; it is replaced there by the interim process of §7 — not waived.)*

## 5. Review gates

No normative change merges without passing the applicable gates in [`process/review-gates.md`](process/review-gates.md): Architecture, Security, Performance, Testability, Documentation, Backward-Compatibility, and Migration. Gates scale with impact — an editorial fix clears them trivially; a new IR opcode does not.

## 6. Conflict resolution

1. Discuss in the RFC/PR thread.
2. Escalate to the Domain Lead.
3. Escalate to the Steering Council, which decides by simple majority. The Council *documents its reasoning* in an ADR.

## 7. Interim governance during bootstrap

*Provisional — pending [RFC-0012](rfcs/0012-interim-review-process.md) reaching `Accepted` under its own §4.2–4.5. Staged in the same change; this section settles (drops "provisional") when that cooling-off completes. See RFC-0012 §4.7, "Adoption sequencing" — the normative change must not go live before the instrument that legitimizes it clears its own gate.*

The roles and quorum in §2 and §4 assume the project is staffed. It is not yet: [`MAINTAINERS.md`](MAINTAINERS.md) records every Domain Lead as vacant and no Steering Council, so the §4 consensus test (Domain Lead + ≥2 Reviewers) and the §8 amendment rule are currently **unsatisfiable**. Until a domain is staffed, that domain operates under the **interim review process** defined in [RFC-0012](rfcs/0012-interim-review-process.md) and [`process/interim-review.md`](process/interim-review.md):

- Acceptance authority for an unstaffed domain is held **temporarily** by the founding maintainer (bootstrap authority), exercised **only** through a compensating-control package — a cooling-off period, an adversarial self-review, an independent adversarial pass, and provenance stamping — in place of the reviewers who do not yet exist.
- Every interim acceptance is logged in the [interim-acceptance ledger](process/interim-acceptance-ledger.md) and **owes an independent re-review before v1.0** — by a seated human reviewer or, per [RFC-0013](rfcs/0013-agent-reviewers-and-the-agent-review-quorum.md), an **independent agent review quorum** (recorded as `cleared (agent)`, with disclosed provenance). Agents unblock the review quorum; they do not fill Domain Lead or Steering Council seats or end the bootstrap period (RFC-0013 §4.7).
- The interim rules **lapse automatically, per domain**, the moment that domain records a Domain Lead + ≥2 Reviewers, and bootstrap authority ends project-wide once ≥3 Domain Leads seat a Steering Council.

This is a founding act, adopted where §8 cannot yet be satisfied, and bound to its own supersession. See RFC-0012 §4.7 for the reasoning, disclosed rather than hidden.

## 8. Amending governance

*The bootstrap carve-out in the final paragraph of this section is **provisional** — pending [RFC-0012](rfcs/0012-interim-review-process.md) acceptance under its own §4.2–4.5 (RFC-0012 §4.7).*

This document is amended only by RFC, a 15-working-day FCP, and a two-thirds Steering Council supermajority. Amendments are announced project-wide. **Until a Steering Council exists**, this rule is unsatisfiable; governance is amended under the bootstrap authority of §7 (RFC-0012), and any such amendment — including RFC-0012 itself — is subject to ratification, revision, or retirement by the Council once seated.
