# RFC-0002: Replay semantics and the recorded-reasoning carve-out to the determinism guarantee

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Authors** | Dhiraj Salian (Phase 2 hardening review) |
| **Domain / Book** | AOS IR & Runtimes / Books 04, 06 (and Book 01) |
| **Shepherd (Domain Lead)** | Compiler/Runtime Domain Lead |
| **Created** | 2026-07-15 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | TBD |

> Draft raised by the Phase 2 hardening pass (adversarial review toward v1.0). Numbering provisional until a maintainer reserves it at PR time.

## 1. Executive Summary
The specification states its flagship determinism guarantee unconditionally in three places — *"identical Low IR + identical resolved bindings + identical inputs ⇒ identical observable behavior"* ([Book 04 §Ch04 §6](../spec/book-04-aos-ir/04-low-ir.md), [Book 01 §Ch05 §1](../spec/book-01-foundations/05-determinism-and-determinization.md), [Book 04 §Ch07 §4](../spec/book-04-aos-ir/07-serialization-and-content-addressing.md)). But a plan may legitimately contain a live `CapturedReasoning` node, whose output is non-deterministic by construction (the spec assumes no model determinism). For such a plan the guarantee is *literally false* unless replay substitutes the **recorded** reasoning output rather than re-invoking the model. The spec gestures at this ("replayable-with-record", [Book 06 §Ch03 §1](../spec/book-06-runtimes/03-execution-semantics.md)) but never makes it normative, and never carves the exception into the determinism definition. This RFC (a) states the carve-out precisely, (b) defines two execution modes — **fresh** and **replay-with-record** — with normative semantics for each, and (c) specifies where recorded reasoning outputs live and how they bind at replay.

## 2. Problem Statement
Determinism is the mission ([Book 01](../spec/book-01-foundations/05-determinism-and-determinization.md)). The determinism guarantee is stated as a function of `(Low IR, bindings, inputs)`. A `CapturedReasoning` instruction ([Book 04 §Ch04 §3](../spec/book-04-aos-ir/04-low-ir.md)) "produces a typed value" from a model call that is *not* a function of those three things — the spec explicitly declines to assume model determinism ([Book 08 §Ch05 §5](../spec/book-08-planners/05-non-determinism-boundary.md)).

Consequences of leaving this unstated:
- **The headline invariant is false as written** for any plan with an un-determinized `Reasoning` node — which is the common case before determinization has accrued evidence. A reviewer, conformance-suite author, or runtime implementer cannot tell whether "replay" means *re-run the model* (non-reproducible) or *inject the recorded output* (reproducible).
- **Golden-file replay tests** ([Book 05, Book 10](../spec/book-04-aos-ir/07-serialization-and-content-addressing.md)) are undefined: they "compare hashes" of executions that cannot match if reasoning is re-run.
- **Determinization evidence** ([Book 10 §Ch06](../spec/book-10-experience/06-determinization-engine.md)) depends on comparing reasoning outputs across runs; without a defined capture-and-address model for those outputs, "same output" is not decidable.

Cost of doing nothing: the central guarantee of the platform is ambiguous at exactly the point where non-determinism is admitted, undermining conformance, replay, audit, and determinization simultaneously.

## 3. Alternatives Considered
- **Do nothing.** Rejected: leaves the flagship invariant false-as-written and replay semantics implementation-defined, which will produce divergent runtimes and a meaningless conformance suite.
- **Re-run reasoning on replay (no record substitution).** Rejected: replay would not reproduce the original execution, breaking audit ("what actually happened") and golden-file tests; contradicts "replayable-with-record".
- **Forbid live `Reasoning` nodes in executable Low IR (force determinize-or-reject).** Rejected: contradicts [Book 08 §Ch05 §3](../spec/book-08-planners/05-non-determinism-boundary.md) (residual reasoning that genuinely depends on execution-time data must be deferred) and would make the platform unable to run novel work.
- **Adopt an explicit two-mode replay model with a recorded-reasoning ledger (chosen).** Mirrors deterministic-replay systems that record non-deterministic inputs and replay them (e.g. Temporal's event-history replay, `rr` record/replay debuggers): the machine is deterministic *relative to a recorded log of non-deterministic results*.

## 4. Proposed Design
**4.1 Restated determinism guarantee (normative).** The determinism guarantee is amended everywhere it appears to:

> Given identical Low IR, identical resolved bindings, identical declared inputs, **and identical recorded outputs for every `CapturedReasoning` node and every `Time`/`Random` effect**, execution produces identical observable effects and outputs.

For a plan with no `Reasoning`/`Time`/`Random` nodes the recorded set is empty and the guarantee reduces to the current three-part form.

**4.2 Two execution modes (normative).**
- **Fresh execution.** Each `CapturedReasoning` node invokes the model (or its determinized `Capability` if substituted, [Book 05 §Ch06](../spec/book-05-compiler/06-determinization-passes.md)); each `Time`/`Random` effect reads its live typed input. The runtime **MUST** report every such output as a typed, secret-free `RuntimeEvent` on its existing `events()` stream ([Book 06 §Ch02 §5](../spec/book-06-runtimes/02-runtime-interface.md)) before dependent instructions consume it; the Runtime Manager reflects these into the execution's **reasoning ledger** (§4.3). Recording thus preserves P4 — the runtime neither writes Resources nor publishes to the bus directly.
- **Replay-with-record.** Given an execution's Low IR hash + resolved bindings + reasoning ledger, the runtime **MUST NOT** invoke the model for any `CapturedReasoning` node present in the ledger; it **MUST** inject the recorded typed output. `Time`/`Random` are likewise injected from the ledger. A ledger miss (a node with no recorded output) is a **fail-closed** error ([Book 03 §Ch13 §1](../spec/book-03-kernel/13-failure-modes-and-degradation.md)), not a silent fall-through to fresh reasoning.

**4.3 The reasoning ledger (normative).** Each `Execution` Resource ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)) carries, in `status`, a **reasoning ledger**: an ordered, append-only map from `(instructionId, canonical-input-hash)` to the recorded typed output, plus the model/capability identity that produced it. The ledger:
- **MUST** be secret-free (P7): recorded outputs are typed values that, like all IR-adjacent data, never contain secret material.
- **MUST** be content-addressed by the canonical hash of the recorded output ([Book 04 §Ch07](../spec/book-04-aos-ir/07-serialization-and-content-addressing.md)) so "same reasoning output" is decidable — this is the input [Book 10 §Ch06 §2](../spec/book-10-experience/06-determinization-engine.md) already assumes exists.
- **MUST** be part of the Experience record ([Book 10 §Ch03](../spec/book-10-experience/03-capture-pipeline.md)) and retained under the same policy as the Execution.

The ledger is not a new concept so much as a formalization: Experience records already carry "reasoning traces" as typed/hashed values ([Book 10 §Ch02 §4](../spec/book-10-experience/02-experience-anatomy.md)). This RFC promotes them from an informally-described capture artifact to a normative structure that replay is *bound to* and determinization discovery is *defined over*.

**4.4 Relationship to determinization.** Determinization discovery ([Book 10 §Ch06](../spec/book-10-experience/06-determinization-engine.md)) mines the ledger: input-equivalence and output-stability are computed over `(canonical-input-hash → output-hash)` pairs. This RFC gives that mechanism its data model. (Shadow re-sampling for drift is out of scope here; see the companion drift RFC.)

## 5. Tradeoffs
**Gain:** the flagship invariant becomes true and testable; replay, audit, and golden-file tests get precise semantics; determinization discovery gets a defined substrate.
**Give up:** every `Execution` now carries a ledger, increasing status size and retention cost for reasoning-heavy plans; runtimes must implement two modes and a record path on the hot path.

## 6. API Changes
Fully additive; no method signatures change:
- Runtime interface ([Book 06 §Ch02](../spec/book-06-runtimes/02-runtime-interface.md)): the execution mode (`fresh | replay`) and, in replay mode, the ledger to inject ride in the existing `executionCtx` of `execute(runtimeGraph, executionCtx)` — alongside the binding tokens it already carries. The ledger *sink* is the existing `events()` stream (a new typed `RuntimeEvent` kind, §4.2). `describe()` additionally declares replay-mode support, feeding runtime selection.
- No new method exposes secrets; the [Book 06 §Ch06](../spec/book-06-runtimes/06-secret-materialization.md) prohibition on reading secrets is unchanged (the ledger is secret-free).
- Kernel API: `Execution` status schema gains the ledger field (additive, P6).

## 7. Resource Changes
`Execution` ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)): `status.reasoningLedger` (additive, secret-free, retained with the Execution). No lifecycle change; the Execution remains one-shot/terminal ([Book 02 §Ch03 §6](../spec/book-02-resource-model/03-desired-vs-actual-state.md)).

## 8. Event Changes
Additive: a `reasoning.recorded` domain Event (secret-free, references the ledger entry by hash) so Experience assembly and audit see reasoning capture. Per-subject ordering on the `Execution` subject suffices ([Book 03 §Ch03 §5](../spec/book-03-kernel/03-event-bus.md)).

## 9. Security Impact
No new secret flow: the ledger is typed values only and is bound by the same P7 rule as IR. Replay-with-record **MUST** re-run the runtime policy checkpoint ([Book 14 §Ch06](../spec/book-14-observability-governance/06-runtime-policy-enforcement.md)) and approval checks ([Book 11 §Ch07](../spec/book-11-security/07-approval-engine.md)) against *live* state — replaying recorded reasoning does **not** replay stale approvals or bypass fail-closed gates. This must be stated normatively so replay cannot become an approval-bypass path. Security review required (touches the determinism invariant P1).

## 10. Performance Impact
Ledger write is on the hot path but small (one content-addressed entry per reasoning node / `Time` / `Random`). Retention cost scales with reasoning density × execution volume; bounded by the kind's retention policy ([Book 02 §Ch04 §5](../spec/book-02-resource-model/04-lifecycle-model.md)). Replay avoids model-call cost entirely, so replay is strictly cheaper than fresh.

## 11. Testing Strategy
- Conformance: a runtime **MUST** pass a fresh→replay equivalence suite (record a fresh run, replay it, assert identical observable effects) — this becomes the operational definition of the invariant.
- Property test: for plans with reasoning, fresh runs may differ but every replay of a fixed ledger is identical.
- Failure-injection: ledger miss ⇒ fail-closed, never silent fresh reasoning.

## 12. Documentation Changes
Amend the determinism statement in [Book 01 §Ch05 §1](../spec/book-01-foundations/05-determinism-and-determinization.md), [Book 04 §Ch04 §6](../spec/book-04-aos-ir/04-low-ir.md), [Book 04 §Ch07 §4](../spec/book-04-aos-ir/07-serialization-and-content-addressing.md); add the two-mode model and ledger to [Book 06 §Ch03](../spec/book-06-runtimes/03-execution-semantics.md); extend [Book 10 §Ch03](../spec/book-10-experience/03-capture-pipeline.md). New Glossary entries: *Reasoning ledger*, *Replay-with-record*, *Fresh execution*.

## 13. Migration Strategy
Additive and pre-1.0; no stored artifacts change meaning. Existing `Execution` schemas gain an optional field; runtimes without replay mode remain conformant for fresh execution until the conformance suite ([Book 06 §Ch07](../spec/book-06-runtimes/07-conformance-suite.md)) requires replay at a named milestone.

## 14. Risks
- **Ledger bloat** for reasoning-heavy workspaces — mitigated by retention policy and by determinization reducing reasoning nodes over time.
- **Replay-as-bypass**: if §9's live-recheck rule is omitted, replay could re-execute effects without fresh approval — mitigated by making the runtime-checkpoint re-run mandatory in replay mode.
- **Unknown unknown:** interaction of replay with partial-failure/compensation ([Book 06 §Ch03 §4](../spec/book-06-runtimes/03-execution-semantics.md)) — flagged for the compensation RFC.

## 15. Future Improvements
Ledger compression/deduplication across executions sharing reasoning outputs (content addressing already enables it); selective ledger retention tied to determinization-candidate status.

---
### Resolved questions
*(none yet)*

### Unresolved questions
- Should `Network(read)` results also be ledgered to make full-plan replay reproducible, or remain "declared effect, re-evaluated"? (Leaning: out of scope; `Network(read)` is a declared input by [Book 06 §Ch03 §7](../spec/book-06-runtimes/03-execution-semantics.md), but a "sealed replay" mode may want it.)
- Exact tolerance semantics when a `determinize`-eligible node's recorded output is used as determinization evidence vs. as replay input.
