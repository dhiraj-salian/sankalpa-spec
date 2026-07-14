# Book 05 · Chapter 03 — Optimization Passes

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P13 and IR-P10.*

> Optimization passes make plans cheaper, faster, and more reliable **without changing what they mean** (IR-P10). This chapter defines the optimization stage: its correctness contract, the catalog of core optimizations, and the interaction with effects and cost. Optimization runs *before* policy validation (Ch 01 §2) so governance sees the plan's true, post-optimization effect set.

## 1. The optimization contract

Every optimization is a transform pass (Ch 02) and inherits its contract, with these emphases specific to optimization:

- **Semantics-preserving, always (IR-P10).** An optimization may change *how much work* happens but never the *observable result*. The effect/dataflow analyses (Ch 02 §1) must *prove* a change unobservable before it is applied; unprovable ⇒ not applied.
- **Effect-monotone.** An optimization may only **remove provably-dead effects** or leave the effect set unchanged; it may never add an effect or drop a live one (Ch 02 §4). After optimization the module's declared effects still exactly cover what it does (deny-by-default, Book 04 §Ch06).
- **Determinism-preserving.** An optimization may never introduce a dependency on `Time`/`Random`/`Reason` (Book 04 §Ch06 §4), nor reorder across an effect boundary the analysis cannot prove commutative.
- **Optional and safe to skip.** Every optimization is optional: skipping all of them yields a correct (if slower) compilation. Correctness never depends on an optimization having run. This is what lets optimizations be pluggable (P11) without risking soundness.

## 2. Core optimizations

The following are the baseline optimizations the compiler provides. Each states what it does and the analysis that justifies it.

### 2.1 Dead-step elimination
Remove a step whose outputs no observable output transitively depends on **and** whose effects are pure/dead. Justified by dataflow + effect analysis. A step with a live external effect (e.g. `Network(write)`) is **never** dead, even if its outputs are unused — its side effect is observable.

### 2.2 Common-subplan elimination (CSE)
Identify structurally identical, effect-compatible subgraphs (by content hash, Book 04 §Ch07) and compute them once, sharing the result. Justified by content addressing + purity/idempotency analysis. Only applies where re-execution would be observably equivalent to sharing (pure, or idempotent-read of a stable source per policy, Book 04 §Ch06 §4).

### 2.3 Capability fusion
Fuse adjacent `CapabilityInvocation`s into a single invocation when a fused Capability exists that is provably equivalent (same typed inputs→outputs, effect set = union), reducing round-trips. Justified by type + effect equivalence. The fused Capability is a registered Capability (Book 03 §Ch06), so fusion is governed and auditable.

### 2.4 Constant folding & propagation
Evaluate pure data nodes (Book 04 §Ch03 §3.4) over literal inputs at compile time; propagate the results. Justified by purity. MUST NOT fold anything touching a `SecretRef` (opaque, Book 04 §Ch05 §2.2) or a non-pure effect.

### 2.5 Parallelization / serialization tuning
Where High IR granted `Parallel` (Book 04 §Ch03 §3.3), decide (informed by cost analysis) how much concurrency to *request* in the schedule; where determinism/ordering requires, serialize. This adjusts the schedule hint, never the results (legal orders are observationally equivalent by the effect/idempotency contract, Book 04 §Ch04 §2).

### 2.6 Caching insertion
Mark content-addressed, policy-stable read subgraphs as cacheable so repeated executions reuse results. Justified by effect analysis (read of a source policy marks stable) — the execution-time analog of CSE and a direct lever on cost.

## 3. Cost model and Experience (P13 linkage)

Optimizations that involve a *choice* (fusion vs. not, degree of parallelism, what to cache) consult a **cost analysis** (Ch 02 §1) whose model is fed by **Experience** (Book 10): observed latencies, costs, and failure rates of Capabilities and runtimes. This makes the compiler *cost-aware* and improves its choices over time — a benign, semantics-preserving form of the platform learning. Because these are only *choices among equivalent options*, a wrong cost estimate can make a plan slower but never incorrect (§1).

## 4. What optimization is NOT allowed to do

To keep the contract unambiguous, optimization **MUST NOT**:
- Remove or reorder a live external effect, or merge two non-idempotent writes.
- Fold, read, or otherwise derive a value from a `SecretRef` (P7).
- Convert a `Reasoning` node into a deterministic one — that is **determinization** (Ch 06), a distinct, evidence-gated pass, not an optimization. (Optimization never *assumes* a reasoning result.)
- Change observable types or weaken effect declarations.
- Depend on having run: no later stage may require an optimization's effect for correctness.

## 5. Verification backstop

As with all transforms, optimized IR is **re-verified** (Book 04 §Ch08) and checked for semantics preservation before the pipeline proceeds (Ch 02 §4). An optimization that produced ill-formed, under-declared, or non-deterministic IR is caught here and its output discarded. Optimizations are therefore *safe by construction*: even a buggy optimization pass cannot corrupt a compilation — at worst it is rejected.

## 6. Invariants (normative summary)

1. Every optimization is semantics-preserving, effect-monotone (remove-dead-only), and determinism-preserving; unprovable changes are not applied.
2. Optimizations are optional and safe to skip; correctness never depends on one having run.
3. Live external effects and non-idempotent writes are never removed or merged; secrets are never folded/derived.
4. Turning reasoning deterministic is determinization (Ch 06), never an optimization; optimization never assumes a reasoning result.
5. Choice-based optimizations use an Experience-fed cost model; a wrong estimate affects speed, never correctness.
6. Optimized IR is re-verified and semantics-checked; buggy optimizations are rejected, not trusted.
