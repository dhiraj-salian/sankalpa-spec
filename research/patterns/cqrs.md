# Pattern Study: CQRS

*Status: Accepted · Informs: Books 03 (Kernel API), 14 (observability), 02 §Ch03 (desired vs. actual).*

## 1. Pattern in one paragraph
CQRS (Command Query Responsibility Segregation) generalizes Bertrand Meyer's command–query separation from methods to architecture: **the model you write through and the model you read from need not be the same model**. Writes go to a model shaped for validating invariants; reads are served from projections shaped for the questions asked, updated asynchronously. The payoff is that read and write scale, evolve, and are optimized independently; the price is that reads are stale, and the staleness is now the application's problem — and the user's.

## 2. Core ideas
1. **Separate the write model from the read model.** One enforces invariants; the others answer questions. Neither compromises for the other.
2. **Commands express intent and can be rejected;** queries are side-effect-free and never change state. Mixing them is what CQS forbids at method level and CQRS forbids at system level.
3. **Projections/read models** are derived, disposable views — rebuildable from the write model's authoritative record.
4. **Asynchronous propagation** from write to read, which makes eventual consistency explicit rather than accidental.
5. **Reads scale independently** — different stores, denormalization, caching, all without touching the write path.
6. **It pairs naturally with event sourcing** ([study](event-sourcing.md)) — the event stream is a convenient thing to project from — but neither pattern requires the other.

## 3. Design decisions & trade-offs
- **Two models** buy independent optimization and scaling, at the cost of a second model to build, keep correct, and evolve — and of two places to look when data is wrong.
- **Async projections** buy write throughput and read flexibility, at the cost of read-your-own-writes being broken by default: the user submits a change and does not see it, which is a UX problem no amount of architecture diagrams solves.
- **Rebuildable projections** buy operational forgiveness (a bad projection is a bug, not a data-loss event), at the cost of a rebuild path that must actually be maintained and exercised.
- **The pattern's real cost is complexity in the wrong place:** CQRS applied to a domain with simple invariants is pure overhead, and it is one of the most over-applied patterns in the catalog. Its author has spent years telling people not to use it.

## 4. Relevance to Sankalpa
Two levels. Structurally, the Kernel API is command/query separated: `Create`/`Update`/`Delete`/`Invoke` are commands that can be rejected by the request pipeline; `Get`/`List`/`Watch` are queries (Book 03 §Ch02 §2). Conceptually, the deeper CQRS-shaped decision is **desired vs. actual state** (Book 02 §Ch03): `spec` is the write model (what was asked for), `status` is a read model (what is), and they have **separate write paths with separate capabilities** — a component that observes cannot redirect intent (Book 03 §Ch02 §2.1). Book 14's metrics, traces, and audit are projections of the event stream, exactly in the CQRS sense.

## 5. What we adopt
- **Command/query separation in the Kernel API verb set** (Book 03 §Ch02 §2): a small, uniform verb vocabulary where the effectful verbs are explicit and gated (`Invoke` is "the *only* way to cause an effect through the Kernel"), and queries are non-mutating.
- **Separate write paths for `spec` and `status`, gated by separate capabilities** (Book 03 §Ch02 §2.1). This is CQRS's separation used for *authority*, not scale, and it is the more valuable use: it is what stops a Controller reporting observations from also being able to rewrite the intent it is reconciling toward (P8).
- **Projections as derived, rebuildable views.** Metrics, traces, logs, and audit views (Book 14) are projections; the Event stream plus ARM is the record (Book 02 §Ch08 §5).
- **Derive events from committed state, never dual-write** (Book 14 §Ch01 §5) — the discipline that keeps a projection honest.
- **Eventual consistency made explicit.** Book 02 §Ch08's guarantees state what is linearizable (single-Resource operations) and what is not, rather than letting callers assume.
- **Commands are rejectable and typed.** Every Kernel command passes authentication, capability check, admission, and policy before executing (Book 03 §Ch02 §3), returning typed errors (§4) — a command that cannot be *refused* is not a command.

## 6. How faithfully we apply it (and where we deviate)
- **We adopt the separation, not the two-datastore architecture.** ARM is a single authoritative store (Book 02 §Ch08), and `Get` is linearizable — not a projection served from an async replica. We take CQRS's *conceptual* split (spec vs. status, command vs. query) and refuse its *physical* one for the control plane, because a control plane that cannot read its own writes is a control plane whose controllers fight each other.
- **`status` is not an async projection.** It is written by the owning Controller through `UpdateStatus` and read linearizably. It is CQRS-shaped in *authority* (a separate write path, separate capability) but not in *propagation*.
- **Watch is level-based, not a projection feed.** Kubernetes-style resync (Book 02 §Ch08 §3) means consumers reconcile from observed state rather than depending on a complete, ordered projection — a level-triggered stance that is deliberately more forgiving than CQRS's edge-triggered projection updates.
- **Reconciliation replaces the write→read propagation.** The gap between "what was asked" and "what is" is not projection lag; it is a *real physical gap* that a Controller is actively closing (P3, Book 07 §Ch01). CQRS's staleness is an artifact of the architecture and is regrettable; ours is a fact about the world and is the domain. This is the deviation that matters: we borrow the vocabulary and the authority split, and our eventual consistency comes from reconciliation theory, not from CQRS.
- **No command bus, no handler/aggregate ceremony.** Those are implementation-shaped, and Book 00 §Ch05 keeps the spec above that.
- **Where genuine read-model divergence exists, it is admitted as such:** the knowledge vault ⇄ graph (Book 09 §Ch05) is two *authoritative* representations reconciled bidirectionally (Book 15 §Ch04) — explicitly not a write model plus a projection, which is why RFC-0006 needs base-version stamping rather than a rebuild.

## 7. Open questions
- **Is CQRS load-bearing here at all?** This study's own answer, after §6, is largely no: `spec/` gets its command/query split from resource-oriented API design ([Kubernetes](../prior-art/kubernetes.md)), its `spec`/`status` split from the capability model (P8), and its eventual consistency from reconciliation (P3) — none of which needs CQRS to explain it. Book 01 §Ch06 does not list CQRS among the influences, and that omission is defensible. The pattern's one genuinely load-bearing contribution is the *authority* separation of the two write paths (Book 03 §Ch02 §2.1), which CQRS names better than reconciliation theory does. Keeping the study in the index is a claim that naming it is worth something; that claim is arguable and should be argued rather than assumed.
- **Audit is a projection that cannot be rebuilt, which makes it a record.** Book 11 §Ch09 §6 says audit is "retained per policy… often longer than the Resources they describe" and Book 02 §Ch04 §5 permits Resource reclamation. So the audit projection deliberately outlives its sources — which means it is not a disposable fold over an authoritative log (§2.3) but a *record* in its own right, with every obligation that implies (tamper-evidence, Book 11 §Ch09 §4; erasure, see the [event sourcing study](event-sourcing.md) §7). CQRS's "projections are rebuildable" premise does not hold for the projection that matters most, and the spec is right not to pretend otherwise — but the consequence is that the pattern's operational forgiveness is unavailable exactly where one would want it.
- **No read path is specified as serving from an async projection.** Book 14 §Ch08's operability surfaces state no staleness bounds, which is fine only if nothing serves stale. Worth confirming rather than assuming, since it is the difference between "CQRS-shaped in authority" (§6) and CQRS with unstated lag.

## 8. References
- Meyer, *Object-Oriented Software Construction* (command–query separation); Young, "CQRS Documents"; Fowler, "CQRS" (bliki) and his cautions on applicability; Young's later public warnings against over-application; the [event sourcing](event-sourcing.md) study.
