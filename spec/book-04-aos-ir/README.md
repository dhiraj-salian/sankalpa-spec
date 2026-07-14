# Book 04 — AOS IR

*Status: **Draft-complete** (all chapters authored) · Nature: Normative. · Reflects: RFC-0001; realizes principles P1, P7, P9.*

## Scope
The Agent Operating System Intermediate Representation — the **only** executable representation. Specifies the two levels (High, Low), the type system, the effect system, the grammar/opcodes, serialization, and verification. This Book is, with Book 02, the semantic heart of Sankalpa.

## Chapters
1. **`01-why-an-ir.md`** — The LLVM/MLIR-inspired case for a compiler IR between non-deterministic planning and deterministic execution (RFC-0001).
2. **`02-ir-design-principles.md`** *(Normative)* — Deterministic evaluation, typed, effect-annotated, reference-based secrets, versioned, verifiable.
3. **`03-high-ir.md`** *(Normative)* — Planner-facing, runtime-agnostic form: steps, data dependencies, capability invocations, control flow, declared effects. Grammar and examples.
4. **`04-low-ir.md`** *(Normative)* — Compiler-produced form: fixed evaluation order, resource/secret bindings by reference, retry/idempotency, error handling. Grammar and examples.
5. **`05-type-system.md`** *(Normative)* — Value and capability types; type checking; rejection of ill-typed IR.
6. **`06-effect-system.md`** *(Normative)* — Effect taxonomy (network, filesystem, secret use, cost, side effects); deny-by-default for un-annotated effects; how Policy consumes effects.
7. **`07-serialization-and-content-addressing.md`** *(Normative)* — Canonical serialization; content addressing for caching and replay.
8. **`08-verification.md`** *(Normative)* — Structural, type, and policy well-formedness checks independent of any runtime.
9. **`09-versioning-and-compatibility.md`** *(Normative)* — IR schema versioning; additive evolution; MAJOR-break policy and lowering-compatibility shims.
10. **`10-worked-examples.md`** *(Informative)* — Full Intent → High IR → Low IR examples for representative tasks.
