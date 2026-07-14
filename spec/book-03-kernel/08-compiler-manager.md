# Book 03 · Chapter 08 — Compiler Manager

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P9, P13. Companion to Book 05 (Compiler), Book 04 (AOS IR).*

> The Compiler Manager is the Kernel's entry point to the compile pipeline. It orchestrates the journey from High IR to a runtime-agnostic Low IR — verify → optimize → policy-validate → lower — as a governed, cached, resource-backed process. Book 05 specifies the passes; this chapter specifies the *Manager* that runs them.

## 1. Position

The Compiler Manager sits between planning (Book 08, which produces High IR as an `IRModule`) and execution (Scheduler + Runtime Manager, §Ch07, which run the result). It owns the `Compilation` Resource (Book 02 §Ch07) and exposes compilation through the Kernel API — nothing invokes the compiler except through it (P4).

It is **core** because it hosts the policy-enforcement point (P9) and the determinism gate (P1); the individual **passes and backends** it runs are extensible (plugins/Packages, P11), but the orchestration and the invariants are not.

## 2. Responsibilities

1. **Accept** a High IR `IRModule` for compilation, producing a `Compilation` Resource that records inputs, requested passes, and results.
2. **Verify** the input (Book 04 §Ch08) *first* — no un-verified IR is optimized, policy-checked, or lowered.
3. **Orchestrate passes** — run the optimization pipeline, then the policy-validation pass, then lowering to Low IR (Book 05 §01–§05), in a defined, deterministic order.
4. **Enforce policy** (P9) — ensure the policy-validation pass runs and that a policy rejection stops compilation with an explainable diagnostic (Book 04 §Ch06 §5).
5. **Cache** — key pass outputs and whole compilations by content hash (Book 04 §Ch07) so identical inputs are not recompiled.
6. **Feed determinization** (P13) — surface repeated-reasoning signals to the Experience engine and apply determinization passes when a cached Capability exists (Book 05 §06).
7. **Emit Events & produce outputs** — a verified Low IR `IRModule`, or a typed compilation failure, with Events at each stage (P5).

## 3. The compilation pipeline (orchestration view)

```
High IR IRModule
   │  Compiler Manager creates Compilation (Pending)
   ▼
[VERIFY high]           Book 04 §Ch08  ── fail ─► Compilation Failed (diagnostic)
   ▼
[OPTIMIZE passes]       Book 05 §03    (semantics-preserving, IR-P10)
   ▼
[POLICY VALIDATE]       Book 05 §04 / Book 11 §06  ── reject ─► Compilation Failed (policy diagnostic)
   ▼
[DETERMINIZE (opt)]     Book 05 §06    (fold repeated reasoning → Capability, P13)
   ▼
[LOWER → Low IR]        Book 05 §05    (semantics-preserving)
   ▼
[VERIFY low]            Book 04 §Ch08  ── fail ─► Compilation Failed (internal error)
   ▼
Low IR IRModule (verified)  → Compilation Succeeded
```

Rules:
- **Verify-first, verify-again.** Input High IR is verified before anything; output Low IR is verified before it is handed on. A verification failure of *output* is an internal compiler defect (Book 04 §Ch08 §1), surfaced distinctly from input failures.
- **Policy is mandatory and pre-lowering.** The policy-validation pass runs on High IR *before* lowering (Book 04 §Ch06 §5); the Compiler Manager MUST NOT lower a plan that policy rejected. This is one of the two enforcement points of P9 (the other is the runtime checkpoint, Book 14 §06).
- **Determinism preserved.** Every pass and the lowering are semantics-preserving (IR-P10); the Compiler Manager MUST reject a pass/backend that cannot guarantee this (Book 05 §02).

## 4. Caching and content addressing (P13 substrate)

- The Compiler Manager keys results by the content hash of inputs (Book 04 §Ch07): an identical High IR (and identical pass configuration and policy version) yields the cached Low IR without re-running passes.
- A `Compilation` need not re-execute if its input hash and configuration are unchanged (Book 02 §Ch07). This makes compilation cheap on repeat and is the mechanical basis for recognizing "we have compiled this before."
- Cache correctness depends on canonical serialization (Book 04 §Ch07 §2); the Compiler Manager MUST canonicalize before hashing/caching.

## 5. Passes and backends as extensions

- Optimization passes and lowering **backends** are registered (often via Packages, Book 12) and run by the Compiler Manager through the pass/backend interface (Book 05 §02, §05). The Manager knows the *interface*, never a specific backend (§Ch04 §3).
- Extension passes are **untrusted**: the Manager runs them under the same discipline as other plugins (isolation, least-privilege, §Ch09) and re-verifies their output (§3). A malicious or buggy pass cannot smuggle non-determinism or an undeclared effect past output verification.

## 6. Errors and diagnostics

- Compilation failures are typed and staged (verification failure vs. policy rejection vs. lowering failure vs. internal error) with structured, secret-free diagnostics (Book 05 §07, P7) so a planner or its author can correct the input.
- Failures are recorded on the `Compilation` Resource and emitted as Events; they feed Experience (Book 10) so recurring failure modes inform future planning.

## 7. Invariants (normative summary)

1. All compilation flows through the Compiler Manager and is recorded as a `Compilation` Resource; nothing else invokes the compiler (P4).
2. Input High IR is verified before any pass; output Low IR is verified before hand-off; output-verification failure is flagged as an internal defect.
3. The policy-validation pass runs on High IR before lowering and can stop compilation with an explainable diagnostic (P9); rejected plans are never lowered.
4. Every pass and lowering is semantics-preserving; passes that cannot guarantee this are rejected (IR-P10).
5. Results are cached by canonical content hash; identical inputs+config are not recompiled (P13 substrate).
6. Passes/backends are extensions run under plugin isolation and re-verified; diagnostics are typed, staged, and secret-free.
