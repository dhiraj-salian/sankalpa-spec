# Prior-Art Study: MLIR

*Status: Accepted · Informs: Book 04 (two-level IR), Book 05 (lowering framework).*

## 1. System in one paragraph
MLIR is LLVM's answer to LLVM's own mistake: by the time a program reaches LLVM IR, the high-level structure a domain needs for good decisions has already been destroyed, so every domain (ML frameworks, HLS, hardware compilers) built its own private IR and re-solved the same infrastructure problems badly. MLIR provides **IR infrastructure rather than an IR**: multiple coexisting *dialects* at different abstraction levels in one module, with **progressive lowering** between them, and shared machinery (verification, passes, printing) across all of them. It is the direct ancestor of our High/Low IR split (Book 04).

## 2. Core ideas
1. **Dialects.** Operations, types, and attributes are namespaced and extensible; a dialect is a domain's vocabulary, not a fork of the compiler.
2. **Progressive lowering.** Rather than one cliff from source to machine, a program descends through levels, each lowering step small, local, and verifiable. Abstraction is *spent* deliberately, not thrown away at the door.
3. **Mixed abstraction in one module.** High and low dialects can coexist during lowering, so a transition is a rewrite, not a rewrite of everything at once.
4. **Everything is verifiable.** Every dialect declares invariants; the shared verifier enforces them at every step, so a bad lowering is caught at the step that caused it.
5. **Shared infrastructure, declarative definition.** Ops are declared (TableGen/ODS) and get parsing, printing, verification, and pass plumbing for free — the cost of a new abstraction level is low by design.
6. **Region-based structure.** Nested regions keep control-flow and scoping structure legible instead of flattening it to branches.

## 3. Design decisions & trade-offs
- **Extensible dialects** buy domain fidelity, at the cost of fragmentation: dialects proliferate, and interoperability between them becomes the new hard problem.
- **Progressive lowering** buys verifiability and reuse of lowering steps, at the cost of pipeline length and conceptual overhead.
- **Universal verification** buys early, precise failure attribution, at the cost of writing invariants for everything.
- **Infrastructure, not policy** buys broad adoption, at the cost of MLIR having no opinion about what your compiler *should* do — you must supply the discipline.

## 4. Relevance to Sankalpa
MLIR settles a question Book 04 would otherwise have to argue from scratch: whether one IR level suffices. It does not, and MLIR is the evidence. Our High IR (Book 04 §Ch03) is domain-level — Capability invocations, reasoning nodes, human-meaningful structure — and our Low IR (§Ch04) is execution-level, with lowering (Book 05 §Ch05) as the deliberate descent between them. MLIR is also the strongest argument that policy belongs on the *high* level: governance decisions need the abstraction LLVM would have already discarded (Book 05 §Ch01 §2, "policy before lowering").

## 5. What we adopt
- **Progressive lowering as the governing idea** (Book 01 §Ch06's stated borrowing): High IR → Low IR → RuntimeGraph, each step small and re-verified (Book 05 §Ch01 §2).
- **Two abstraction levels rather than one** — MLIR's core lesson, taken at the smallest dose that solves our problem (§6).
- **Verify at every level, with each level declaring its own invariants** (Book 04 §Ch08); the compiler re-verifies its own output (Book 05 §Ch01 §2).
- **Namespaced extension instead of core modification.** `Custom(namespace, name, payload)` effects (Book 04 §Ch06 §2) are the dialect idea applied to our extension point: a Package may add vocabulary, but it stays policy-visible and declared — never an unknown that slips past the verifier.
- **Structure preserved, not flattened.** Explicit dataflow and nested structure in High IR (IR-P9) rather than an early collapse to a flat graph.
- **Lowering steps as reusable, registered units** — the backend/lowering framework (Book 05 §Ch05) mirrors dialect conversion.

## 6. What we reject / change
- **Open-ended dialect proliferation.** MLIR's cost is that everyone invents a dialect and interop rots. We fix **exactly two levels** (Book 04 §Ch03–04), and they are specified, not extensible — because our IR is a *governance* artifact: policy (Book 11 §Ch06) is written against a known vocabulary, and an unbounded dialect space would mean unbounded policy surface. Extension happens at declared points (Capabilities, `Custom` effects, backends), not by minting abstraction levels.
- **Mixed abstraction within a module.** MLIR allows dialects to coexist mid-lowering; we require each artifact to be wholly High or wholly Low, so "what is this thing" is never ambiguous — verification, content addressing (Book 04 §Ch07), and caching all key off it.
- **Infrastructure without opinion.** MLIR is deliberately policy-free; our pipeline mandates a policy checkpoint and a semantics-preservation contract (Book 05 §Ch01 §3). We want the opinion; it is the point.
- **Compile-time as the only concern.** MLIR's dialects lower toward machine code; our Low IR lowers toward *durable, distributed, effectful* execution (Book 06), where retries and compensation matter more than instruction selection.

## 7. Open questions
- **Where does a `Custom` effect stop being an extension and start being a dialect in disguise?** Book 04 §Ch06 §2 admits `Custom(namespace, name, payload)` and keeps it policy-visible and declared, which is the guardrail. But policy (Book 11 §Ch06) is written against the effect lattice, and a `Custom` effect's *refinement relation* to the core kinds is supplied by its declarer. A Package that declares a `Custom` effect the workspace's policy has no rule for is, from governance's seat, an effect that exists and is ungoverned — deny-by-default catches the undeclared, not the declared-but-unruled. MLIR's dialect fragmentation is the shape of this risk; the question is whether the boundary is drawn tightly enough.

*Checked and answered, contrary to an earlier draft of this study: the "third level" question was mis-framed — the pipeline already has three (High IR → Low IR → RuntimeGraph, Book 05 §Ch05), with backend lowering owning the runtime-specific step, which is exactly where per-runtime divergence is absorbed.*

## 8. References
- Lattner et al., "MLIR: Scaling Compiler Infrastructure for Domain Specific Computation" (CGO 2021); MLIR Language Reference and Dialect Conversion docs; MLIR's "progressive lowering" rationale documents.
