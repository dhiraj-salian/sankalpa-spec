# Book 03 · Chapter 07 — Scheduler and Runtime Manager

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0001; realizes principles P4, P8, P9, P11. Companion to Book 06 (Runtimes).*

> These two Managers turn compiled IR into scheduled, placed execution. The **Scheduler** decides *whether and when* work runs under capacity and policy; the **Runtime Manager** decides *where* — which Runtime plugin executes it — and owns runtime lifecycle. A defining rule lives here: **the runtime is selected after planning**, never before (RFC-0001).

## 1. Why runtime selection comes last

A planner emits runtime-agnostic High IR (Book 04 §Ch03); the compiler produces runtime-agnostic Low IR (Book 04 §Ch04). Only once the *what* and the *how-mechanically* are fixed does the system choose *where* to run. This ordering is load-bearing:
- It keeps planners ignorant of runtimes (P1, P11) — a planner never encodes "use n8n."
- It lets the **same** Low IR target different runtimes and produce observably equivalent executions (Book 04 §Ch04 §7, IR-P10) — the portability guarantee.
- It lets selection use information available only late: current capacity, cost, policy, and runtime health.

## 2. The Scheduler

### 2.1 Responsibilities
1. **Admission** — decide whether a unit of work (a `Compilation` or an `Execution`, Book 02 §Ch07) may start now, given capacity, quotas, and policy.
2. **Placement queueing** — order and gate pending work; apply fairness across workspaces/tenants.
3. **Backpressure** — when the system is saturated, hold or shed work deterministically rather than overcommit (§Ch13).
4. **Deadlines & priorities** — honor per-work deadlines and priorities without starving low-priority tenants.

### 2.2 Rules
- The Scheduler operates on **Resources** (Book 02): an `Execution` in `Pending` is a scheduling candidate; the Scheduler advances it toward `Progressing` by binding it to a runtime (via the Runtime Manager) when admitted. It writes only its own status concerns (P4, one-writer rule, Book 02 §Ch03 §4).
- Admission is **policy-aware** (P9): scheduling MUST NOT admit work that control-plane policy forbids (e.g. a workspace over budget); this complements the compiler's pre-execution policy pass.
- Scheduling is **tenancy-fair and capability-checked** (P8): a workspace cannot consume another's capacity, and only authorized subjects can submit work.
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
- Rescheduling decisions are Events (P5) and feed Experience (Book 10) for future cost/health-aware selection.

## 7. Invariants (normative summary)

1. Runtime is selected after planning and (usually) after Low IR exists; planners and IR never name a runtime (P1, P11, IR-P7).
2. The Scheduler decides admission/timing under capacity, quota, fairness, deadline, and policy (P9), tenancy-fair and capability-checked (P8).
3. The Runtime Manager selects a runtime as an explainable, auditable function of declared requirements + live signals; unsatisfiable requirements fail explicitly.
4. Any selected runtime executes the Low IR observably-equivalently; a runtime that cannot honor an instruction's ExecPolicy is not a candidate (IR-P10).
5. Runtimes are untrusted, isolated plugins granted least-privilege; secrets materialize only at execution (P7).
6. Unsafe re-execution is never performed; rescheduling requires established idempotency/compensation, and all such decisions are audited and feed Experience.
