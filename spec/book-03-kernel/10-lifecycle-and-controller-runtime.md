# Book 03 · Chapter 10 — Lifecycle Manager and Controller Runtime

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P5. Companion to Book 07 (Controllers), Book 02 §Ch04 (Lifecycle).*

> Principle **P3** — everything has a lifecycle and a controller — is realized by two collaborating pieces of the core: the **Lifecycle Manager** enforces the kind-independent lifecycle rules (phases, finalizers, deletion, GC); the **Controller Runtime** hosts and schedules the reconcilers that do the domain-specific work. Book 07 specifies how to *write* a controller; this chapter specifies the *Kernel machinery that runs them*.

## 1. Division of labor

- **Lifecycle Manager** — the *rules*: which phase transitions are legal (Book 02 §Ch04 §3), how finalizers gate deletion (§Ch04 §4), and how retention/cascading GC reclaim terminal and owned Resources (§Ch04 §5). It is kind-independent and enforces these uniformly so no controller can, e.g., skip finalizers or perform an illegal transition.
- **Controller Runtime** — the *engine*: it hosts Controllers (core and plugin-provided), watches Resources on their behalf, delivers reconcile triggers, and applies scheduling discipline (work queues, rate limiting, back-off, leader election).

Together they make reconciliation (Book 02 §Ch03) an operational reality rather than a per-kind reinvention.

## 2. Lifecycle Manager

### 2.1 Responsibilities
1. **Transition enforcement** — validate every `phase` change against the legal transition graph (Book 02 §Ch04 §3); reject illegal transitions (e.g. `Succeeded → Active`).
2. **Deletion orchestration** — on a delete request, set `deletionTimestamp`, move to `Terminating`, and hold physical removal until all finalizers clear (§Ch04 §4).
3. **Finalizer discipline** — ensure each responsible controller runs its finalizer idempotently and removes only its own; surface stuck termination as a condition rather than force-dropping cleanup (which would leak external resources).
4. **Retention & GC** — apply each kind's retention policy and owner-based cascading GC (§Ch04 §5) through the deletion path, never as a side channel.

### 2.2 Rules
- Physical removal happens **only** when the finalizer list is empty; the Lifecycle Manager MUST NOT bypass this except via the explicit, privileged, audited force-delete path (Book 11).
- A `spec` change re-reconciles within the current phase; it MUST NOT reset lifecycle or identity (Book 02 §Ch04 §6).
- Every transition and deletion emits an Event (P5), keeping the lifecycle observable and auditable.

## 3. Controller Runtime

### 3.1 Responsibilities
1. **Host** Controllers — core reconcilers and plugin-provided ones (via the Controller SDK, Book 07 §03), each isolated per its trust level (plugin controllers are untrusted, §Ch09 §2).
2. **Watch & trigger** — subscribe to the Event Bus (§Ch03) on each controller's behalf and deliver reconcile triggers for the kinds/subjects it owns.
3. **Work queues** — per-controller queues keyed by subject `id`, coalescing duplicate triggers so a burst of Events collapses into a single reconcile of current state (level-triggered, §Ch03 §4).
4. **Rate limiting & back-off** — bound reconcile frequency and apply exponential back-off with jitter on repeated failure, protecting the Kernel and external systems from hot loops.
5. **Leader election / single-writer** — ensure exactly one active reconciler owns a given Resource's `status` at a time (Book 02 §Ch03 §4), even across replicated Kernel instances, so there are no dual writers.
6. **Crash recovery** — after a restart, re-list and re-reconcile owned Resources from current state; because reconcile is idempotent and level-triggered, no trigger is "lost" (Book 07 §05).

### 3.2 Guarantees to a controller
The Controller Runtime guarantees each hosted controller:
- **At-least-once, coalesced triggers**: it will be woken for changes to its subjects; duplicates are collapsed; it must reconcile from observed state, not trigger contents.
- **Single active ownership**: no other reconciler concurrently writes its subjects' status.
- **Bounded execution**: rate limits and back-off prevent runaway loops; a slow/failing reconcile is retried with back-off, not spun.
- **Isolation**: a plugin controller's fault is contained; its subjects' reconciliation stalls with a surfaced condition rather than crashing the runtime (§Ch13).

## 4. Interaction

```
Event (Ch03) ─► Controller Runtime work queue (coalesced, per subject id)
      │  (leader-elected single owner; rate-limited)
      ▼
Controller.reconcile(subject)   # level-triggered, idempotent (Book 07)
      │  observes current state, drives actual→desired, writes own status
      ▼
Kernel API UpdateStatus / Invoke ─► Resource Manager (Ch05) ─► Event (Ch03) ─► ...
      │
Lifecycle Manager: validates transitions, runs finalizers on delete, GC
```

The loop is closed and uniform: controllers act through the Kernel API (P4), the Resource Manager commits and emits (§Ch05), the Event Bus re-triggers as needed (§Ch03), and the Lifecycle Manager polices transitions and teardown throughout.

## 5. Why this belongs in the core

Reconciliation machinery is *mechanism*, shared by every kind; putting it in the core (rather than reimplementing per controller) is what makes P3 uniform and cheap. Controllers themselves may be plugins (a Package can ship a controller for its own kinds, Book 12), but the *runtime that hosts them* and the *lifecycle rules they must obey* are core and non-replaceable — otherwise a plugin could evade finalizers, dual-write, or hot-loop the system. This is the §Ch01 §4 litmus test applied to reconciliation.

## 6. Invariants (normative summary)

1. Phase transitions are validated against the legal graph; illegal transitions are rejected; every transition/deletion emits an Event (P3, P5).
2. Physical deletion occurs only when finalizers are clear; stuck termination surfaces as a condition, never a silent forced drop (except the audited privileged path).
3. The Controller Runtime delivers coalesced, at-least-once, level-triggered reconcile triggers per subject; controllers reconcile from observed state.
4. Exactly one active reconciler owns a Resource's status at a time, even across replicas (no dual writers); reconciles are rate-limited with back-off.
5. Crash recovery re-lists and re-reconciles from current state; no trigger is lost because reconcile is idempotent and level-triggered.
6. Lifecycle rules and the Controller Runtime are core and non-replaceable; hosted controllers may be untrusted plugins and are isolated accordingly.
