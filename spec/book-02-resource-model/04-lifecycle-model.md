# Book 02 · Chapter 04 — Lifecycle Model

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0004 (terminal conditions, not phases); realizes principles P3, P5, P6.*

> Every Resource has a lifecycle (P3). This chapter defines the kind-independent lifecycle: the `phase` enum, the legal transitions, finalizers, deletion semantics, and garbage collection. Kinds MAY define sub-states inside a phase but MUST map onto these phases.

## 1. Why a uniform lifecycle

`conditions` (Ch 02 §5) capture orthogonal facts; `phase` captures the single coarse answer to *"where in its life is this Resource?"*. A uniform phase vocabulary lets tooling, scheduling, retention, and audit treat every kind alike, while each kind refines the meaning of a phase through its `conditions` and `actualState`.

## 2. The phase model

`status.phase` **MUST** be one of the following. Every kind maps to this set; a kind **MUST NOT** invent a new top-level phase.

| Phase | Meaning | Terminal? |
|-------|---------|-----------|
| `Pending` | Admitted and persisted; the Controller has not yet begun (or cannot yet begin) reconciliation. | No |
| `Progressing` | The Controller is actively driving actual toward desired state. | No |
| `Active` | Converged and continuously maintained. The steady state for long-lived kinds. | No |
| `Succeeded` | Terminal success for one-shot kinds (e.g. an `Execution` that completed). | Yes |
| `Failed` | Terminal failure; the Controller has given up per the kind's failure policy. | Yes |
| `Terminating` | Deletion requested; finalizers running (§4). | Transitional → deleted |

Notes:
- Long-lived kinds (Service, Runtime, Policy) live in `Active`; they never reach `Succeeded`.
- One-shot kinds (Execution, Compilation) end in `Succeeded`/`Failed` and are later reclaimed (§5).
- `phase` is a summary; the authoritative fine-grained state is always `conditions` + `actualState`.
- **Outcomes finer than the two terminal phases are `conditions`, never new phases.** An Execution that rolled back is `Failed` with a `Compensated` condition; one stopped early is `Failed`/`Cancelled`; one left externally inconsistent is `Failed` with a `CompensationFailed` condition (RFC-0004, Book 06 §Ch03 §4). This is the direct consequence of "a kind MUST NOT invent a new top-level phase": vocabulary like *Compensated*/*Cancelled*/*CompensationFailed* lives in `conditions`, which is why consumers (Experience capture, audit) MUST read conditions, not just phase, to distinguish these.

## 3. Legal transitions

Controllers **MUST** respect this transition graph. Illegal transitions (e.g. `Succeeded → Active`) **MUST** be rejected/ignored by the Controller and never persisted.

```
                 ┌─────────────► Failed ◄────────────┐
                 │                                     │
 (create) → Pending → Progressing → Active            │
                 │          │           │             │
                 │          └────► Succeeded           │
                 │                                     │
 (delete req) ───┴──────────► Terminating ──► (removed)┘
```

Rules:
- Any non-terminal phase **MAY** transition to `Terminating` on a deletion request.
- Any non-terminal phase **MAY** transition to `Failed` per the kind's failure policy.
- `Succeeded` and `Failed` are terminal for actual-state mutation (Ch 03 §6); the only onward transition is to `Terminating` (deletion).
- Re-entry into `Progressing` from `Active` is legal and expected on drift or a `spec` change (Ch 03 §2).

## 4. Deletion and finalizers

Deletion is **not** immediate erasure; it is a reconciled process, so that a Resource can clean up external effects it created (revoke a capability grant, tear down a runtime graph, release a secret reference) before it disappears.

### 4.1 Requesting deletion
A delete request sets `metadata.deletionTimestamp` and moves `phase` to `Terminating`. The Resource **MUST** remain readable while terminating. Its `spec` **MUST NOT** be further mutated; only finalizer removal and `status` updates are permitted.

### 4.2 Finalizers
`metadata.finalizers` is an ordered list of named cleanup hooks. While it is non-empty, the Resource Manager **MUST NOT** physically remove the Resource. Each responsible Controller:

1. Observes `deletionTimestamp` set and its finalizer present.
2. Performs its cleanup **idempotently** (Ch 03 §3.2) — external teardown, event emission.
3. Removes *its own* finalizer.

When the last finalizer is removed, the Resource Manager physically deletes the Resource and emits a final `deleted` Event. A Controller **MUST NOT** remove a finalizer it does not own.

### 4.3 Stuck termination
If a finalizer cannot complete (e.g. an external system is unreachable), the Controller **MUST** keep the Resource in `Terminating` and surface a condition (`reason=FinalizerBlocked`). Forced removal is a privileged, audited operation (Book 11) and **MUST NOT** be the default path — silently dropping cleanup would leak external resources.

## 5. Retention and garbage collection

Terminal Resources (`Succeeded`/`Failed`) accumulate; they are reclaimed by policy, not left forever and not deleted instantly (their history feeds Experience, Book 10, and audit, Book 14).

- Each kind **MUST** declare a **retention policy**: how long terminal instances are kept, and what is preserved (often the derived Experience outlives the Execution Resource).
- **Owner-based cascading GC:** when a Resource is deleted, Resources that list it in `ownerRefs` (Ch 06) **MUST** be garbage-collected per their `ownerRef` deletion policy (`Cascade` | `Orphan`).
- GC itself proceeds through the deletion/finalizer path (§4); it is reconciliation, not a side channel.

## 6. Interaction with versioning

Lifecycle and versioning (Ch 05) are orthogonal: a `spec` change (new `generation`) does **not** reset the lifecycle; it triggers re-reconciliation within the current phase (typically `Active → Progressing → Active`). A Resource keeps one identity (`metadata.id`) across its whole life regardless of how many generations it passes through.

## 7. Worked example — an Execution's life

```
Pending        # admitted; scheduler has not placed it
Progressing    # runtime selected, graph executing; Events streaming
Succeeded      # terminal; actualState frozen; Experience produced (Book 10)
Terminating    # retention window elapsed; finalizer releases secret refs
(removed)      # last finalizer cleared; final "deleted" Event; Experience retained
```

## 8. Invariants (normative summary)

1. `status.phase` is one of the six defined phases; kinds do not invent new phases.
2. Only legal transitions are persisted; terminal phases do not mutate actual state.
3. Deletion is a reconciled, finalizer-gated process; a Resource is physically removed only when its finalizer list is empty.
4. Controllers run their own finalizers idempotently and remove only their own.
5. Terminal Resources are reclaimed by declared retention + owner-based cascading GC, both via the deletion path.
6. A `spec` change re-reconciles; it does not reset lifecycle or identity.
