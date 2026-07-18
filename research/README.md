# Research (Phase 0)

*Status: Accepted (Phase 0 gate met — see [Provenance and its limits](#provenance-and-its-limits)) · Nature: Informative — research is **input** to the specification, never normative on its own.*

Phase 0 of the [roadmap](../ROADMAP.md) is deliberate, disciplined study before design. We do not invent what has been solved; we learn from the systems worth learning from, and we are explicit about **what we adopt, what we reject, and why**. Research documents feed RFCs and ADRs; they do not themselves change the spec.

## Provenance and its limits

Two facts about how these studies came to exist. Both weaken the evidentiary weight of the set, and a reader should know them before relying on it.

**They were authored and accepted without independent review.** Every Domain Lead in [`MAINTAINERS.md`](../MAINTAINERS.md) is vacant and the Steering Council is unseated, so no independent reviewer existed to accept them. The studies were drafted in a single pass and accepted by the founding maintainer. Research is non-normative and additive, and [`CONTRIBUTING.md`](../CONTRIBUTING.md) subjects it to lighter review than normative text — but "accepted" here means *one person judged them sound*, not that they cleared the adversarial reading the [review gates](../process/review-gates.md) impose on `spec/`.

*What that cost, concretely:* the studies were first drafted against chapter **outlines** rather than chapter **text**, and their **Open questions** sections were consequently near-worthless — a subsequent audit against the full text found roughly three quarters of them already answered, several emphatically, and one study's headline objection (that determinization sits in unresolved tension with IR-P10) flatly contradicted by Book 05 §Ch06 §3, which draws the distinction the study claimed was missing. Those sections have since been rewritten: each now carries only questions verified against the cited text, and closes with a *"Checked and answered"* note pointing at the chapter that resolves what an earlier draft got wrong. Those notes are deliberate — a question the spec has already answered is worth recording as a pointer, and the record of having been wrong is worth more than a silent deletion. The **adopt/reject** sections were re-checked in the same pass. Treat this as calibration: the set was confidently wrong in a way one reading fixed, which is what "no independent review" buys you.

**They were written after the specification, not before it.** The exit criterion below says Phase 1 begins only once these are accepted. That is not what happened: `spec/` reached Draft-complete and Phase 2 hardening produced Final RFCs while `research/` held one exemplar study. So these documents did not *inform* the design in the way the roadmap intends — they were reconstructed against a design already made, and a study written to explain a decision is a weaker instrument than one written to inform it. Where a study reports agreement with the spec, discount it accordingly; where it raises an objection anyway (see the microkernel study on core size, or the CQRS study questioning its own inclusion), that objection survived the bias and is worth more.

Neither fact is a reason to redo the work. Both are reasons to treat the set as a **map of the design's intellectual debts**, which is what it is good for, rather than as evidence the design is correct — which it cannot be, having been written last.

## Structure

- [`prior-art/`](prior-art/README.md) — studies of specific systems (Linux, LLVM, Kubernetes, Temporal, Git, Terraform, Docker, Cloudflare Workers, PostgreSQL, Rust).
- [`patterns/`](patterns/README.md) — studies of architectural patterns (compiler design, DDD, CQRS, hexagonal, microkernel, event sourcing, capability-based security).

## Exit criterion for Phase 0

Every study listed in the [prior-art](prior-art/README.md) and [patterns](patterns/README.md) indexes reaches an accepted state with a completed **"What we adopt / What we reject / Why"** section. Only then does Phase 1 (authoring the spec) formally begin. Studies may be revised later as understanding deepens.

**Met, out of order.** All 18 studies are accepted and carry the required sections. The gate's *sequencing* intent was not honored — Phase 1 proceeded ahead of it — and no amount of backfilling repairs that; see [Provenance and its limits](#provenance-and-its-limits).

## Study template

Each study MUST contain:

1. **System/pattern in one paragraph** — what it is and the problem it solves.
2. **Core ideas** — the 3–7 ideas that make it work.
3. **Design decisions & trade-offs** — the consequential choices its designers made.
4. **Relevance to Sankalpa** — which layer(s)/Book(s) it informs.
5. **What we adopt** — concrete ideas we take, and where.
6. **What we reject** — ideas that do not fit, and why.
7. **Open questions** — what it raises for our design.
8. **References** — primary sources.
