# Book 05 · Chapter 02 — Pass Framework

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P9, P11 and IR-P8, IR-P10.*

> Every transformation and analysis the compiler performs is a **pass**. This chapter defines the pass abstraction, the analysis/transform distinction, ordering and dependencies, and the contract a pass MUST satisfy — including the semantics-preservation and determinism obligations that make the pipeline trustworthy even when passes are third-party plugins.

## 1. The pass abstraction

A **pass** is a unit of compiler work over an IR module. There are two kinds:

- **Analysis pass** — computes information *about* a module without changing it (effects summary, dataflow/dominance, cost model, reuse/duplication map). Produces an **analysis result** other passes consume.
- **Transform pass** — rewrites a module into another module. MUST be semantics-preserving (§4).

```
Pass:
  id            string           # stable identifier
  kind          Analysis | Transform
  level         High | Low        # which IR level it operates on (Book 04)
  requires      list<AnalysisId>  # analyses it depends on
  invalidates   list<AnalysisId>  # analyses its rewrite makes stale (transform only)
  run(module, analyses) -> module' | analysisResult
```

Passes are pure functions of `(module, required analyses)` (§3); they hold no hidden state between runs. This is what makes compilation reproducible and cacheable (Book 04 §Ch07).

## 2. Analysis/transform separation

The separation is strict and load-bearing:
- A transform pass **MUST NOT** compute analyses ad hoc; it declares `requires` and receives them. This lets the framework cache and share analyses, and lets a transform be reasoned about in terms of what it consumes.
- A transform pass **MUST** declare which analyses it `invalidates`; the framework recomputes them before a later pass that requires them. A pass that under-declares invalidation is a defect: a stale analysis could make a subsequent transform unsound.
- Analysis results are content-addressed by `(analysisId, module hash)` and cached (Book 04 §Ch07); an unchanged module reuses analyses.

## 3. Purity and determinism (IR-P8 support)

Every pass **MUST** be:
- **Deterministic** — same inputs ⇒ same output, always. No clocks, randomness, ambient config, or network. (Configuration a pass needs is an explicit, hashed input, so it participates in cache keys.)
- **Total** — terminates for every valid input, producing either a result or a diagnostic (Ch 07); a pass may not hang the compiler.
- **Isolated** — for plugin passes, run under plugin isolation (Book 03 §Ch09) with least-privilege; a pass has no ambient authority and cannot reach secrets, storage, or the network (P8).

Determinism of passes is why identical High IR compiles to identical Low IR (a cache hit, Book 03 §Ch08 §4) — a property the whole determinization story (Ch 06) rests on.

## 4. Semantics preservation (IR-P10) — the transform contract

A transform pass **MUST** preserve the observable semantics of the module: the optimized/lowered module, executed with equal inputs and resolved bindings, produces the same observable effects and outputs as the original (Book 04 §Ch02 IR-P1/IR-P10). Concretely a transform:

- **MUST NOT** add, remove, or alter an externally-visible effect (Book 04 §Ch06) except by removing a *provably dead* one (an effect no observable output depends on and that is itself pure/idempotent-safe to drop). Removing a genuine write effect is forbidden.
- **MUST NOT** change types observably or weaken the effect declaration below what the module actually does (deny-by-default still holds after the pass).
- **MUST NOT** introduce non-determinism — it may not turn a pure construct into one depending on `Time`/`Random`/`Reason` (Book 04 §Ch06 §4).
- **MAY** reorder, deduplicate, fuse, or eliminate constructs only insofar as the effect/dataflow analyses prove the change unobservable.

The framework **enforces** this defensively: after the pass pipeline, and again after lowering, the output is **re-verified** (Book 04 §Ch08). A pass that produces IR failing verification — or that a semantics-preservation check flags — is rejected, and its output discarded. Trust is never extended to a pass's claim of correctness; it is checked.

## 5. Ordering and the pipeline manager

- Passes run in a **defined order** determined by the fixed pipeline stages (Ch 01 §2) and by `requires`/`invalidates` dependencies. The order within a stage is deterministic and versioned (it is part of the compilation configuration, and thus part of the cache key).
- The framework computes a valid schedule (a topological order honoring dependencies) and MUST reject a configuration with a dependency cycle among passes.
- Because pass order affects output, it is **pinned per compilation** and recorded on the `Compilation` Resource (Ch 08); changing the pipeline configuration is a versioned change (P6) so historical compilations remain reproducible.

## 6. Extensibility (P11) and trust

- Optimization and determinization passes and lowering backends are **registered** — often via Packages (Book 12) — through the pass/backend interface. The compiler knows the *interface*, never a specific pass (Book 03 §Ch04 §3).
- Extension passes are **untrusted**: isolated (Book 03 §Ch09 §2), least-privilege (P8), and their output re-verified (§4). A malicious pass cannot smuggle non-determinism, an undeclared effect, or a secret into IR, because verification (Book 04 §Ch08) would reject the result.
- A pass declares the `irVersion` range it supports (Book 04 §Ch09); the framework refuses to run a pass against an incompatible IR version rather than risk misinterpretation.

## 7. Invariants (normative summary)

1. All compiler work is passes; analysis passes observe, transform passes rewrite; the two are strictly separated with declared `requires`/`invalidates`.
2. Every pass is deterministic, total, and (for plugins) isolated and least-privilege; configuration is an explicit hashed input.
3. Transform passes preserve observable semantics: no added/removed/altered effect except provably-dead removal, no introduced non-determinism, no weakened effect declaration.
4. Output is re-verified after the pipeline and after lowering; a pass whose output fails verification or semantics checks is rejected and discarded.
5. Pass order is dependency-driven, deterministic, pinned per compilation, and versioned so compilations reproduce.
6. Passes/backends are untrusted extensions run behind a stable interface with version compatibility enforced.
