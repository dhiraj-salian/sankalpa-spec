# Book 06 · Chapter 03 — Execution Semantics

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P3, P5 and IR-P1, IR-P10. Companion to Book 04 §Ch04 (Low IR), Book 03 §Ch13 (failure).*

> This chapter specifies what it *means* to execute a RuntimeGraph correctly: the determinism obligation, idempotency and retries, timeouts, partial-failure and compensation, and how execution progress becomes Events and an Execution Resource. These are the runtime-side counterparts to the `ExecPolicy` the compiler fixed in Low IR.

## 1. The determinism obligation (IR-P1)

Execution MUST be **reproducible in its observable behavior**: given the same RuntimeGraph (hence the same Low IR, Book 04 §Ch04 §6), the same resolved bindings, and the same inputs, execution produces the same observable effects and outputs. The runtime realizes this by:

- Executing only what the graph specifies, in a legal order of the schedule's partial order (Book 04 §Ch04 §2) — all legal orders are observationally equivalent by the effect/idempotency contract.
- Consulting non-determinism **only** where the IR declared it: `CapturedReasoning` nodes (a model call) and `Time`/`Random` effects modeled as explicit inputs (Book 04 §Ch06 §4). The runtime MUST NOT introduce its own clocks, randomness, or ambient state into instruction results.

Where a `CapturedReasoning` node remains (not determinized, Book 05 §Ch06), its non-determinism is confined to that instruction: its typed output is captured and recorded in the Execution/Experience so the run is *auditable and replayable-with-record* even though the reasoning itself was non-deterministic.

## 2. Idempotency and retries

Retries are the most dangerous execution behavior because a naive retry can double an external side effect. The rules (set in Low IR, Book 04 §Ch04 §4; enforced at execution here):

- A runtime MUST retry an instruction **only** if its `ExecPolicy.idempotency` is `Idempotent` or `IdempotentWithKey`. It MUST NOT retry a `NonIdempotent` instruction (Book 04 §Ch08 §2.5).
- For `IdempotentWithKey`, the runtime MUST use the idempotency key so that a retried write is deduplicated by the external system (or by the runtime's own dedup) — the key is what makes re-execution safe.
- Retries follow the policy's bounds (max attempts, backoff); on exhaustion the instruction fails and `onError` applies (§4).
- Because delivery and reconcile are at-least-once across the platform (Book 03 §Ch03 §4), *the whole execution path must be idempotent* — this is why idempotency is a first-class IR concern, not a runtime afterthought.

## 3. Timeouts and cancellation

- Every instruction with a `timeout` MUST be bounded by it; on expiry the instruction fails and `onError` applies. This prevents a hung external call from stalling an execution indefinitely.
- Cooperative `cancel` (Book 06 §Ch02 §4) stops further instructions and triggers compensation for completed, non-idempotent effects so external state is left consistent.

## 4. Partial failure and compensation (saga semantics)

External effects cannot be rolled back by a transaction (there is no cross-Resource transaction anywhere in Sankalpa, Book 02 §Ch08 §4). Instead, Low IR encodes **saga-style** recovery, which the runtime executes:

- On an instruction failure, the runtime applies `onError`: `Fail` (propagate), `Compensate(ref)` (run the compensating Capability to undo the effect), `Fallback(region)` (run an alternative sub-region), or `Ignore` (only where explicitly justified, Book 04 §Ch04 §4).
- **Compensation is mandatory where declared.** A runtime MUST run the specified compensation for a completed, non-idempotent effect when the execution fails or is cancelled — leaving, e.g., a sent message retracted or a created record deleted, per the compensating Capability.
- A partially-completed execution MUST reach an **explained terminal state** (Book 03 §Ch13 §3): the Execution's status/conditions record what completed, what compensated, and what could not — never a silent partial success.

## 5. Execution as a Resource (P2, P3, P5)

An execution is an **`Execution` Resource** (Book 02 §Ch07), and its semantics follow the ARM lifecycle:

- It moves `Pending → Progressing → Succeeded|Failed` (Book 02 §Ch04); it is one-shot and terminal (Book 02 §Ch03 §6) — a completed execution never mutates further.
- Its `status.actualState` tracks progress; its `conditions` explain readiness/failure; every meaningful transition emits a secret-free Event (P5, P7).
- It is **owned by** its `RuntimeGraph`/`Goal` and **owns** the `Experience` it produces (Book 02 §Ch06, §Ch07). The full chain Intent → Goals → Compilation → RuntimeGraph → Execution → Experience is thereby traceable and auditable (Book 15 §01).

## 6. Events and observability (P5, P10)

- The runtime reports RuntimeEvents (Book 06 §Ch02 §5); the Runtime Manager reflects them as domain Events on the Event Bus (Book 03 §Ch03), preserving P4.
- Events carry trace context (Book 14 §03), reference IR by content hash (Book 04 §Ch07), and are **secret-free** (P7). They are the raw material for both the audit trail (Book 14 §05) and the Experience record (Book 10 §03).
- Execution metrics (durations, retries, cost, outcome) are captured for the cost model (Book 05 §Ch03 §3) and determinization discovery (Book 10 §06).

## 7. Determinism vs. the real world

A subtle but important boundary: Sankalpa guarantees the *observable behavior of the plan* is reproducible given equal inputs and bindings — **not** that the external world is unchanging. If an external API returns different data on two runs, that difference enters through a **declared effect** (`Network(read)`) as an input to the rest of the plan, exactly as the effect system models it (Book 04 §Ch06 §4). Determinism is therefore *relative to declared inputs and effects*: everything the plan does with a given set of inputs is reproducible; the inputs themselves are honestly modeled as effects, not hidden. This is what makes replay (Book 04 §Ch07 §4) and determinization (Book 05 §Ch06) sound.

## 8. Invariants (normative summary)

1. Execution is observably reproducible given equal RuntimeGraph, bindings, and inputs; non-determinism occurs only where the IR declared it and is captured for audit/replay.
2. Retries occur only on idempotent/keyed-idempotent instructions and use the key to deduplicate; non-idempotent instructions are never blindly retried.
3. Timeouts bound every so-declared instruction; cancellation is cooperative and triggers compensation for completed non-idempotent effects.
4. Partial failure runs the declared saga recovery (compensate/fallback) and reaches an explained terminal state; silent partial success is prohibited.
5. An execution is a one-shot `Execution` Resource with a terminal lifecycle, emitting secret-free Events and owning its Experience; the full intent→experience chain is traceable.
6. Determinism is relative to declared inputs and effects; external variability enters only through declared effects, keeping replay and determinization sound.
