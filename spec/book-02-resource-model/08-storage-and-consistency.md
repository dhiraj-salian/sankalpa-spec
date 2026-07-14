# Book 02 · Chapter 08 — Storage and Consistency

*Nature: **Normative** (properties), **Informative** (mechanism sketches). · Reflects: ADR-0001, ADR-0002; realizes principles P4, P5, P6, P10.*

> This chapter specifies the storage *properties* ARM requires — not a storage *product*. Following ADR-0001, the specification mandates guarantees; an implementation chooses a backend that provides them. This preserves the freedom to evolve or swap storage over a decade.

## 1. Why properties, not a product

Mandating a specific database (as Kubernetes effectively does with etcd) couples the platform's lifespan to that product's. Sankalpa instead states the invariants any conforming store **MUST** satisfy. A reference implementation will pick a concrete backend (Roadmap Phase 3–4); alternative implementations may pick others as long as they meet §2–§6.

## 2. Required consistency guarantees

A conforming ARM store **MUST** provide:

1. **Linearizable single-Resource operations.** A read of a Resource returns its latest committed state; a successful write is immediately visible to subsequent reads of that Resource. Per-Resource, there are no lost updates and no stale reads after commit.
2. **Optimistic concurrency via `resourceVersion`.** Every write carries the `resourceVersion` the client observed; the store commits only if it still matches, else rejects with a conflict (Ch 05 §1). This provides compare-and-swap without locks.
3. **Atomic status/spec separation.** A `spec` write and a `status` write are independently committed and independently capability-gated (P8); a `status` write never mutates `spec` or `generation` (Ch 02 §4).
4. **Durable, monotonic ordering per Resource.** Writes to a single Resource are totally ordered; `resourceVersion` is monotonic per Resource.

The store is **not** required to provide multi-Resource transactions (see §4).

## 3. Watch and the Event Bus

Consumers observe change through **watch**, which underpins level-triggered reconciliation (Ch 03 §3.1) and the Event Bus (P5, Book 03 §03).

- A watch **MUST** deliver changes such that a consumer can always recover the *current* state (level, not just edge): a watcher that reconnects **MUST** be able to resync from a known `resourceVersion` or via a full list + resume, without missing the latest state.
- Watch delivery is **at-least-once**; consumers **MUST** be idempotent (Ch 03 §3.2). Exactly-once is neither required nor assumed.
- Emitted domain Events (P5) are derived from committed writes; an Event **MUST NOT** be emitted for an uncommitted write, and every committed mutation **MUST** eventually produce its Event.

## 4. No cross-Resource transactions — and why

ARM deliberately does **not** offer multi-Resource ACID transactions. This is a design decision, not a limitation to be fixed:

- **Rationale.** Cross-Resource atomicity would centralize coupling in the store and make the control plane a distributed monolith. Instead, multi-Resource consistency is achieved by **reconciliation**: a Controller makes a single-Resource change and other Controllers converge in response (Ch 03 §4). This is eventually consistent by construction and matches the Kubernetes lesson ([prior-art study](../../research/prior-art/kubernetes.md)).
- **Consequence.** Designs **MUST NOT** assume two Resources change atomically. Where an invariant must span Resources, it is expressed as a *desired state* that a Controller drives toward, with conditions surfacing the transient inconsistency (P10) — never as an assumed atomic multi-write.
- **Ownership + finalizers** (Ch 04, Ch 06) provide the safe teardown ordering that transactions would otherwise be misused for.

## 5. History, event sourcing, and reconstruction

The Resource holds **current state only**; history lives in the Event stream (Ch 02 §6). This supports an event-sourced view without mandating event sourcing as the storage mechanism:

- The Event stream **MUST** be sufficient, together with the current state, to audit *how* a Resource reached its state (Book 14 §05) and to build Experience (Book 10).
- An implementation **MAY** use event sourcing internally (events as source of truth, state as projection) or a state-primary store that emits events on commit — either satisfies §2–§3 as long as events and committed state never disagree.
- Whatever the mechanism, a committed state and its Event history **MUST** be mutually consistent: no Event without a commit, no unaudited commit.

## 6. Secrets are never stored here (P7)

The ARM store holds Resources; it **MUST NOT** hold secret values. `Secret` Resources carry only reference metadata (Ch 06 §4, Ch 07 §5). Secret material is custodied exclusively by the Secret Broker's own protected store (Book 11 §04), under different controls. A conforming ARM store that is fully compromised **MUST NOT** thereby expose any secret value — because none is present to expose. This property is a primary reason secrets are references, not values.

## 7. Scale, retention, and tenancy

- **Retention.** Terminal Resources and their Events are retained per the kind's retention policy (Ch 04 §5); the store **MUST** support policy-driven reclamation without violating §5 (audit/Experience must survive as required).
- **Tenancy.** `workspace` (Ch 02 §2) is an isolation boundary; the store **MUST** enforce that reads/writes/watches are scoped to authorized workspaces (P8), so a compromised or buggy consumer cannot observe another tenant's Resources.
- **Scaling.** The store **SHOULD** scale horizontally with the number of Resources and watchers; §4's avoidance of cross-Resource transactions is what makes this tractable. Concrete scaling targets are set per implementation via a [performance review](../../templates/performance-review-template.md).

## 8. Invariants (normative summary)

1. Single-Resource operations are linearizable; concurrency is optimistic via `resourceVersion`.
2. `spec` and `status` writes are independently committed and capability-gated; `status` never touches `generation`.
3. Watch supports level-based resync and at-least-once, idempotent consumption; every committed mutation yields its Event, and no Event precedes a commit.
4. No cross-Resource transactions; multi-Resource consistency is achieved by reconciliation and surfaced via conditions.
5. Current state + Event history are mutually consistent and sufficient for audit and Experience.
6. The ARM store never holds secret values; a full compromise of it exposes no secret (P7).
7. Storage is scoped and access-controlled per workspace (P8).
