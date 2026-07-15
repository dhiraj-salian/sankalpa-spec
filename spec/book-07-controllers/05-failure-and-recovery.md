# Book 07 · Chapter 05 — Failure and Recovery

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P5. Companion to Book 03 §Ch13 (Kernel failure & degradation), §Ch10 (Controller Runtime).*

> Controllers fail — they crash, hang, or restart — and the system must recover without lost work or corrupted state. This chapter specifies controller failure and recovery: crash recovery from committed state, at-least-once reconcile semantics, and how controllers converge after partial failure. It is the controller-level expression of the Kernel's deterministic-recovery discipline (Book 03 §Ch13 §4).

## 1. Recovery leans on the model, not heroics

The reconciliation model (§Ch01) is *designed* to make recovery trivial, so recovery requires no special machinery — just the properties already guaranteed:
- **State is the source of truth; Events derive from it** (Book 03 §Ch03 §6). After any failure there is no ambiguous in-memory state to reconcile against storage — the committed Resource state is authoritative.
- **Reconcile is level-triggered and idempotent** (§Ch01 §2–3). So re-running reconcile after a crash — for the same or updated state — is always safe and correct.

Together these mean a controller can crash at any point and recover by simply *reconciling current state again*. There is no journal to replay, no partial-operation to unwind — the next reconcile observes reality and converges.

## 2. Crash recovery

When a controller (or its host) restarts (Book 03 §Ch10 §3.1):
1. The Controller Runtime **re-lists** the Resources the controller owns (from authoritative storage) and **re-enqueues** them for reconciliation.
2. Each is reconciled from **current state** (§Ch01 §2); because reconcile is idempotent, re-processing a Resource that was already converged is a no-op, and one that was mid-change is driven the rest of the way.
3. No trigger is "lost": a wake-up that arrived during the outage is subsumed by the re-list (the controller reconciles current state regardless of which events it did or did not see, §Ch04 §3).

Recovery is thus **complete** (every owned Resource is re-reconciled) and **safe** (idempotency makes re-reconcile harmless) — without any controller-specific recovery code.

## 3. At-least-once reconcile semantics

The runtime provides **at-least-once** reconcile (Book 03 §Ch10 §3.2), never exactly-once:
- A controller MUST assume `reconcile` may be invoked more than once for the same state — after a crash, a duplicate trigger, or a requeue. Idempotency (§Ch01 §3) is what makes this safe.
- External actions during reconcile use idempotency keys / existence checks (§Ch03 §2) so that an at-least-once reconcile does not produce at-least-once *external effects*. This mirrors the execution discipline (Book 04 §Ch04 §4): the whole platform is at-least-once, so everything downstream must be idempotent.

Exactly-once reconcile is neither promised nor needed; at-least-once + idempotent is the correct, achievable contract (Book 03 §Ch03 §4).

## 4. Partial failure within a reconcile

A single reconcile may itself fail partway (it created one downstream Resource but crashed before the second):
- On the next reconcile, the controller **observes what already exists** (via existence checks / idempotency keys) and completes only what remains — it does not blindly recreate. Level-triggering makes "what's already done" observable, and idempotency makes "finish the rest" safe.
- The controller **progresses or explains** (§Ch01 §4): if it cannot complete (a dependency is missing), it sets a condition and requeues with backoff (§Ch02 §4) rather than failing silently.
- Partial failure therefore never leaves corrupt or invisible state: the gap is observable (conditions, `observedGeneration`) and the next reconcile closes it.

## 5. Containment of a failing controller

A controller that crashes, hangs, or persistently errors is **contained** (Book 03 §Ch10 §3.2, §Ch13 §2):
- Its failure degrades *its own kind's* reconciliation, not the Controller Runtime or other controllers (domain isolation, Book 03 §Ch13 §2).
- Hangs are bounded by the runtime's limits/timeouts (§Ch02 §2); a runaway controller is terminated, not allowed to consume the host.
- A **plugin** controller (untrusted, Book 11 §10) is additionally sandboxed; its fault stalls its subjects with a surfaced condition rather than harming the core.
- The failure is attributed and audited (Book 11 §09) and feeds Experience (Book 10), so a chronically failing controller is visible to operators (Book 14 §Ch08).

## 6. Deterministic recovery

Because state is authoritative and reconcile is level-triggered and idempotent, recovery is **deterministic** (Book 03 §Ch13 §4): given the committed state, a recovered controller converges to the same place regardless of *how* it failed — crash, restart, partition, or duplicate. There is no failure-specific recovery path to get wrong, because there is only one path: reconcile current state. This is the controller-level guarantee behind the Kernel's promise that the system recovers deterministically.

## 7. Invariants (normative summary)

1. Recovery requires no special machinery: authoritative committed state plus level-triggered idempotent reconcile make re-reconciling current state a complete, safe recovery.
2. On restart, the runtime re-lists and re-enqueues owned Resources; each is reconciled from current state; no trigger is lost.
3. Reconcile is at-least-once; controllers assume multiple invocations for one state and use idempotency keys/existence checks so external effects are not duplicated.
4. Partial failure within a reconcile is completed on the next reconcile by observing what exists and finishing the rest; the gap is always observable, never corrupt or invisible.
5. A failing controller is contained to its own kind, bounded by limits, isolated if a plugin, and attributed/audited/surfaced to operators.
6. Recovery is deterministic: given committed state, a recovered controller converges to the same result regardless of how it failed.
