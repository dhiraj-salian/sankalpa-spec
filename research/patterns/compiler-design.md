# Pattern Study: Compiler Design

*Status: Accepted · Informs: Books 04 (AOS IR), 05 (Compiler), 08 (planners as the front-end), 06 (backends).*

## 1. Pattern in one paragraph
The classical compiler architecture splits translation into a **front-end** (source → IR), a **middle-end** (IR → better IR, target-independent), and a **back-end** (IR → target). The split exists to solve an economic problem: without a common IR, supporting *n* languages on *m* targets costs n×m implementations; with one, it costs n+m. Everything else — passes, verification, semantics preservation, diagnostics — is machinery to make that split safe. Sankalpa is, structurally, a compiler: this is the pattern the whole middle of the architecture instantiates.

## 2. Core ideas
1. **The three-stage split**, with the IR as the contract between stages and the only thing all stages agree on.
2. **The IR is the architecture.** Its design determines what can be analyzed, optimized, checked, and targeted — everything downstream is constrained by what the IR made visible.
3. **Passes over a shared representation**, with analyses (compute facts) separated from transforms (rewrite), because a transform invalidates facts and the framework must know which.
4. **Semantics preservation.** Every transform must preserve meaning; this is the axiom that makes the whole pile trustworthy, and the one that cannot be compromised for speed.
5. **Verification at stage boundaries** — a malformed IR is rejected where it was produced, not diagnosed three stages later as inexplicable behavior.
6. **Progressive lowering.** Abstraction is spent deliberately, in steps, each verifiable ([MLIR study](../prior-art/mlir.md)).
7. **Diagnostics are a product.** A compiler that rejects correctly but explains badly is a bad compiler; users experience the error messages, not the optimizer.
8. **Separation of concerns by *what is known*:** the front-end knows the source, the middle knows nothing about targets, the back-end knows one target.

## 3. Design decisions & trade-offs
- **A common IR** buys n+m and reuse of the whole middle-end, at the cost of an abstraction that fits no source or target perfectly — the IR is always someone's compromise.
- **The abstraction level of the IR** is *the* consequential choice: too low and high-level intent is unrecoverable (LLVM's problem, MLIR's motivation); too high and lowering carries all the complexity.
- **Optional, ordered passes** buy modular optimization, at the cost of the phase-ordering problem: pass interactions are emergent and no ordering is optimal.
- **Static checking** buys pre-execution guarantees, at the cost of rejecting some correct programs — the trade every type system makes.
- **Optimizing aggressively** buys performance, and in the C tradition was paid for with undefined behavior — the license to assume, and the security bugs that followed.

## 4. Relevance to Sankalpa
This is not an analogy; it is the design. Book 04 §Ch01 argues the n×m case for planners × runtimes; Book 05 §Ch01 is the pipeline; §Ch02 is the pass framework; §Ch05 is lowering; §Ch07 is diagnostics. The mapping is exact but for one substitution, and the substitution changes everything: **the front-end is a language model** (Book 08). Every classical assumption that "the front-end is deterministic and its output is a faithful translation of the source" fails, and Book 08 §Ch05's non-determinism boundary is the repair.

## 5. What we adopt
- **The three-stage split**, as: planner (front-end, Book 08) → High IR → compiler passes (middle-end, Book 05) → lowering → Low IR → runtime backend (Book 06). RFC-0001 makes the IR the sole executable representation, which is what makes the split load-bearing rather than decorative.
- **The IR as the architecture** (Book 04) — its type system (§Ch05) and effect system (§Ch06) determine what policy can even be written against, so IR design *is* security design here.
- **Analysis/transform separation and a pipeline manager owning order** (Book 05 §Ch02 §2, §5).
- **Semantics preservation as a contract** (IR-P10), strengthened past the classical norm into checkable effect-graph conservation (Book 05 §Ch02 §4.1) and opt-in translation validation (§4.3) — because our passes may be third-party and untrusted, so "the pass author was careful" is not an available assumption.
- **Verify at every boundary** (Book 04 §Ch08), including the compiler re-verifying its own output (Book 05 §Ch01 §2).
- **Progressive lowering** across exactly two levels (Book 04 §Ch03–04).
- **Diagnostics as a specified surface** (Book 05 §Ch07) — with a twist the classical pattern does not have: our "programmer" receiving the diagnostic may be a planner (Book 08), a human, or both, and a rejection must be actionable by whichever it is.
- **Static rejection over runtime surprise** — deny-by-default effects (Book 04 §Ch06 §3), policy before lowering (Book 05 §Ch01 §2).

## 6. How faithfully we apply it (and where we deviate)
- **The front-end is non-deterministic.** No compiler has this. A parser's output is a function of its input; a planner's is not. Consequences: the IR must be *verified on entry* rather than trusted (Book 05 §Ch01 §2 — the gate "that stops planner non-determinism from entering"); non-determinism that survives into execution must be *typed* (`Reason`, Book 04 §Ch06); and "compile the same source twice, get the same IR" is false, which breaks the reproducibility assumption classical compilers rest on. Content addressing (Book 04 §Ch07) recovers reproducibility *downstream of the IR* — from High IR onward we are a classical compiler, and upstream of it we are not.
- **A mandatory governance pass.** No compiler has a checkpoint that cannot be disabled or reordered; ours does (Book 05 §Ch04, P9), and it sits *after* optimization so it sees the real effect set (§Ch01 §2).
- **Passes are untrusted.** Classically, passes are in-tree and trusted; ours may be Packages, run untrusted with output re-verified (Book 05 §Ch01 §3).
- **Optimizing for the wrong thing.** Classical middle-ends optimize cycles. Ours optimize cost, latency, *and* reasoning-eliminated (Book 05 §Ch06) — the objective function is different, so the pass literature transfers structurally but not numerically.
- **No undefined behavior.** We take the determinism (IR-P1) and refuse the optimization licence; our performance ceiling is network and model latency anyway.
- **Determinization has no analog, and the classical axiom does not stretch to cover it.** A pass that rewrites the program from *production evidence of past executions* (Book 05 §Ch06, Book 10 §Ch06) is not profile-guided optimization: PGO changes strategy, never the meaning of a construct. Replacing a reasoning step with a deterministic Capability is a semantic substitution justified by evidence — which, judged by semantics preservation alone, a classical compiler would call a miscompilation. Sankalpa's answer is not to stretch the axiom but to **split it**, and the split is the interesting contribution: Book 05 §Ch06 §3 gate 6 governs the *intended* semantic change by evidence (Book 10 §Ch06 §3's thresholds), reversibility (§Ch06 §4), and drift retirement (RFC-0003's shadow sampling); effect-graph conservation under a *declared refinement relation* catches *unintended* collateral change; and value-level equivalence is claimed by neither, requiring `validated: true` (Ch 02 §4.3) where it matters. So the safety apparatus is not doing the axiom's job badly — it is doing a *different* job the axiom was never able to do, and the spec is explicit about which gate answers which question. Classical compiler design has no vocabulary for this because no classical compiler is permitted to be right *on the evidence* rather than right *by construction*.

## 7. Open questions
- **The diagnostic audience problem is real and unaddressed.** §5 notes that a rejection may be read by a planner, a human, or both. Book 05 §Ch07 specifies diagnostics as a surface, but a compiler whose "programmer" is a language model raises a question no classical compiler has: is a policy rejection (Book 05 §Ch04) *fed back* to the planner to re-plan against, and if so, has an attacker who controls planner input just acquired an oracle that enumerates the policy by trial and error? Book 08 §Ch05 does not ask this, and it sits exactly on the P1/P9 seam.

*Checked and answered, contrary to an earlier draft of this study — including one claim this study got materially wrong:*
- *Intra-stage pass ordering (Book 05 §Ch02 §5 — deterministic, versioned, part of the cache key, topologically scheduled, cycles rejected, pinned per compilation). The phase-ordering free-for-all is foreclosed.*
- ***IR-P10 vs. determinization is explicitly reconciled, not in tension.** This study's §6 framed determinization as a semantics-changing transform the semantics-preservation axiom cannot account for. Book 05 §Ch06 §3 gate 6 and invariant 4 draw the distinction precisely: substitution's **intended** semantic change (reasoning → Capability) is governed by evidence (§2–3) and drift retirement (§5); its **unintended** collateral change is what effect-graph conservation catches, evaluated under the pass's **declared refinement relation** (`Reason` removed, declared, hence checked rather than assumed); and **neither** gate decides value-level equivalence — for that a pass declares `validated: true` (Ch 02 §4.3). The three-way split is sharper than this study credited.*
- *Post-mortem reproducibility (Book 05 §Ch08 — `Compilation` is a Resource recording the input `IRModule` by hash, the config, the diagnostics, and an `inputHash` over input+config+policyVersion; with RFC-0002's reasoning ledger, the compile is reproducible even though the front-end is not re-runnable).*

## 8. References
- Aho, Lam, Sethi & Ullman, *Compilers: Principles, Techniques, and Tools*; Muchnick, *Advanced Compiler Design and Implementation*; Appel, *Modern Compiler Implementation*; the [LLVM](../prior-art/llvm.md) and [MLIR](../prior-art/mlir.md) studies; Leroy, "Formal Verification of a Realistic Compiler" (CompCert, 2009); Pnueli et al. on translation validation.
