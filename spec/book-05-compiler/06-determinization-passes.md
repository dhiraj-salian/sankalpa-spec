# Book 05 · Chapter 06 — Determinization Passes

*Nature: **Normative**. · Reflects: RFC-0001, RFC-0003 (`shadowSource`/drift); realizes principle P13 (and P1). Companion to Book 10 §06 (Experience/determinization engine), Book 03 §Ch06 (Capability Manager).*

> Determinization is Sankalpa's compounding advantage made mechanical: **repeated non-deterministic reasoning is converted into reusable deterministic Capabilities** (P13). This chapter specifies the compiler side — how a determinized Capability, once discovered, is substituted into a plan, and the strict safety gates that keep the substitution sound. Discovery (mining Experience for candidates) is specified in Book 10 §06; the two meet here.

## 1. The idea and the boundary

A `Reasoning` node (Book 04 §Ch03 §3.2) is the one place non-determinism is allowed. Some reasoning, run repeatedly with equivalent typed inputs, reliably yields equivalent outputs (classify a ticket, extract a field, choose a route). When that is *demonstrated* by evidence, the reasoning can be replaced by a deterministic **Capability** — cheaper, faster, reproducible, and governable like any other Capability.

The boundary is critical: determinization is **not** an optimization (Ch 03 §4) and **not** an assumption. An optimizer may never presume a reasoning result. Determinization acts **only** on reasoning explicitly marked eligible (`determinize: true`, Book 04 §Ch03 §3.2) **and** backed by sufficient evidence (§3). Absent both, the `Reasoning` node stays exactly as the planner emitted it.

## 2. The two-phase mechanism

Determinization spans two subsystems; the compiler owns phase B.

- **Phase A — Discovery (Book 10 §06).** The Experience/determinization engine mines Experience records for recurring reasoning: same reasoning shape, equivalent typed inputs (matched by content hash of inputs + signature, Book 04 §Ch07), stable outputs, over a threshold of occurrences with acceptable variance. When criteria are met, it **synthesizes and registers** a candidate `Capability` (typed signature identical to the reasoning's, effects a subset — `Reason` removed) via the Capability Manager (Book 03 §Ch06 §6), with provenance `determinized` and a link to the supporting evidence.

- **Phase B — Substitution (this chapter).** During compilation, the **determinization pass** (a transform pass, Ch 02) checks each eligible `Reasoning`/`CapturedReasoning` node: if a registered, applicable determinized Capability exists whose evidence still holds, it **replaces** the reasoning node with a `CapabilityInvocation` of that Capability, recording the substitution (Book 04 §Ch04 §3). The pass MUST mark the resulting invocation with a **`shadowSource`** — the identity of the reasoning it replaced — so the runtime can reconstruct and shadow-sample the original reasoning for drift detection (Book 10 §Ch06 §5); without it, the substituted node has no data source to detect drift against. Otherwise it leaves the node untouched.

Phase B runs **before lowering** (Ch 01 §2) so lowering works on the final graph.

## 3. Safety gates (the pass MUST enforce)

Substitution changes what executes, so it is gated harder than any optimization:

1. **Eligibility.** Only nodes marked `determinize: true` are candidates. A planner opts a reasoning into determinization; the system never silently determinizes reasoning the planner marked volatile.
2. **Type identity.** The Capability's signature MUST exactly match the reasoning's typed inputs/outputs (Book 04 §Ch05). A mismatch disqualifies substitution.
3. **Effect subset.** The Capability's declared effects MUST be a **subset** of the reasoning's (Book 04 §Ch06 §7) — `Reason` removed, no new effect introduced. Deny-by-default still holds after substitution.
4. **Evidence sufficiency & freshness.** The supporting evidence (occurrence count, variance bound, recency) MUST meet the policy threshold *at compile time*. Stale or thin evidence disqualifies substitution — determinization is revocable if the world changed.
5. **Policy permission.** Determinization itself is policy-governed (P9): a workspace MAY forbid or constrain determinizing certain reasoning classes (e.g. anything touching a `pii` effect). A disallowed substitution is not applied.
6. **Verification backstop.** The substituted module is re-verified and semantics-checked (Ch 02 §4). Because the Capability is typed, effect-subset, and evidence-backed, the substitution is a valid, effect-monotone transform.

If any gate fails, the pass **MUST** leave the reasoning node as-is. The safe default is *do not determinize*.

## 4. Reversibility

Determinization is **not permanent**. Because provenance and evidence are recorded (Book 03 §Ch06 §6, Book 10 §06):
- If a determinized Capability later proves wrong, the engine **retires** it (revokes the grant / marks it deprecated, Book 03 §Ch06 §7). Drift is detected by **shadow sampling** (Book 10 §Ch06 §5): a policy-governed fraction of executions re-runs the `shadowSource` reasoning alongside the Capability and compares outputs — necessary because the substituted node no longer calls the model on its own. Subsequent compilations fail the freshness gate (§3.4) and fall back to the original `Reasoning` node — the plan degrades gracefully to non-deterministic reasoning rather than executing a stale answer.
- This reversibility is why substitution is safe to attempt: the worst case is a retired Capability and a return to reasoning, never a silently-wrong deterministic execution.

## 5. Why this belongs in the compiler

Substitution must happen where the plan is transformed, typed, effect-checked, and verified — i.e. the compiler — so that a determinized Capability enters a plan under the *same* soundness guarantees as any other transform (Ch 02 §4). Doing it anywhere else (e.g. at planning or runtime) would bypass verification and risk unsound substitution. Discovery, by contrast, is a *learning* activity over historical data and rightly lives in the Experience engine (Book 10). The split keeps each side in its competence.

## 6. The compounding effect (P13, realized)

Over time, a workspace's most-repeated reasoning becomes a growing library of governed, deterministic, cheap Capabilities. Plans that once required many `Reason` steps increasingly resolve to `CapabilityInvocation`s — faster, reproducible, and auditable — while genuinely novel reasoning still flows through the LLM. This is the mission of Book 01 §05 made operational: *the system continuously evolves toward deterministic execution*, one determinized Capability at a time, without ever sacrificing safety.

## 7. Invariants (normative summary)

1. Determinization substitutes a registered deterministic Capability for an *eligible*, *evidence-backed* reasoning node before lowering; it is never an optimization and never an assumption.
2. Substitution requires: planner eligibility, exact type identity, effect-subset, sufficient/fresh evidence, and policy permission; failing any gate leaves the reasoning untouched (safe default).
3. Discovery (Experience engine, Book 10) synthesizes/registers determinized Capabilities; the compiler only substitutes and re-verifies.
4. Substituted modules are re-verified and semantics-checked; the transform is effect-monotone and type-preserving.
5. Determinization is reversible: retired Capabilities cause future compilations to fall back to the original reasoning; no stale deterministic answer is ever executed.
6. The mechanism drives the system toward determinism over time while confining non-determinism to genuinely novel reasoning.
