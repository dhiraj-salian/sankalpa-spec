# Book 06 · Chapter 07 — Conformance Suite

*Nature: **Normative**. · Reflects: RFC-0001, AEP-0002; realizes principles P1, P7, P11 and IR-P10. Companion to Book 04 §Ch08 (verification), the AEP process.*

> A runtime's promise — "I will execute any plan I claim to support, faithfully and safely" — must be *provable*, not merely asserted. The conformance suite is the executable test corpus a runtime MUST pass to claim conformance at a given stability level. It turns "trust me" into "here is the passing run," and it is what makes runtime plugins safely interchangeable (P11).

## 1. Purpose

The suite exists to guarantee, independently of a runtime's author, that:
- What a runtime `supports` (Ch 02 §2), it executes **faithfully** (IR-P10).
- It **never leaks secrets** (P7).
- It handles failure, retries, and compensation **safely** (Ch 03).
- Its RuntimeGraph output is **valid and behavior-preserving** (Book 05 §Ch05 §5).

Conformance is always **to a specific version** (of AEP-0002 and of the IR, Book 04 §Ch09) — never to "latest" — so a runtime's claim is precise and durable (versioning-and-stability).

## 2. Structure of the suite

The suite is a corpus of `(RuntimeGraph or Low IR module, inputs, expected observable outcome)` cases plus assertions, organized by the property each group proves.

### 2.1 Fidelity (IR-P10)
- **Effect fidelity:** the runtime produces exactly the declared effects — no more (no undeclared calls), no fewer (no dropped writes). Golden-effect assertions.
- **Schedule fidelity:** results are identical across legal total orders of the schedule's partial order (Book 04 §Ch04 §2); the suite runs the same module multiple times and asserts observational equivalence.
- **Output fidelity:** typed outputs match expectations for given inputs.
- **Cross-runtime equivalence:** the same Low IR run on this runtime and on a reference runtime yields observably-equivalent outcomes (the portability assertion, Ch 04 §6).

### 2.2 Determinism & replay (IR-P1)
- Same RuntimeGraph + same bindings + same inputs ⇒ same observable behavior, across repeated runs (golden-file replay, Book 04 §Ch07 §4).
- Declared `Time`/`Random` inputs are honored as inputs, not smuggled ambiently (Book 04 §Ch06 §4).

### 2.3 Execution policy (Ch 03)
- **Idempotency/retry:** retried idempotent/keyed instructions do not double external effects; non-idempotent instructions are never retried.
- **Timeouts:** timed-out instructions fail and trigger `onError` within bound.
- **Compensation/saga:** on induced failure, declared compensation runs and external state is left consistent; partial failures reach explained terminal states (never silent partial success).
- **Cancellation:** cooperative cancel stops work and compensates completed non-idempotent effects.

### 2.4 Secret safety (P7) — the highest-stakes group
- **No leakage:** across all cases, **no** materialized secret value appears in any Event, log, trace, metric, RuntimeGraph artifact, output, or persisted state. The suite injects marked "canary" secrets and asserts they never surface anywhere observable.
- **Materialization discipline:** secrets are materialized only at the declaring instruction, only under the authorizing capability (Ch 06 §2); an attempt to materialize without the grant fails closed.
- **Ephemerality:** materialized values are not retained across executions.

### 2.5 Isolation & safety (P8)
- The runtime performs no effect outside its granted capabilities and resource limits; attempts are contained (Book 11 §10).
- A crashing/hanging case is contained without affecting the harness or other runtimes (Book 03 §Ch13).

### 2.6 Interface & versioning (AEP-0002)
- `describe`/`supports`/`lower`/`execute`/`cancel`/`events`/`health`/`init`/`shutdown` behave per Ch 02.
- Version negotiation refuses incompatible IR/interface versions rather than degrading (Ch 02 §3).
- **Honesty:** a runtime that reports `supports = false` for a feature is *not* penalized for lacking it; it is penalized only for reporting `true` and then failing. This is the crux — conformance rewards honest capability advertisement (Ch 04 §3.1, Ch 05 §7).

## 3. RuntimeGraph conformance (the `lower` output)

Because a runtime is also its own lowering backend (Ch 02 §2, Book 05 §Ch05), the suite checks its `lower` output:
- The RuntimeGraph preserves the declared effects and the `ExecPolicy` of the Low IR (Book 05 §Ch05 §5).
- Binding sites map to secret-injection slots that resolve **only** at execution (Ch 06).
- The RuntimeGraph contains **no** secret material and is as safe to store/inspect as the IR (Book 04 §Ch07 §5).

A runtime whose `lower` output fails these checks does not conform, regardless of how well it executes — because unfaithful lowering would corrupt the plan before execution even begins.

## 4. Stability levels and conformance

Conformance is graded to the AEP-0002 stability level a runtime claims (the AEP process):
- **Experimental:** passes the core fidelity, determinism, and **secret-safety** groups (secret safety is required even at Experimental — it is never optional).
- **Beta:** adds full execution-policy (compensation, cancellation) and isolation groups, with real adopters.
- **Stable:** the entire suite, cross-runtime equivalence, and a track record; breaking changes then require a major AEP bump + migration (Book 04 §Ch09).

## 5. Two conforming runtimes agree

The defining guarantee: **two runtimes that both conform (at a version) produce observably-equivalent executions for every module they both support.** This is what makes runtimes genuinely interchangeable (P11) and portability operational (Ch 04 §6) — a deployment can swap or add runtimes trusting the suite, not the vendor.

## 6. Relationship to the whole test story

The runtime conformance suite is one member of a family of conformance suites the spec defines — verifier (Book 04 §Ch08 §6), planner (Book 08 §07), Kernel API (Book 03 §Ch02 §7), package format (Book 12). Together they make "conforms to Sankalpa vX" a precise, testable claim across every extension boundary, which is the foundation of a trustworthy ecosystem (AEP-0000).

## 7. Invariants (normative summary)

1. A runtime MUST pass the conformance suite for the AEP-0002/IR version it claims; conformance is always version-specific.
2. The suite proves fidelity, determinism/replay, execution-policy safety, secret-safety, isolation, and honest capability advertisement.
3. Secret-safety (no leakage anywhere, materialization discipline, ephemerality) is required at every stability level, including Experimental.
4. The runtime's `lower` (RuntimeGraph) output is conformance-checked to preserve effects/ExecPolicy, resolve secrets only at execution, and contain no secret material.
5. Conformance rewards honest `supports` reporting; a runtime is penalized only for claiming a capability it then fails to honor.
6. Two conforming runtimes produce observably-equivalent executions for every module they both support, making runtimes interchangeable.
