# Prior-Art Study: PostgreSQL

*Status: Accepted · Informs: Books 05 (compiler, cost-based optimization), 12 (extensions), 02 §Ch08 (storage and consistency).*

## 1. System in one paragraph
PostgreSQL is the long game made visible: three decades of a project that grew enormously in capability while breaking its users almost never. Two things in it are worth studying. Its **cost-based query planner** takes a declarative statement of *what* the user wants and chooses *how* to execute it, using statistics gathered from real data — the closest prior art to a compiler that improves plans from observed evidence. Its **extension system** let the ecosystem add types, index methods, and entire foreign-data subsystems without forking the core, and did it without the extension surface becoming a liability.

## 2. Core ideas
1. **Declarative in, plans out.** SQL says what; the planner decides how. The user never writes an execution strategy — the same inversion as Intent → IR.
2. **Cost-based optimization over statistics.** The planner enumerates alternatives and picks by estimated cost from collected statistics — a *model* of reality, refreshed by `ANALYZE`.
3. **Extensibility built into the type system.** New types, operators, index access methods, and aggregates are first-class registrations, not patches — an ambition set in the original Postgres research papers.
4. **`EXPLAIN` as a first-class product.** The plan is inspectable, and `EXPLAIN ANALYZE` shows estimate vs. actual — the system tells you where its model was wrong.
5. **MVCC.** Readers never block writers; each transaction sees a consistent snapshot.
6. **The write-ahead log as the state of record**, with replication and point-in-time recovery falling out of it.
7. **Stability as culture.** Extensive review, a slow release cadence, and a strong reluctance to break compatibility — features get rejected for years until they are right.

## 3. Design decisions & trade-offs
- **Cost-based planning** buys good plans across wildly different data shapes, at the cost of *estimate error*: bad statistics produce bad plans, and the failure is silent, non-local, and maddening to debug. The planner is only as good as its model of reality.
- **Statistics over real data** buys adaptivity, at the cost of plan instability — the same query can regress overnight because the data moved.
- **In-process C extensions** buy full power and speed, at the cost of no isolation: a bad extension segfaults the database. The ecosystem's power and its fragility have the same cause.
- **MVCC** buys concurrency, at the cost of bloat and vacuum — a maintenance burden paid forever for a benefit taken continuously.
- **Slow, conservative evolution** buys trust that compounds for decades, at the cost of losing features to competitors for years at a time. This trade is why it won.

## 4. Relevance to Sankalpa
Two things. First, Book 05: the planner is the only mature prior art for *choosing among semantically equivalent execution strategies using evidence gathered from production* — which is precisely what our optimization passes (§Ch03) and determinization (§Ch06, Book 10 §Ch06) attempt. Second, Book 12: PostgreSQL is the counter-example to the belief that a rich extension ecosystem must destabilize a core, and its `EXPLAIN` culture is the model for our compiler diagnostics (Book 05 §Ch07) and Compilation record (§Ch08).

## 5. What we adopt
- **Declarative-in, plan-out** as the shape of the whole system (Book 01 §Ch02): the user states Intent, the machine chooses execution. SQL is the proof this inversion is usable by non-experts at scale.
- **Evidence-driven optimization.** Statistics gathered from real executions feeding plan choice is the direct ancestor of the Experience loop (Book 10) and determinization's evidence thresholds (Book 10 §Ch06 §3) — do not synthesize a Capability from one observation any more than you would plan from one row.
- **`EXPLAIN` as an obligation.** The compiler produces a record, not just an artifact (Book 05 §Ch01 §5); diagnostics are a specified surface (§Ch07). `EXPLAIN ANALYZE`'s estimate-vs-actual is exactly the drift check RFC-0003 makes of a determinized Capability.
- **The optimizer is a *guess* and must be treatable as one.** PostgreSQL's hard-won lesson is that a wrong plan needs an escape hatch and an explanation. Determinization reversibility (Book 10 §Ch06 §5) is our version.
- **Extension without forking the core** (P11) — new types, new methods, registered through declared interfaces (Book 12 §Ch02 §2).
- **Stability as culture, enforced by process.** Slow, reviewed, compatibility-preserving evolution (`process/versioning-and-stability.md`); PostgreSQL is the evidence that this compounds rather than costs.
- **A log as the state of record** with reconstruction from it (Book 02 §Ch08 §5, Book 14 §Ch01 §5's no-dual-write rule).
- **Snapshot semantics for readers** informing Book 02 §Ch08 §2's consistency guarantees.

## 6. What we reject / change
- **In-process, unisolated extensions.** This is PostgreSQL's one clear anti-pattern for us: a C extension has full process privilege, so the ecosystem is safe only because it is small, expert, and audited by convention. Our Packages come from a marketplace of strangers (Book 12 §Ch06), so isolation (Book 11 §Ch10) and explicit capability grants (P8) are mandatory — we want the extensibility without inheriting the trust model that made it work.
- **Superuser.** `CREATE EXTENSION` requiring superuser, and extensions then holding it, is ambient authority in its purest form (Book 11 §Ch03 §1). Install ≠ privilege (Book 12, RFC-0008).
- **Cost estimates as the only signal.** The planner guesses cost from statistics *before* running. We have something PostgreSQL does not: recorded outcomes of prior real executions (Book 10 §Ch03), so our "statistics" are history, not extrapolation — evidence thresholds (Book 10 §Ch06 §3) rather than a cost model.
- **Silent plan regression.** A Postgres plan can change overnight with no announcement. Our compilations are content-addressed, cached, and recorded (Book 05 §Ch01 §4–5); a changed plan is an evented, attributable fact (P5, P10), not a surprise in the latency graph.
- **Optimization without a governance checkpoint.** The planner optimizes for cost alone; our pipeline puts a mandatory policy pass between optimization and lowering (Book 05 §Ch01 §2), and no pass may bypass it.
- **A mandated storage engine.** We specify storage *properties* (Book 02 §Ch08 §1); MVCC informs the guarantees, it is not a requirement.
- **SQL's compatibility burden.** PostgreSQL carries decades of syntax it cannot remove. Our deprecation path is governed and finite (`versioning-and-stability`).

## 7. Open questions
- **Should Intent or Policy be able to constrain plan choice?** PostgreSQL's most-debated refusal is query hints: it holds that a wrong plan is a planner bug to fix, not a knob to expose, and pays for that position in occasional user agony. Our analogous surface is real — Book 06 §Ch04 §3.2 already lets policy *pin* a runtime (and fails closed if the pinned one is unfaithful), which is a hint by another name at the selection layer. Whether the same should exist at the *planning* layer, and whether that reopens the "the human writes the plan" door P1 and Book 01 §Ch02 deliberately closed, is unasked.

*Checked and answered, contrary to an earlier draft of this study: the drift question — our evidence apparatus is materially stronger than `ANALYZE`, not a longer-feedback-loop re-derivation of it. Book 10 §Ch06 §5 confronts the exact trap (once a `Reasoning` node is substituted the model is no longer called, so there is no fresh result to diverge from) and answers with **shadow sampling** (RFC-0003): a `shadowSource` on the substituted invocation, a policy-governed sampling rate with a normative floor > 0 for high-consequence effect classes, comparison under the *same* equivalence relation and stability bound used at synthesis (raw hash inequality would retire healthy Capabilities on noise), version-bucketed observations so a model upgrade opens a re-validation window rather than mass-retiring the workspace's accrued asset, and retirement falling back to the original reasoning. `ANALYZE` has no analog to any of that. Marketplace quality floor (Book 12 §Ch06 — all three: verified publisher identity and signed, provenance-attested artifacts (§Ch04), conformance badges as verifiable claims (§Ch02 §6), publishing validation that MAY require passing declared suites, plus moderation).*

## 8. References
- Stonebraker & Rowe, "The Design of POSTGRES" (1986); the PostgreSQL planner/optimizer documentation and `EXPLAIN` docs; Selinger et al., "Access Path Selection in a Relational Database Management System" (1979); PostgreSQL extension-building documentation; the project's release-policy and compatibility conventions.
