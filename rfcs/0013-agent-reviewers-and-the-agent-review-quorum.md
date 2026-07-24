# RFC-0013: Agent reviewers and the agent review quorum for the bootstrap period

| Field | Value |
|-------|-------|
| **Status** | Proposed |
| **Authors** | Dhiraj Salian (founding maintainer) |
| **Domain / Book** | Governance / `GOVERNANCE.md`, `process/` |
| **Shepherd (Domain Lead)** | *Governance seat vacant — accepted under bootstrap authority (RFC-0012 §4.7)* |
| **Created** | 2026-07-24 |
| **Amends** | RFC-0012 §4.4, §4.5, §4.8 (the reviewer-independence and high-stakes rules) |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | — · re-review tracked in the [interim-acceptance ledger](../process/interim-acceptance-ledger.md) |

> This RFC changes a governance rule and therefore requires an RFC ([`rfc-process.md`](../process/rfc-process.md)). It amends RFC-0012 rather than rewriting it (RFC-0012 is append-only, GOVERNANCE §1.5). Like RFC-0012, it is adopted under bootstrap authority (RFC-0012 §4.7) and subject to Steering Council ratification once one exists.

## 1. Executive Summary

RFC-0012 unblocked *acceptance* under a solo maintainer, but it left **v1.0 blocked on independent human reviewers who do not yet exist**: the ROADMAP Phase 2 gate requires "two independent reviewers sign off per Book," RFC-0012 §4.5 defines an independent reviewer as a "distinct, attributable **natural person**," and §4.8 requires a **named human** for security-invariant / AOS-IR / Kernel-API changes. With zero reviewers seated and no recruitment timeline, the specification would sit **Draft-complete and un-releasable indefinitely** — the very stall Alternative (2) of RFC-0012 was written to avoid, now relocated to the release gate.

This RFC defines a rigorous **agent reviewer** instrument and permits an **agent review quorum** — **≥2 independent agent reviewers** — to (a) clear interim-acceptance-ledger entries and (b) satisfy the ROADMAP v1.0 two-reviewer gate, **including high-stakes §4.8 changes**, in place of the human reviewers who do not yet exist. It does so under strict **independence**, **anti-steering**, and **provenance** rules whose non-negotiable core is: an agent-reviewed artifact — up to and including a v1.0 release — is **labeled as agent-reviewed** and never masquerades as human-independent review. It changes no `spec/` Book; it changes *who* may satisfy the review quorum while the project is unstaffed.

## 2. Problem Statement

- **The gate is unsatisfiable, not merely unmet.** Two independent humans per Book, zero seated, no timeline (`process/v1.0-readiness.md` P2). Solo work cannot substitute (RFC-0012 §4.5 bars self-clearance).
- **RFC-0012 deliberately excluded agents from the quorum.** §4.4: an AI pass "does not 'count' toward the eventual ≥2-Reviewer quorum." §4.8: a *named human* is required for the least-reversible changes. Those rules were correct for their purpose (an author's own single AI pass is weak and steerable) — but they also mean the project cannot reach v1.0 without people it does not have.
- **The cost of doing nothing** is a permanent Draft-complete plateau: the hardening the ROADMAP calls Phase 2 cannot be *ratified*, only accumulated, and the spec can never claim the stability that unlocks implementation (Phase 3+).
- **Agent review is not hypothetical scrutiny.** The project's own record shows adversarial agent passes catching real, security-relevant defects (the Phase 0 audit; the RFC-0010 secret-egress and RFC-0011 effect-retargeting findings). The question this RFC answers is not *whether* agents can find harm — they demonstrably do — but *under what discipline* their review may carry the weight of the quorum.

## 3. Alternatives Considered

1. **Wait for human reviewers (status quo).** *Pro:* preserves the human-independence guarantee. *Con:* indefinite stall with no timeline; the project cannot recruit reviewers for an unreleased spec any more than it could recruit Domain Leads (RFC-0012 Alt 2). Rejected: relocates the deadlock rather than resolving it.
2. **Agents as a mandatory pre-gate only, no quorum credit.** *Pro:* strengthens quality; keeps the human watershed. *Con:* does not unblock v1.0 at all. Rejected against the stated goal.
3. **Capped tier — an agent counts as at most *one* of the two reviewers, a human required as the second and for §4.8.** *Pro:* keeps one human in the loop for the stability claim. *Con:* still blocked whenever *no* human exists — which is the defining bootstrap condition. Considered and rejected by the founding maintainer as still-blocking; its human-in-the-loop discipline is retained as the **recommended, not required** form (§4.5, §4.6).
4. **Chosen: a disciplined agent review quorum that fully substitutes for human reviewers**, under independence + anti-steering + honest-provenance rules, with agent-cleared status kept distinct and upgradeable to human review. *Pro:* unblocks v1.0; supplies real adversarial scrutiny; is *more* auditable than human review (the whole transcript is recordable and reproducible). *Con:* correlated model blind spots and no accountable human behind the invariants (§5, §14). Chosen because it is the least-dishonest arrangement that does not stall — provided the provenance never lies about what reviewed it.

## 4. Proposed Design

The interim process gains a defined reviewer class — the **agent reviewer** — and a **quorum** of them may stand in for the human reviewers RFC-0012 §4.5 and the ROADMAP v1.0 gate require, *only while a domain is unstaffed by humans* (RFC-0012 §4.6 exit is unchanged). Everything else RFC-0012 requires (cooling-off, adversarial self-review, the ledger, the stamp) still applies.

### 4.1 What an agent reviewer is (and is not)

An **agent reviewer** is an AI system performing a **cold, adversarial review of a finished artifact**: a separate session/instance given only the artifact and its spec context and tasked to find concrete harm — the same shape as the independent adversarial pass RFC-0012 §4.4 already defines, elevated to a role that can carry quorum weight when the conditions below hold.

It is **not** the author's own in-session AI pass (RFC-0012 §4.4 "same session … not independent"), and it is **not** a Domain Lead or Steering Council seat (§4.7): it supplies *review*, never *accountability*.

### 4.2 Independence of agent reviewers (MUST)

The **≥2** agent reviewers counting toward a Book's quorum MUST satisfy:

- **Distinct sessions.** Each runs in its own cold session with only the artifact + cited spec context — never the session that authored the artifact or its self-review.
- **Distinct model families (SHOULD, strong).** The two reviewers SHOULD be from **different model families/vendors**, because correlated training distributions are the agent analogue of a shared blind spot. If they are the same family, that MUST be disclosed and a **third** independent reviewer of a different family is REQUIRED — two same-family agents alone do not meet the quorum.
- **Not the author.** No agent reviewer may be the model/session that generated the change under review.
- **Recorded identity.** Each reviewer's **model id and version** are recorded (a review is only reproducible if the reviewer is pinned; model drift otherwise silently invalidates it).

### 4.3 Anti-steering — evidence, not honor (MUST)

Because the founding maintainer runs the agent reviewers, the defense against a steered or curated panel is the **complete, public transcript**, mirroring RFC-0012 §4.2/§4.4:

- The **verbatim prompt/instruction** given to each reviewer is recorded — not paraphrased.
- **Every run is recorded**, not a chosen-favorable one (RFC-0012 §4.4 "no run-shopping" applies in full).
- **Raw findings and their disposition** are recorded in the PR and linked from the ledger; each dismissed finding carries a stated technical reason a later human can weigh.
- A panel whose prompt steers toward a clean result, or that hides adverse runs, **does not satisfy this section** and is a governance defect, not a pass. The transcript makes steering an attributable act, not a silent gap.

### 4.4 The agent review quorum (MUST)

A **cleared agent review** is **≥2 independent agent reviewers** (§4.2) with **no unresolved blocking finding** (a finding of concrete technical harm blocks until resolved, RFC-0012 §4.4). A cleared agent review is, for the purposes of ledger clearance (RFC-0012 §4.5) and the ROADMAP Phase 2 v1.0 gate ("two independent reviewers sign off per Book"), **equivalent to two independent reviewers** — with the provenance distinction of §4.6 always attached.

This **amends RFC-0012 §4.5**: "distinct, attributable natural persons" is broadened to "distinct, attributable **reviewers** — natural persons **or** independent agent reviewers meeting §4.2–4.3." The **no-self-clearance** rule survives unchanged: the founding maintainer may *operate* the agent reviewers but the reviewers must be independent of the authoring session, and the maintainer's own judgment is not itself a reviewer.

### 4.5 High-stakes changes (amends RFC-0012 §4.8)

For changes that introduce or alter a **security invariant (SI-1…SI-13), the AOS IR, or the Kernel API**, RFC-0012 §4.8 required a *named human or deferral*. This RFC replaces "required" with a **stronger agent panel**:

- **≥3 independent agent reviewers spanning ≥2 model families**, plus the **10-working-day cooling-off** (unchanged).
- A **named human reviewer remains RECOMMENDED** and, when available, MUST be recorded and counted — but is **no longer REQUIRED**; the change is no longer deferred solely for want of a human.
- The extra friction is retained because these decisions are the least reversible (`versioning-and-stability.md` MAJOR surface); flattening scrutiny here entirely would repeat the mistake RFC-0012 §4.8 was written to correct.

### 4.6 Provenance — the non-negotiable (MUST)

Full substitution is permitted **only** because it is disclosed. No reader may mistake agent review for human-independent review.

- **Artifact stamp.** An artifact cleared by agent quorum carries `Reviewed by: agent quorum (RFC-0013)` alongside its RFC-0012 interim stamp.
- **Ledger status.** A ledger entry cleared by agent quorum becomes **`cleared (agent)`**, distinct from `cleared (human)`; it records the reviewers (model ids/versions), the panel size, and a link to the transcript. `cleared (agent)` is **upgradeable**: a later human review may re-clear it to `cleared (human)`.
- **Release label.** A v1.0 reached via agent review MUST be published as **v1.0 with a disclosed `review basis: agent quorum`** in [`CHANGELOG.md`](../CHANGELOG.md), [`ROADMAP.md`](../ROADMAP.md), and [`process/v1.0-readiness.md`](../process/v1.0-readiness.md). The version number is not qualified (it is a real v1.0 under this governance), but the **review basis is stated wherever the release is described**. Erasing that qualifier over time is a provenance defect (§14).

### 4.7 Reviewers vs. roles — agents do not staff the project

Agent reviewers satisfy the **review quorum**; they do **not** fill Domain Lead or Steering Council seats, and they do **not** end the bootstrap period. RFC-0012 §4.6 is unchanged: bootstrap authority still lapses per domain on **human** staffing, and project-wide at ≥3 **human** Domain Leads. Agents unblock *review throughput*; **human accountability** for the specification remains the goal, not a box this RFC checks. This keeps the door open for the humans the project still needs and prevents "agent-reviewed" from being read as "governed."

### 4.8 Self-application

RFC-0013 is itself cleared under its own rules: an agent review quorum (§4.4) attacks the finished RFC, its transcript recorded, before it is accepted (after the RFC-0012 §4.2 cooling-off). It does not claim a human reviewed it.

## 5. Tradeoffs

**What we gain:** an unblocked, honest path to v1.0 that does not wait on people who do not exist; real adversarial scrutiny with a demonstrated record of catching security-relevant defects; and a review that is *more* auditable and reproducible than human review — the entire prompt, panel, and findings are recorded and re-runnable.

**What we give up / accept honestly:**
- **Correlated blind spots.** Independent humans fail independently; agents of similar training may fail *together*, missing an entire class of defect with false confidence. Different-family panels (§4.2) reduce but do not eliminate this.
- **No accountable human** stands behind an agent-cleared invariant. If it is wrong, there is a transcript, not a person with standing — and §4.8 changes are the least reversible.
- **Steering risk** is bounded only by transcript audit (§4.3), which no one may perform until a human arrives.
- **A v1.0 stability claim** would rest on agent judgment. This RFC keeps that *labeled* (§4.6); it does not pretend the label makes the judgment human-equivalent.

## 6. API Changes

N/A — governance process only; no Kernel API, extension API, or wire format is affected.

## 7. Resource Changes

N/A — no ARM Resource is added or changed.

## 8. Event Changes

N/A — no Event on the Event Bus is added or changed.

## 9. Security Impact

No security **invariant** changes (SECURITY.md, Book 11 §Ch11 untouched). The governance-security consideration is sharp and owned: **§4.5 permits the least-reversible changes (SI-1…SI-13, IR, Kernel API) to reach v1.0 with no human reviewer.** This is the strongest case against this RFC (it is exactly what the RFC-0012 adversarial review argued was unsafe). It is bounded, not dismissed: a ≥3-agent, ≥2-family panel plus the 10-day cooling-off; a **disclosed** agent-review basis so downstream implementers know a `cleared (agent)` Security Book was never seen by an accountable human; and an **upgradeable** ledger status so the first available human reviewer re-clears the security surface first. Net: the *authority* to clear security changes is extended to an agent quorum under recorded, disclosed controls; the *invariants* and their eventual human review are preserved as an outstanding, visible obligation.

## 10. Performance Impact

N/A — process document, no runtime component. Throughput cost: running a ≥2-family panel (≥3 for high-stakes) per artifact, plus the unchanged cooling-off.

## 11. Testing Strategy

Audit-against-record, plus mechanical CI: a stamped/`cleared (agent)` entry MUST have the required transcript artifacts (≥2 recorded model ids, ≥2 families or a disclosed third, all runs, verbatim prompt); a v1.0 release label MUST carry its `review basis` qualifier where described. These are addable to the same CI that RFC-0012 §11 defers.

## 12. Documentation Changes

- **Amended:** [`process/interim-review.md`](../process/interim-review.md) (agent quorum in the checklist), [`process/review-gates.md`](../process/review-gates.md) (agent reviewers answer the gates under quorum), [`process/interim-acceptance-ledger.md`](../process/interim-acceptance-ledger.md) (`cleared (agent)` status), [`process/v1.0-readiness.md`](../process/v1.0-readiness.md) (agents satisfy P2 and the reviewer columns, with provenance), [`GOVERNANCE.md`](../GOVERNANCE.md) §7, [`MAINTAINERS.md`](../MAINTAINERS.md) (an agent-reviewer standard).
- **Glossary:** *Agent reviewer, Agent review quorum, `cleared (agent)`.*

## 13. Migration Strategy

- **Existing ledger backlog** (the grandfathered class + RFC-0012/0013 entries) may now be cleared by agent quorum; each becomes `cleared (agent)` with its transcript, and remains upgradeable to `cleared (human)`.
- **No dual-run window** — the rule applies from acceptance; human staffing still reverts a domain to the full process (RFC-0012 §4.6).

## 14. Risks

- **Correlated blind spots.** *Mitigation:* different-family panels (§4.2); *residual:* real — agents can be confidently, jointly wrong. The upgradeable ledger is the backstop.
- **Steering / prompt-curation.** *Mitigation:* verbatim prompt + all runs recorded (§4.3). *Residual:* no one audits the transcript until a human arrives.
- **Provenance erosion.** The `review basis: agent quorum` qualifier gets dropped and a v1.0 is later read as human-reviewed. *Mitigation:* the qualifier is required in three places (§4.6) and CI-checkable (§11). *Watching:* whether it survives.
- **Over-trust by implementers.** Downstream code relies on an agent-cleared invariant as if human-vetted. *Mitigation:* the disclosed basis; the security surface re-cleared first when a human arrives (§9).
- **Model drift.** A reviewer's model changes and the review is no longer reproducible. *Mitigation:* record model id + version (§4.2).
- **"Good enough" lock-in.** Agent review being sufficient removes the incentive to ever recruit humans (§4.7's whole concern). *Mitigation:* agents never staff roles or end bootstrap; the human obligation stays visible in the ledger and readiness tracker. *Residual:* the incentive weakens regardless — named, not hidden.

## 15. Future Improvements

- A **human "reviewer-of-record"** path that upgrades `cleared (agent)` → `cleared (human)`, prioritizing the security surface.
- **Cross-vendor hardening** (require, not just recommend, ≥2 families for all quorum reviews once tooling exists).
- The **CI transcript/provenance check** (§11) bound to the interim stamp and ledger.

---
### Resolved questions
- *May an agent quorum fully replace human reviewers, including at v1.0 and for §4.8?* **Yes** — by founding-maintainer decision, under §4.2–4.6, with a human recommended (not required) for high-stakes and provenance always disclosed.
- *Do agents fill governance roles?* **No** (§4.7) — review only; bootstrap ends on human staffing.

### Unresolved questions
- Whether a same-family two-agent panel should ever suffice (currently no — §4.2 requires a third).
- The exact CI provenance check (§11) — deferred to a follow-up editorial PR; not load-bearing for adoption.
