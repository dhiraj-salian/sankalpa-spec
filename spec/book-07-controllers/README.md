# Book 07 — Controllers & Reconciliation

*Status: Draft skeleton · Nature: Normative. · Reflects: ADR-0002; realizes P3, P5.*

## Scope
The controller model: how every Resource is driven from Actual State to Desired State by a reconciliation loop, and how the Controller Runtime hosts these loops. This is the mechanism behind "everything has a controller."

## Chapters
1. **`01-reconciliation-model.md`** *(Normative)* — The observe → diff → act → emit loop; level-triggered (not edge-triggered) reconciliation; idempotency of reconcile.
2. **`02-controller-runtime.md`** *(Normative)* — Hosting, scheduling, work-queues, rate limiting, and back-off for controllers.
3. **`03-controller-contract.md`** *(Normative)* — The Controller SDK surface (AEP-governed): watch, reconcile, status updates, finalizers, requeue.
4. **`04-consistency-and-conflicts.md`** *(Normative)* — Optimistic concurrency, conflict handling, and ordering guarantees relative to the Event Bus.
5. **`05-failure-and-recovery.md`** *(Normative)* — Crash recovery, at-least-once reconcile semantics, and how controllers converge after partial failure.
6. **`06-writing-a-controller.md`** *(Informative)* — Worked example of a controller for a core Resource kind.
