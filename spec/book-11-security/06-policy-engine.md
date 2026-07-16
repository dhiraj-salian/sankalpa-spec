# Book 11 · Chapter 06 — The Policy Engine

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0001; realizes principle P9 (and P8). Companion to Book 05 §Ch04 (policy-validation pass), Book 14 §06 (runtime checkpoint).*

> Principle **P9 — everything is governed by policy** requires that governance be *mechanical and pre-emptive*, not advisory and after-the-fact. The Policy Engine is the component that makes policy machine-checkable and enforces it at defined checkpoints. This chapter specifies the policy model, the three enforcement points, conflict resolution, and the fail-closed guarantee.

## 1. Policy is code, checked before action

Policy in Sankalpa is **machine-checkable rules over structured facts** — never prose to be manually interpreted. Because the platform produces analyzable artifacts at every stage (effect-annotated IR, typed Resources, capability grants), policy can be *evaluated automatically* against them *before* an action occurs. This is the difference between governance that *constrains* and governance that merely *documents*.

Policies are **`Policy` Resources** (Book 02 §Ch07): versioned (P6), scoped (to a workspace, kind, or effect class), and shippable as Packages (Book 12). Because they are versioned Resources, policy changes are auditable and participate in cache keys (Book 05 §Ch08 §3) so nothing runs against a stale validation.

## 2. Three enforcement points (defense in depth)

Policy is enforced at three checkpoints, each seeing what only it can:

1. **Control-plane checkpoint** — at the Kernel API request pipeline (Book 03 §Ch02 §3 step 4). Governs *operations on Resources*: who may create/update/delete/invoke what. Sees the request and the caller's capabilities.
2. **Compile-time checkpoint** — the policy-validation pass on High IR (Book 05 §Ch04). Governs *what a plan would do*: its declared effects, capability/secret use, and cost, on the human-meaningful artifact, **before** lowering. Denies or annotates-with-guards.
3. **Runtime checkpoint** — immediately before execution (Book 14 §06). Re-evaluates policy against **live** conditions only knowable then: current budget, present Approval decisions (Ch 07), runtime health, time-of-day. A plan that passed compile-time policy may still be stopped here if conditions changed.

The three are deliberately redundant (Ch 01 §6): static analysis (2) cannot see live state; the control plane (1) cannot see plan internals; the runtime (3) is the last gate before an irreversible effect. Together they close the gaps each alone would leave.

## 3. What policy reasons over

Policy rules evaluate structured, **non-secret** facts:
- **Effects** (Book 04 §Ch06) — the primary subject: forbid/require-gate/bound `Network(write)`, `SecretUse(class)`, `Cost`, `StateWrite`, etc.
- **Capabilities & entitlements** (Ch 03) — which capabilities a subject may exercise in a context.
- **Secret classes by reference** (Ch 04) — e.g. "materializing a `payments` secret requires Approval" — reasoned over the *reference/class*, never the value (P7).
- **Types & data classifications** — e.g. "no Capability tagged `pii-export` for workspace W."
- **Cost & quota** — budgets and rate limits.
- **Context** — workspace, submitter role, time, approval state.
- **Grant re-authorization thresholds** (Book 12 §Ch05 §3.1) — when a Package upgrade may carry grants forward versus require fresh consent: e.g. "always re-consent `payments`", "re-consent on MAJOR", "auto-carry same-key patch". Absent or undecided policy is **fail-safe**: re-authorize on any signing-identity change and for any sensitive-class capability.

Policy MUST NOT require secret values to evaluate (P7); it governs secret *use* by reference and class.

## 4. Outcomes and conflict resolution

Per applicable policy, evaluation yields **allow**, **allow-with-guard** (insert an approval/redaction/budget-check — Book 05 §Ch04 §3), or **deny** (with an explainable diagnostic — Book 05 §Ch07).

Conflicts among policies are resolved by defined **precedence** (e.g. a workspace deny overrides a permissive default; explicit deny beats allow). Crucially:
- An **unresolved conflict is a deny** (fail-closed, Ch 01 §6). Governance never defaults to permissive when uncertain.
- A policy that **cannot be evaluated** (missing data, engine error) causes the guarded action to be **refused**, not admitted (Book 03 §Ch13 §1). "We couldn't check" is treated as "not allowed."

## 5. Explainability (governance must teach)

Every deny or required guard produces a **structured, secret-free, deterministic** diagnostic (Book 05 §Ch07): the policy id/version, the offending fact (effect/node/capability), a human explanation, and — where possible — how to comply. Rationale: opaque governance breeds workarounds and mistrust; explainable governance lets planners adapt (Book 05 §Ch04 §4) and humans understand the guardrail. A rejection that only says "no" is a governance defect.

## 6. Relationship to capabilities (P8 + P9)

Capabilities and policy are complementary and *both* required (Ch 03 §6):
- **Capabilities** define what a subject *may ever* do (static authority).
- **Policy** defines what is *allowed here and now* (contextual governance).
- An action proceeds only if **both** permit it; **either** can forbid it. A subject may hold a capability yet be denied by policy in this context, and vice-versa. This dual gate is why, e.g., a runtime that *can* materialize a secret (capability) is still stopped if policy requires an absent Approval (Ch 07).

## 7. Invariants (normative summary)

1. Policy is machine-checkable rules over structured facts, enforced pre-emptively; policies are versioned, scoped `Policy` Resources.
2. Policy is enforced at three checkpoints — control-plane, compile-time (pre-lowering), and runtime (pre-execution) — each seeing what only it can (defense in depth).
3. Policy reasons over effects, capabilities, secret classes (by reference), types, cost, and context — never over secret values (P7).
4. Outcomes are allow / allow-with-guard / deny; unresolved conflicts and un-evaluable policy both deny (fail-closed).
5. Denials and guards produce structured, secret-free, deterministic, explainable diagnostics.
6. Capabilities (may-ever) and policy (allowed-here-now) both gate every action; either can forbid it.
