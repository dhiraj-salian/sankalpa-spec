# Book 06 · Chapter 03 — Execution Semantics

*Nature: **Normative**. · Reflects: RFC-0001, RFC-0002 (replay modes & reasoning ledger), RFC-0004 (compensation failure), RFC-0005 (secret carve-out); realizes principles P1, P3, P5 and IR-P1, IR-P10. Companion to Book 04 §Ch04 (Low IR), Book 03 §Ch13 (failure).*

> This chapter specifies what it *means* to execute a RuntimeGraph correctly: the determinism obligation, idempotency and retries, timeouts, partial-failure and compensation, and how execution progress becomes Events and an Execution Resource. These are the runtime-side counterparts to the `ExecPolicy` the compiler fixed in Low IR.

## 1. The determinism obligation (IR-P1)

Execution MUST be **reproducible in its observable behavior**: given the same RuntimeGraph (hence the same Low IR, Book 04 §Ch04 §6), the same resolved bindings, and the same inputs, execution produces the same observable effects and outputs. The runtime realizes this by:

- Executing only what the graph specifies, in a legal order of the schedule's partial order (Book 04 §Ch04 §2) — all legal orders are observationally equivalent by the effect/idempotency contract.
- Consulting non-determinism **only** where the IR declared it: `CapturedReasoning` nodes (a model call) and `Time`/`Random` effects modeled as explicit inputs (Book 04 §Ch06 §4). The runtime MUST NOT introduce its own clocks, randomness, or ambient state into instruction results.

**"Resolved bindings" includes materialized secret values.** A `SecretRef` is a binding site resolved *at execution* (Book 04 §Ch04 §5), so the value the Broker returns is part of the resolved bindings this guarantee quantifies over — and it is the one such input that is **secret-free-by-necessity unrecordable**: P7 forbids putting it in the reasoning ledger or any durable artifact, so "the same bindings" cannot be made checkable by recording it. The guarantee is therefore stated **modulo** the materialized secret values, which the platform pins **within** an Execution (§Ch06 §2.1, intra-execution stability) but does not carry **across** a record→re-execution boundary: a rotation between runs changes a resolved binding that no record can pin. Reconstruction replay is unaffected — it performs no external effect and injects recorded outcomes, so it materializes no secret at all; **re-execution** replay re-materializes live and therefore compares the reference's `rotationGeneration` against the recorded one, failing closed on a change (§2 of Book 11 §04 §7) rather than silently performing a secret-dependent effect under a credential the recorded run never used.

Where a `CapturedReasoning` node remains (not determinized, Book 05 §Ch06), its non-determinism is confined to that instruction: its typed output is captured and recorded so the run is *auditable and replayable-with-record* even though the reasoning itself was non-deterministic. This is the **recorded-reasoning carve-out** to the determinism guarantee (RFC-0002, Book 01 §05 §1); the following makes it normative.

**The reasoning ledger.** Each `Execution` carries, in `status`, an ordered, append-only **reasoning ledger**: a sequence of entries `(instructionId, invocationIndex, canonical-input-hash, recorded typed output, producer identity)`, where `invocationIndex` numbers successive dynamic invocations of the same static instruction (loops, retries). The ledger MUST be secret-free (P7 — recorded outputs are typed values, never secret material) and content-addressed by the canonical hash of each output (Book 04 §Ch07) so "same reasoning output" is decidable — the substrate determinization discovery (Book 10 §06) and shadow drift detection (Book 10 §Ch06 §5) are defined over. The runtime MUST report each recorded output as a typed, secret-free RuntimeEvent (§2 of Book 06 §Ch02) before dependent instructions consume it; the Runtime Manager reflects these into the ledger, preserving P4 (the runtime writes no Resources and publishes to no bus directly).

**Two execution modes.** A runtime executes in one of two modes:
- **Fresh execution.** Each `CapturedReasoning` node invokes the model (or its determinized Capability if substituted); each `Time`/`Random` effect reads its live typed input. Every such output is recorded to the ledger.
- **Replay-with-record.** Given a Low IR hash + resolved bindings + reasoning ledger, the runtime MUST NOT invoke the model for any `CapturedReasoning` node present in the ledger; it MUST inject the recorded typed output (and injected `Time`/`Random`), verifying the entry's canonical-input-hash against live inputs and failing closed on mismatch or on a **ledger miss** — never silently falling through to fresh reasoning. Replay has two variants: **reconstruction** performs no external effect (every effectful instruction's result is injected from the recorded outcome; this is what golden-file conformance and audit reconstruction use, and it MUST NOT be able to cause any external effect), while **re-execution** re-fires effects live and therefore re-runs the runtime policy checkpoint (Book 14 §06) and approval checks (Book 11 §07) against current state — replay is never an approval-bypass path. A runtime declaring replay support MUST implement reconstruction.

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

**When compensation itself fails (RFC-0004).** Compensation runs under its own bounded `ExecPolicy` (retry bounds, timeout), and a `NonIdempotent` compensation is attempted at most once (§2). When a mandated compensation fails after exhausting that policy — the compensator errors, times out, or is unsafe to retry — the Execution MUST reach terminal `Failed` with the distinguished condition **`CompensationFailed`**, whose message records, per effect: what completed, what compensation was attempted, why it failed, and the **residual external state** left inconsistent (by reference, secret-free). It MUST NOT report clean rollback. `CompensationFailed` is a *condition* on the `Failed` phase, never a new phase (Book 02 §Ch04 §2). Additional rules:
- **Best-effort continuation.** When multiple effects need compensating, failure of one compensator MUST NOT halt the others; the runtime continues best-effort with every remaining independent compensation and records a per-effect outcome (`compensated` / `compensation-failed` / `not-attempted`).
- **One-level cascade.** A compensator's own compensable effects are not themselves compensated; on its failure they are reported as residual state.
- **Mandatory escalation.** A `CompensationFailed` terminal MUST emit a high-severity `execution.compensation_failed` Event and operator alert (Book 14 §08), be recorded in the tamper-evident audit trail with full attribution (Book 11 §09), and surface as a durable **`RemediationTask`** (Book 02 §Ch07) — not merely a log line. Resolving a task whose residual touches a high-consequence effect class MUST require two-party acknowledgment (Book 11 §07); lower-consequence residuals MAY be single-party.
- **Cancellation and approval-deny** (§3, Book 11 §07 §3) both trigger compensation and inherit all of the above when that compensation fails.
- **Replay interaction.** Reconstruction replay of a `CompensationFailed` execution reproduces the recorded outcome and re-attempts nothing; re-attempting a recorded failed compensation is exclusively the operator-authorized remediation path.

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

1. Execution is observably reproducible given equal RuntimeGraph, bindings, inputs, **and recorded reasoning/`Time`/`Random` outputs**; non-determinism occurs only where the IR declared it and is captured to the reasoning ledger. Replay-with-record injects recorded outputs (fail-closed on ledger miss); reconstruction performs no effects and defines conformance, re-execution re-fires effects under live policy.
2. Retries occur only on idempotent/keyed-idempotent instructions and use the key to deduplicate; non-idempotent instructions are never blindly retried.
3. Timeouts bound every so-declared instruction; cancellation is cooperative and triggers compensation for completed non-idempotent effects.
4. Partial failure runs the declared saga recovery (compensate/fallback) and reaches an explained terminal state; silent partial success is prohibited. When compensation itself fails, the Execution reaches `Failed`/`CompensationFailed` (a condition, not a phase) with residual state recorded, continues best-effort across remaining compensations, and escalates via a durable `RemediationTask`.
5. An execution is a one-shot `Execution` Resource with a terminal lifecycle, emitting secret-free Events and owning its Experience; the full intent→experience chain is traceable.
6. Determinism is relative to declared inputs and effects; external variability enters only through declared effects, keeping replay and determinization sound.
