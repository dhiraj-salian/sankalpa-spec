# Book 03 · Chapter 07 — Scheduler and Runtime Manager

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0001, RFC-0007 (admission liveness); realizes principles P4, P8, P9, P11. Companion to Book 06 (Runtimes).*

> These two Managers turn compiled IR into scheduled, placed execution. The **Scheduler** decides *whether and when* work runs under capacity and policy; the **Runtime Manager** decides *where* — which Runtime plugin executes it — and owns runtime lifecycle. A defining rule lives here: **the runtime is selected after planning**, never before (RFC-0001).

## 1. Why runtime selection comes last

A planner emits runtime-agnostic High IR (Book 04 §Ch03); the compiler produces runtime-agnostic Low IR (Book 04 §Ch04). Only once the *what* and the *how-mechanically* are fixed does the system choose *where* to run. This ordering is load-bearing:
- It keeps planners ignorant of runtimes (P1, P11) — a planner never encodes "use n8n."
- It lets the **same** Low IR target different runtimes and produce observably equivalent executions (Book 04 §Ch04 §7, IR-P10) — the portability guarantee.
- It lets selection use information available only late: current capacity, cost, policy, and runtime health.

## 2. The Scheduler

### 2.1 Responsibilities
1. **Run-admission** — decide whether a unit of work (a `Compilation` or an `Execution`, Book 02 §Ch07) may *start now*, given capacity, quotas, and policy (§2.3).
2. **Placement queueing** — order and gate pending work; apply fairness across workspaces/tenants (§2.4).
3. **Backpressure** — when the system is saturated, **hold or shed** work deterministically rather than overcommit (§2.5, §Ch13).
4. **Deadlines & priorities** — honor per-work deadlines and priorities **without starving low-priority tenants** (§2.4, §2.5).

> **"Admission" has two senses; do not conflate them.** *Store-admission* is the write succeeding — the Resource exists and its `phase` is `Pending` (Book 02 §Ch04 §2, whose wording "Admitted and persisted" refers to this). *Run-admission* is this Scheduler's decision to advance `Pending → Progressing`. §2.3–2.5 govern the gap between the two: a Resource can be store-admitted for some time before it is run-admitted, and that gap must never be silent or unbounded.

### 2.2 Scheduling inputs: class and deadline

Honoring "deadlines and priorities" requires them to exist in the model. Schedulable one-shot kinds (`Execution`, `Compilation`, Book 02 §Ch07) carry, in `spec`:

```yaml
schedulingClass: <named priority class>   # e.g. interactive | batch | best-effort
deadline:        <absolute timestamp | relative duration from creation>   # optional
```

- `schedulingClass` is a **named class mapped to a priority by workspace policy** (Book 11 §06), not a fixed core enum: the core fixes the *ordering semantics* and the §2.4 starvation floor, while tenancy shapes the classes. A subject may request only classes its capability permits (P8), so **priority is governed, not self-asserted** — otherwise a tenant could self-escalate and starve others.
- `deadline` is optional. Its absence disables the deadline-driven terminal (§2.5) **only** — the §2.5 backpressure budget still bounds the wait. A hold is therefore always both *visible* and *bounded*; never indefinite.
- Both are `spec` fields, so they version (P6) and audit (P10) like anything else.

### 2.3 Run-admission

A `Pending` unit is a run-admission candidate. Admission is **policy-aware** (P9) and **capability-checked** (P8) — see §2.6 — and, when granted, the Scheduler binds the unit to a runtime (via the Runtime Manager) and advances it to `Progressing`.

### 2.4 Fairness and starvation-freedom

Priority alone **cannot** deliver "without starving low-priority tenants": under sustained load from a high-priority tenant, a low-priority unit waits forever. The Scheduler's effective ordering therefore **MUST** guarantee **bounded wait** — no admitted-eligible unit waits unboundedly while others make progress — by implementing at least one of:
- **Aging** — a `Pending` unit's effective priority rises monotonically with wait time, so any unit eventually reaches the front; and/or
- **Per-tenant weighted-fair reservation** — each workspace (Book 11 §08 §5) holds a guaranteed share of capacity independent of another tenant's priority, so a high-priority tenant cannot consume another tenant's reservation.

Reservation shares are **static** (a per-workspace quota), not derived from learned demand: fairness is a *guarantee*, and Book 10 §Ch05 confines learned data to choices among options *already semantically equivalent and already permitted* — a wrong cost estimate may make a plan slower, "never incorrect, unfaithful, or non-compliant". A wrong learned reservation would be non-compliant with this invariant, so it is not that class of decision.

The normative guarantee: **every schedulable unit is, within bounded (load-dependent) time, either run-admitted, held with a surfaced condition (§2.5), or terminated (§2.5) — never silently starved.** The floor holds regardless of how policy maps classes to priorities, so a misconfigured map cannot reintroduce starvation.

### 2.5 Hold, shed, and the pending-work terminals

Saturation has exactly two responses, and a `Pending` unit **MUST** be visibly in one of them:
- **Hold (transient backpressure).** The unit stays `Pending` **with a surfaced `AdmissionPending` condition** (Book 02 §Ch03 §3.5) naming the reason (capacity, quota, policy), so "queued, will retry" is observable rather than an opaque `Pending`. Aging (§2.4) applies while held.
- **Shed (unadmittable).** When a unit cannot be run-admitted within its `deadline`, or — absent a deadline — when a bounded admission-retry/backpressure budget is exhausted, the Scheduler **MUST** transition it to terminal `Failed` with `reason=Shed`, Evented and explained. The budget is a policy default, per-kind overridable; it is **never unbounded**, which is what makes "rather than growing unbounded queues" (§Ch13 §2) true for async work and not only for synchronous API requests.

**`DeadlineExceeded`.** If a unit with a `deadline` reaches it before run-admission (in `Pending`), or before completion where the deadline is execution-spanning (in `Progressing`), it **MUST** transition to terminal `Failed` with `reason=DeadlineExceeded` — fail-closed, with an explaining condition and an Event. Where a `Progressing` unit had already produced effects, its **compensation runs**, and if compensation itself fails the `CompensationFailed` condition and `RemediationTask` escalation apply (Book 06 §Ch03 §4): deadline expiry never leaves silent partial state. A rescheduled Execution (§6) **carries its original deadline**, so rescheduling cannot launder a missed one.

Both reasons are conditions on the existing `Failed` phase; no new phase is introduced (Book 02 §Ch04 §2).

### 2.6 Rules
- The Scheduler operates on **Resources** (Book 02): an `Execution` in `Pending` is a scheduling candidate; the Scheduler advances it toward `Progressing` by binding it to a runtime (via the Runtime Manager) when admitted. It writes only its own status concerns (P4, one-writer rule, Book 02 §Ch03 §4).
- Admission is **policy-aware** (P9): scheduling MUST NOT admit work that control-plane policy forbids (e.g. a workspace over budget); this complements the compiler's pre-execution policy pass.
- Scheduling is **tenancy-fair and capability-checked** (P8): a workspace cannot consume another's capacity — mechanized by §2.4's reservation/aging floor, not asserted — and only authorized subjects can submit work, at only the `schedulingClass` their capability permits (§2.2).
- The Scheduler makes **no** runtime-specific decision beyond "which registered Runtime satisfies the requirements" — it asks the Runtime Manager; it does not embed runtime knowledge (§Ch04 §3).

## 3. The Runtime Manager

### 3.1 Responsibilities
1. **Registry** — maintain `Runtime` Resources (Book 02 §Ch07): each registered runtime plugin, its declared capabilities, cost profile, and health.
2. **Selection** — given a Low IR module's requirements (required Capabilities, declared effects, constraints) and live signals (capacity, cost, policy, health), select a suitable Runtime (Book 06 §04).
3. **Lifecycle** — load, health-check, and retire Runtime plugins via the Plugin Manager (§Ch09); isolate them (Book 11 §10).
4. **Lowering hand-off** — invoke the selected runtime's backend to lower Low IR → RuntimeGraph (Book 05 §05, Book 06 §02), then drive execution and collect Events.
5. **Secret hand-off** — arrange, through the Secret Broker, that the runtime can materialize required `SecretRef`s **at execution only** (P7, Book 06 §06).

### 3.2 Selection contract
- Selection is a **pure function of declared requirements + live signals**; it MUST be explainable (which runtime, why) and auditable (P10). An unsatisfiable requirement (no healthy runtime provides a needed Capability/effect) fails with a typed, explainable error — never a silent wrong placement.
- Selection MUST preserve semantics: any runtime selected MUST be able to execute the Low IR observably-equivalently (IR-P10). If a runtime cannot honor an instruction's `ExecPolicy` (e.g. compensation, Book 04 §Ch04 §4), it is not a valid candidate.
- Selection respects **policy and cost**: policy may pin, forbid, or prefer runtimes per workspace; cost models (fed by Experience, Book 10) inform choice among valid candidates.

## 4. Interaction between the two

```
Compilation done → Low IR (Book 05)
      │
Scheduler: admit? (capacity, quota, policy, deadline) ──no──► hold/shed (Ch13)
      │ yes
Runtime Manager: select Runtime (requirements + signals) ──none──► typed error
      │
Runtime backend: lower Low IR → RuntimeGraph (Book 06 §02)
      │
Execute (Book 06) → Events (Ch03) → Experience (Book 10)
```

The Scheduler owns *when*; the Runtime Manager owns *where*; neither owns *what* (the compiler/planner do). This separation keeps each Manager cohesive and keeps runtime specifics out of the core (§Ch04 §3).

## 5. Plugins, isolation, and trust

Runtimes are **plugins** (P11), loaded and isolated by the Plugin Manager (§Ch09) and sandboxed per Book 11 §10. The Runtime Manager treats them as untrusted: it grants each runtime only the attenuated capabilities its execution needs (§Ch06 §5), materializes secrets only at execution, and contains failures so a crashing runtime degrades its executions, not the Kernel (§Ch13).

## 6. Failure and rescheduling

- If a selected runtime becomes unhealthy before or during execution, the Runtime Manager marks it and the Scheduler MAY reschedule the `Execution` onto another valid runtime **iff** the Low IR's idempotency/compensation contract (Book 04 §Ch04 §4) makes re-execution safe. Where it does not, the Execution fails with a clear terminal status and its compensation runs.
- A rescheduled `Execution` re-enters run-admission (§2.3) **carrying its original `deadline`** (§2.2): rescheduling MUST NOT launder a missed deadline by restarting the clock.
- Rescheduling decisions are Events (P5) and feed Experience (Book 10) for future cost/health-aware selection.

## 7. Invariants (normative summary)
1. Runtime is selected after planning and (usually) after Low IR exists; planners and IR never name a runtime (P1, P11, IR-P7).
2. The Scheduler decides run-admission/timing under capacity, quota, fairness, deadline, and policy (P9), tenancy-fair and capability-checked (P8). Priority and deadline are first-class `spec` fields; `schedulingClass` is capability-gated, so priority is governed, never self-asserted.
3. Starvation-freedom is mechanized, not asserted: bounded wait via aging and/or static per-tenant weighted-fair reservation, and the floor holds regardless of the policy class→priority map.
4. Every schedulable unit is, within bounded time, run-admitted, **held with a surfaced `AdmissionPending` condition**, or terminated `Failed` with `DeadlineExceeded`/`Shed` — never an invisible or unbounded stall. An execution-spanning deadline expiry compensates, escalating per `CompensationFailed` if compensation itself fails; a rescheduled Execution carries its original deadline.
5. The Runtime Manager selects a runtime as an explainable, auditable function of declared requirements + live signals; unsatisfiable requirements fail explicitly.
6. Any selected runtime executes the Low IR observably-equivalently; a runtime that cannot honor an instruction's ExecPolicy is not a candidate (IR-P10).
7. Runtimes are untrusted, isolated plugins granted least-privilege; secrets materialize only at execution (P7).
8. Unsafe re-execution is never performed; rescheduling requires established idempotency/compensation, and all such decisions are audited and feed Experience.
