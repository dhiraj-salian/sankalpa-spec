# Book 01 · Chapter 05 — Determinism and Determinization

*Nature: **Normative** (defines terms the whole spec depends on). · Reflects: RFC-0001, RFC-0002 (recorded-reasoning carve-out), RFC-0003 (drift/reversibility); realizes P1, P13. Companion to Book 04 §Ch02, Book 05 §Ch06, Book 10 §Ch06.*

> "Deterministic execution" is the mission's key phrase (§Ch01), and "determinization" is its engine (P13). This chapter defines both precisely — what determinism *means* here, where non-determinism is *permitted*, and how the system *converts* one into the other. Imprecision about these terms would undermine the whole specification, so they are pinned down here.

## 1. What determinism means (and does not)

**Determinism in Sankalpa means: given the same Low IR, the same resolved inputs, the same resolved bindings, and the same recorded outputs for every `CapturedReasoning` node and every `Time`/`Random` effect, execution produces the same observable effects and outputs — reproducibly** (Book 04 §Ch02 IR-P1, Book 06 §Ch03 §1). For a plan with no reasoning/`Time`/`Random` nodes the recorded set is empty and this reduces to the three-part form (Low IR + inputs + bindings) — the common case for a fully-determinized plan.

Two clarifications guard against over-claiming:
- **Determinism is relative to declared inputs and effects** (Book 06 §Ch03 §7). Sankalpa does *not* claim the external world is unchanging. If an external API returns different data on two runs, that difference enters through a **declared effect** (`Network(read)`, Book 04 §Ch06) as an input — honestly modeled, not hidden. Everything the plan *does with* a given set of inputs is reproducible; the inputs themselves are declared effects.
- **Determinism is about the machine, not the model.** The spec assumes *no* model determinism (Book 08 §Ch05 §5). A model call is a `Reasoning` node whose actual output is **captured to the execution's reasoning ledger** (Book 06 §Ch03 §1, Book 10 §03); the deterministic guarantee holds *around* it because replay injects the recorded output rather than re-invoking the model. This is the **recorded-reasoning carve-out**: without it the guarantee would be literally false for any un-determinized reasoning node, which is the common state before determinization has accrued evidence.

So "deterministic execution" is a precise, achievable property — reproducibility of the plan's behavior given its declared inputs — not a claim to control reality.

## 2. Where non-determinism is permitted

Non-determinism is **permitted in exactly two forms**, both explicit and typed (Book 04 §Ch02 IR-P1):
1. **`Reasoning` nodes** (Book 04 §Ch03 §3.2) — a decision delegated to a planner/model, producing a typed output. This is the sanctioned home of genuine reasoning.
2. **Declared `Time`/`Random` effects** (Book 04 §Ch06 §4) — modeled as explicit typed inputs, so reasoning *about* the plan stays deterministic even where the plan consumes a clock or randomness.

Everywhere else, determinism is required and enforced by verification (Book 04 §Ch08 §2.5). Non-determinism is thus never *diffuse* — it is localized to named, typed, effect-declared points the deterministic machine can reason about, govern, cache, and replay.

## 3. What determinization is

**Determinization is the process of converting repeated non-deterministic reasoning into a reusable deterministic Capability** (P13, the mission's second clause). It is Sankalpa's compounding advantage made mechanical:
- When a `Reasoning` node recurs with **equivalent typed inputs** and **stable outputs**, over an evidence threshold, it can be replaced by a deterministic `Capability` (Book 10 §Ch06 discovers, Book 05 §Ch06 substitutes).
- After determinization, what was a model call becomes a governed, typed, cheaper, reproducible capability invocation — and the non-determinism boundary (§Ch03 §4) has receded by one node.

Determinization is *not* an optimization or an assumption (Book 05 §Ch06 §1): it acts only on eligible, evidence-backed reasoning, under verification, and it is **reversible** — a determinized Capability that drifts is retired, and compilation falls back to reasoning (Book 10 §Ch06 §5). Drift is *measured*, not assumed: a policy-governed fraction of executions re-runs the original reasoning in **shadow** alongside the substituted Capability and compares outputs (Book 10 §Ch06 §5), so retirement is evidence-driven rather than a bare promise. The worst case is a return to reasoning, never a silently-wrong deterministic answer.

## 4. Why this is the platform's compounding advantage

The mission is not just "make execution deterministic" but "*continuously evolve toward* deterministic execution" (§Ch02 §3). Determinization is the *continuously evolve* part:
- Every execution is a potential lesson (Book 10). Over time, a workspace's most-repeated reasoning becomes a growing library of deterministic Capabilities.
- The system therefore *gets better at its own work the more it is used* — cheaper, faster, more reliable — while genuinely novel reasoning still flows through the model.
- This is the difference between a tool (static) and a platform that *learns to be more deterministic* (compounding). It is the single most distinctive claim of the architecture, and it is realized by the effect system (which instruments reasoning, Book 04 §Ch06 §7), content addressing (which makes "same reasoning" decidable, Book 04 §Ch07), and the discovery/substitution split (Books 10/05).

## 5. Determinism and the mission, unified

Putting §1–§4 together: Sankalpa turns intent into *deterministic execution* (determinism, §1) by confining non-determinism to sanctioned points (§2), and it *evolves toward* determinism by converting repeated reasoning at those points into capabilities (determinization, §3–4). The first is the static guarantee; the second is the dynamic trajectory. Together they are the mission (§Ch01 §1). Every mechanism in Books 04–10 exists to make this pair real and safe.

## 6. Invariants (normative summary)

1. Determinism means reproducible observable behavior given the same Low IR, inputs, bindings, **and recorded reasoning/`Time`/`Random` outputs** (the recorded-reasoning carve-out, Book 06 §Ch03 §1); it is relative to *declared* inputs/effects and does not assume an unchanging world or a deterministic model.
2. Non-determinism is permitted only as typed `Reasoning` nodes and declared `Time`/`Random` effects; everywhere else determinism is required and verification-enforced; non-determinism is never diffuse.
3. Determinization converts repeated, equivalent-input, stable-output reasoning into governed deterministic Capabilities — not an optimization or assumption, evidence-gated, verified, and reversible.
4. Determinization is the platform's compounding advantage: the system evolves toward determinism the more it is used, while novel reasoning still flows through the model.
5. Determinism (static guarantee) and determinization (dynamic trajectory) together constitute the mission; every mechanism in Books 04–10 serves them.
