# Book 08 · Chapter 07 — Conformance Suite

*Nature: **Normative**. · Reflects: RFC-0001, AEP-0001; realizes principles P1, P7, P8, P11. Companion to Book 04 §Ch08 (verification), Book 06 §Ch07 (runtime conformance).*

> A planner's promise — "I will turn Goals into valid, safe, verifiable High IR, and confine my non-determinism" — must be *provable*. The planner conformance suite is the executable corpus a planner MUST pass to claim conformance at a stability level. It is what makes planners safely interchangeable (P11) and what lets an untrusted, model-driven planner be trusted *as a component* even though its reasoning is not.

## 1. Purpose and what it can (and cannot) test

A planner is non-deterministic (§Ch01), so the suite **cannot** assert "for this Intent, produce exactly this IR." Instead it asserts **properties that must hold for every plan a planner produces**, regardless of the specific plan:

- Every emitted plan is **valid, verifiable** High IR (§Ch03 §3).
- Every emitted plan is **secret-safe** (P7) and **runtime-agnostic** (IR-P7).
- Non-determinism is **properly boxed** in typed `Reasoning` nodes (§Ch05).
- The planner **degrades honestly** under uncertainty (clarifies rather than guesses dangerously).
- The planner holds **no execution authority** and causes no effects (P8, §Ch04).

Conformance is always **to a specific version** of AEP-0001 and the High IR (Book 04 §Ch09) — never "latest."

## 2. Structure of the suite

The suite is a corpus of `(Intent/Goals + context)` inputs plus **property assertions** over whatever plan the planner returns. Because outputs vary, assertions are *universal properties*, not golden outputs.

### 2.1 Output validity (the core property)
- **Always verifiable.** For every input, `plan` returns High IR that passes verification (Book 04 §Ch08) — well-formed, fully typed, fully effect-annotated — *or* a `ClarificationRequest`/`Diagnostic`. It MUST NEVER return malformed IR or non-IR executable content.
- **Effect completeness.** Every node in every emitted plan declares its effects (deny-by-default holds); the suite injects capability descriptors with known effects and asserts the plan's effect annotations match.
- **Type soundness.** Emitted plans type-check against the provided capability signatures.

### 2.2 Secret safety (P7) — required at every level
- **No secret ingress.** Given context seeded with marked "canary" values in *non-secret* fields, and given that the interface exposes no secret method, the planner's outputs and any observable planner behavior (logs, model-egress records the harness can see) contain **no** secret material and no fabricated secrets.
- **`SecretRef` discipline.** When a plan must use a secret, it emits a `SecretRef` inside a capability invocation and **never** routes a `SecretRef` into a `Reasoning` node — verification catches violations, and the suite asserts the planner does not produce such plans.

### 2.3 Non-determinism boundary (§Ch05)
- **Reasoning localization.** Non-deterministic decisions appear only as typed `Reasoning` nodes; no other construct is non-deterministic (the emitted plan passes the determinism checks of Book 04 §Ch08 §2.5).
- **Honest determinize marking.** The planner marks `determinize` eligibility sensibly (stable-if-repeated reasoning as eligible); the suite checks the *presence and typing* of the marking, not the model's answers.
- **Stability characterization.** A planner that `describe`s itself as deterministic MUST produce identical IR for identical inputs across runs; a model-driven planner is not held to output-identity, only to the universal properties.

### 2.4 Uncertainty handling (Ch 02)
- **Clarifies, doesn't dangerously guess.** For inputs with high-consequence ambiguity, the planner returns a `ClarificationRequest` or routes through Approval — it does not silently emit a plan with irreversible effects on a guessed interpretation.
- **Records assumptions.** For low-consequence ambiguity, assumptions are recorded on the Goals/IR (Ch 02 §4).
- **Grounded Goals.** Derived Goals carry success criteria and are grounded in provided capabilities (Ch 02 §3); a Goal without success criteria fails conformance.

### 2.5 Isolation & authority (P8, §Ch04)
- **No effects.** Across all cases, the planner performs **no** external effect: it holds capability *descriptors*, not execution grants, so any attempt to invoke an effectful capability is denied/contained.
- **Bounded resources.** The planner respects the resource limits from `init`; a runaway planning case is contained (Book 03 §Ch13).
- **Model egress is a grant.** If the planner calls a model, it does so only through its granted egress capability, honoring policy (e.g. on-prem-only) — the suite verifies it cannot exceed the grant.

### 2.6 Interface & versioning (AEP-0001)
- describe/deriveGoals/plan/init/health/shutdown behave per §Ch03; version negotiation refuses incompatible IR/interface versions rather than degrading.

## 3. Stability levels and conformance

Graded to the AEP-0001 level a planner claims (the AEP process):
- **Experimental:** output-validity + **secret-safety** + non-determinism-boundary (secret safety is never optional, even here).
- **Beta:** adds full uncertainty-handling and isolation/authority groups, with real adopters.
- **Stable:** the whole suite plus a track record; breaking changes then require a major AEP bump + migration (Book 04 §Ch09).

## 4. Why property-testing a non-deterministic component works

It might seem impossible to conformance-test something non-deterministic. The resolution is the same idea that underpins the whole platform: **we do not certify the reasoning; we certify the output contract.** The planner may reason in wildly different ways run to run, but *every* output must satisfy the universal properties above — and the strongest of them (validity, secret-safety) are *also* enforced at runtime by verification (Book 04 §Ch08) and policy (Book 05 §Ch04). Conformance is the planner's *pre-flight* guarantee; verification is the *always-on* backstop. A planner that slips a bad plan past conformance still cannot execute it — the backstop catches it. Conformance raises quality; verification guarantees safety.

## 5. Two conforming planners are interchangeable

The defining guarantee: **any conforming planner (at a version) can be substituted for another** — because both are guaranteed to emit only valid, secret-safe, properly-bounded High IR from Goals. A deployment can swap LangGraph for a rule engine, or route Goal classes to different planners, trusting the suite (P11). This is what makes "not tied to any LLM or framework" (Book 01 §01) an operational reality rather than a slogan.

## 6. Invariants (normative summary)

1. A planner MUST pass the AEP-0001/IR-version conformance suite it claims; conformance is version-specific and property-based, not golden-output-based.
2. The suite proves: every output is verifiable High IR (or a clarification/diagnostic), secret-safe, runtime-agnostic, with non-determinism boxed in typed `Reasoning` nodes.
3. Secret-safety and output-validity are required at every stability level, including Experimental.
4. The suite verifies honest uncertainty handling (clarify vs. dangerous guess), grounded Goals with success criteria, no execution authority, and bounded/policy-governed model egress.
5. Conformance is a pre-flight guarantee; verification and policy are the always-on backstops — a plan that slips past conformance still cannot execute if unverifiable or forbidden.
6. Any two conforming planners are interchangeable, making the platform genuinely planner- and LLM-agnostic (P11).
