# RFC-0007: Scheduling admission liveness — priority, deadlines, starvation-freedom, and the pending-work terminal

| Field | Value |
|-------|-------|
| **Status** | Final |
| **Authors** | Dhiraj Salian (Phase 2 hardening review) |
| **Domain / Book** | Kernel / Book 03 (and Book 02) |
| **Shepherd (Domain Lead)** | Kernel Domain Lead |
| **Created** | 2026-07-16 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | — · re-review tracked in the [interim-acceptance ledger](../process/interim-acceptance-ledger.md) |

> Raised by the Phase 2 hardening pass (adversarial review toward v1.0). Number 0007 reserved; open for review by the Kernel Domain Lead and Reviewers. Second hardening batch (0005–0011). Shares the "no silent stall / explained terminal" spine with RFC-0004 (the `CompensationFailed` condition); independent of it.

> **Accepted 2026-07-16.** FCP (accept disposition) was called and concluded the same day by the Kernel Domain Lead. For the solo-maintainer repo the 10-working-day window was shortened and the ≥2-Reviewer gate ([process §7](../process/rfc-process.md)) waived — both recorded here for auditability, not pretended. No blocking objections; all design questions resolved (see *Resolved questions*), and the review pass fixed the defects it found. Per [process §8](../process/rfc-process.md) this RFC becomes **normative only on reflection into `spec/`**; status advances to **Final** once the Documentation Changes (§12) land in Books 02 §04, 03 §07/§13, 14 §02 and the Glossary.

> **Final 2026-07-16.** §12 reflected into `spec/` (Book 03 §07 §2.1–2.6/§6/§7, §13 §2; Book 02 §04 §2, §07; Book 14 §02 §3) and the Glossary in this change; the RFC is now normative. See CHANGELOG.

## 1. Executive Summary
The Scheduler is specified to "apply fairness across workspaces/tenants," "honor per-work **deadlines** and **priorities** without **starving** low-priority tenants," and "hold or **shed** work deterministically" ([Book 03 §Ch07 §2.1](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md)). These are stated as **guarantees but backed by no mechanism, no Resource fields, and no lifecycle outcome**:

- **No priority or deadline exists in the model.** No schedulable Resource (`Execution`, `Compilation`, [Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)) carries a priority or a deadline field. "Honor per-work deadlines and priorities" references inputs that do not exist.
- **No starvation-freedom mechanism.** "Without starving low-priority tenants" is asserted with nothing behind it — naive priority scheduling starves low priority under sustained high-priority load. No aging, weighting, or reservation is specified.
- **No terminal for pending-but-unadmittable work.** `Pending` is defined as "Admitted and persisted; the Controller has not yet begun **or cannot yet begin** reconciliation" ([Book 02 §Ch04 §2](../spec/book-02-resource-model/04-lifecycle-model.md)). An `Execution` the Scheduler can never give capacity sits in `Pending` **indefinitely and invisibly**. Backpressure ([Book 03 §Ch13 §2](../spec/book-03-kernel/13-failure-modes-and-degradation.md)) sheds *new* work at the Kernel API with typed `Busy`/`Unavailable`, but an *already-persisted* `Pending` unit has no `DeadlineExceeded`/`Shed` terminal — a silent stall the spec forbids everywhere else ([Book 03 §Ch13 §3](../spec/book-03-kernel/13-failure-modes-and-degradation.md), [Book 08 §02 §4](../spec/book-08-planners/02-intent-and-goal-derivation.md)).

This RFC makes admission liveness real: (a) **priority/scheduling class** and optional **deadline** as first-class fields on schedulable kinds; (b) a normative **starvation-freedom** rule (bounded wait via aging and/or per-tenant weighted-fair reservation); (c) a **`DeadlineExceeded`** terminal; and (d) a defined **hold-vs-shed** distinction so pending work is either visibly held with a surfaced condition or reaches an explained terminal — never an invisible unbounded stall.

## 2. Problem Statement
Everything else in Sankalpa reaches an *explained terminal* or a *surfaced condition* rather than stalling silently — partial execution failure ([Book 03 §Ch13 §3](../spec/book-03-kernel/13-failure-modes-and-degradation.md)), unresolved clarification ([Book 08 §02 §4](../spec/book-08-planners/02-intent-and-goal-derivation.md)), expired approval ([Book 13 §06 §4](../spec/book-13-interfaces/06-approval-and-human-in-the-loop.md)), compensation failure (RFC-0004). Scheduling admission is the exception:

- **The promise names inputs that don't exist.** [Book 03 §Ch07 §2.1.4](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md) commits the Scheduler to honoring "per-work deadlines and priorities." No `spec` field on `Execution`/`Compilation` ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)) expresses either. A conformance author cannot test "honors deadlines" against a model with no deadline; two implementations will invent incompatible fields.
- **The fairness guarantee is unbacked.** [Book 03 §Ch07 §2.1.2/§2.1.4](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md) and invariant 2 ("tenancy-fair … without starving") assert starvation-freedom. Priority without aging or reservation *cannot* deliver it: under sustained load from a high-priority tenant, a low-priority unit waits forever. The mechanism that would make the promise true is absent.
- **`Pending` is a liveness black hole.** Because `Pending` explicitly includes "cannot yet begin" ([Book 02 §Ch04 §2](../spec/book-02-resource-model/04-lifecycle-model.md)) and the phase graph allows `Pending` to persist, an unadmittable `Execution` neither progresses nor terminates. The degradation model sheds *new* admissions ([Book 03 §Ch13 §2](../spec/book-03-kernel/13-failure-modes-and-degradation.md)) and the Kernel API sheds *synchronous requests* ([Book 03 §Ch02 §… deadlines & backpressure](../spec/book-03-kernel/02-kernel-api.md)), but neither governs an async work Resource already sitting in `Pending`. There is no `DeadlineExceeded`, no `Shed`, no surfaced backpressure condition required — so the submitter cannot tell "queued" from "will never run."
- **Two meanings of "admission" collide.** The `Pending` definition calls the work "Admitted" (admitted to the *store*), while [Book 03 §Ch07 §2.1.1](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md) makes "Admission" the decision to *start running*. The overload is itself a spec ambiguity to resolve.

Cost of doing nothing: the platform's liveness story has a hole exactly where load meets multi-tenancy; "fair, deadline-honoring, non-starving" is unfalsifiable and unimplementable-to-spec; and a starved or expired `Execution` violates the no-silent-stall guarantee the rest of the system upholds.

## 3. Alternatives Considered
- **Do nothing.** Rejected: leaves fairness/deadline promises unbacked and `Pending` an unbounded silent stall.
- **Pure priority, no aging/reservation.** Rejected: cannot satisfy the existing "without starving" invariant; low-priority tenants starve under sustained load.
- **Unbounded FIFO queue, no shed, no deadline.** Rejected: contradicts "hold or shed deterministically … rather than growing unbounded queues" ([Book 03 §Ch13 §2](../spec/book-03-kernel/13-failure-modes-and-degradation.md), [Book 03 §Ch02](../spec/book-03-kernel/02-kernel-api.md)); an unbounded queue *is* the silent stall.
- **Reject-at-submission only (never enqueue).** Rejected: too blunt — transient saturation should hold-and-retry with visibility, not immediately fail work that could run seconds later; only *unadmittable-within-deadline* work should terminate.
- **First-class priority/deadline + aging/weighted-fair + explicit hold/shed/deadline terminals (chosen).** Backs each existing promise with a mechanism and a lifecycle outcome, reusing the phase model, conditions, and Event stream already in ARM — no bespoke scheduler state machine.

## 4. Proposed Design
**4.1 Priority and deadline as first-class fields (normative).** Schedulable one-shot kinds (`Execution`, `Compilation`, [Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)) gain, in `spec`:

```yaml
schedulingClass: <named priority class>     # e.g. interactive | batch | best-effort; workspace-policy-bounded
deadline:        <absolute timestamp | relative duration from admission>   # optional; when the outcome is worthless
```

- `schedulingClass` maps to a numeric priority via workspace policy ([Book 11 §06](../spec/book-11-security/06-policy-engine.md)); a subject may only request classes its capability permits (P8), so priority is governed, not self-asserted.
- `deadline` is optional. Its absence means only that no **deadline-driven** terminal (§4.3) applies — **not** that the unit may wait forever: the bounded admission-retry/backpressure budget of §4.4 still applies, and its exhaustion sheds the unit. A hold is therefore always both **visible** (a surfaced condition, §4.4) and **bounded** (by deadline or budget), never indefinite — as [Book 03 §Ch13 §2](../spec/book-03-kernel/13-failure-modes-and-degradation.md) requires ("rather than growing unbounded queues"). Its presence binds §4.3.
- These are `spec` fields (desired intent), so they participate in versioning (P6) and audit (P10) like everything else.

**4.2 Starvation-freedom (normative).** The Scheduler's effective ordering **MUST** guarantee **bounded wait**: no admitted-eligible unit waits unboundedly while lower-or-equal-total-demand tenants make progress. A conforming Scheduler **MUST** implement at least one of:
- **Aging** — a `Pending` unit's effective priority rises monotonically with wait time, so any unit eventually reaches the front; and/or
- **Per-tenant weighted-fair reservation** — each workspace ([Book 11 §08 §5](../spec/book-11-security/08-identity-users-sessions.md)) receives a guaranteed share of capacity independent of another tenant's priority, so a high-priority tenant cannot consume another tenant's reservation.

The guarantee is stated normatively: *every schedulable unit is, within bounded (load-dependent) time, either admitted to run, held with a surfaced condition (§4.4), or terminated per its deadline (§4.3) — never silently starved.* This replaces the bare "without starving" assertion in [Book 03 §Ch07 §2.1.4](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md).

**4.3 `DeadlineExceeded` terminal (normative).** If a unit with a `deadline` reaches it before admission (in `Pending`), or before completion where the deadline is execution-spanning (in `Progressing`), the Scheduler/owning Controller **MUST** transition it to terminal `Failed` with `reason=DeadlineExceeded` ([Book 02 §Ch04 §2/§3](../spec/book-02-resource-model/04-lifecycle-model.md)), fail-closed, with a condition explaining it ([Book 02 §Ch03 §3.5](../spec/book-02-resource-model/03-desired-vs-actual-state.md)) and a Event (§8). Where a `Progressing` unit had already produced effects, its **compensation runs** and, if compensation itself fails, the escalation of RFC-0004 applies — deadline expiry never leaves silent partial state.

**4.4 Hold vs. shed (normative).** The Scheduler **MUST** distinguish two responses to saturation, and a `Pending` unit **MUST** be in exactly one, visibly:
- **Hold (transient backpressure).** The unit stays `Pending` **with a surfaced `AdmissionPending` condition** ([Book 02 §Ch03 §3.5](../spec/book-02-resource-model/03-desired-vs-actual-state.md)) naming the reason (capacity, quota, policy) — so "queued and will retry" is observable, never an opaque `Pending`. Aging (§4.2) applies while held.
- **Shed (unadmittable).** When a unit cannot be admitted within its `deadline`, or — for a unit with no deadline — when a bounded admission-retry/backpressure budget is exhausted, the Scheduler **MUST** transition it to terminal `Failed` with `reason=Shed` (distinct from `DeadlineExceeded`), Evented and explained. This aligns the async work-Resource path with the Kernel API's synchronous `Busy`/`Unavailable` shed ([Book 03 §Ch02](../spec/book-03-kernel/02-kernel-api.md), [Book 03 §Ch13 §2](../spec/book-03-kernel/13-failure-modes-and-degradation.md)): both surfaces shed deterministically and legibly, differing only in mechanism (typed error vs. terminal Resource state).

**4.5 "Admission" disambiguated (normative).** The `Pending` phase description ([Book 02 §Ch04 §2](../spec/book-02-resource-model/04-lifecycle-model.md)) is clarified: *store-admission* (the write succeeded; the Resource exists in `Pending`) is distinct from *run-admission* (the Scheduler's decision to advance `Pending → Progressing`, [Book 03 §Ch07 §2.1.1](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md)). This RFC's terminals govern the gap between the two.

## 5. Tradeoffs
**Gain:** the fairness/deadline/shed promises become mechanized and testable; low-priority tenants are provably non-starved; a pending unit is always either visibly held or explicitly terminated; the async and synchronous shed paths are consistent; priority is governed (capability-gated classes), not self-asserted.
**Give up:** two new `spec` fields on schedulable kinds; the Scheduler must track wait-time/age and per-tenant shares (modest bookkeeping); a hold→shed budget must be configured for deadline-less work; workspace policy must define the class→priority map.

## 6. API Changes
Additive; no method signatures change:
- `Execution`/`Compilation` `spec` gain `schedulingClass` and optional `deadline` (additive, P6).
- No new Kernel API method; the Scheduler already operates on these Resources ([Book 03 §Ch07 §2.2](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md)). Class→priority resolution is a policy read ([Book 11 §06](../spec/book-11-security/06-policy-engine.md)).
- `status` gains the `AdmissionPending` condition and the `DeadlineExceeded`/`Shed` terminal reasons — additive condition/enum values.

## 7. Resource Changes
- `Execution`, `Compilation` ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)): additive `spec.schedulingClass`, `spec.deadline`; `status` reasons `DeadlineExceeded`, `Shed`, and condition `AdmissionPending`. No new phase — reuses `Pending`/`Failed` ([Book 02 §Ch04 §2](../spec/book-02-resource-model/04-lifecycle-model.md)); the phase graph is unchanged (both are `→ Failed`, already legal).

## 8. Event Changes
Additive domain Events (P5, secret-free): `execution.admissionPending` (held, with reason), `execution.deadlineExceeded`, `execution.shed` (and the `Compilation` analogs). Per-subject ordering on the work Resource suffices ([Book 03 §Ch03 §5](../spec/book-03-kernel/03-event-bus.md)). These feed metrics ([Book 14 §Ch02](../spec/book-14-observability-governance/02-metrics.md)) — queue depth, wait-time percentiles, shed/deadline rates as SLIs — and Experience ([Book 10 §Ch05](../spec/book-10-experience/05-feedback-into-planning.md)) for load-aware cost models.

## 9. Security Impact
Priority is **capability-gated**: a subject requests a `schedulingClass` only within its grant (P8, [Book 11 §06](../spec/book-11-security/06-policy-engine.md)), so a tenant cannot self-escalate to starve others — closing a denial-of-service-by-priority path the current unmechanized promise leaves open. Per-tenant reservation (§4.2) is a tenancy-isolation guarantee ([Book 11 §08 §5](../spec/book-11-security/08-identity-users-sessions.md)): one workspace cannot starve another, strengthening the existing "a workspace cannot consume another's capacity" invariant ([Book 03 §Ch07 §2.2](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md)) from assertion to mechanism. No secret flow. All admission decisions remain auditable (P10).

## 10. Performance Impact
Aging/fair-share tracking is O(pending units) bookkeeping, standard scheduler cost. Deadline enforcement is a timer per deadline-bearing unit. Shedding *bounds* queue growth (the point of §Ch13 §2). Net: bounded queues and predictable latency replace an unbounded, silently-stalling queue.

## 11. Testing Strategy
- Starvation-freedom: sustained high-priority load from tenant A **MUST NOT** prevent tenant B's unit from admitting within a bounded time (aging) or its reserved share (reservation).
- Deadline: a `Pending` unit past its `deadline` reaches `Failed/DeadlineExceeded`; a `Progressing` unit past an execution-spanning deadline compensates then terminates.
- Hold vs. shed: transient saturation ⇒ `AdmissionPending` condition then admit; sustained unadmittable-within-budget ⇒ `Failed/Shed`. No unit sits in `Pending` with no condition.
- Governance: requesting a `schedulingClass` beyond the subject's grant is denied (P8).

## 12. Documentation Changes
Rewrite [Book 03 §Ch07 §2.1](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md) to specify the mechanism (priority classes, aging/reservation, hold/shed, deadline terminal) behind each responsibility; amend [Book 02 §Ch04 §2](../spec/book-02-resource-model/04-lifecycle-model.md) to disambiguate store- vs run-admission and list the new `Failed` reasons; extend [Book 03 §Ch13 §2](../spec/book-03-kernel/13-failure-modes-and-degradation.md) to cover async-work shed alongside API shed; add SLIs to [Book 14 §Ch02](../spec/book-14-observability-governance/02-metrics.md). New Glossary entries: *Scheduling class*, *Run-admission*, *Aging*, *Shed (work)*, *DeadlineExceeded*.

## 13. Migration Strategy
Additive and pre-1.0. Absent `schedulingClass` defaults to a policy-defined default class; absent `deadline` means no deadline-driven terminal, with the §4.4 backpressure budget still bounding the hold (visible and bounded, never indefinite and never silent). Existing `Execution`/`Compilation` schemas gain optional fields; a Scheduler that only implements aging (not reservation) is conformant. No stored artifact changes meaning.

## 14. Risks
- **Deadline-less work held forever.** Mitigated by §4.4's mandatory `AdmissionPending` visibility and the bounded backpressure budget → `Shed`; "held" is never invisible, and operators/submitters always see it.
- **Class→priority policy misconfiguration** could recreate starvation — mitigated by the mandatory aging/reservation floor (§4.2), which holds regardless of class mapping.
- **Interaction with runtime rescheduling** ([Book 03 §Ch07 §6](../spec/book-03-kernel/07-scheduler-and-runtime-manager.md)): a rescheduled Execution re-enters admission; its original deadline **MUST** carry through so rescheduling cannot launder a missed deadline — flagged for the reflecting PR.
- **Interaction with compensation/RFC-0004**: an execution-spanning `DeadlineExceeded` triggers compensation, which may itself fail — resolved by deferring to RFC-0004's `CompensationFailed` condition and `RemediationTask` escalation, not re-specifying it here.

## 15. Future Improvements
Preemption of best-effort work by interactive work under reservation pressure; deadline-propagation across a plan's dependency graph (a Goal's deadline deriving its Executions' deadlines); admission-control feedback into planning cost models ([Book 10 §Ch05](../spec/book-10-experience/05-feedback-into-planning.md)) so a planner can prefer cheaper plans under sustained load.

---
### Resolved questions
- **Fixed core enum or policy-mapped named classes?** **Named classes + policy map** (§4.1). The core fixes what must be uniform — the *ordering semantics* and the §4.2 starvation-freedom floor — while workspace policy ([Book 11 §Ch06](../spec/book-11-security/06-policy-engine.md)) maps names to priorities. Hard-coding a class list in the core would bake one tenancy model into the substrate, against P11; and because the floor holds regardless of the mapping, a misconfigured map cannot reintroduce starvation.
- **Global default shed budget, or per-kind?** A **global policy default, per-kind overridable** (§4.4). Requiring every schedulable kind to define its own is authoring burden for no benefit, while a single global value cannot express that a `Compilation` and an `Execution` tolerate different waits. The load-bearing part is the floor: if neither is set, the global default applies — the budget is **never** unbounded, so deadline-less work can never hold invisibly forever.
- **Reservation shares: static, or Experience-derived?** **Static** per-workspace quota for v1; the learned variant stays in §15. Fairness is a *guarantee*, and deriving it from a learned model would make a guarantee depend on inference. This follows the spec's own line on learned data ([Book 10 §Ch05](../spec/book-10-experience/05-feedback-into-planning.md)): cost models may only choose *among options already semantically equivalent and already permitted* — a wrong estimate may make a plan slower, "never incorrect, unfaithful, or non-compliant." A wrong learned reservation *would* be non-compliant with the fairness invariant, so it does not belong in that class of decision.

### Unresolved questions
*(none — all resolved ahead of review)*
