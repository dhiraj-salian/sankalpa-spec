# RFC-0004: Compensation-failure condition and escalation

| Field | Value |
|-------|-------|
| **Status** | Proposed |
| **Authors** | Dhiraj Salian (Phase 2 hardening review) |
| **Domain / Book** | Runtimes & Kernel / Books 06, 03, 02 |
| **Shepherd (Domain Lead)** | Runtime Domain Lead |
| **Created** | 2026-07-15 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | TBD |

> Raised by the Phase 2 hardening pass. Number 0004 reserved; open for review by the Runtime Domain Lead and Reviewers. Shares a replay boundary with RFC-0002 (§4.5).

## 1. Executive Summary
Sankalpa handles partial execution failure with saga-style compensation: on failure, the runtime runs the declared compensating Capability to undo an effect ([Book 06 §Ch03 §4](../spec/book-06-runtimes/03-execution-semantics.md)). But the spec never defines what happens when **compensation itself fails** — the compensating Capability errors, times out, or is non-idempotent and unsafe to retry. The failure taxonomy ([Book 03 §Ch13 §6](../spec/book-03-kernel/13-failure-modes-and-degradation.md)) has no row for it. The result is an execution stranded with inconsistent external state and no defined terminal, no operator surface, and no audit obligation — a silent hole beneath the "no silent partial success" guarantee. This RFC defines a `CompensationFailed` terminal condition, a mandatory escalation path, and the accompanying Event and taxonomy entry.

## 2. Problem Statement
Compensation is the *only* mechanism reconciling external effects on failure, since Sankalpa has no cross-Resource/external transactions ([Book 02 §Ch08 §4](../spec/book-02-resource-model/08-storage-and-consistency.md)). The spec mandates compensation "where declared" and requires an "explained terminal state" ([Book 06 §Ch03 §4](../spec/book-06-runtimes/03-execution-semantics.md), [Book 03 §Ch13 §3](../spec/book-03-kernel/13-failure-modes-and-degradation.md)) — but the explained-terminal contract assumes compensation *succeeds*. When it doesn't:
- External state is left inconsistent (a message sent but its retraction failed; a record created but its delete failed) with **no defined system state** describing it.
- The failure taxonomy ([Book 03 §Ch13 §6](../spec/book-03-kernel/13-failure-modes-and-degradation.md)) omits it, so there is no "safe behavior" row an implementer can conform to.
- There is no mandated escalation, so the most dangerous outcome in the system (irreversible inconsistency) is the least specified.
- The assumption is pervasive: the runtime checkpoint's fail-closed rule also ends with "completed non-idempotent effects are compensated" ([Book 14 §Ch06 §4](../spec/book-14-observability-governance/06-runtime-policy-enforcement.md)), and the runtime interface's cancellation contract requires the runtime to "complete or compensate" every partially-applied write ([Book 06 §Ch02 §4](../spec/book-06-runtimes/02-runtime-interface.md)) — every one of these paths dead-ends if the compensator itself fails.

Cost of doing nothing: implementations diverge, and the platform's strongest safety claim — that partial failure is always explained and never silent — has a gap exactly where the stakes are highest.

## 3. Alternatives Considered
- **Do nothing.** Rejected: leaves the highest-consequence failure mode undefined.
- **Retry compensation indefinitely.** Rejected: compensation may be non-idempotent (unsafe to retry, [Book 06 §Ch03 §2](../spec/book-06-runtimes/03-execution-semantics.md)) or the target may be permanently gone; unbounded retry stalls the Execution against the "reach a terminal state" contract.
- **Treat compensation failure as ordinary `Failed`.** Rejected: conflates "cleanly rolled back and failed" with "left externally inconsistent" — operators and audit must distinguish these; they have very different remediation.
- **Distinct `CompensationFailed` terminal *condition* + mandatory escalation (chosen).** A condition on the existing `Failed` phase — not a new phase, respecting the two-terminal lifecycle rule ([Book 02 §Ch04 §2](../spec/book-02-resource-model/04-lifecycle-model.md)) — makes the inconsistency a first-class, attributable, operator-actionable state, consistent with the "explain, never hide" discipline ([Book 02 §Ch03 §3.5](../spec/book-02-resource-model/03-desired-vs-actual-state.md)).

## 4. Proposed Design
**4.1 Terminal condition (normative).** When a mandated compensation fails after exhausting its own bounded `ExecPolicy` ([Book 04 §Ch04 §4](../spec/book-04-aos-ir/04-low-ir.md)), the Execution **MUST** reach terminal `Failed` with a distinguished condition `CompensationFailed` whose message records, per effect: what completed, what compensation was attempted, why it failed, and the residual external state left inconsistent (by reference, secret-free). The Execution **MUST NOT** report clean rollback.

**4.2 Bounded compensation (normative).** Compensation runs under its own `ExecPolicy` (retry bounds, timeout). Retry of a compensating action is subject to the same idempotency rule as any effect ([Book 06 §Ch03 §2](../spec/book-06-runtimes/03-execution-semantics.md)): a `NonIdempotent` compensation is attempted at most once. On exhaustion, §4.1 applies — compensation is bounded, never unbounded.

When multiple effects require compensation, failure of one compensator **MUST NOT** halt the others: the runtime continues best-effort with every remaining independent compensation and records a per-effect outcome (`compensated` / `compensation-failed` / `not-attempted`, with reason). Stopping at the first failure would maximize residual inconsistency for no benefit; §4.1's terminal condition reports the full per-effect set.

Cascading compensation is bounded to **one level**: a compensator's own compensable effects are not themselves compensated — on its failure, everything it leaves behind is reported as residual state under §4.1.

**4.3 Mandatory escalation (normative).** A `CompensationFailed` terminal **MUST**:
- emit a high-severity `execution.compensation_failed` Event ([Book 14 §Ch01](../spec/book-14-observability-governance/01-event-model.md)) and an operator alert ([Book 14 §Ch08](../spec/book-14-observability-governance/08-operability.md));
- be recorded in the tamper-evident audit trail ([Book 11 §Ch09](../spec/book-11-security/09-audit-and-attribution.md)) with full attribution (which runtime, which grant, which effects);
- surface as a durable operator remediation task, not merely a log line — the inconsistency requires human reconciliation and the system must make that unavoidable to miss.

**4.4 Interaction with cancellation and approval-deny.** Cancellation ([Book 06 §Ch03 §3](../spec/book-06-runtimes/03-execution-semantics.md)) and approval-deny ([Book 11 §Ch07 §3](../spec/book-11-security/07-approval-engine.md)) both trigger compensation; both inherit §4.1–§4.3 when that compensation fails. (Whether irreversible effects should be schedule-barriered behind their approval is addressed separately in the approval-ordering finding.)

**4.5 Interaction with replay (RFC-0002, normative).** *Reconstruction replay* of an execution that terminated `CompensationFailed` reproduces the recorded outcome — including the compensation failure — and performs no effects; it never re-attempts compensation. *Re-execution replay* is a new execution and follows §4.1–§4.3 afresh for its own effects. No replay mode implicitly re-attempts the *recorded* failed compensation against the residual external state: re-attempt is exclusively the operator-authorized remediation path (§4.3), audited per §9. This closes the interaction flagged in RFC-0002 §14.

## 5. Tradeoffs
**Gain:** the worst failure mode becomes explained, attributable, and operator-actionable; conformance has a concrete target; "no silent partial success" holds end-to-end.
**Give up:** implementations must add a distinct terminal path and an escalation/remediation surface; more operator-facing machinery.

## 6. API Changes
None to the Kernel API surface beyond the additive Event; runtimes must expose compensation outcome (success/failure + residual) via existing RuntimeEvents ([Book 06 §Ch02 §5](../spec/book-06-runtimes/02-runtime-interface.md)).

## 7. Resource Changes
`Execution` ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)): `CompensationFailed` becomes a defined condition reason; no new phase (it is a `Failed` execution, [Book 02 §Ch04](../spec/book-02-resource-model/04-lifecycle-model.md)). A new **`RemediationTask`** kind carries §4.3's durable operator obligation: §14 requires it be a retained ARM Resource that cannot be silently dropped, which a generic operator-task kind does not guarantee — this decides the reuse-vs-new question in favor of a dedicated, minimal kind (spec: references the Execution and residual effects; lifecycle: open → acknowledged → resolved, resolution audited).

## 8. Event Changes
Additive, high-severity `execution.compensation_failed` (secret-free; references effects and residual state by id). Consumers: operability/alerting ([Book 14 §Ch08](../spec/book-14-observability-governance/08-operability.md)), audit ([Book 11 §Ch09](../spec/book-11-security/09-audit-and-attribution.md)), Experience ([Book 10 §Ch03](../spec/book-10-experience/03-capture-pipeline.md)).

## 9. Security Impact
Attribution is mandatory (P10): a compensation failure names the runtime, grant, and effects. No secret enters the condition/Event (P7) — residual state is described by reference. Forced/manual remediation of the residual inconsistency is a privileged, audited operation ([Book 11](../spec/book-11-security/09-audit-and-attribution.md)), analogous to forced finalizer removal ([Book 02 §Ch04 §4.3](../spec/book-02-resource-model/04-lifecycle-model.md)). Security review recommended.

## 10. Performance Impact
Negligible; the path is a rare failure branch. Escalation/alerting cost is bounded by failure frequency.

## 11. Testing Strategy
- Failure-injection: force compensation to fail (unreachable target, erroring compensator) and assert `CompensationFailed` terminal + Event + audit entry + remediation task.
- Property: a `NonIdempotent` compensation is attempted at most once.
- Conformance: no execution reaches `Succeeded` or clean-`Failed` when a mandated compensation failed.

## 12. Documentation Changes
Add a taxonomy row to [Book 03 §Ch13 §6](../spec/book-03-kernel/13-failure-modes-and-degradation.md) ("Compensation failure → `CompensationFailed` terminal condition + escalation"); extend [Book 06 §Ch03 §4](../spec/book-06-runtimes/03-execution-semantics.md) with §4.1–§4.5; note the condition in [Book 02 §Ch04](../spec/book-02-resource-model/04-lifecycle-model.md). Glossary: *Compensation failure*, *Residual inconsistency*.

**Terminal-state vocabulary reconciliation (found during this review).** [Book 10 §Ch03 §3](../spec/book-10-experience/03-capture-pipeline.md) finalizes Experience "on Execution terminal (Succeeded/Failed/Compensated/Cancelled)" — but the lifecycle model ([Book 02 §Ch04 §2](../spec/book-02-resource-model/04-lifecycle-model.md)) defines exactly two terminal phases, `Succeeded` and `Failed`, and forbids kinds from inventing top-level phases. This RFC's position — compensated and cancelled outcomes are `Failed` (or `Succeeded`) with distinguishing *conditions* (`CompensationFailed`, `Cancelled`, `Compensated`), never new phases — should be reflected into Book 10 §Ch03 to close the drift.

## 13. Migration Strategy
Additive and pre-1.0; introduces a new condition reason and Event, no change to existing valid artifacts.

## 14. Risks
- **Alert fatigue** if compensation failures are frequent — signals a deeper design problem in the affected plan, which is the correct thing to surface; tune severity/routing in operability.
- **Remediation-task lifecycle** must not itself be droppable — modeled as an ARM Resource with retention so it cannot be silently lost.
- **Cascading compensation** (a compensator that itself has compensable effects) — bounded to one level, remainder reported as residual (§4.2); matches saga practice, where compensations are not themselves compensated.

## 15. Future Improvements
Structured remediation runbooks attached to `CompensationFailed`; automated re-attempt workflows a human can authorize; linking residual-inconsistency records to Knowledge ([Book 09](../spec/book-09-knowledge/README.md)) so recurring compensation gaps become lessons.

---
### Resolved questions
- **Reuse an operator-task kind vs. introduce `RemediationTask`?** Introduce a dedicated `RemediationTask` kind (§7) — the non-droppable/retention requirement (§14) decides it.
- **Depth limit for cascading compensation?** One level; a failed compensator's own effects are reported as residual, never compensated in turn (§4.2).
- **Does replay re-attempt failed compensation?** Never implicitly; only via the operator-authorized remediation path (§4.5).

### Unresolved questions
- Exact `RemediationTask` schema and whether resolution requires two-party acknowledgment for high-consequence residuals.
