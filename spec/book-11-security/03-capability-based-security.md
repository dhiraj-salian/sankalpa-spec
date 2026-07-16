# Book 11 · Chapter 03 — Capability-Based Security

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0008 (grants bind to verified identity), RFC-0005 (revocation pierces the secret memoization); realizes principle P8 (and P9). Companion to Book 03 §Ch06 (Capability Manager).*

> Principle **P8 — no ambient authority** is the backbone of Sankalpa's authorization model. This chapter specifies *capability-based security*: authority as unforgeable references to specific actions, granted explicitly, attenuated for delegation, and revocable. It is the model that makes untrusted plugins safe to run and unauthorized effects impossible. The *component* that implements it is the Capability Manager (Book 03 §Ch06); this chapter specifies the *security model* it enforces.

## 1. Ambient authority is the thing we reject

Most systems use **ambient authority**: a subject's identity implicitly confers a broad set of permissions (a process runs "as a user" and can do anything that user can). Ambient authority is the root cause of confused-deputy attacks, privilege creep, and the blast radius of a single compromise. Sankalpa rejects it outright (P8): **a subject can do only what its held capabilities explicitly permit — nothing follows from mere identity.**

## 2. What a capability is

A **capability** is an **unforgeable reference that both designates a specific action and conveys the authority to perform it** (the object-capability model — KeyKOS, E). In Sankalpa (Book 03 §Ch06 §1) the two senses are unified: a `Capability` describes a typed behavior; a *grant* of it to a subject conveys the authority to invoke it.

Properties a capability MUST have:
- **Unforgeable.** It cannot be guessed, constructed, or forged. Possession is the *only* way to hold authority; there is no "ask for permission by name" path that ambient identity could satisfy.
- **Specific.** It authorizes a *particular* action with a bounded signature and effect set (Book 04 §Ch05–06) — not a category of power.
- **Transferable only by explicit delegation** (§4) — and only attenuating.
- **Revocable** (§5).
- **Bound to the grantee identity it was authorized against.** A grant records the **authorized-against identity** — for a Package (Book 12 §Ch05 §3), its version, publisher identity, and signing-key/artifact identity (Book 12 §Ch04). Authority is conveyed to the *verified code that was authorized*, not to a name in perpetuity, so substituting the code re-opens the authorization question (Book 12 §Ch05 §3.1). Without this, "explicit grant" would be defeated by the ordinary upgrade path: a compromised or newly-transferred publisher could ship different code under the same name and inherit sensitive authority no one reviewed.

## 3. No authority without a grant

The core rule, enforced at every `Invoke` (Book 03 §Ch02 §2.2, §Ch06 §4):
- To cause any effect, a subject MUST hold a valid grant for the exact Capability, whose signature and effect set cover the request.
- Holding a grant for Capability A conveys **zero** authority over Capability B. There is no transitive escalation, no "admin" capability that implies others, no inheritance of authority from identity or membership.
- A subject with **no** grants can do **nothing** effectful — the default is powerlessness, and power is added explicitly, one capability at a time.

This is what bounds every adversary in Ch 01 §3: a compromised plugin, a malicious Package's code, a prompt-injected planner — each can do only what it was *explicitly granted*, which is the minimum its function requires.

## 4. Attenuation: safe delegation

Authority is delegated by **attenuation** — deriving a *weaker* capability from one held (Book 03 §Ch06 §3):
- A holder MAY derive a capability with **narrower** inputs, **tighter** effect scope, reduced quota, or a shorter validity — a subset of its own authority.
- A holder MUST NOT derive a **stronger** capability; amplification is impossible and the Capability Manager enforces subset-only derivation. This is the mathematical guarantee behind least-privilege delegation: you can only ever hand out *less* than you hold.
- **Example:** the Runtime Manager holds a capability to materialize any secret an execution declares; it attenuates this to a grant that lets a *specific* runtime materialize *only the specific secrets of the execution it is running* (Book 06 §Ch06). The runtime cannot widen it.

Attenuation is what lets the trusted core safely delegate slivers of authority to untrusted plugins: the plugin gets exactly what it needs, provably no more.

## 5. Revocation

A grant is a Resource (Book 02) with a lifecycle; revocation (Book 03 §Ch06 §7):
- Transitions the grant to a terminal state and emits an audit Event (Ch 09).
- MUST be **prompt and complete**: after revocation, subsequent invocations under it are denied, even past any caching. A just-revoked grant cannot be used for a new effect.
- In-flight invocations complete or are cancelled per the Capability's policy; no *new* invocation succeeds.
- The "even past any caching" rule has one named cache to speak to: the **per-Execution secret memoization** (Book 06 §Ch06 §2.1). It caches a materialized *value* for stability under rotation, never an *authorization* — every `SecretUse` re-runs the capability and policy checks, so a revoked grant is denied mid-Execution even where a value is already memoized. A memoization that elided those checks would silently defeat prompt revocation, which is why the split is normative there.

Revocation is what makes capabilities *governable over time*: authority can be withdrawn (a plugin retired, a determinized Capability found faulty — Book 05 §Ch06 §4) without restarting the world.

## 6. Capabilities meet the effect system (P9)

Authorization (capabilities) and governance (policy) meet at the effect system (Book 04 §Ch06):
- A grant authorizes a bounded **effect set**; an invocation whose declared effects exceed the grant's coverage is denied (Book 03 §Ch06 §3). So a capability to "read the CRM" cannot be stretched to "write the CRM" — the effects differ.
- Policy (Ch 06) further constrains *which* granted capabilities may be exercised *in a given context* (workspace, approval state). Capabilities say *what a subject may ever do*; policy says *what is allowed here and now*. Both must permit an action; either can forbid it (fail-closed, Ch 01 §6).

## 7. Why this makes the platform safe

Capability security is the linchpin that reconciles Sankalpa's ambitions with its threat model (Ch 01):
- **Untrusted plugins become safe** because they hold only attenuated, least-privilege grants and no ambient power (Ch 02 §3).
- **Confused-deputy and escalation attacks are structurally impossible** — there is no ambient authority to confuse and no transitivity to escalate.
- **Blast radius is minimized** — a compromise reaches only the specific, attenuated authority the compromised component held.
- **Secrets are protected** because materialization is itself capability-gated (Ch 04), so even a component holding a secret reference cannot resolve it without the separate authority.

## 8. Invariants (normative summary)

1. Authority is capability-based, never ambient: identity/membership alone confers no permission (P8).
2. A capability is an unforgeable, specific, revocable reference that both designates an action and conveys authority to perform it, and is bound to the verified grantee identity it was authorized against — substituting the grantee's code re-opens the authorization question (Book 12 §Ch05 §3.1).
3. Every effect requires a held grant covering the request's signature and effects; holding one capability conveys no authority over any other; there is no transitive escalation.
4. Delegation is by attenuation only — derived capabilities are strict subsets; amplification is impossible.
5. Revocation is prompt and complete; no new invocation succeeds under a revoked grant — including past the per-Execution secret memoization, which caches a value, never an authorization (Book 06 §Ch06 §2.1).
6. Capabilities (what a subject may ever do) and policy (what is allowed here and now) both gate every action; either can forbid it, fail-closed.
