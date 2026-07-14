# Book 08 · Chapter 05 — The Non-Determinism Boundary

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P13 and IR-P1. Companion to Book 04 §Ch03 §3.2 (Reasoning nodes), Book 05 §Ch06 (determinization).*

> Planning is non-deterministic; everything downstream is deterministic. This chapter specifies **exactly where** that boundary sits, how non-determinism is *represented* (as typed `Reasoning` nodes) so it does not contaminate the deterministic machine, and how the boundary is designed to *recede over time* through determinization (P13). This is the most conceptually important chapter in Book 08.

## 1. Two kinds of non-determinism, one boundary

There are two distinct non-deterministic activities in planning, and the boundary treats them differently:

1. **Plan-construction non-determinism.** *Producing the plan itself* is non-deterministic: the same Intent may yield different (but valid) High IR on different runs, because a model reasoned to build it. This non-determinism is **resolved at plan time** — once the planner emits verified High IR, the plan is fixed.
2. **Residual runtime reasoning.** Some decisions *cannot* be made at plan time because they depend on data seen only at execution ("classify this incoming ticket," "summarize whatever the query returns"). These are **deferred into the plan** as typed `Reasoning` nodes (Book 04 §Ch03 §3.2), to be resolved at execution.

The boundary rule: **plan-construction non-determinism ends when verified High IR is emitted; residual reasoning is confined to explicitly-typed `Reasoning` nodes and nowhere else.** Everything not inside a `Reasoning` node (or a declared `Time`/`Random` effect, Book 04 §Ch06 §4) is deterministic.

## 2. How non-determinism is contained, not leaked

The containment is structural, enforced by the type and effect systems (Book 04 §Ch05–06) and by verification (Book 04 §Ch08):

- A `Reasoning` node has a **typed output** and declares the **`Reason`** effect. Downstream nodes consume only its typed output — never the reasoning process, prompt, or model text. The non-determinism is boxed inside the node's boundary; its *output* is an ordinary typed value (IR-P1).
- A `Reasoning` node receives **no secret-derived context** (§Ch04, Book 04 §Ch03 §3.2) — verification rejects a `SecretRef` flowing into reasoning.
- **Nothing else** in the plan may be non-deterministic: a data node must be pure, a capability invocation must be a deterministic function of its inputs and declared effects (Book 04 §Ch06 §4). A planner cannot smuggle non-determinism into, say, a `Convert` or a capability call — verification enforces determinism constraints (Book 04 §Ch08 §2.5).

Thus a plan is *deterministic except at named points*, and those points are typed, effect-declared, and auditable. The deterministic machine (Books 05–06) can reason about, cache, replay, and govern the whole plan precisely because non-determinism is not diffuse — it is localized.

## 3. Why not resolve all reasoning at plan time?

One might ask: why leave `Reasoning` nodes at all — why not have the planner decide everything upfront? Because some decisions *genuinely depend on execution-time data* that does not exist at plan time. Forcing an early decision would mean either guessing (unsound) or fetching that data during planning (which would drag effects and secrets into the planner, violating §Ch04). Deferring the decision into a typed `Reasoning` node is the honest, safe representation: the plan says "at this point, reason over *this typed input* to produce *this typed output*," and the decision is made at execution with the real data — under governance, and captured for audit and replay (Book 06 §Ch03 §1).

## 4. The boundary recedes over time (P13)

The boundary is not fixed — it is designed to **move toward determinism** as the system learns (Book 01 §05):

- A `Reasoning` node marked `determinize: true` (Book 04 §Ch03 §3.2) is a candidate for **determinization** (Book 05 §Ch06): when Experience shows the same reasoning yields stable outputs for equivalent inputs, it is folded into a deterministic **Capability**, and future compilations substitute the Capability for the `Reasoning` node (Book 05 §Ch06 §2).
- After substitution, what was a non-deterministic point becomes a deterministic capability invocation — the boundary has receded by one node. Genuinely novel reasoning still flows through the model; solved reasoning does not.
- The planner participates by marking eligibility honestly: it flags reasoning it believes is *stable-if-repeated* as `determinize: true`, and volatile reasoning as `false`. This is a judgment the planner is well-placed to make and the determinization engine later validates against evidence (Book 05 §Ch06 §3).

So the planner is not just the home of non-determinism — it is the *source of the signals* that let the platform reduce non-determinism over time. Every plan it emits is both an answer now and a candidate lesson for later.

## 5. Determinism relative to the model

A subtle clarification: even *within* a single `Reasoning` node, "non-deterministic" is relative. If the underlying model is run at temperature zero on fixed inputs it may be effectively deterministic; if not, it may vary. The spec does **not** assume model determinism. Instead it treats a `Reasoning` node as *potentially* non-deterministic and captures its actual output in the Execution/Experience record (Book 06 §Ch03 §1, Book 10) so the run is auditable and replayable-with-record regardless. This is why the boundary is drawn at the *node*, not at the model: the platform's guarantees do not depend on how the model behaves inside the box.

## 6. Invariants (normative summary)

1. Plan-construction non-determinism ends when verified High IR is emitted; residual non-determinism is confined to typed `Reasoning` nodes (and declared `Time`/`Random` effects) and nowhere else.
2. A `Reasoning` node exposes only a typed output and declares the `Reason` effect; downstream consumes the output, never the reasoning; it receives no secret-derived context.
3. All non-reasoning constructs are deterministic; verification rejects non-determinism outside the sanctioned points (IR-P1).
4. Reasoning is deferred into the plan (not guessed at plan time or resolved by fetching data into the planner) precisely when it depends on execution-time data.
5. The boundary recedes over time: `determinize`-eligible reasoning is folded into deterministic Capabilities as Experience accrues (P13); the planner supplies the eligibility signals.
6. The spec assumes no model determinism; a `Reasoning` node's actual output is captured for audit/replay, so guarantees hold regardless of model behavior.
