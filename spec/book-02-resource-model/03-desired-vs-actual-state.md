# Book 02 · Chapter 03 — Desired vs. Actual State

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P5, P10. Companion to Book 07 (Controllers).*

> This chapter defines the reconciliation *contract* from the Resource's point of view: what desired and actual state mean, what convergence and drift are, and what a Controller owes a Resource. The *mechanism* (work queues, back-off, crash recovery) is specified in Book 07.

## 1. The two states

Every Resource carries two states (Ch 02):

- **Desired state** = `spec`. The intended condition, authored by the holder of the create/update capability.
- **Actual state** = `status.actualState`. The observed condition, authored only by the owning Controller.

The purpose of the entire control plane is to make actual state match desired state, continuously, in the face of change and failure. This is principle **P3** made operational.

## 2. Convergence and drift

Define, for a Resource `R` with owning Controller `C`:

- `R` is **converged** when `status.observedGeneration == metadata.generation` **and** `status.actualState` satisfies the intent expressed by `spec`, as judged by `C`. A converged Resource **SHOULD** report `Ready = True`.
- `R` is **reconciling** when `status.observedGeneration < metadata.generation`, i.e. `spec` changed and `C` has not yet acted on the new generation.
- `R` has **drifted** when it was converged but actual state has since diverged from `spec` without a `spec` change (external change, failure, expiry). `C` **MUST** detect drift and re-reconcile.

Convergence is **kind-specific** in its details but **uniform** in its shape: every Controller answers the same question — *"does actual satisfy desired for the current generation?"* — and records the answer in `observedGeneration` + `conditions`.

## 3. The reconciliation contract

A Controller `C` owning kind `K` **MUST** uphold the following. The full loop is specified in Book 07 §01; here is what it guarantees *to the Resource*.

### 3.1 Level-triggered, not edge-triggered
`C` **MUST** reconcile from the *observed current state*, not from the event that woke it. Missing, duplicated, or reordered wake-ups **MUST NOT** cause incorrect convergence. Rationale: events (P5) are notifications, not commands; correctness must survive an unreliable notification channel. This mirrors the Kubernetes controller model ([prior-art study](../../research/prior-art/kubernetes.md)).

### 3.2 Idempotent reconcile
Reconciling `R` when already converged **MUST** be a no-op with no external side effects. `C` **MUST** be safe to invoke arbitrarily many times for the same generation. Rationale: at-least-once delivery and crash recovery (Book 07 §05) will re-invoke reconcile.

### 3.3 Records what it observed
On each reconcile, `C` **MUST** set `status.observedGeneration = metadata.generation` once it has acted on that generation, update `status.actualState`, and update `conditions` (each with its own `observedGeneration`). `C` **MUST NOT** write `spec`, `generation`, or another kind's `status` that it does not own.

### 3.4 Emits events on transition
`C` **MUST** emit an Event (P5) on every meaningful transition — phase change, condition flip, actual-state change, and terminal success/failure — so that observers, Experience capture (Book 10), and audit (Book 14) see a complete history. Event payloads **MUST NOT** carry secret values (P7).

### 3.5 Progresses or explains
For every reconcile that leaves `R` un-converged, `C` **MUST** either make forward progress or set a condition explaining why it cannot (e.g. `Ready=False, reason=SecretUnavailable`). Silent non-progress is a defect. Un-actionable states **MUST** surface as `False`/`Unknown` conditions, never as an empty status.

## 4. Ownership: exactly one writer of actual state

Each Resource has **exactly one** owning Controller responsible for its `status`. Multiple controllers **MAY** read a Resource, but only its owner writes `status`. Rationale: two writers of actual state produce races and contradictory observations. Cross-Resource effects are achieved by a controller writing its *own* Resources and letting the other owners react — never by writing across ownership boundaries. Concurrency control for the single writer is specified in Ch 08 (optimistic concurrency via `resourceVersion`) and Book 07 §04.

## 5. Observability of the gap (P10)

Because desired and actual are both first-class and every transition emits an Event, the *gap* between them is always inspectable:

- `generation` vs. `observedGeneration` reveals pending reconciliation.
- `conditions` reveal *why* a Resource is or is not `Ready`.
- The Event stream reveals *how* it got there.

Tooling, dashboards (Book 13), and the Experience engine (Book 10) rely on this being uniform across all kinds. A kind that hides its gap (e.g. by overloading `spec` with observations, or by not setting `observedGeneration`) violates this chapter.

## 6. Ephemeral and one-shot Resources

Some kinds (e.g. a single `Execution`, a `Compilation`) reach a **terminal** actual state and never drift afterward. For these:

- Convergence is *terminal*: once `phase` is `Succeeded`/`Failed` (Ch 04), the Controller **MUST NOT** further mutate actual state.
- Re-reconcile after terminal **MUST** be a no-op (§3.2 still holds).
- Retention and garbage collection of terminal Resources are governed by Ch 04 and Ch 08, not by continued reconciliation.

This lets the same uniform contract serve both long-lived Resources (a `Service` that must be kept converged forever) and one-shot Resources (an `Execution` that converges once), without special-casing the model.

## 7. Invariants (normative summary)

1. Desired = `spec`; actual = `status.actualState`; convergence requires `observedGeneration == generation` and actual satisfying desired.
2. Reconciliation is level-triggered and idempotent.
3. Exactly one Controller writes a Resource's `status`; no controller writes across ownership.
4. Every reconcile records `observedGeneration`, updates actual state and conditions, and either progresses or explains non-progress.
5. Every meaningful transition emits a secret-free Event.
6. Terminal Resources stop mutating actual state and are reclaimed by lifecycle/retention, not by reconciliation.
