# Book 07 · Chapter 04 — Consistency and Conflicts

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P4, P6. Companion to Book 02 §Ch08 (storage/consistency), Book 03 §Ch03 (Event Bus).*

> Reconciliation runs concurrently across many controllers and (with replication) many instances. This chapter specifies how consistency is maintained: optimistic concurrency, single-writer ownership, ordering relative to the Event Bus, and how cross-Resource consistency is achieved *without* transactions. These rules are what let a mesh of concurrent controllers converge to a coherent whole.

## 1. Optimistic concurrency, not locks

Controllers coordinate on shared Resources through **optimistic concurrency** (Book 02 §Ch08 §2), never locks:
- Every status write carries the `resourceVersion` the controller observed; the Resource Manager commits only if it still matches (Book 02 §Ch05 §1). A stale write is rejected with a conflict.
- On conflict, the controller **re-reads and re-reconciles** (§Ch01 §2) — because it is level-triggered, a conflict is not an error to escalate but a signal to re-observe and retry. This is why controllers must tolerate conflict as a normal outcome.
- Optimistic concurrency scales (no lock contention) and is safe (no lost updates) — the same properties that let the storage layer avoid cross-Resource transactions (Book 02 §Ch08 §4).

## 2. Single-writer ownership eliminates most conflict

The deepest conflict-avoidance is structural: **exactly one controller writes a given Resource's actual state** (§Ch01 §5, Book 02 §Ch03 §4), enforced by leader election across replicas (§Ch02 §5). Because there is one writer per Resource's status, most would-be conflicts simply cannot arise — two controllers never fight over one Resource's actual state. Conflicts remain possible only between a controller and (a) a `spec` writer (a different authority, different field) or (b) a brief leadership overlap (handled by `resourceVersion`, §1). Single-writer turns a hard distributed-consensus problem into a rare, backstopped edge case.

## 3. Ordering relative to the Event Bus

Controllers reconcile in response to Events, so ordering guarantees matter (Book 03 §Ch03 §5):
- **Per-subject ordering is guaranteed**: Events about one Resource arrive in order, so a controller sees a coherent progression of the Resource it owns.
- **Cross-subject ordering is NOT guaranteed**: Events about different Resources may arrive in any relative order. Controllers MUST NOT assume a global order (Book 03 §Ch03 §5).
- Because reconciliation is level-triggered (§Ch01 §2), cross-subject ordering *does not matter for correctness*: a controller reconciles from current state regardless of the relative order in which it learned of changes. Ordering assumptions would be a bug; level-triggering makes them unnecessary.

## 4. Cross-Resource consistency without transactions

Sankalpa has **no cross-Resource transactions** (Book 02 §Ch08 §4); consistency spanning Resources is achieved by reconciliation (§Ch01 §6):
- Where an invariant spans Resources, one controller makes a single-Resource change and other controllers converge in response (each writing *its own* Resources).
- This is **eventual consistency by construction**, with the transient gap **observable** via conditions and `observedGeneration` (Book 02 §Ch03 §5). A design MUST NOT assume two Resources change atomically; it expresses the desired cross-Resource state and lets controllers drive toward it, surfacing the transient inconsistency (Book 02 §Ch08 §4).
- **Ownership and finalizers** (§Ch03 §4, Book 02 §Ch06) provide the safe *ordering* of multi-Resource teardown that a transaction would otherwise be misused for.

## 5. Convergence guarantee

Despite concurrency, conflict, and unordered cross-subject events, the system **converges**: because every controller is level-triggered and idempotent (§Ch01), and state is the authoritative source with Events derived from it (Book 03 §Ch03 §6), repeated reconciliation drives every Resource to satisfy its desired state (or to a surfaced, explained blocked condition). Convergence does not depend on timing, ordering, or delivery reliability — only on controllers correctly implementing the model (§Ch03). This is the formal payoff of the whole design: correctness is a property of the *model*, not of getting concurrency timing right.

## 6. Invariants (normative summary)

1. Controllers coordinate via optimistic concurrency (`resourceVersion`), never locks; a conflict triggers re-read and re-reconcile, treated as normal, not as an error to escalate.
2. Single-writer ownership (one controller per Resource's status, leader-elected across replicas) structurally eliminates most conflicts; the remainder are backstopped by `resourceVersion`.
3. Per-subject Event ordering is guaranteed and cross-subject ordering is not; level-triggering makes cross-subject ordering irrelevant to correctness — assuming a global order is a bug.
4. Cross-Resource consistency is achieved by reconciliation (each controller writing its own Resources), yielding eventual consistency with an observable transient gap; no design assumes atomic multi-Resource change.
5. The system converges regardless of timing, ordering, or delivery reliability, provided controllers implement the level-triggered idempotent model — correctness is a property of the model.
