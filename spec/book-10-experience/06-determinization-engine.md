# Book 10 · Chapter 06 — The Determinization Engine

*Nature: **Normative**. · Reflects: RFC-0001; realizes principle P13 (and P1, P9). Companion to Book 05 §Ch06 (determinization passes), Book 03 §Ch06 (Capability Manager), Book 08 §05 (non-determinism boundary).*

> This is the discovery half of determinization — the mechanism by which Sankalpa *learns* that a piece of repeated reasoning has become predictable and can be turned into a deterministic Capability. The compiler's substitution half is Book 05 §Ch06; the two meet at the Capability Manager (Book 03 §Ch06 §6). This chapter is where the platform's mission — *continuously evolve toward deterministic execution* (Book 01 §05) — becomes a concrete, evidence-driven algorithm.

## 1. Discovery vs. substitution (the two halves)

Determinization spans two subsystems, deliberately split by competence (Book 05 §Ch06 §5):

- **Discovery (this chapter, Experience Manager).** A *learning* activity over historical Experience: find reasoning that recurs with stable outcomes, and — when the evidence is strong enough — synthesize and register a deterministic Capability. This belongs in Experience because it reasons over accumulated data.
- **Substitution (Book 05 §Ch06, compiler).** A *transform* activity during compilation: replace an eligible `Reasoning` node with the registered Capability, under verification. This belongs in the compiler because it must be typed, effect-checked, and verified.

Keeping discovery out of the compiler keeps compilation deterministic and fast; keeping substitution out of Experience keeps unsound substitutions impossible. Each half does only what it is competent to do safely.

## 2. What discovery mines

The engine mines **reasoning traces** (Book 10 §02 §4) across Experiences. For each candidate `Reasoning` shape it asks:
- **Recurrence.** Does the same reasoning shape (by plan-position/hash) appear across many Experiences?
- **Input-equivalence.** Do occurrences share **equivalent typed inputs** (by content hash, Book 04 §Ch07)? Because inputs are typed and content-addressed, "same input" is a *decidable* check, not a fuzzy guess.
- **Output-stability.** For equivalent inputs, are the typed outputs **stable** (identical or within a defined tolerance)?
- **Eligibility.** Did the planner mark the node `determinize: true` (Book 04 §Ch03 §3.2, Book 08 §05 §4)? Volatile reasoning the planner marked `false` is never a candidate.

Only reasoning that is recurrent, input-equivalent, output-stable, and eligible is a determinization candidate.

## 3. Evidence thresholds (the gate to synthesis)

The engine synthesizes a Capability only when **evidence crosses a policy-defined threshold**:
- a minimum number of occurrences,
- a maximum output variance (stability bound),
- a recency requirement (the pattern holds *recently*, not just historically),
- optionally, human confirmation for consequential classes (mirroring §Ch04 §4).

Thresholds are **policy-governed** (P9, Book 11 §06): a workspace controls how aggressively its reasoning is determinized, and may forbid determinizing certain effect classes (e.g. anything touching `pii`). Conservative-by-default: thin or stale evidence yields *no* synthesis. The cost of a wrong determinization (executing a stale answer) is higher than the cost of not determinizing (paying for reasoning again), so the gate favors safety.

## 4. Synthesis and registration

When the threshold is met, the engine:
1. **Synthesizes a Capability** with a **typed signature identical** to the reasoning's inputs/outputs (Book 04 §Ch05) and an **effect set that is a subset** of the reasoning's — `Reason` removed, only genuinely-needed effects (e.g. a `Network(read)`) retained (Book 04 §Ch06 §7).
2. **Registers it** with the Capability Manager (Book 03 §Ch06 §6), provenance `determinized`, linked to the supporting Experiences (evidence) by reference.
3. **Makes it available** to the compiler's substitution pass (Book 05 §Ch06 §2), which will fold eligible reasoning into it on future compilations — *only* if the six substitution gates still hold at compile time (Book 05 §Ch06 §3).

The engine only *proposes* (synthesizes+registers); the compiler *disposes* (substitutes under verification). Discovery can never itself alter a running plan.

## 5. Reversibility and drift (the safety net)

Determinization is **not permanent** (Book 05 §Ch06 §4). The engine continuously watches new Experience for **drift**:
- If a determinized Capability's outputs start diverging from fresh reasoning (the world changed), or its error rate rises, the engine **retires** it — revoking/deprecating it via the Capability Manager (Book 03 §Ch06 §7).
- After retirement, future compilations fail the freshness gate (Book 05 §Ch06 §3.4) and **fall back to the original `Reasoning` node** — the plan degrades gracefully to non-deterministic reasoning rather than executing a stale answer.

This closed monitor-and-retire loop is why determinization is *safe to attempt*: the worst case is a retired Capability and a return to reasoning, never a silently-wrong deterministic execution. The engine both creates and *maintains* determinism.

## 6. The compounding effect (P13 realized)

Over time, a workspace accrues a growing library of governed, deterministic Capabilities for its most-repeated reasoning. The consequence (Book 01 §05, Book 08 §05 §4):
- Plans increasingly resolve `Reasoning` nodes to `CapabilityInvocation`s — **faster, cheaper, reproducible, auditable**.
- Genuinely novel reasoning still flows through the model; solved reasoning does not.
- The **non-determinism boundary recedes** (Book 08 §05 §4), one determinized Capability at a time.

This is the mechanical realization of *"whenever repeated reasoning is observed, the system should attempt to transform it into reusable deterministic capabilities"* (Book 01 mission). Experience is the eye that observes the repetition; this engine is the hand that transforms it; the compiler and Capability Manager are where the transformation is made sound.

## 7. Governance, audit, and tenancy

- Every synthesis, registration, and retirement is **audited** (Book 11 §09) and emits Events (P5) — the platform's *learning* is as accountable as its *doing*.
- Determinization is **policy-governed** (§3) and **tenant-scoped** (Book 11 §08 §5): a workspace's reasoning is determinized only within its own boundary and rules; cross-tenant determinization requires an explicit grant.
- Synthesized Capabilities are ordinary governed Capabilities (Book 03 §Ch06): capability-gated, effect-bounded, revocable — nothing about being machine-discovered exempts them from the security model.

## 8. Invariants (normative summary)

1. Determinization splits into discovery (Experience, this chapter) and substitution (compiler, Book 05 §Ch06); discovery proposes, the compiler disposes under verification.
2. Discovery mines reasoning traces for recurrence, decidable input-equivalence, output-stability, and planner eligibility; only all four make a candidate.
3. Synthesis occurs only above policy-defined evidence thresholds (occurrences, variance, recency, optional human confirmation); thin/stale evidence yields no synthesis (conservative-by-default).
4. Synthesized Capabilities have identical typed signatures and subset effects, are registered with `determinized` provenance and evidence links, and are used only via the compiler's gated substitution.
5. The engine monitors for drift and retires faulty Capabilities; future compilations then fall back to reasoning — never a silently-wrong deterministic answer.
6. Determinization is audited, policy-governed, tenant-scoped, and its outputs are ordinary governed Capabilities; it realizes the toward-determinism mission by receding the non-determinism boundary over time (P13).
