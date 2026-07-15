# Book 07 · Chapter 03 — The Controller Contract

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P5, P8, P11. This chapter specifies the **Controller SDK** surface (an AEP-governed interface for third-party controllers).*

> This chapter specifies the contract a controller MUST honor — the SDK surface it implements and the obligations it accepts. Because third parties ship controllers for their own kinds (via Packages, Book 12), this contract is an extension interface (AEP-governed, like the planner and runtime interfaces): honor it and your controller is hosted safely by the Controller Runtime (§Ch02).

## 1. The Controller SDK surface

A controller implements a small interface the Controller Runtime (§Ch02) drives:

```
Controller (Controller SDK):
  describe() -> ControllerDescriptor
        # which kind(s) it owns; required capabilities; supported apiVersions

  reconcile(subjectRef) -> ReconcileResult
        # the core loop (§Ch01): observe current state, drive actual→desired,
        # write own status, return {done | requeue(after) | error}

  finalize(subjectRef) -> FinalizeResult
        # cleanup on deletion (§4); idempotent; remove own finalizer when complete

  # Lifecycle
  init(grantSet, limits) / health() / shutdown()
```

- `reconcile` is invoked by the runtime with a subject id; the controller reads current state (it is *given the id, not the event* — level-triggered, §Ch01 §2) and acts.
- `reconcile` returns a result telling the runtime what to do next: `done` (converged), `requeue(after)` (revisit later — e.g. waiting on a dependency), or `error` (retry with backoff, §Ch02 §4).
- The controller **never** implements watching, queueing, coalescing, or leader election — the runtime provides those (§Ch02 §1). This keeps the contract small and the controller focused on domain logic.

## 2. Obligations of a controller

A conforming controller MUST:

1. **Be level-triggered** (§Ch01 §2): reconcile from observed current state, never assume the trigger carries the change.
2. **Be idempotent** (§Ch01 §3): reconciling converged state is a no-op; external actions use idempotency keys/existence checks so re-invocation does not double-act.
3. **Write only its own status** (§Ch04, Book 02 §Ch03 §4): update `observedGeneration`, `actualState`, and `conditions` for the kind it owns; never write `spec`, never write another kind's status.
4. **Progress or explain** (§Ch01 §4): every un-converged reconcile makes progress or sets an explanatory condition; never stall silently.
5. **Emit Events** (P5): meaningful transitions emit Events via the status writes the runtime reflects (Book 03 §Ch03).
6. **Act within its grants** (P8): perform effects only through capabilities it holds; a controller has no ambient authority.
7. **Handle deletion via `finalize`** (§4): run cleanup idempotently and remove its own finalizer.

These obligations are exactly §Ch01's model plus the security discipline; the SDK makes them the *contract*, and the conformance suite (§5) tests them.

## 3. What the controller receives and returns

- **Receives:** a subject id (to reconcile/finalize), its capability grants and limits (at `init`), and read access (via the Kernel API) to the Resources it is authorized for (§Ch02 §6, Book 11 §08).
- **Returns:** status updates for its own kind and a `ReconcileResult` directing requeue/backoff. It causes cross-Resource effects only by writing its *own* Resources (which other controllers react to, §Ch04) or by invoking capabilities it holds — never by reaching across ownership.

## 4. Finalizers and deletion

A controller participates in the finalizer-gated deletion process (Book 02 §Ch04 §4, Book 03 §Ch10 §2):
- When a subject it owns is being deleted (`deletionTimestamp` set) and its finalizer is present, the runtime invokes `finalize`.
- The controller performs its cleanup **idempotently** — teardown of external effects it created (revoke a grant, tear down a runtime graph, release a secret reference) — then removes **its own** finalizer.
- It MUST NOT remove a finalizer it does not own; when a finalizer cannot complete, it keeps the subject `Terminating` and surfaces a condition (`FinalizerBlocked`, Book 02 §Ch04 §4.3) rather than silently dropping cleanup (which would leak external resources).

## 5. Third-party controllers and conformance (P11)

- Controllers may be **core** (for core kinds) or **third-party** (shipped by Packages for their own kinds, Book 12). Third-party controllers implement this same SDK and are hosted identically — the mechanism does not distinguish, but the *trust* does: third-party controllers are untrusted, isolated, least-privilege (§Ch02 §6, Book 11 §10).
- A conformance suite (like those for runtimes/planners, Book 06 §Ch07, Book 08 §Ch07) tests a controller's obligations: level-triggered, idempotent, own-status-only, progress-or-explain, finalizer-correct, grant-bounded. A controller that passes is safely interchangeable and hostable (P11); the runtime does not trust a controller's claim — it verifies behavior.

## 6. Invariants (normative summary)

1. A controller implements a small SDK (describe/reconcile/finalize/init/health/shutdown); the runtime provides watching, queueing, coalescing, and leader election — not the controller.
2. `reconcile` is given a subject id (not an event) and returns done/requeue/error; the controller reads and acts on current state.
3. A conforming controller is level-triggered, idempotent, writes only its own status, progresses-or-explains, emits Events, acts within its grants, and handles deletion via idempotent finalizers.
4. A controller causes cross-Resource effects only by writing its own Resources or invoking held capabilities — never across ownership; it has no ambient authority (P8).
5. Controllers may be core or third-party; third-party controllers use the same SDK, are untrusted/isolated/least-privilege, and are validated by a conformance suite that tests behavior rather than trusting claims (P11).
