# Book 05 · Chapter 08 — Compilation as a Resource

*Nature: **Normative**. · Reflects: RFC-0001, ADR-0002; realizes principles P2, P5, P6, P10, P13. Companion to Book 02 §Ch07, Book 03 §Ch08.*

> The *act of compiling* is itself a first-class `Compilation` Resource (Book 02 §Ch07). This closes the loop with the rest of the platform: compilation becomes observable, auditable, reproducible, cacheable, and a source of Experience — exactly like every other thing Sankalpa does. This chapter specifies the `Compilation` Resource and why modeling compilation as a Resource matters.

## 1. Why compilation is a Resource (P2)

Everything is a Resource (P2), and compilation is no exception. A one-off "compile function" would be invisible: you could not see why a plan failed, reproduce a past compilation, cache its result, or learn from recurring failures. As a Resource, a `Compilation` gets — for free — the uniform lifecycle, event emission, versioning, and observability the whole platform provides (Book 02). Compilation earns its keep in the same currency as executions and intents.

## 2. The `Compilation` Resource

```
Compilation (core.sankalpa.dev, kind=Compilation):
  spec:
    input        Ref<IRModule>        # the High IR to compile (by id/hash)
    config       CompileConfig        # pinned pass pipeline + order (Ch 02 §5)
    policyVersion Ref                 # the Policy set/version in force (Ch 04 §2)
    constraints  object               # target constraints, if any (informs later selection)
  status:
    phase        Pending|Progressing|Succeeded|Failed   # (Book 02 §Ch04)
    stagesRun    list<StageResult>    # verify/optimize/policy/determinize/lower/verifyLow
    diagnostics  list<Diagnostic>     # (Ch 07)
    output       Ref<IRModule>?       # verified Low IR, on success
    inputHash    Hash                 # canonical hash of input+config+policyVersion (Book 04 §Ch07)
    outputHash   Hash?
    substitutions list<Determinization> # reasoning→Capability foldings applied (Ch 06)
    metrics      CompileMetrics       # durations, cache hits, cost
```

- The `Compilation` is **owned** by (has an `ownerRef` to) the `IRModule`/`Goal` that triggered it (Book 02 §Ch06); its output Low IR `IRModule` is owned by the `Compilation` (Book 02 §Ch07 catalog).
- It is a **one-shot** kind (Book 02 §Ch04): it converges to `Succeeded`/`Failed` and is then retained per policy (§5).

## 3. Reproducibility via content addressing (P6, P13 substrate)

The `inputHash` is the canonical hash of `(input High IR, config, policyVersion)` (Book 04 §Ch07). This is the crux of compilation's reproducibility and caching:

- **Deterministic result.** Given the same `inputHash`, the compiler produces the same `output` Low IR (same `outputHash`) — because passes are deterministic (Ch 02 §3) and the pipeline order is pinned (Ch 02 §5). A `Compilation` is thus a *pure function* recorded as a Resource.
- **Cache.** The Compiler Manager keys results by `inputHash` (Book 03 §Ch08 §4): an identical `inputHash` need not recompile; it reuses the prior `Compilation`'s `output`. `metrics.cacheHits` records this.
- **Policy participates in identity.** Because `policyVersion` is part of `inputHash`, a policy change correctly invalidates cached compilations of affected plans (Ch 04 §2) — you never execute a plan validated against a stale policy.

## 4. Observability and audit (P5, P10)

- Every stage transition and diagnostic emits an Event (Book 03 §Ch03), secret-free (P7). The `Compilation`'s `status.stagesRun` is a durable record of *what the compiler did*: which passes ran, what they found, what was determinized, how long it took.
- This makes compilation **auditable**: for any executed plan you can point to the exact `Compilation` that produced its Low IR, the policy version it satisfied, and the determinizations applied. Combined with the Execution's record, the chain Intent → Goals → Compilation → Execution → Experience is fully traceable (Book 15 §01).

## 5. Lifecycle, retention, and Experience (P13)

- A `Compilation` reaches a terminal phase and is retained per its kind's retention policy (Book 02 §Ch04 §5) — long enough for audit and for Experience assembly, even after the Low IR it produced has itself been superseded.
- The Experience engine (Book 10) consumes `Compilation` records: recurring `Failed` compilations (by diagnostic code, Ch 07) reveal systematic planning or policy friction; the `substitutions` and `metrics` feed the cost model (Ch 03 §3) and the determinization discovery loop (Ch 06 §2). Compilation is therefore not a dead-end artifact but an input to the platform getting better.

## 6. Concurrency and idempotency

- Compiling the same `inputHash` concurrently MUST NOT run the pipeline twice wastefully: the Compiler Manager deduplicates by `inputHash` (a cache/single-flight), returning one `Compilation` result. This follows from content addressing and the no-double-work principle behind P13.
- Re-requesting a completed `Compilation` is idempotent — it returns the existing terminal Resource (Book 02 §Ch03 §3.2).

## 7. Invariants (normative summary)

1. Every compilation is a first-class `Compilation` Resource with spec (input, pinned config, policy version, constraints) and status (stages, diagnostics, output, hashes, substitutions, metrics).
2. `inputHash` = canonical hash of input + config + policy version; identical `inputHash` yields identical output (deterministic, pure-function compilation) and is cache-reused.
3. A policy-version change invalidates affected cached compilations; no plan executes against a stale policy validation.
4. Stage transitions and diagnostics emit secret-free Events, making compilation observable and the Intent→…→Experience chain fully auditable.
5. `Compilation` is one-shot, retained per policy, and consumed by Experience to improve planning, policy, cost modeling, and determinization.
6. Compilation of an identical `inputHash` is deduplicated (single-flight) and idempotent on re-request.
