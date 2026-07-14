# Book 10 · Chapter 05 — Feedback into Planning

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P9, P13. Companion to Book 08 (Planners), Book 05 §Ch03 (cost model), Book 06 §Ch04 (runtime selection).*

> The second feedback path: Experience conditions **future planning** and the deterministic **cost models** the compiler and runtime selection rely on. This chapter specifies how experience improves *what gets planned* and *how it gets compiled and placed* — while preserving the invariant that this feedback never leaks secrets and never compromises determinism.

## 1. Two consumers: the planner and the deterministic machine

Experience feeds planning in two distinct ways, and the distinction matters:

1. **The planner (non-deterministic).** Experience-derived **Knowledge** (via §Ch04, Book 09) and success/failure history condition how a planner derives Goals and builds plans (Book 08 §02–§03) — as *non-secret context*, exactly like any other Knowledge (Book 08 §04).
2. **The deterministic machine.** Experience **metrics** feed the compiler's **cost model** (Book 05 §Ch03 §3) and **runtime selection** (Book 06 §Ch04 §3) — as data driving *choices among semantically-equivalent options*.

Both improve outcomes, but only the second is deterministic. Keeping them separate preserves the platform's core discipline: the planner is where non-deterministic learning may influence *what to do*; the compiler/runtime only let learning influence *how efficiently to do it*, never *what it means* (P1, IR-P10).

## 2. Feedback into the planner

Through Knowledge (§Ch04), Experience makes planners better at:
- **Forming Goals** — success criteria and objective shapes that worked before (Book 08 §02 §3).
- **Choosing approaches** — plans/Capabilities that succeeded, avoiding anti-patterns that failed policy or execution (Book 05 §Ch07 §6).
- **Calibrating confidence and clarification** — knowing which ambiguities historically mattered, so the planner clarifies the right things (Book 08 §02 §4).

Critically, this feedback reaches the planner **only as non-secret Knowledge context** (Book 08 §04, §Ch04 §6). It does **not** bypass the planner interface or inject raw Experience as prompt text (that would be memory-as-context, which Book 10 §01 §3 rejects). The planner consumes structured, provenanced Knowledge and still emits **verified** High IR (Book 08 §01) — learning improves the plan, but the output is verified exactly as before. A better-informed planner is still an untrusted planner whose output is checked (Book 11 §10 §4).

## 3. Feedback into the cost model (deterministic)

Experience `metrics` (Book 10 §02 §3) feed a **cost model** the compiler and Runtime Manager consult:
- **Compiler optimization choices** (Book 05 §Ch03 §3) — whether to fuse, how much to parallelize, what to cache — informed by observed latencies/costs.
- **Runtime selection among valid candidates** (Book 06 §Ch04 §3.4) — choosing the cheaper/healthier runtime *after* the fidelity and policy gates.

The safety property (Book 05 §Ch03 §1, Book 06 §Ch04 §3): the cost model only influences choices among options that are **already semantically equivalent and already permitted**. A wrong cost estimate can make a plan *slower or pricier* but **never incorrect, unfaithful, or non-compliant** — the fidelity gate (Book 06 §Ch04 §3.1) and semantics-preservation (Book 05 §Ch02 §4) are upstream of any cost decision. This is why it is safe to let learned, imperfect data drive these choices.

## 4. The determinization signal (forward to §Ch06)

The most powerful planning feedback is **determinization** (§Ch06, Book 05 §Ch06): repeated reasoning discovered in Experience becomes a deterministic Capability, so future planning increasingly resolves to capability invocations rather than `Reasoning` nodes (Book 08 §05 §4). This is feedback into planning of a special kind — it does not just make the planner *smarter*, it makes future plans *more deterministic*. It is significant enough to warrant its own chapter (§Ch06); here it is noted as the third and most consequential planning-feedback mechanism.

## 5. Feedback is governed and tenant-scoped (P9, tenancy)

- What Experience-derived Knowledge may influence which plans is **policy-governed** (Book 11 §06): a workspace controls what its history is allowed to drive, and cross-tenant learning requires an explicit grant (Book 11 §08 §5). One tenant's Experience never conditions another's planning by default.
- The cost model is **per-tenant/per-context** where appropriate, so one workspace's cost profile does not distort another's choices.

## 6. Feedback never compromises the invariants

Restating the guardrails that make this path safe:
- **No secrets** (P7): planner-facing feedback is non-secret Knowledge; cost metrics are non-secret quantities (Book 10 §02 §6).
- **No determinism loss** (P1, IR-P10): cost feedback only picks among equivalent options; the planner's output is still verified; determinization is gated and reversible (Book 05 §Ch06 §3–4).
- **No governance bypass** (P9): feedback influences choices *within* what policy already permits; it can make the system prefer a compliant option, never prefer a forbidden one.

Learning makes Sankalpa *better and cheaper*, strictly inside the envelope its invariants define. That envelope is never widened by learning.

## 7. Invariants (normative summary)

1. Experience feeds planning two ways: as non-secret Knowledge conditioning the (non-deterministic) planner, and as metrics driving the (deterministic) cost model for compiler/runtime choices.
2. Planner-facing feedback arrives only as structured, provenanced, non-secret Knowledge context; it never injects raw Experience as prompt text and never bypasses verification of the planner's output.
3. The cost model only influences choices among already-equivalent, already-permitted options; a wrong estimate affects speed/cost, never correctness, fidelity, or compliance.
4. Determinization (Ch 06) is the most consequential planning feedback: it makes future plans more deterministic, not just the planner smarter.
5. Feedback is policy-governed and tenant-scoped; cross-tenant learning requires an explicit grant.
6. Feedback improves outcomes strictly within the invariant envelope (no secrets, no determinism loss, no governance bypass); learning never widens that envelope.
