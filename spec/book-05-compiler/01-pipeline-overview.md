# Book 05 · Chapter 01 — Pipeline Overview

*Nature: **Informative**. · Reflects: RFC-0001; realizes principles P1, P9, P13. Companion to Book 04 (AOS IR), Book 03 §Ch08 (Compiler Manager).*

## 1. What the compiler is for

The compiler is the deterministic machine between planning and execution. It takes the **High IR** a planner emitted (Book 04 §Ch03) and produces a verified, runtime-agnostic **Low IR** (Book 04 §Ch04) ready for a runtime backend to lower and execute. In doing so it discharges three of the platform's defining responsibilities:

1. **Determinism (P1).** It transforms IR only in semantics-preserving ways (IR-P10) and hands execution a fully-specified, reproducible artifact.
2. **Governance (P9).** It is one of the two mandatory policy checkpoints — the *pre-execution* one that runs on the human-meaningful High IR before anything is committed to execution mechanics.
3. **Determinization (P13).** It is where repeated reasoning is recognized and folded into reusable deterministic Capabilities — the platform's compounding advantage.

Book 04 defined the *language* (the IR); this Book defines the *machine that processes it*.

## 2. The pipeline

The Compiler Manager (Book 03 §Ch08) orchestrates a fixed sequence of stages. The order is not arbitrary — each stage depends on the guarantees the previous one established.

```
High IR (IRModule, verified on entry — Book 04 §Ch08)
   │
   ▼  ── ANALYSIS ──────────────────────────────────────────
[ build analyses ]     effects, dataflow, cost, reuse (Ch 02 §analysis passes)
   │
   ▼  ── OPTIMIZATION ──────────────────────────────────────
[ optimization passes ]   semantics-preserving (Ch 03, IR-P10)
   │
   ▼  ── GOVERNANCE ────────────────────────────────────────
[ policy-validation pass ]  on High IR, before lowering (Ch 04, P9)
   │        └── reject ─► Compilation Failed (policy diagnostic, Ch 07)
   ▼  ── DETERMINIZATION (optional) ────────────────────────
[ determinization passes ]  fold repeated reasoning → Capability (Ch 06, P13)
   │
   ▼  ── LOWERING ──────────────────────────────────────────
[ lower High → Low ]   fix order, retries, error handling, bindings (Ch 05)
   │
   ▼
Low IR (IRModule) ── VERIFY ──► verified Low IR  (Book 04 §Ch08)
   │        └── fail ─► Compilation Failed (internal defect, Ch 07)
   ▼
→ Scheduler + Runtime Manager select a runtime; backend lowers Low → RuntimeGraph (Book 03 §Ch07, Book 06)
```

Why this order:
- **Verify before anything** — the compiler never processes un-verified IR (Book 04 §Ch08 §1); this is the gate that stops planner non-determinism from entering.
- **Optimize before policy** — optimization may eliminate dead effects/steps, so policy validates the *actual* effect set the plan will exhibit, not a pre-optimization overestimate. (Optimization is semantics-preserving, so it cannot *hide* a real effect — only remove genuinely-dead ones, Ch 03 §3.)
- **Policy before lowering** — governance decides on the human-meaningful High IR (Book 04 §Ch06 §5); a rejected plan is never committed to execution mechanics.
- **Determinize before lowering** — folding a reasoning node into a Capability changes what gets lowered; doing it first keeps lowering working on the final graph.
- **Lower, then verify again** — the compiler re-verifies its *own* output before hand-off, catching internal defects (Book 03 §Ch08 §3).

## 3. What is fixed vs. what is extensible

- **Fixed (core, non-replaceable):** the pipeline order, the mandatory policy checkpoint, the verify-before/verify-after gates, and the semantics-preservation requirement. These enforce P1/P9 and cannot be a plugin (Book 03 §Ch01 §4).
- **Extensible (plugins/Packages, P11):** individual optimization passes, determinization strategies, and lowering **backends** (per-runtime). They register through the pass/backend interface (Ch 02, Ch 05) and are run untrusted, with their output re-verified.

This mirrors LLVM: a fixed pipeline skeleton with a rich, pluggable set of passes and targets. It lets the compiler grow smarter (new optimizations, new runtimes) without weakening its guarantees.

## 4. Everything is content-addressed and cached

Because IR is canonical and content-addressed (Book 04 §Ch07), every stage is cacheable by input hash: an identical High IR under an identical pass/policy configuration yields the cached Low IR without re-running (Book 03 §Ch08 §4). This is what makes compilation cheap on repeat and is the substrate on which determinization (Ch 06) recognizes "we have done this before."

## 5. The compiler produces a record, not just an artifact

Each run is a **`Compilation`** Resource (Book 02 §Ch07, Ch 08): it records the input hash, the passes run, diagnostics, and output hashes. Compilation is thus itself observable, auditable, reproducible, and feeds Experience (Book 10) — the same first-class-Resource discipline applied to the act of compiling.

## 6. Reading order

The chapters proceed down the pipeline: the [pass framework](02-pass-framework.md) (the abstraction all passes obey), [optimization](03-optimization-passes.md), the [policy-validation pass](04-policy-validation-pass.md), the [lowering framework](05-lowering-framework.md), the [determinization passes](06-determinization-passes.md), the [diagnostic model](07-diagnostics-and-errors.md), and [compilation as a Resource](08-compilation-as-a-resource.md).
