# Book 04 · Chapter 08 — Verification

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P7, P9 and IR-P1…P8.*

> Verification is the gate that makes the IR's promises mechanically true. It is a **total function** over IR that consults **no runtime** (IR-P8) and either accepts a module or produces a precise diagnostic. Execution admits only verified IR; nothing runs that has not passed.

## 1. Position in the pipeline

Verification runs at every boundary where IR crosses from a less-trusted producer to the deterministic machine:

```
Planner ──High IR──► [VERIFY] ──► optimize ──► [policy] ──► lower ──► Low IR ──► [VERIFY] ──► RuntimeGraph ──► execute
```

- **After planning** (High IR): the primary defense that stops planner/model non-determinism from entering execution as ill-formed IR (Book 08). A planner that emits invalid High IR fails here and the plan does not proceed.
- **After lowering** (Low IR): confirms the compiler's own output is well-formed before it is handed to a runtime backend.
- Verification is idempotent and cheap relative to execution; re-verifying already-verified content (by hash, Ch 07) is a fast cache hit.

## 2. What verification checks

Verification is the conjunction of independent, total checks. A module is **valid** iff *all* pass. Each maps to an IR principle.

### 2.1 Structural well-formedness (IR-P9)
- The module parses/canonicalizes (Ch 07).
- Data edges connect existing ports; every input port is fed exactly once (or is a declared module input); the dataflow graph is acyclic; control-flow regions are well-nested.
- Every referenced Capability/Resource `Ref` is resolvable by id (Book 02 §Ch06); dangling references fail.

### 2.2 Type correctness (IR-P2, Ch 05)
- Every value/port is typed; every edge satisfies assignability; every invocation matches its `Signature`; all conversions are explicit and checked; `SecretRef` opacity and `Dynamic` narrowing hold. (Full obligations: Ch 05 §7.)

### 2.3 Effect soundness (IR-P3, Ch 06)
- Every construct declares its effects; the module effect set is exactly the checked union; no construct can perform an undeclared effect (deny-by-default). `Custom` effects are declared, not implicit.

### 2.4 Secret safety (IR-P4, P7)
- No literal or annotation contains secret material; secrets appear only as `SecretRef` values; no construct derives a value from a `SecretRef`; the canonical form (and hash) contains no secret material (Ch 07 §5).

### 2.5 Determinism constraints (IR-P1)
- Non-determinism appears only in `Reasoning`/`CapturedReasoning` (or declared `Time`/`Random` effects modeled as explicit inputs, Ch 06 §4). No other construct depends on hidden state.
- **Low-IR only:** every retry attaches to an idempotent (or keyed-idempotent) instruction; every failable effect declares `onError`; schedules respect data/control order (Ch 04 §4). Violations (e.g. retry on a non-idempotent `Network(write)`) fail verification.

### 2.6 Termination/cost sanity (IR-P8)
- Unbounded `Loop`s declare a termination bound or a `Cost` effect (Ch 03 §3.3) so the verifier can confirm the module cannot be *statically* proven non-terminating without acknowledgement. Verification does not solve the halting problem; it requires that potentially-unbounded constructs are *declared* as such, turning an undecidable question into a checkable annotation.

## 3. Verification is not policy — but it enables policy

Verification checks **well-formedness** (is this valid IR?); the **policy-validation pass** (Book 05 §04) checks **permission** (is this allowed here?). They are separate: policy runs *on verified* IR, because policy reasons about effects and structure that verification has already proven sound. A module can be perfectly valid (verifies) yet forbidden (policy rejects). Keeping them separate keeps each simple and independently testable.

## 4. Diagnostics

- Verification failures **MUST** produce structured diagnostics (Book 05 §07): an error code, the offending node/edge/type by canonical id, a human message, and — where possible — a suggested fix. Diagnostics are how a planner (or its author) learns to emit valid IR.
- Diagnostics **MUST NOT** contain secret material (P7) even when reporting on a `SecretRef` node — they report the reference and the violated rule, never a value.
- A diagnostic is deterministic for a given module (same input ⇒ same diagnostics), so failures are reproducible.

## 5. Totality and independence (IR-P8)

- **Total:** verification terminates for every input, valid or not. No input can hang the verifier; §2.6 is how potentially-unbounded IR is handled without a non-terminating analysis.
- **Runtime-independent:** verification consults no runtime and resolves no bindings; it checks references exist and types/effects are sound, not what a network call would return. This is what lets IR be verified once and trusted everywhere it is later executed (portability, IR-P7).
- **Composable via hashing:** because sub-modules and types are content-addressed (Ch 07), verification results can be memoized by hash — a large module reusing verified sub-graphs re-verifies only what changed.

## 6. Conformance

An implementation's verifier is itself subject to a conformance suite (developed with Book 05 testing): a corpus of valid and invalid modules with expected verdicts and diagnostics. Two conforming verifiers **MUST** agree on validity for every module in the suite. This makes "verified IR" a portable, trustworthy property across implementations (the guarantee planners and runtimes rely on).

## 7. Invariants (normative summary)

1. Only verified IR is optimized, lowered, or executed; verification runs after planning and after lowering.
2. Verification is the conjunction of structural, type, effect, secret-safety, determinism, and termination/cost checks — each total.
3. Verification consults no runtime and resolves no bindings; it is runtime-independent and memoizable by content hash.
4. Verification (well-formedness) is separate from policy validation (permission); policy runs on verified IR.
5. Diagnostics are structured, deterministic, secret-free, and actionable.
6. Conforming verifiers agree on validity for the conformance corpus, making "verified" portable.
