# RFC-0012: Interim review process for the bootstrap period

| Field | Value |
|-------|-------|
| **Status** | Proposed |
| **Authors** | Dhiraj Salian (founding maintainer) |
| **Domain / Book** | Governance / `GOVERNANCE.md`, `process/` |
| **Shepherd (Domain Lead)** | *Governance seat vacant — accepted under bootstrap authority this RFC defines (§4.7)* |
| **Created** | 2026-07-18 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | TBD |

> This RFC changes a governance rule and therefore requires an RFC ([`rfc-process.md`](../process/rfc-process.md) §"When an RFC is required"). It is **Proposed**, not Accepted: under the very process it defines it cannot be accepted until a cooling-off period elapses (§4.2). It documents the authority under which it will then be accepted, and does not pretend that authority is the normal quorum.

## 1. Executive Summary

Sankalpa's documented process requires, for any normative change, approval from the owning **Domain Lead** plus **≥2 Reviewers** ([`GOVERNANCE.md`](../GOVERNANCE.md) §4, [`rfc-process.md`](../process/rfc-process.md) §7). [`MAINTAINERS.md`](../MAINTAINERS.md) currently records **every** Domain Lead as vacant and the Steering Council as unseated, so that quorum is not merely unmet — it is **unsatisfiable**. The project has nonetheless produced eleven Final RFCs and a Draft-complete 16-Book specification, by a single maintainer, waiving the quorum ad hoc and in prose (e.g. RFC-0011: *"the ≥2-Reviewer gate waived … recorded here, not pretended"*). That practice is honest but undefined, inconsistent, and gives **nothing back** for the scrutiny it waives. This RFC defines an **interim review process** that (a) states the temporary **bootstrap authority** under which a solo maintainer may accept changes, (b) requires a **compensating-control package** — cooling-off, adversarial self-review, an independent adversarial pass, and provenance stamping — in exchange for the waived reviewers, (c) logs every interim acceptance to a standing **re-review ledger** that must be cleared by real reviewers before v1.0, and (d) **auto-lapses per domain** the moment that domain is properly staffed. It changes no `spec/` Book; it changes how changes are reviewed until the project has people to review them.

## 2. Problem Statement

The gap is between the process as written and the process as practiced.

- **As written** (GOVERNANCE §4): Accepted requires Domain Lead approval, ≥2 Reviewer approvals, a 10-working-day FCP, and no blocking objection. Amending governance itself (GOVERNANCE §7) requires an RFC, a 15-working-day FCP, and a **two-thirds Steering Council supermajority**.
- **As staffed** (MAINTAINERS.md): zero Domain Leads, zero Reviewers, no Steering Council. Both rules above are unsatisfiable by their own terms.
- **As practiced**: the founding maintainer authors, self-accepts, and records a prose waiver per artifact. RFC-0011 accepted on the **same day** it was proposed, shortening the FCP to zero and waiving the reviewer gate. The Phase 0 research was accepted by one person and — as its own [provenance note](../research/README.md#provenance-and-its-limits) records — a later audit found roughly three-quarters of its "open questions" already answered in text the author had not read.

The cost of leaving this undefined:

1. **No compensating scrutiny.** The waiver is pure subtraction: a reviewer is removed and nothing replaces the function reviewers serve (catching the author's blind spots). The Phase 0 audit is direct evidence that a solo author is *confidently wrong* in ways a second pass catches.
2. **Fictional roles in the record.** RFC-0011 names a "Compiler/Runtime Domain Lead" shepherd who does not exist. The record asserts approvals that no person gave.
3. **No bound and no exit.** Nothing says when solo authority ends, what ends it, or what happens to the pile of unreviewed-but-accepted material when reviewers finally arrive.
4. **The bootstrap paradox is unaddressed.** The rule for amending governance cannot be satisfied, so *no* governance change — including this one — is reachable by the book. Pretending otherwise is the alternative to naming it.

Doing nothing means continuing to accrue normative decisions with no scrutiny, no provenance, and no re-review obligation — the opposite of the specification-first discipline (ADR-0001) the project exists to uphold.

## 3. Alternatives Considered

1. **Do nothing — continue ad-hoc prose waivers.** *Pro:* zero process work; maximum velocity. *Con:* everything in §2 persists; the record keeps asserting approvals nobody gave; there is no re-review obligation, so the debt is invisible and unpayable. Rejected: the honesty the project already reaches for (the research provenance note, RFC-0011's "not pretended") deserves a mechanism, not a per-artifact improvisation.
2. **Freeze all normative work until reviewers exist.** *Pro:* no unreviewed decisions; maximally safe. *Con:* the project cannot recruit Domain Leads for a system that does not yet exist on paper — staffing *follows* a credible spec, so freezing is a deadlock. Rejected: it optimizes for a purity the bootstrap phase cannot afford.
3. **Lower the quorum by fiat** (e.g. "one approval suffices for now") **without compensating controls.** *Pro:* simple. *Con:* this is the current practice with a nicer label; it subtracts scrutiny and adds nothing. Rejected for the same reason as (1).
4. **Require a genuine outside reviewer for every change, found ad hoc.** *Pro:* real independent eyes. *Con:* stalls indefinitely whenever no volunteer is available — which is the defining condition of the bootstrap period; it makes progress hostage to luck. Considered and folded into the chosen design as the *preferred but not sole* form of the adversarial pass (§4.4).
5. **Chosen: bootstrap authority + a compensating-control package + a re-review ledger + per-domain auto-lapse.** *Pro:* keeps the project moving, gives real scrutiny back for the waived reviewers, records provenance, bounds itself, and pays down its own debt on a defined schedule. *Con:* more process than a solo author would otherwise run; the compensating controls are weaker than true independent review (acknowledged, §5). Chosen because it is the least-dishonest arrangement that does not deadlock.

Prior art: this is the standard *constitutional convention* move — a founding body adopts, under its own provisional authority, the minimum rules needed to operate until the permanent bodies exist (cf. the IETF's early "rough consensus" before formal structure; Rust's pre-RFC "the core team decides" period; the way most open-source projects run on a BDFL until a foundation is seated). None of these pretended a quorum they lacked; each named the interim authority and planned its own supersession.

## 4. Proposed Design

The interim process **replaces the unsatisfiable quorum** (GOVERNANCE §4's "Domain Lead + ≥2 Reviewers") for a domain, *only while that domain is unstaffed* (§4.6), with the rule below. It changes nothing else: every review gate ([`review-gates.md`](../process/review-gates.md)) still applies, the append-only record still holds, and "Final" still requires reflection into normative text.

### 4.1 Bootstrap authority

While a domain's Domain Lead seat is vacant, **acceptance authority for that domain is held by the founding maintainer** named in [`MAINTAINERS.md`](../MAINTAINERS.md) §Bootstrap. This authority is:

- **Temporary and self-limiting** — it lapses per domain on staffing (§4.6).
- **Not a Domain Lead role** — it confers acceptance authority under interim rules only, not the standing authority of a seated Lead, and it MUST be exercised through the compensating controls below.
- **Recorded** — every exercise of it is stamped (§4.5) and logged (§4.5), so the record never asserts an approval that a non-existent reviewer gave.

### 4.2 Cooling-off (MUST)

A change MUST NOT be accepted on the day it reaches `Proposed` (complete per template). At least **3 working days** MUST elapse between `Proposed` and `Accepted`. The author MUST NOT accept a change in the same working session in which they finalized its draft. This replaces the 10-working-day FCP, whose purpose (a window for community objection) is void when there is no community; the cooling-off window's purpose is to force a *second sitting* by the one person available, which is the minimum defense against same-session blindness. RFC-0011's same-day acceptance is precisely what this forecloses.

**Evidence, not honor.** A solo maintainer cannot be *prevented* from ignoring this window, but the elapsed time is **externally auditable**: the commit that sets `Proposed` and the merge commit that sets `Accepted` are timestamped in git history, and the interval MUST satisfy the window. A same-day or backdated acceptance is visible in the record. As with the MAINTAINERS.md control (§14), the defense is the public, tamper-evident log, not a promise.

### 4.3 Adversarial self-review (MUST)

The author MUST file an **interim self-review** ([template](../templates/interim-self-review-template.md)) that answers **every** applicable review gate (Architecture, Security, Performance, Testability, Documentation, Backward-Compatibility, Migration) with a concrete finding or an explicit **"N/A — because …"**. The self-review MUST be *adversarial in intent*: the author states the strongest case **against** their own change and what would falsify it. A self-review that only defends the change does not satisfy this section.

### 4.4 Independent adversarial pass (MUST)

At least one **adversarial review by a party other than the author** MUST be recorded in the PR:

- **Preferred:** a solicited outside reader, even if not a seated Reviewer.
- **Otherwise:** a documented **AI adversarial pass** — the tool, the prompt/instruction, and the raw findings recorded in the PR thread, with each finding either addressed in the change or dismissed with a stated technical reason.

A pass that reports no findings MUST state **what it looked for**; "looks good" is not a recorded pass. A finding of concrete technical harm is a **blocking objection** under GOVERNANCE §4's standard and MUST be resolved before acceptance. (An AI pass is explicitly *weaker* than a human reviewer and does not "count" toward the eventual ≥2-Reviewer quorum; it is a bootstrap compensating control, not a substitute for the role — see §5.)

**Independence of an AI pass (MUST).** An AI adversarial pass authored *in the same session, from the same context, as the artifact it reviews* is **not independent** — it shares the author's blind spots by construction. To qualify, an AI pass MUST be run as a **distinct pass whose task is to attack the finished artifact** (a separate session/instance, given the artifact and told to find harm), and its raw findings and their disposition MUST be recorded. This is still weaker than a human and is marked so; but a rubber-stamp from the same reasoning that produced the change is worth nothing and does not satisfy §4.4.

### 4.5 Provenance and the ledger (MUST)

- **Stamp.** Every artifact accepted under interim rules MUST carry, in its header/status line, `Reviewed under: interim process (RFC-0012)`. No reader may mistake it for independently reviewed.
- **Ledger.** Every interim acceptance MUST be entered in [`process/interim-acceptance-ledger.md`](../process/interim-acceptance-ledger.md): artifact, domain, date, accepting authority, the adversarial-pass form used, and re-review status (`pending`). The ledger is a **standing re-review queue**.
- **Re-review before v1.0.** A ledger entry is cleared only by a **seated Reviewer or Domain Lead** (not the founding maintainer acting under bootstrap authority). Every entry touching a Book MUST be cleared before that Book can pass the Phase 2 **v1.0 gate** ([`ROADMAP.md`](../ROADMAP.md): "two independent reviewers sign off per Book"). The interim process therefore **cannot by itself produce v1.0** — it produces material that remains owed a real review. Note the two axes are orthogonal: an RFC may be **Final** (reflected into normative text) *and* still `pending` in the ledger (not independently reviewed).
- **No self-clearance (MUST).** A ledger entry MUST NOT be cleared by the person who accepted it under bootstrap authority — **even after that person becomes a seated Reviewer or Domain Lead.** The whole value of the ledger is a *second* set of eyes; letting the bootstrap author later self-certify their own backlog would reopen at the exit exactly the independence gap the ledger exists to close. An entry authored *and* bootstrap-accepted by the same person needs a genuinely different reviewer. This is the one place the ROADMAP's "two *independent* reviewers" is load-bearing for the founding maintainer personally.

### 4.6 Exit — per-domain auto-lapse

- **Per domain.** The interim rules for a Book/domain lapse **automatically** the moment [`MAINTAINERS.md`](../MAINTAINERS.md) records a **Domain Lead and ≥2 Reviewers** for that domain. From that instant, changes in that domain use the full process (GOVERNANCE §4, rfc-process §7). No vote is needed; the trigger is the MAINTAINERS.md change, which is itself governed.
- **Project-wide.** The **bootstrap authority** (§4.1) ends when **≥3 Domain Leads** are seated, at which point a Steering Council is constitutible (GOVERNANCE §2). The Council SHOULD then **ratify, revise, or retire** this RFC under the normal amendment rule (GOVERNANCE §7) — its first legitimate act on the governance it inherited.
- **The ledger survives.** Auto-lapse does not clear ledger entries; each is cleared only by an actual re-review (§4.5). Staffing a domain enables its backlog to be paid down, it does not forgive it.

### 4.7 The bootstrap paradox, stated plainly

GOVERNANCE §7 requires a Steering Council supermajority to amend governance. No Council exists, so §7 cannot be satisfied for *any* governance change, including this one. This RFC is therefore adopted as a **founding act**: the founding maintainer, where no Council can yet be convened, adopts the minimum governance needed to operate legitimately, and binds that adoption to its own supersession (§4.6) once a Council exists. The circularity — using bootstrap authority to legitimize bootstrap authority — is inherent to standing up a constitution and is **disclosed here rather than hidden** inside a same-day acceptance. This RFC's own acceptance will follow its own §4.2–4.5: a cooling-off, a filed self-review, a recorded adversarial pass, a stamp, and a ledger entry.

### 4.8 High-stakes changes — no special treatment

Changes to **security invariants** (Book 11 / [`SECURITY.md`](../SECURITY.md)), the **AOS IR** (Book 04), and the **Kernel API** (Book 03) are accepted under the **same** interim rules as any other change — no extra gate, no deferral, no distinct status. This is a deliberate, owned decision. Its rationale: freezing exactly these areas would halt the Phase 2 hardening the project most needs, and the compensating-control package (§4.2–4.5) plus the mandatory pre-v1.0 re-review (§4.5) bound the risk. Its honest cost (§5): the most consequential and least-reversible decisions are being made in the lowest-scrutiny period; the Security gate's checklist (Book 11 §Ch11, SI-1…SI-13) and the ledger are the backstops, not a claim that solo review is sufficient for them.

## 5. Tradeoffs

**What we gain:** an honest, consistent, bounded way to keep designing while unstaffed; real scrutiny (a forced second sitting, an adversarial self-argument, an independent pass) in place of a bare waiver; a provenance trail so nothing masquerades as reviewed; a debt ledger that makes the bootstrap cost visible and payable; and a self-terminating design that hands authority to real reviewers automatically.

**What we give up / accept honestly:**
- The compensating controls are **weaker than independent human review**. A cooling-off is not a second reviewer; an AI adversarial pass is not a person with standing. This process reduces solo-author risk; it does not eliminate it, and §4.8 concentrates the hardest decisions in this window by the author's choice.
- **More ceremony** than a solo maintainer would otherwise run (a self-review artifact and a recorded pass per change). Mitigated by scaling with impact (§ editorial/research changes clear it trivially, as gates already do).
- The re-review ledger **defers, rather than avoids, the cost** of real review. That is the intended shape: pay a little now (compensating controls), pay the rest when reviewers arrive (ledger clearance before v1.0).

## 6. API Changes

N/A — because this RFC changes governance process only. No Kernel API, extension API, or wire format is affected.

## 7. Resource Changes

N/A — because no ARM Resource is added or changed. (The interim-acceptance ledger is a process document, not a Resource.)

## 8. Event Changes

N/A — because no Event on the Event Bus is added or changed.

## 9. Security Impact

No security **invariant** changes (SECURITY.md, Book 11 §Ch11 are untouched). There is a governance-security consideration: §4.8 permits security-invariant changes to be accepted under interim rules. This RFC does not weaken any SI-1…SI-13; it defines *who may accept a change to them and under what compensating scrutiny* while the Security Domain Lead seat is vacant. The Security gate ([`review-gates.md`](../process/review-gates.md) §2) still applies to every such change and is answered in the interim self-review; every such acceptance is stamped and ledgered for mandatory re-review by a seated Security reviewer before v1.0. Net: the *authority* to accept security changes is temporarily broadened to the founding maintainer under recorded controls; the *invariants* and their eventual independent review are preserved.

## 10. Performance Impact

N/A — because this is a process document with no runtime component. The only "performance" cost is human throughput: the cooling-off (§4.2) adds a minimum 3-working-day latency to each acceptance, deliberately.

## 11. Testing Strategy

Governance process is verified by **audit against the record**, not automated tests. Concretely: (a) CI already checks link integrity, template completeness, and glossary coverage ([`review-gates.md`](../process/review-gates.md) §Automation) and will check that every artifact bearing the interim stamp has a corresponding ledger entry (a mechanical cross-reference, addable to CI); (b) the ledger is itself the test oracle for the v1.0 gate — a Book with any `pending` entry cannot pass. A follow-up may add a CI check that fails if a stamped artifact is missing from the ledger or vice-versa.

## 12. Documentation Changes

- **New:** this RFC; [`process/interim-review.md`](../process/interim-review.md) (operational detail); [`process/interim-acceptance-ledger.md`](../process/interim-acceptance-ledger.md) (the ledger, seeded); [`templates/interim-self-review-template.md`](../templates/interim-self-review-template.md).
- **Amended:** [`GOVERNANCE.md`](../GOVERNANCE.md) (new §7 "Interim governance during bootstrap"; amendment section renumbered); [`process/README.md`](../process/README.md) (index); [`process/rfc-process.md`](../process/rfc-process.md) (§7 notes the interim substitution); [`process/review-gates.md`](../process/review-gates.md) (how gates are satisfied under interim rules); [`MAINTAINERS.md`](../MAINTAINERS.md) (a §Bootstrap naming the founding maintainer and this authority).
- **Glossary:** *Bootstrap authority, Interim review process, Compensating control, Interim-acceptance ledger.*

## 13. Migration Strategy

- **Existing accepted artifacts are grandfathered.** RFCs 0001–0011, ADRs 0000–0002, the 16 Books, and the Phase 0 research were accepted under *pre-RFC-0012* interim practice, without these compensating controls. They are entered in the ledger as a single **grandfathered class**, `pending` re-review, rather than retrofitted with self-reviews now (make-work the bootstrap phase cannot afford). They are already destined for the Phase 2 two-reviewer gate; the ledger simply makes that debt explicit.
- **This RFC is the first artifact accepted under the new rules** and demonstrates them end-to-end (Proposed today; self-review filed; adversarial pass recorded; acceptance only after cooling-off).
- **No dual-run window is needed** — the interim rules apply from acceptance; the full process resumes per-domain automatically on staffing (§4.6).

## 14. Risks

- **The controls become theater.** An author can file a shallow self-review and a rubber-stamp AI pass. *Mitigation:* the self-review MUST be adversarial and the pass MUST record what it looked for (§4.3–4.4); the ledger keeps the artifact visibly owing a real review regardless. Residual risk acknowledged: a determined solo author can still under-scrutinize. The ledger + v1.0 gate is the real backstop.
- **The ledger is ignored.** A backlog nobody clears. *Mitigation:* the v1.0 gate is hard — a Book with `pending` entries cannot be released; the ledger is not advisory. *Watching:* whether ledger clearance actually happens once reviewers arrive, or whether it is waved through.
- **Bootstrap authority overstays.** The founding maintainer keeps accepting after staffing would allow the full process. *Mitigation:* auto-lapse is triggered by the MAINTAINERS.md change, not by the maintainer's choice (§4.6). *Residual:* the maintainer also controls MAINTAINERS.md; the honest defense is the public record, not a lock.
- **§4.8 bites.** A security-invariant or IR change accepted solo turns out wrong and has propagated. *Mitigation:* the pre-v1.0 re-review and the standing invariant checklist; but this is the risk the author chose to accept over freezing the work. Named, not hidden.
- **Unknown unknowns:** whether an AI adversarial pass meaningfully catches what a human would (early evidence — the Phase 0 audit — is encouraging but is not a controlled comparison).

## 15. Future Improvements

- A **CI check** binding the interim stamp and the ledger (flag any stamped artifact absent from the ledger, or vice-versa).
- A lightweight **"Reviewer-of-record" registry** so that even a single recurring outside reader can be credited toward the ≥2-Reviewer quorum over time, easing the exit.
- **Tiered cooling-off** (longer for T3 / high-stakes, shorter for editorial) if a flat 3 days proves mis-calibrated.
- Guidance on **standing up the Steering Council** the moment 3 Domain Leads exist, so the exit (§4.6) is exercised promptly rather than drifting.

---
### Resolved questions
- *Do high-stakes changes get extra interim friction?* **No** (§4.8) — resolved by the founding maintainer: same rules as any change, with the ledger and the pre-v1.0 gate as backstops. The trade-off is stated in §5 and §14 rather than mitigated away.
- *Per-domain or project-wide exit?* **Per-domain auto-lapse** (§4.6), with a project-wide end to bootstrap authority at ≥3 Domain Leads.
- *What replaces the waived reviewers?* The **full compensating-control package** (§4.2–4.5), not a bare waiver.

### Unresolved questions
- The exact CI cross-check between stamp and ledger (§11, §15) is deferred to a follow-up editorial PR; it is not load-bearing for adopting the process.
- Whether an AI adversarial pass should ever be creditable toward the exit quorum (§15) — left open; currently it explicitly is not (§4.4).
