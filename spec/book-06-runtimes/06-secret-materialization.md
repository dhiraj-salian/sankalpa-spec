# Book 06 · Chapter 06 — Secret Materialization

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P7, P8. Companion to Book 11 §04 (Secret Broker), Book 04 §Ch04 §5 (binding sites).*

> This chapter specifies the **one and only** point in Sankalpa where a secret becomes a value: at execution, inside a runtime, materialized by the Secret Broker from a reference, under a capability grant. Everywhere else — planning, IR, compilation, storage, logs, Events — secrets exist solely as references (P7). Getting this chapter right is what makes principle P7 mechanically true rather than aspirational.

## 1. The single materialization point

Across the whole intent→execution arc, a secret **value** appears in exactly one place: within a runtime's isolated execution context, for the duration of the effect that needs it. This is the culmination of the reference-only discipline carried through every prior layer:

```
Secret (ARM: reference metadata only, Book 02 §Ch06 §4)
   │  planner sees a SecretRef value, never a value (Book 08 §04)
   │  compiler lowers SecretRef → binding site, resolved at Execution (Book 04 §Ch04 §5)
   │  backend maps binding site → runtime's secret-injection slot, still a reference (Book 05 §Ch05 §6)
   ▼
RUNTIME, at execution: presents binding token → Secret Broker → VALUE injected into execution context
   │  used for the effect; never logged, emitted, or persisted (§5)
   ▼
execution ends → materialized value discarded (§5)
```

There is no earlier materialization and no other path. The runtime interface (Ch 02 §6) offers **no** method to read a secret ahead of time or in bulk — the capability to leak simply does not exist in the contract.

## 2. The materialization protocol

At the moment an instruction with a `SecretUse` effect (Book 04 §Ch06 §6) is about to execute, the runtime performs:

1. **Present the binding token.** From `executionCtx` (Ch 02 §2), the runtime presents the binding site's reference token to the **Secret Broker** (Book 11 §04) — a reference, not a request for "all secrets."
2. **Capability check (P8).** The Broker verifies the runtime holds the capability authorizing resolution of *this* secret (Book 03 §Ch06). Holding the reference is necessary but **not sufficient**; the authorizing grant (`grantedBy`, Book 04 §Ch04 §5) must also be present. No ambient authority.
3. **Policy check (P9).** The Broker confirms any policy conditions on this materialization are satisfied — e.g. a required Approval decision (Book 11 §07) is present, or a runtime-checkpoint condition holds (Book 14 §06).
4. **Scoped injection.** On success, the Broker injects the value **directly into the runtime's execution context** for this instruction — not returned up any API, not placed in IR, not in an Event. The injection is scoped to the smallest unit that needs it.
5. **Use and discard.** The runtime uses the value for the effect and discards it when the instruction (or execution) completes (§5).

Any failure at steps 2–3 is **fail-closed** (Book 03 §Ch13 §1): the secret is not materialized, the instruction fails, and the execution reaches an explained terminal state — it never proceeds with a missing or unauthorized secret.

## 3. What the runtime may and may not do

**May:** materialize a secret it is authorized for, at the instruction that declared `SecretUse`, and use it to perform that instruction's effect.

**MUST NOT:**
- Materialize a secret outside an execution, ahead of need, or in bulk (the interface cannot express this, Ch 02 §6).
- Log, emit as an Event, trace, or otherwise surface a materialized value (P7; Book 03 §Ch03 §8, §Ch12 §3, Book 14 §04).
- Persist a materialized value beyond the execution's need, or cache it across executions.
- Return a materialized value up the Kernel API or into any Resource, IR, or artifact.
- Pass a materialized value into a `Reasoning`/model call (P7; Book 04 §Ch03 §3.2, Book 08 §04) — reasoning context is secret-free by construction.

A runtime that violates any of these has a security defect (Book 11), treated with the same gravity as a leak in IR.

## 4. Why the Broker, and why separate storage

- The Secret Broker is the **sole custodian** of secret values, backed by a **protected store separate from the ARM store** (Book 02 §Ch08 §6, Book 03 §Ch11 §1.3). A full compromise of the Resource store exposes no secret — because none is there.
- Centralizing materialization in one audited component means there is exactly one place to reason about, harden, and audit secret exposure (Book 11 §04) — rather than N runtimes each inventing their own secret handling.
- Every materialization is **audited** (P10): the Broker records that runtime R materialized secret S for execution E under grant G — by reference, never the value — so secret *use* is fully traceable without secret *exposure*.

## 5. Ephemerality

Materialized values are **ephemeral** by contract:
- Held only for the instruction/execution that needs them, in the isolated runtime context (Book 11 §10).
- Never written to durable storage, logs, caches, or Events.
- Discarded when no longer needed; the runtime MUST NOT retain them across executions.

This bounds the exposure window to the minimum: a secret is a value only briefly, in one isolated place, for one purpose.

## 6. Credential acquisition (forward pointer)

How secrets *enter* the Broker in the first place — secure one-time HTTPS pages, OAuth-preferred acquisition, rotation — is specified in Book 11 §05. This chapter concerns only the *use* side: turning a stored, referenced secret into a transient value at execution. The two together (acquisition into the Broker; materialization from the Broker) complete the secret lifecycle, and neither ever puts a value into IR, planning, logs, or ARM.

## 7. Invariants (normative summary)

1. A secret becomes a value at exactly one point: inside a runtime's isolated execution context, at the instruction that declared `SecretUse`, materialized by the Secret Broker from a reference.
2. Materialization requires both the reference *and* the authorizing capability (P8) and satisfies policy/approval conditions (P9); failure is fail-closed.
3. The runtime interface offers no way to read secrets except this by-reference, at-execution materialization; ahead-of-time or bulk reads are inexpressible.
4. Materialized values are never logged, emitted, traced, persisted, cached across executions, returned up any API, or passed into reasoning (P7).
5. The Broker is the sole custodian with storage separate from ARM; every materialization is audited by reference, never by value.
6. Materialized values are ephemeral — held only as long as needed, in isolation, then discarded.
