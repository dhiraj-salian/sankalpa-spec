# Book 07 · Chapter 01 — The Reconciliation Model

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P5. Companion to Book 02 §Ch03 (desired vs. actual), Book 03 §Ch10 (Controller Runtime).*

> Principle **P3** — everything has a lifecycle and a controller — rests on one idea: **reconciliation**. A controller continuously drives a Resource's actual state toward its desired state. Book 02 §Ch03 specified this contract from the *Resource's* point of view; this Book specifies it from the *controller's* point of view. This chapter fixes the reconciliation model that every controller in Sankalpa obeys — the mechanism that keeps the system correct under change and failure.

## 1. The reconcile loop

A controller runs a loop for each Resource it owns:

```
observe   — read the Resource's current desired state (spec) and actual state (status.actualState)
diff      — compute the gap: does actual satisfy desired for the current generation?
act       — take the steps that move actual toward desired (via Kernel API / Invoke)
emit      — update status (observedGeneration, actualState, conditions) and emit Events
```

This is the whole model. Its power is in the discipline around it (§2–§4), not its complexity. Every controller — the Intent controller (Book 08), the Knowledge sync controller (Book 09 §Ch05), the Package install controller (Book 12), the Execution controller (Book 06) — is a specialization of this one loop.

## 2. Level-triggered, not edge-triggered

The single most important property (Book 02 §Ch03 §3.1): a controller reconciles from the **observed current state**, not from the event that woke it. The Event is a *hint that something may have changed*; it is never a command carrying the change.

Consequences (all load-bearing):
- **Robust to unreliable delivery.** The Event Bus is at-least-once and per-subject-ordered but not globally ordered (Book 03 §Ch03 §4). A missed, duplicated, or reordered wake-up cannot cause incorrect convergence, because the controller re-reads current state and acts on *that*.
- **Self-healing.** If actual state drifts (external change, failure, expiry) without any event, the next reconcile still detects and corrects it (Book 02 §Ch03 §2).
- **No lost updates.** There is no "I missed the event so I missed the change" — the truth is the Resource, and the controller always re-reads it.

This is the Kubernetes lesson (Book 02 §Ch01, prior-art study): correctness lives in re-reading state, not in perfect event delivery. Sankalpa adopts it wholesale.

## 3. Idempotent reconcile

A reconcile MUST be **idempotent** (Book 02 §Ch03 §3.2): reconciling an already-converged Resource is a no-op with no external side effect, and reconciling the same state repeatedly is safe. Rationale:
- At-least-once delivery (§2) and crash recovery (§Ch05) *will* re-invoke reconcile for the same state; if that were not safe, the system would double-act.
- Idempotency is what lets the controller be woken freely (coalesced, retried, replayed) without harm.

Where an action has an external effect (creating a downstream Resource, invoking a Capability), the controller uses idempotency keys / existence checks so that a repeated reconcile does not duplicate the effect — the same discipline the IR enforces for execution (Book 04 §Ch04 §4).

## 4. Progress or explanation (P5, observability)

Every reconcile that leaves a Resource un-converged MUST either **make forward progress** or **set a condition explaining why it cannot** (Book 02 §Ch03 §3.5): `Ready=False, reason=SecretUnavailable`, etc. Silent non-progress is a defect. This is what makes the system observable (P10) and debuggable: the gap between desired and actual is always visible, and *why* it persists is always stated (Book 14 §Ch08). A controller that stalls without explanation hides a problem; a controller that stalls *with* a condition surfaces it.

## 5. Single writer of actual state

Exactly **one** controller owns a given Resource's `status` (Book 02 §Ch03 §4). Multiple controllers may *read* a Resource, but only its owner *writes* its actual state. Cross-Resource effects are achieved by a controller writing its *own* Resources and letting other owners react (§Ch04) — never by writing across ownership. This prevents races and contradictory observations, and it is enforced by the Controller Runtime's single-active-owner guarantee (Book 03 §Ch10 §3.1).

## 6. Reconciliation is how multi-Resource consistency is achieved

Sankalpa has no cross-Resource transactions (Book 02 §Ch08 §4). Reconciliation is the answer: where an invariant spans Resources, one controller makes a single-Resource change and others converge in response. The whole intent→execution pipeline is a chain of reconciliations — an Intent controller derives Goals, a planning step produces IR, a compilation controller compiles it, a scheduler places it, a runtime executes it — each a controller reacting to the state the previous one produced. This is why reconciliation is not just a substrate detail but the *organizing principle* of how work flows through the system.

## 7. Invariants (normative summary)

1. Every controller runs the observe → diff → act → emit loop; every controller in the system is a specialization of this one model.
2. Reconciliation is level-triggered: the controller acts on observed current state, never on event contents; this makes it robust to unreliable delivery and self-healing against drift.
3. Reconcile is idempotent; repeated reconciliation of the same state is a safe no-op, using idempotency keys/existence checks for external effects.
4. Every un-converged reconcile makes forward progress or sets a condition explaining why not; silent non-progress is a defect (P5, P10).
5. Exactly one controller writes a Resource's actual state; cross-Resource effects flow through each owner reacting to others' Resources, never cross-ownership writes.
6. Reconciliation is how multi-Resource consistency is achieved (there being no cross-Resource transactions) and is the organizing principle of how work flows through the pipeline.
