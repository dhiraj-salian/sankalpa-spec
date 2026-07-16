# Book 03 · Chapter 13 — Failure Modes and Degradation

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0004 (compensation failure), RFC-0007 (async-work shed); realizes principles P1, P3, P4, P7, P8, P9. Companion to Book 07 §05 (controller recovery), Book 14 §08 (operability).*

> A decade-scale platform is defined as much by how it *fails* as by how it works. This chapter specifies the Kernel's failure behavior: which invariants hold **even when the core itself is partially broken**, how the system degrades rather than collapses, and how it recovers. The governing rule: **fail safe, degrade gracefully, recover deterministically.**

## 1. The invariants never degrade

Some properties are **non-negotiable even under failure**. The Kernel MUST be designed so that a failure *removes capability*, never *relaxes an invariant*:

- **Secret-freedom (P7).** No failure mode may cause a secret value to enter IR, an Event, a log, or planner context. If the Secret Broker is unavailable, executions needing secrets **fail**; they do not proceed with a placeholder or a cached value in an unsafe place.
- **No ambient authority (P8).** If the Capability/Security Manager cannot verify authority, the request is **denied**, never allowed by default. Auth failure is fail-*closed*.
- **Policy enforcement (P9).** If policy cannot be evaluated, the guarded action is **refused**, not admitted. A plan that cannot be policy-validated is not lowered or executed.
- **Determinism (P1).** No failure/retry path may execute unverified IR or re-run a non-idempotent effect unsafely (§3).

These four are **fail-closed**: the safe state is *deny/stop*. The rest of this chapter is about degrading *availability* while preserving these.

## 2. Degradation model — shed, don't topple

Under overload or partial outage, the Kernel degrades in a defined order rather than failing globally:

1. **Backpressure first.** The Scheduler (§Ch07) and Kernel API (§Ch02 §4) shed new work rather than growing unbounded queues. Admitted work in flight is protected before new work is accepted. The two surfaces shed *deterministically and legibly*, differing only in mechanism: the API returns a typed `Busy`/`Unavailable`, while an already-persisted async unit is either **held with a surfaced `AdmissionPending` condition** or reaches terminal `Failed`/`Shed` when its deadline or backpressure budget is exhausted (§Ch07 §2.5). Neither surface may queue unboundedly, and neither may stall silently.
2. **Domain isolation.** A failing Manager degrades *its domain*, not the whole Kernel (§Ch04 §3). If the Compiler Manager is unavailable, existing Executions continue and new compilations queue/fail; reconciliation of unrelated Resources proceeds.
3. **Plugin containment.** A crashing/hanging plugin is contained by the Plugin Manager (§Ch09 §4): its work fails or reschedules per safety (§3), the Kernel and other plugins are unaffected.
4. **Read-mostly survival.** If the store's write path is impaired, the Kernel SHOULD continue serving reads/watches (observability, audit) while failing writes with `Unavailable`, so operators can see the system even when it cannot change it.

Degradation decisions are Events (P5) and feed Experience (Book 10) and operator signals (Book 14 §08).

## 3. Failure during execution — safety over completion

Executions touch the outside world, so their failure handling is the most delicate. The rules derive from the IR contract (Book 04 §Ch04 §4):

- **Retries require established idempotency.** The Kernel/runtime MUST NOT retry a `NonIdempotent` instruction; only `Idempotent`/`IdempotentWithKey` instructions retry (Book 04 §Ch08 §2.5). A failure of a non-idempotent write surfaces as terminal or triggers **compensation** (saga rollback), never a blind re-run.
- **Rescheduling is gated.** Moving an Execution to another runtime after a failure is permitted only when the Low IR's idempotency/compensation contract makes re-execution safe (§Ch07 §6).
- **Partial failure is explained, not hidden.** A partially-completed Execution reaches a terminal `Failed` with conditions describing what completed, what compensated, and what did not (Book 02 §Ch03 §3.5). Silent partial success is prohibited.
- **Compensation can itself fail.** When a mandated compensation fails after exhausting its bounded policy (the compensator errors, times out, or is non-idempotent), the Execution reaches terminal `Failed` with a distinguished **`CompensationFailed`** condition recording the residual inconsistency, and MUST escalate to a durable `RemediationTask` (Book 06 §Ch03 §4). This is the highest-consequence failure mode — irreversible external inconsistency — so it is made first-class, attributable, and operator-actionable rather than left silent.
- **At-least-once everywhere.** Because delivery and reconcile are at-least-once (§Ch03 §4, Book 02 §Ch03), *everything downstream must be idempotent*; this is why idempotency is a first-class IR concern rather than an afterthought.

## 4. Recovery — deterministic, from committed state

Recovery leans on the same properties that make the system correct in the first place:

- **State is the source of truth; Events derive from it** (§Ch03 §6). After a Kernel restart, there is no ambiguous in-memory state to reconcile against storage — the store is authoritative and Events are consistent with it.
- **Controllers re-list and re-reconcile** from current state (§Ch10 §3.1, Book 07 §05). Because reconcile is level-triggered and idempotent, no wake-up is "lost" and re-processing is safe.
- **Consumers resync** from a known `resourceVersion` (§Ch03 §4); no consumer silently misses the latest state.
- **Compilations are replayable by content hash** (§Ch08 §4); interrupted compilation re-runs from cached pass outputs.
- **Executions replay by Low IR hash + resolved bindings** (Book 04 §Ch04 §6) *only where idempotency permits*; otherwise recovery is via compensation, not replay.

Recovery is therefore **deterministic**: given the committed state, the system converges to the same place regardless of how it crashed.

## 5. Split-brain and single-writer

- The single-writer guarantee (Book 02 §Ch03 §4, §Ch10 §3.1) must survive Kernel replication. The Controller Runtime uses leader election so that, even during a network partition, **at most one** reconciler writes a given Resource's status. A partitioned replica that cannot confirm leadership MUST stop writing (fail-closed) rather than risk dual-write.
- Optimistic concurrency (`resourceVersion`, Book 02 §Ch08 §2) is the backstop: even a stale writer's commit is rejected, so a brief leadership overlap cannot corrupt state.

## 6. Failure taxonomy (summary)

| Failure | Safe behavior |
|---------|---------------|
| Secret Broker down | Executions needing secrets fail-closed; nothing proceeds unsafely (P7). |
| Security/Capability check unavailable | Deny (fail-closed, P8). |
| Policy uncheckable | Refuse guarded action (fail-closed, P9). |
| Store write path impaired | Fail writes `Unavailable`; keep serving reads/watches if possible. |
| Manager crash | Degrade that domain; isolate; others continue. |
| Plugin crash/hang | Contain; fail/reschedule its work per idempotency; Kernel unaffected. |
| Overload | Backpressure/shed with typed errors; protect in-flight work. |
| Kernel restart | Recover deterministically from committed state; controllers re-reconcile; consumers resync. |
| Partition / split-brain | Single-writer via leader election; non-leader stops writing; `resourceVersion` backstops. |
| Compensation itself fails | Terminal `Failed`/`CompensationFailed` condition recording residual inconsistency; best-effort across remaining compensations; mandatory escalation to a durable `RemediationTask` + high-severity Event + audit (Book 06 §Ch03 §4). |

## 7. Invariants (normative summary)

1. Security, secret, policy, and determinism invariants (P7, P8, P9, P1) are **fail-closed**: failure removes capability, never relaxes an invariant.
2. The system degrades in order — backpressure, domain isolation, plugin containment, read-mostly survival — rather than failing globally.
3. Retries and rescheduling occur only where idempotency/compensation make re-execution safe; partial failures reach explained terminal states, never silent partial success.
4. Recovery is deterministic from committed state: state is authoritative, Events derive from it, controllers re-reconcile idempotently, consumers resync.
5. Single-writer survives replication via leader election with a `resourceVersion` backstop; a replica that cannot confirm leadership stops writing.
