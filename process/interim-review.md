# Interim Review Process (bootstrap period)

*Status: Proposed · Reviewed under: interim process (RFC-0012) · Instrument owner: founding maintainer until a Steering Council is seated · Defined by: [RFC-0012](../rfcs/0012-interim-review-process.md).*

> This document is the operational companion to [RFC-0012](../rfcs/0012-interim-review-process.md). The RFC states *why* and *what*; this states *how to actually do it* in a PR. It is in force **only while a domain is unstaffed** (§4) and **lapses automatically** as domains are staffed. When the full process ([`GOVERNANCE.md`](../GOVERNANCE.md) §4, [`rfc-process.md`](rfc-process.md) §7) is satisfiable for a domain, use that instead.

## 1. When this applies

Use the interim process for a change **iff** the change's owning domain has **no seated Domain Lead** in [`MAINTAINERS.md`](../MAINTAINERS.md). If a Domain Lead and ≥2 Reviewers are seated for that domain, the interim process does **not** apply there — use the full process.

Editorial changes (typos, links, formatting) never needed the quorum and still don't: one maintainer merges, no ledger entry. **But the tier decision is not the author's private call.** Under the full process a Domain Lead confirms the tier ([`review-gates.md`](review-gates.md)); with that seat vacant, the author proposes a tier **and records it in the PR**, and the independent adversarial pass (§2) is told the proposed tier and MAY challenge it. A pass arguing that a change classed T0/T1 "editorial" in fact alters normative meaning is raising a **blocking objection** — the change is re-tiered and takes the compensating controls its true tier demands. Down-tiering to escape the package is thus a challengeable, recorded act, not a silent exit.

## 2. The interim acceptance checklist

A normative change (RFC, ADR, AEP, or spec-text PR) is **Accepted under interim rules** only when **all** of the following are true and visible in the PR:

- [ ] **Proposed** — the artifact is complete per its template (no blank/TODO sections).
- [ ] **Cooling-off** — ≥ **3 working days** have elapsed since it reached `Proposed`, and acceptance is a *separate sitting* from drafting. *(RFC-0012 §4.2.)*
- [ ] **Adversarial self-review filed** — the [interim self-review template](../templates/interim-self-review-template.md) is filled: every applicable review gate answered with a finding or "N/A — because …", **including the strongest case against the change**. *(§4.3.)*
- [ ] **Independent adversarial pass recorded** — a solicited outside reader if one can be found; otherwise a documented AI adversarial pass (tool + verbatim instruction + **all** raw runs, not a curated one), each finding addressed or dismissed with a stated technical reason. A no-findings pass states what it looked for. *(§4.4.)*
- [ ] **High-stakes: a named outside human** — if the change introduces or alters a **security invariant, the AOS IR, or the Kernel API**, an AI pass does **not** suffice: a named human must be recorded, the cooling-off is **10 working days**, and if no human is available the change is **`Deferred`, not accepted**. *(§4.8.)*
- [ ] **No unresolved blocking objection** — a finding of concrete technical harm blocks acceptance until resolved.
- [ ] **Provenance stamped** — the artifact's header carries `Reviewed under: interim process (RFC-0012)`. *(§4.5.)*
- [ ] **Ledger entry added** — a row in [`interim-acceptance-ledger.md`](interim-acceptance-ledger.md), status `pending`. *(§4.5.)*

Only then may the founding maintainer, acting under **bootstrap authority** (RFC-0012 §4.1), mark the artifact `Accepted` and merge. Reflection into normative text advances it to `Final` as usual — but `Final` does **not** clear the ledger entry (§4).

**Clearing a ledger entry (RFC-0013).** A `pending` entry is cleared by an independent re-review — a seated human (`cleared (human)`) or an **agent review quorum** (`cleared (agent)`): **≥2 independent agent reviewers** in distinct cold sessions, of distinct model families (or a disclosed third, RFC-0013 §4.2), never the authoring session, with the verbatim prompt, all runs, model ids/versions, and findings recorded and linked. High-stakes entries (security invariant / AOS IR / Kernel API) need the stronger **≥3-agent, ≥2-family** panel (RFC-0013 §4.5). The clearing party is never the founding maintainer's own judgment.

## 3. The review gates, under interim rules

The gates in [`review-gates.md`](review-gates.md) are unchanged in *what* they ask. What changes is *who answers them*: instead of the gate owners (who don't exist yet), the **author answers each gate adversarially in the self-review**, and the **independent adversarial pass** probes those answers. This is a weaker instrument than the named owners and is recorded as such. A T3 change (new IR op, Kernel API change, security-invariant change) still requires the filled security/performance/testing artifacts the gates demand — the interim process does not waive them, it only changes who signs them off — and, per [RFC-0012 §4.8](../rfcs/0012-interim-review-process.md#48-high-stakes-changes--a-named-outside-human-or-defer), a T3 change to a **security invariant, the AOS IR, or the Kernel API** additionally requires a **named outside human** (an AI pass does not suffice) or is deferred.

## 4. Exit

- **Per domain:** the moment [`MAINTAINERS.md`](../MAINTAINERS.md) records a Domain Lead + ≥2 Reviewers for a domain, this process stops applying there; that domain reverts to the full process. Automatic, no vote. *(RFC-0012 §4.6.)*
- **Project-wide:** bootstrap authority ends when ≥3 Domain Leads are seated (a Steering Council becomes constitutible); the Council should then ratify, revise, or retire RFC-0012 under [`GOVERNANCE.md`](../GOVERNANCE.md) §8.
- **The ledger outlives the exit:** entries are cleared only by an actual re-review, never by staffing alone.

## 5. Relationship to v1.0

The interim process **cannot produce a v1.0 spec**. The Phase 2 gate ([`ROADMAP.md`](../ROADMAP.md)) requires two independent reviewers to sign off per Book; every ledger entry touching a Book must be cleared by a seated reviewer first. Interim acceptance keeps the design *moving*; it does not make it *stable*. Those are different claims, kept separate on purpose.
