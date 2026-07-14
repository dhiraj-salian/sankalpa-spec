# Book 05 — The Compiler

*Status: **Draft-complete** (all chapters authored) · Nature: Normative. · Reflects: RFC-0001; realizes principles P1, P9, P13.*

## Scope
The middle-end that turns High IR into runtime-specific execution graphs: the pass framework, optimization, policy validation, and lowering. Backends (per-runtime lowering) are specified in Book 06 and governed by the runtime AEP.

## Chapters
1. **`01-pipeline-overview.md`** — High IR → Optimize → Policy Validation → Low IR → Lower → RuntimeGraph. Ownership and Events at each stage.
2. **`02-pass-framework.md`** *(Normative)* — Pass abstraction, ordering, analysis vs. transform passes, and pass-level determinism/idempotency requirements.
3. **`03-optimization-passes.md`** *(Normative)* — Semantics-preserving optimizations (dead-step elimination, common-subplan elimination, capability fusion, caching of content-addressed subgraphs).
4. **`04-policy-validation-pass.md`** *(Normative)* — The pre-compilation policy checkpoint (P9): consuming effect annotations, rejecting disallowed plans, producing `ir.policy.validated/rejected` events.
5. **`05-lowering-framework.md`** *(Normative)* — Low IR → RuntimeGraph; the backend interface; the requirement that lowering preserve observable behavior.
6. **`06-determinization-passes.md`** *(Normative)* — Recognizing repeated reasoning and folding it into cached deterministic Capabilities (P13), using Experience data (Book 10).
7. **`07-diagnostics-and-errors.md`** *(Normative)* — The compiler diagnostic model: errors, warnings, and their surfacing to users/planners.
8. **`08-compilation-as-a-resource.md`** *(Normative)* — The `Compilation` Resource: inputs, passes run, results, caching, and reproducibility.
