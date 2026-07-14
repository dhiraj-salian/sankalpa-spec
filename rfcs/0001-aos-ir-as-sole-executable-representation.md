# RFC-0001: AOS IR as the sole executable representation

| Field | Value |
|-------|-------|
| **Status** | Proposed |
| **Authors** | Founding maintainers |
| **Domain / Book** | AOS IR & Compiler / Books 04–05 |
| **Shepherd** | Compiler Domain Lead |
| **Created** | 2026-07-14 |
| **Tracking issue** | TBD |

## 1. Executive Summary
Sankalpa defines **AOS IR** (Agent Operating System Intermediate Representation) as the *only* representation that may be executed. Natural language, planner prompts, and model outputs are never executed directly; they are compiled — through a High IR and a Low IR — into runtime-specific execution graphs. This RFC establishes the two-level IR as a first-class, versioned, runtime-agnostic contract sitting between non-deterministic planning and deterministic execution, mirroring the role LLVM IR plays between language front-ends and machine back-ends.

## 2. Problem Statement
Sankalpa's mission is to turn intent into *deterministic* execution while confining non-determinism to LLM reasoning. If any layer executes natural language or model output directly, four things become impossible: (a) determinism — the same intent may execute differently each time; (b) policy enforcement — you cannot statically validate free-form text; (c) portability — logic is welded to whichever runtime the planner imagined; (d) auditability and optimization — there is no stable artifact to inspect, cache, or improve. We need a single, analyzable, executable artifact that every planner targets and every runtime consumes, with a clean separation between "what to do" (planner-facing) and "how to run it here" (runtime-facing).

## 3. Alternatives Considered
- **Execute planner output directly (no IR).** Rejected: welds planners to runtimes, defeats determinism, makes policy validation and optimization impossible. This is the mainstream "agent framework" approach and precisely what Sankalpa rejects.
- **Single-level IR.** Rejected: conflates runtime-agnostic intent with runtime-specific concerns, forcing either planners to know about runtimes or runtimes to re-derive intent. LLVM's own history shows the value of separating a target-independent IR from target-specific lowering.
- **Reuse an existing workflow DSL (e.g., a specific engine's JSON).** Rejected: couples the platform to one runtime's semantics and versioning; every other runtime becomes a second-class translation target.
- **A graph database as the executable form.** Rejected: storage model, not an execution semantics; lacks a defined evaluation order, type system, and verification story.

Adopted: a **two-level IR** (High → Low) inspired by LLVM (target-independent IR + lowering), MLIR (dialects/progressive lowering), and classical compiler middle-ends.

## 4. Proposed Design
**Two levels, one semantics.**

- **High IR** — planner-facing, runtime-agnostic. Expresses Goals as a typed graph of *steps*, *data dependencies*, *capabilities to invoke*, *control flow*, and *policy-relevant effects*. Planners (Book 08) emit High IR and know nothing about n8n, Docker, Temporal, or Python. High IR is declarative about intent and effects, not about scheduling mechanics.
- **Low IR** — compiler-produced, closer to execution but still runtime-agnostic in *form*. Produced by the compiler middle-end (Book 05) after optimization and **policy validation**. Low IR fixes evaluation order, resource bindings (by reference, including secret references), retry/idempotency semantics, and error handling.
- **Lowering to runtime graphs** — a runtime backend (an AEP-defined plugin) translates Low IR into a runtime-specific execution graph. The **runtime is selected after planning**, never before.

**Pipeline:** `Goals → [Planner] → High IR → [Optimize] → [Policy Validation] → [Lower] → Low IR → [Runtime backend] → execution graph → Execution → Events`.

**Properties the IR MUST have:**
- **Deterministic evaluation semantics.** Given identical Low IR and identical referenced inputs, execution MUST be reproducible. Non-determinism is permitted only inside explicitly-typed "reasoning" steps delegated to a planner/model.
- **Typed.** Every value has a type; every capability declares its input/output types. Ill-typed IR MUST be rejected before execution.
- **Effect-annotated.** Every step declares its externally-visible effects (network, filesystem, secret use, cost) so Policy can validate before compilation and before execution.
- **Reference-based secrets.** Secrets appear only as opaque references resolved by the Secret Broker at execution (Book 11). A secret *value* MUST NOT appear anywhere in IR.
- **Versioned & serializable.** IR is a versioned ARM artifact with a canonical serialization; it is content-addressable for caching and replay.
- **Verifiable.** A verifier MUST be able to check structural, type, and policy well-formedness independently of any runtime.

Detailed grammar, type system, and opcode set are specified in Book 04; the compiler passes in Book 05. This RFC fixes the *contract and invariants*; the concrete schema is developed under it.

## 5. Tradeoffs
We gain determinism, portability, static policy validation, optimization, caching, and auditability. We pay with: the engineering cost of a real compiler and two IR levels; the discipline of keeping planners runtime-ignorant; and the expressiveness limit that anything a runtime can do must be representable in IR (or reached via a typed capability). We accept these costs as the essence of the platform.

## 6. API Changes
Introduces the IR type as a Kernel-managed artifact and the Compiler Manager's compile/verify entry points. Defines the Planner output contract (High IR) and the Runtime input contract (Low IR / lowered graph) — both AEP-governed (planner AEP, runtime AEP).

## 7. Resource Changes
New ARM Resources: `IRModule` (High and Low variants distinguished by level), `Compilation` (an execution of the pipeline with its passes and results), and `RuntimeGraph` (the lowered, runtime-specific artifact). Each has Spec/Status/Desired/Actual/Lifecycle/Controller per Book 02.

## 8. Event Changes
New Events: `ir.high.produced`, `ir.optimized`, `ir.policy.validated` / `ir.policy.rejected`, `ir.lowered`, `runtime.graph.produced`. Each carries content-addressed identifiers, never secret values.

## 9. Security Impact
Strongly positive and load-bearing: makes "natural language is never executable" and "secrets by reference only" *mechanically true*, and gives Policy a typed, effect-annotated artifact to validate before anything runs. Requires a security review of the effect and secret-reference model (attach [security-review-template](../templates/security-review-template.md)). Risk: an under-specified effect system could let effects escape annotation — mitigated by "deny by default" (un-annotated effects are rejected).

## 10. Performance Impact
Compilation adds latency between planning and execution, but is amortizable: IR is content-addressable and cacheable, so repeated intents reuse prior compilations (a direct lever for the determinization mission). Execution itself is *faster and more predictable* than interpreting model output. Needs a performance review of compile-time budgets vs. execution savings (attach [performance-review-template](../templates/performance-review-template.md)).

## 11. Testing Strategy
Property tests: "optimize preserves semantics," "lower preserves observable behavior," "no secret value in any IR/Event." Deterministic replay: identical Low IR + inputs ⇒ identical execution (golden files). Conformance suites for the planner-output and runtime-input contracts. Failure injection at each pass boundary.

## 12. Documentation Changes
Anchors Book 04 (AOS IR) and Book 05 (Compiler); adds Glossary entries for High IR, Low IR, Lowering, Compilation, RuntimeGraph; requires a pipeline sequence diagram in `diagrams/src/`.

## 13. Migration Strategy
N/A for the initial architecture. Post-1.0, IR evolves via versioned schema with additive changes preferred; breaking IR changes require a MAJOR bump and a lowering-compatibility shim so existing RuntimeGraphs remain reproducible.

## 14. Risks
- **Expressiveness gaps** — some desired behavior may be awkward to express in IR early on. Mitigation: extend via typed Capabilities rather than by leaking runtime specifics upward.
- **Two-level overhead** feeling heavy for trivial workflows. Mitigation: the levels are conceptual; trivial IR lowers trivially, and caching hides repeat cost.
- **Contract ossification** — planners/runtimes depending on early IR shapes. Mitigation: AEP stability levels; keep IR Experimental until Book 04 stabilizes.

## 15. Future Improvements
An MLIR-style multi-dialect IR for domain-specific optimization; a cost-based optimizer that uses Experience data; verified lowering (formally checked semantic preservation); and IR-level determinization passes that fold repeated reasoning into cached Capabilities.

---
### Unresolved questions
- The exact type system (structural vs. nominal) — deferred to Book 04.
- Whether High IR and Low IR share one grammar with a "level" attribute or use distinct grammars — deferred to Book 04 with a leaning toward MLIR-style shared infrastructure.
