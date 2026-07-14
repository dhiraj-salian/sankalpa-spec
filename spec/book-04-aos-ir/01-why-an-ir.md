# Book 04 · Chapter 01 — Why an IR

*Nature: **Informative**. · Reflects: RFC-0001; realizes principles P1, P7, P9, P13.*

## 1. The single most important boundary in Sankalpa

There is exactly one place in the system where non-determinism is allowed to enter — planning — and exactly one artifact that crosses from that world into the deterministic world: **AOS IR**. Principle **P1** states it flatly: *natural language is never executable; the only executable representation is AOS IR.* This Book specifies that representation. If Book 02 (ARM) is the system's noun-grammar, Book 04 is its verb-grammar: the precise language in which "what to do" is expressed such that it can be optimized, governed, and executed deterministically.

## 2. What goes wrong without an IR

The mainstream "agent framework" executes model output more or less directly: the model emits a tool call or a snippet, and something runs it. Sankalpa rejects this because it makes four essential properties impossible:

1. **Determinism.** Free-form model output is not reproducible; the same intent can execute differently each time. There is nothing stable to re-run.
2. **Static policy validation (P9).** You cannot soundly analyze arbitrary text/code before it runs. Governance becomes after-the-fact hope.
3. **Portability (P11).** Logic emitted "for tool X" is welded to X. Every other runtime is a lossy translation.
4. **Optimization, caching, and audit.** With no stable artifact there is nothing to inspect, cache, diff, or improve — and thus no path to *determinization* (P13).

An IR is the standard answer to exactly this problem, proven for fifty years in compilers: put a well-defined, analyzable representation between the many possible front-ends and the many possible back-ends.

## 3. The compiler analogy, made precise

Sankalpa's pipeline is a compiler, and it is useful to name the correspondence explicitly:

| Compiler | Sankalpa |
|----------|----------|
| Source language (C, Rust, Swift…) | Natural-language **Intent** |
| Front-end (parser + semantic analysis) | **Planner** (Book 08) — the *only* non-deterministic stage |
| Target-independent IR (LLVM IR) | **AOS High IR** |
| Middle-end (optimization passes) | **Compiler** optimization + policy passes (Book 05) |
| Lowered / target-specific IR | **AOS Low IR** |
| Back-end / codegen (per ISA) | **Runtime backend** lowering to a RuntimeGraph (Books 05–06) |
| Machine (x86, ARM…) | **Runtime** (n8n, Temporal, Bash…) — selected *after* planning |

Two-level IR is deliberate (RFC-0001 §4): **High IR** is runtime-agnostic and planner-facing; **Low IR** is post-optimization, post-policy, and closer to execution while still runtime-agnostic in *form*. The split lets planners stay ignorant of runtimes and lets optimization/policy operate on a clean, high-level artifact before lowering commits to execution mechanics. This is the LLVM (target-independent IR + lowering) and MLIR (progressive lowering across dialects) lesson, adapted.

## 4. Why *two* levels and not one

A single IR would force a false choice: either it carries runtime-agnostic intent (and then something else must still encode execution mechanics — retries, ordering, error handling), or it carries execution mechanics (and then planners must know about them, breaking P1/P11). Two levels resolve this:

- **High IR** answers *"what should happen, and with what declared effects?"* — steps, data dependencies, capability invocations, control flow, effects. No scheduling, no retry policy, no secret binding.
- **Low IR** answers *"in what order, with what error handling, retries, idempotency, and resource/secret bindings?"* — everything needed to execute deterministically, still without naming a specific runtime.

Lowering High → Low is where the compiler *decides* mechanics; lowering Low → RuntimeGraph is where a backend *targets* one runtime.

## 5. What the IR is not

- It is **not** a general-purpose programming language. It is a constrained, analyzable representation. Anything a runtime can do that is not expressible in IR is reached through a typed **Capability** (Book 02 §Capability), never by leaking runtime specifics upward. This keeps the IR small and verifiable.
- It is **not** a prompt format. Prompts and model outputs live *inside* reasoning steps at planning time; they do not survive into executable IR as text to be re-interpreted.
- It is **not** storage. An `IRModule` is an ARM Resource (Book 02 §Ch07) whose *body* is the IR specified here; the body is content-addressed (Ch 07) so identical IR is identical everywhere.

## 6. The payoff

Because a stable, typed, effect-annotated, content-addressed artifact sits at the center:

- Execution is **reproducible** (Ch 02 principle: deterministic evaluation).
- Policy is **enforced before anything runs** (P9, Book 05 §04).
- Identical work is **recognized and cached** by content hash — the mechanical basis of **determinization** (P13, Book 05 §06): repeated reasoning is folded into cached Capabilities.
- Any runtime can be targeted by writing a backend, not by changing planners.

The remaining chapters make this concrete: the [design principles](02-ir-design-principles.md) every IR construct obeys, the [High](03-high-ir.md) and [Low](04-low-ir.md) forms, the [type](05-type-system.md) and [effect](06-effect-system.md) systems, [serialization/content-addressing](07-serialization-and-content-addressing.md), [verification](08-verification.md), [versioning](09-versioning-and-compatibility.md), and [worked examples](10-worked-examples.md).
