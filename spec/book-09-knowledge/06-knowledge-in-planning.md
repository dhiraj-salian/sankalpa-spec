# Book 09 · Chapter 06 — Knowledge in Planning

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P7, P8, P10. Companion to Book 08 §02–§04 (planning, isolation), Book 10 §Ch04–§Ch05 (feedback).*

> This chapter specifies how Knowledge is *consumed* — retrieved and supplied to planners as context — and the guarantees that make that consumption safe and useful. It is the payoff of the whole Book: Knowledge exists to make planning better, and it must do so as **non-secret, provenance-tagged, trust-weighted** context that never compromises the planner-isolation invariants (Book 08 §04).

## 1. How planners retrieve Knowledge

During Goal derivation and planning (Book 08 §02–§03), a planner obtains Knowledge through the Knowledge Manager's retrieval interface (§Ch04 §3), which is part of the **non-secret `context`** the planner interface provides (Book 08 §03 §2). The planner:
- issues relevance retrieval for the current Intent/Goal (§Ch04 §3), receiving related Knowledge traversed and ranked by relevance **and trust** (§Ch02 §3);
- follows relationships to assemble coherent context (an architecture and its capabilities and the org conventions around them);
- weighs each item by its **trust level and provenance** — treating an `Authoritative` fact as stronger than an `Inferred` lesson.

Retrieval is **capability-gated and workspace-scoped** (§Ch04 §5): a planner receives only Knowledge its subject is authorized for, in its own tenancy.

## 2. Knowledge conditions non-deterministic planning — and stops there

Knowledge influences the **non-deterministic** stage (Book 08 §01) and *only* that stage:
- It shapes which **Goals** are derived and which **approaches** the planner considers (Book 08 §02).
- It does **not** enter the deterministic machine (compiler/runtime) except insofar as the planner *chose to encode something into the High IR it emits* — and that IR is **verified** (Book 08 §01, Book 04 §Ch08) regardless of what Knowledge informed it.

This boundary matters: Knowledge makes the planner better-informed, but it never becomes an unverified input to execution and never relaxes an invariant. A plan informed by excellent Knowledge and a plan informed by none are held to the *same* verification and policy (Book 05 §Ch04). Knowledge improves the *quality* of plans; it never expands what plans are *allowed* to do.

## 3. Provenance and trust in use (P10)

Because Knowledge carries provenance and trust (§Ch02 §3), planning can — and MUST — use them:
- **Weighting.** Higher-trust Knowledge is weighted more heavily; low-trust inferences inform but do not override authoritative facts (§Ch04 §3).
- **Citation.** A planner SHOULD be able to *cite* which Knowledge informed a Goal/plan (by reference), so the plan is explainable and the influence is auditable (P10). This citation flows into the Experience (Book 10 §Ch02) and back into the loop.
- **Revisability.** Because influence is provenanced, when Knowledge is later corrected or retired (Book 10 §Ch04 §5), the effect on planning is traceable — you can see which plans were informed by since-retired Knowledge.

Unprovenanced, untrusted Knowledge would make planning a black box that cannot be weighed, cited, or corrected — which is why provenance and trust are mandatory (§Ch02 §1).

## 4. The secret-free guarantee, restated at the point of use (P7)

This is the chapter where Knowledge meets the planner — i.e. where a secret in Knowledge would leak into reasoning context, the exact surface P7 protects most (Book 11 §01). The guarantee is therefore restated at the point of consumption:

- Knowledge supplied to planning **never contains a secret value** (§Ch01 §6, §Ch03 §5, §Ch04 §5). It may reference a secret by class/reference ("this task needs a `payments`-class credential"), which the planner encodes as a `SecretRef` in IR (Book 08 §03 §3) — resolved only at execution (Book 06 §Ch06).
- The Knowledge Manager MUST NOT surface secret material into planning context (Book 03 §Ch11 §2). This is enforced by construction: Knowledge is secret-free everywhere (vault, graph, sync), so there is no secret in Knowledge to surface.
- Combined with the planner-isolation disciplines (Book 08 §04), this closes the loop: the planner receives rich Knowledge context *and* zero secrets — the two are not in tension because Knowledge was built secret-free from the start.

## 5. The closed loop, realized

Knowledge-in-planning is where the learning loop pays off (Book 10 §01 §1, §Ch05):
```
Experience → (promote lessons, Book 10 §Ch04) → Knowledge → (retrieve, this chapter) → better Goals/plans (Book 08)
   → better Execution → richer Experience → ...
```
Each turn, planners form better objectives and choose better approaches because prior, evaluated Experience taught the workspace what "good" looks like — surfaced as durable, provenanced, trust-weighted, secret-free Knowledge. This is the mechanism by which the platform *improves at planning over time* (Book 01 §05), distinct from and complementary to determinization (which improves at *executing* repeated reasoning, Book 10 §Ch06).

## 6. Invariants (normative summary)

1. Planners retrieve Knowledge as non-secret, provenance-tagged, trust-weighted context, capability-gated and workspace-scoped.
2. Knowledge conditions only the non-deterministic planning stage; it enters the deterministic machine solely as verified IR the planner chose to emit, and never relaxes verification or policy.
3. Planning weighs Knowledge by trust, cites the Knowledge that informed a plan (auditable, P10), and the influence is revisable when Knowledge is later corrected/retired.
4. Knowledge supplied to planning never contains a secret value — enforced by Knowledge being secret-free everywhere; secrets appear only as class/reference, encoded as `SecretRef` and resolved at execution (P7).
5. Knowledge-in-planning realizes the closed learning loop, improving the platform's planning over time, complementary to determinization's improvement of repeated execution.
