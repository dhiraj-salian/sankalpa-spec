# Book 05 · Chapter 02 — Pass Framework

*Nature: **Normative**. · Reflects: RFC-0001, RFC-0011 (effect-graph conservation); realizes principles P1, P9, P11 and IR-P8, IR-P10.*

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

The framework **enforces** this defensively, and it is important to be exact about *what* it enforces, because the two checks are different in kind:

- **Re-verification** (Book 04 §Ch08), after the pass pipeline and again after lowering, is a **total predicate on a single module**: it decides whether the *output* is well-formed, well-typed, effect-declared, deterministic, and secret-safe. It never consults the input. `verify(out)` says nothing about `out ≡ in`.
- **Effect-graph conservation** (§4.1) is the **comparative** check: it compares the pass's output *against its input*. This is the check that speaks to IR-P10 at all.

The distinction matters because deciding semantic equivalence of two arbitrary modules is **undecidable**. Re-verification is therefore not a weaker form of the missing check — it is a *different* check. A pass that emits a **well-formed, well-typed, correctly-effect-declared, deterministic, secret-safe** module with **different semantics** passes re-verification cleanly. Passes may be untrusted third-party code (§5), so "trust is never extended to a pass's claim of correctness" is only true to the extent something actually checks the claim.

### 4.1 Effect-graph conservation (mandatory)

After every transform pass, the framework **MUST** compute the **effect graph** of the input and output modules — the multiset of externally-visible effects (Book 04 §Ch06) with their observable dependency order — and **reject** the transform unless:

- **Conservation.** The output's externally-visible effect multiset equals the input's, except for effects the pass declares removed as *provably dead* — permitted only where §4 already allows, and only with the witness of §4.2.
- **Order.** For any two effects with an observable ordering dependency in the input, the output preserves that order.
- **Target identity.** Each conserved effect's declared target/scope (Book 04 §Ch06) is unchanged. This is what catches the wrong-dedup and **retargeting** classes: the count and shape survive, but identity does not.
- **Monotonicity.** The output's effect declaration is not weakened — now checked comparatively rather than assumed.

Conservation is evaluated **modulo a declared effect-refinement relation**. Some legitimate transforms *do* change the effect graph: determinization removes `Reason` (§Ch06); a lowering may split one high-level effect into several. Each such pass/stage **MUST declare its refinement relation** — which input effect refines to which output effects — and conservation is checked against that declaration rather than naive multiset equality. A declared relation is both machine-checkable and human-reviewable; an **undeclared** graph change is a rejection.

### 4.2 Obligations become checkable artifacts

Each "MUST NOT" in §4 becomes an artifact the framework checks rather than a rule the pass is trusted to honor:

- A pass that **removes** an effect **MUST** emit a **dead-effect witness** — the dataflow fact establishing that no observable output depends on it. No witness ⇒ removal rejected. This applies the discipline the IR already uses for the halting problem (Book 04 §Ch08 §2.6): an undecidable *"is this really dead?"* becomes a checkable *"show your witness."*
- A pass that **reorders, dedups, or fuses MUST** declare the analysis fact it relied on (alias, dependency, purity); the framework re-checks that fact against the input rather than trusting the pass's assertion.

### 4.3 Translation validation (opt-in)

A pass **MAY** declare `validated: true` and emit, per compilation, an **equivalence certificate** for *this* input/output pair, checked by a **small trusted validator** in the core. This is decidable **per instance** — it validates one concrete rewrite, not all possible ones — and is the established answer from the compiler literature (Pnueli's translation validation; CompCert; Alive/Alive2 for LLVM). Certificate-checked passes are exempt from nothing: §4.1 still applies. Workspace policy (Book 11 §06) **MAY** require `validated: true` for passes permitted to run on high-consequence plans.

**One case is not left to policy:** any pass running **after the `Approval` instruction has been lowered into the plan** (§Ch04 §3, Book 11 §07) **MUST** be `validated: true`. The approval surface shows the human the plan's *declared* consequence, so a later unvalidated rewrite would make the human approve one thing while the runtime does another — and §4.1 catches a retargeted *effect* but not an intra-value rewrite. A workspace **MUST NOT** be able to opt out of the guarantee at the point where consent binds to specifics.

### 4.4 What this does and does not guarantee

Stated precisely, because a *necessary* condition reads easily as a sufficient one:

- **Conservation is necessary, not sufficient.** It catches added, removed, reordered, duplicated, and **retargeted** effects — the dominant defect class, and the one a malicious pass would use. It does **not** decide full equivalence.
- Absent a §4.3 certificate, a pass's **internal, value-level** correctness is **not** verified. A pass that conserves the effect graph while computing the wrong value inside it is not caught by §4.1.
- What the framework *does* guarantee without a certificate: a pass cannot introduce an **undeclared effect**, **non-determinism**, **ill-typing**, or a **secret leak** (re-verification), and cannot alter the **effect graph** undetected (§4.1).

Trust is never extended to a pass's claim of correctness *to the extent it is checked* — and §4.1–4.4 say exactly how far that extends.

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
4. Two checks of different kinds guard a transform: **re-verification** (a total predicate on the *output alone*, Book 04 §Ch08) and **effect-graph conservation** (the *comparative* check of output against input, §4.1, modulo the pass's declared refinement relation). Re-verification is not a weaker form of conservation — equivalence of two arbitrary modules is undecidable, so it is a different check, and a well-formed output with different meaning passes it cleanly.
5. Pass obligations are checkable artifacts, not trusted assertions: an effect removal requires a **dead-effect witness**; a reorder/dedup/fuse requires the declared analysis fact, re-checked against the input (§4.2).
6. Conservation is **necessary, not sufficient**. Absent a per-compilation **translation-validation certificate** (§4.3), a pass's internal value-level correctness is not verified. `validated: true` is **required** for any pass running after an `Approval` is lowered into the plan, where consent binds to specifics.
7. Pass order is dependency-driven, deterministic, pinned per compilation, and versioned so compilations reproduce.
8. Passes/backends are untrusted extensions run behind a stable interface with version compatibility enforced.
