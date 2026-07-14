# Book 03 · Chapter 06 — Capability Manager

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P8, P9, P13. Companion to Book 11 §03 (capability-based security), Book 02 §Ch07 (Capability).*

> The Capability Manager is where principle **P8 — no ambient authority** is made real. It is the registry and gatekeeper for Capabilities: the typed, reusable units of behavior that are the *only* way to cause an effect through the Kernel (§Ch02 §2.2). Nothing in Sankalpa "just has permission" to do something; it holds a capability, or it cannot.

## 1. Two meanings of "capability", unified

The word carries two senses that this design deliberately unifies:
- **Capability-as-authority** (security): an unforgeable reference that *grants the right* to perform a specific action (the object-capability model — KeyKOS, E).
- **Capability-as-behavior** (function): a typed, reusable, often deterministic unit of work a plan can invoke (Book 02 §Ch07 §4), and the target of determinization (P13).

In Sankalpa these are the same object: to *invoke* a behavior you must *hold the authority* to invoke it. A `Capability` Resource describes the behavior (signature, effects, implementation) and the *grant* of it to a subject conveys the authority. This unification is why `Invoke` (§Ch02) can be both the universal action verb and the universal permission check.

## 2. Responsibilities

1. **Registry** — maintain the set of available Capabilities as Resources (typed signature, declared effects, implementation reference, provenance).
2. **Granting** — issue grants of a Capability to a subject (a session, controller, plugin, or another capability holder), producing an unforgeable reference.
3. **Attenuation** — derive a *weaker* capability from one held (narrower inputs, tighter effect scope, reduced quota) so authority can be delegated least-privilege.
4. **Revocation** — invalidate a grant so future invocations fail, promptly and auditable.
5. **Invocation gating** — on every `Invoke` (§Ch02), verify the caller holds a valid grant whose signature/effects cover the request, then dispatch.
6. **Provenance** — record whether a Capability was hand-authored, packaged (Book 12), or **determinized** from repeated reasoning (P13, Book 10 §06).

## 3. Grants are unforgeable and least-privilege

- A grant is an **unforgeable reference**: it cannot be guessed, constructed, or escalated by a holder. Holding a grant to Capability A conveys **no** authority over Capability B (P8 — no ambient authority, no transitive escalation).
- **Attenuation, never amplification.** A holder may derive a weaker capability (attenuate) but **MUST NOT** derive a stronger one. The Capability Manager enforces that any derived grant's signature and effect set are a *subset* of the parent's. This is what makes delegation safe: a component can hand a plugin exactly the sliver of authority it needs and no more.
- **Effect coverage.** A grant authorizes a bounded effect set (Book 04 §Ch06). An `Invoke` whose actual/declared effects exceed the grant's covered effects **MUST** be denied — the effect system (Book 04) and the capability system meet here.

## 4. Invocation flow

On `Invoke(capabilityRef, inputs)` through the Kernel API:

1. **Grant check** — the caller holds a valid, unrevoked grant for `capabilityRef`.
2. **Type check** — `inputs` type-check against the Capability's signature (Book 04 §Ch05); outputs typed.
3. **Effect/policy check** — the invocation's effects are within the grant's coverage and pass applicable policy (P9, Book 11 §06).
4. **Secret gating** — any `SecretRef` input is resolvable only if the grant (or a companion grant) authorizes it, and only the Secret Broker materializes it at execution (P7, §Ch11).
5. **Dispatch** — the Capability's implementation runs (in-Kernel for core capabilities; via a runtime/plugin for others), typed result or typed error returned.
6. **Audit** — a secret-free Event records who invoked what, with which grant (P5, P10).

Every effectful thing that happens in Sankalpa passes this flow — there is no other route to an effect through the Kernel.

## 5. Capabilities, plugins, and the boundary

- A **plugin** (planner, runtime, backend, provider) receives, at load time, a specific, attenuated set of grants (§Ch09) — never ambient access. A runtime can materialize the secrets its execution needs *only* because it was granted the capability to, and *only* at execution (Book 06 §06).
- The Capability Manager is **core** (not a plugin) precisely because it is the authority gatekeeper; a compromised plugin cannot rewrite it (§Ch01 §4).

## 6. Determinization ties in (P13)

When the Experience/determinization engine (Book 10 §06, Book 05 §06) folds a recurring reasoning into a deterministic Capability, it registers that Capability here with provenance `determinized`, a typed signature, and an effect set that is a subset of the original reasoning's (Book 04 §Ch06 §7). Thereafter plans `Invoke` it like any other Capability — the platform's learned determinism becomes ordinary, governed, capability-gated behavior. The Capability Manager is thus the place where "the system got better at something" is made concrete and safe.

## 7. Revocation and lifecycle

- Grants and Capabilities are Resources (Book 02) with lifecycles: revoking a grant transitions it to a terminal state and emits an Event; in-flight invocations under a just-revoked grant complete or are cancelled per the Capability's policy, but no *new* invocation succeeds.
- Revocation MUST be **prompt and complete**: after revocation, the Capability Manager MUST deny subsequent invocations even under caching. This is a security guarantee (Book 11), not a best effort.

## 8. Invariants (normative summary)

1. Every effect through the Kernel is an `Invoke` of a Capability the caller holds a valid grant for; there is no ambient authority (P8).
2. Grants are unforgeable; holding one conveys no authority over any other; there is no transitive escalation.
3. Derivation only attenuates (subset signature and effects); amplification is impossible.
4. Invocation checks grant, type, effect/policy, and secret authorization before dispatch, and audits after (P5, P7, P9, P10).
5. Plugins receive least-privilege attenuated grants at load; the Capability Manager is core and unrewritable by plugins.
6. Determinized behaviors are registered as provenance-tagged Capabilities with subset effects; revocation is prompt and complete.
