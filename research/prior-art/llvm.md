# Prior-Art Study: LLVM

*Status: Accepted · Informs: Books 04 (AOS IR), 05 (Compiler), 06 (runtime backends).*

## 1. System in one paragraph
LLVM is a compiler infrastructure whose central contribution is the **IR as a product**: a well-specified, typed, target-independent intermediate representation sitting between many front-ends and many back-ends, plus a pass framework that transforms it. The IR is what turns an O(n×m) problem (n languages × m targets) into O(n+m), and it is the direct ancestor of AOS IR (Book 04) and the compiler pipeline (Book 05).

## 2. Core ideas
1. **A single, well-defined IR in the middle.** Front-ends lower *to* it; back-ends lower *from* it. The IR — not any one compiler — is the platform.
2. **SSA form and explicit dataflow.** Values are defined once; data dependencies are visible in the representation rather than inferred, which is what makes analysis and transformation tractable.
3. **A pass framework** with analysis passes (compute facts, mutate nothing) separated from transform passes (rewrite, invalidate analyses), scheduled by a pipeline manager.
4. **Semantics preservation as the transform contract.** A pass may change the program's shape but never its meaning; everything downstream depends on this.
5. **Target independence in the middle-end, target knowledge only in the back-end.** Optimization is written once, not per target.
6. **Verification of the IR as a first-class gate** — a malformed module is caught at the boundary, not diagnosed as mysterious downstream misbehavior.
7. **A textual and a binary form of the same IR**, so the representation is inspectable by humans and cheap for machines.

## 3. Design decisions & trade-offs
- **A typed IR** buys checkable invariants and safer transformation, at the cost of front-end burden and IR verbosity.
- **Library-not-application** buys reuse in unforeseen contexts, at the cost of a large, exposed API surface.
- **A rich, low-level IR** buys optimization power, at the cost of abstraction loss: high-level intent (loop structure, domain semantics) is destroyed by the time it reaches IR, which is precisely the complaint that produced MLIR ([study](mlir.md)).
- **Optional passes with a fixed skeleton** buys extensibility while keeping pipeline behavior predictable.
- **Undefined behavior as an optimization license** buys speed and buys, with it, an entire genre of security bug — a bargain we decline (§6).

## 4. Relevance to Sankalpa
LLVM is the ancestor of the whole middle of Sankalpa. Book 04 §Ch02's IR principles are LLVM's design values restated for our domain; Book 05's pipeline is LLVM's skeleton-plus-passes shape (Book 05 §Ch01 §3 says so explicitly); Book 05 §Ch02's analysis/transform split and semantics-preservation contract are LLVM's, made normative. Runtime backends (Book 06) occupy LLVM's target-backend slot.

## 5. What we adopt
- **The IR-in-the-middle thesis.** Many planners × many runtimes collapses to planners → IR → runtimes (RFC-0001; Book 04 §Ch01). This is the single most consequential borrowing in the project.
- **Explicit dataflow** (IR-P9) and **full typing** (IR-P2) — SSA's lesson that analysis is only as good as what the representation makes visible.
- **The pass framework**: analysis/transform separation (Book 05 §Ch02 §2), pass purity and determinism (§3), a pipeline manager owning order (§5).
- **Semantics preservation as a contract** (IR-P10, Book 05 §Ch02 §4), strengthened into checkable **effect-graph conservation** (§4.1) and opt-in **translation validation** (§4.3) — the latter borrowed from the compiler-verification literature LLVM helped motivate.
- **Verify at the boundary** (Book 04 §Ch08): verify on entry, verify our own output before hand-off (Book 05 §Ch01 §2). A compiler that trusts its own passes is a compiler that ships miscompilations.
- **Fixed skeleton, pluggable passes and backends** (Book 05 §Ch01 §3) — extensibility without weakening guarantees.
- **Two representations, one meaning** — canonical binary plus human-readable text (Book 04 §Ch07 §1).

## 6. What we reject / change
- **A deterministic front-end.** LLVM's front-end is a parser; ours is a *language model* (Book 08). Nothing in LLVM addresses a non-deterministic source, so the non-determinism boundary (Book 08 §Ch05) and the typed `Reason` effect are ours to invent — the single sharpest divergence (Book 01 §Ch06 §2).
- **The abstraction level.** AOS IR's High level is closer to MLIR dialects than to LLVM IR: our nodes are Capability invocations and reasoning steps, not machine-adjacent operations. We keep the *two-level* discipline (High → Low) rather than one low-level IR.
- **Undefined behavior.** We have no UB and want none: IR-P1 requires deterministic evaluation, IR-P3 makes effects explicit and deny-by-default, and "unspecified means the verifier rejects it" (Book 04 §Ch08). Optimization licensed by UB is a trade we can afford to refuse because our performance ceiling is set by network calls and model latency, not instruction selection.
- **Effects as an afterthought.** LLVM's effect knowledge is a scattering of attributes (`readnone`, `nounwind`) bolted on for optimization. Ours is a first-class, policy-visible lattice (Book 04 §Ch06) because it carries *governance*, not just optimization licence.
- **No security pass.** LLVM has no mandatory checkpoint any pass pipeline must clear; our policy-validation pass (Book 05 §Ch04) is non-optional and non-pluggable (P9).
- **No determinization.** Nothing in LLVM converts a source construct into a cheaper one *based on production evidence* (Book 05 §Ch06, Book 10 §Ch06). Profile-guided optimization is the nearest cousin and is not close.
- **Trusted passes.** LLVM passes are in-process and trusted; ours may be third-party Packages, run untrusted with their output re-verified (Book 05 §Ch01 §3).

## 7. Open questions
- LLVM's pass ordering is famously a heuristic pile ("the phase-ordering problem"). Our pipeline fixes the *stage* order (Book 05 §Ch01 §2) but not intra-stage optimization order — do we need a declared ordering discipline before third-party passes multiply?
- Is effect-graph conservation (Book 05 §Ch02 §4.1) strong enough to be worth its cost, given it is a structural check, not a proof? Where does translation validation stop being opt-in?
- LLVM's IR compatibility promise is weak (bitcode compatibility, but the IR churns). Ours is strong (IR-P5, Book 04 §Ch09). Content-addressed, cached, replayable IR (Book 04 §Ch07) may make our promise *harder* than LLVM's — is a migration story specified for every IR change?

## 8. References
- Lattner & Adve, "LLVM: A Compilation Framework for Lifelong Program Analysis & Transformation" (CGO 2004); the LLVM Language Reference; *The Architecture of Open Source Applications* vol. 1, ch. on LLVM; the translation-validation literature (Pnueli et al.; Alive2).
