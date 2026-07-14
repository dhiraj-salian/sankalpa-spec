# Book 08 · Chapter 04 — Planner Isolation and Secrets

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P7, P8. Companion to Book 11 (Security), Book 06 §Ch06 (secret materialization).*

> A model-driven planner is the highest-risk *leak* surface in the entire platform: its reasoning context is exactly where a secret, if present, would be most likely to escape (into a prompt, a log, a cached embedding, the model provider). This chapter specifies the two disciplines that neutralize that risk: **planners never receive secrets** (P7) and **planners hold no execution authority** (P8). Together with output-always-verified (§Ch01 §3), they make it safe to put a non-deterministic model at the top of a deterministic platform.

## 1. The threat this chapter closes

The founding security posture (SECURITY.md, Book 01 §04 P7) identifies the single most consequential leak surface in an AI system: **reasoning context**. Anything placed into a planner's context can end up in a prompt sent to a model provider, in a debug log, in a vector store, or in the plan itself — all durable, high-fanout, hard-to-scrub places. Therefore the design does not try to *carefully handle* secrets in planning; it ensures **there are no secrets in planning to handle**.

## 2. Planners never receive secret values (P7)

This is absolute and structural:

- The planner interface's `context` (§Ch03 §2) is **secret-free**; there is **no** interface method that returns a secret value (§Ch03 §6). A planner cannot request one because the contract cannot express the request.
- Knowledge supplied to planning is **non-secret and provenance-tagged** (Book 09 §06); the Knowledge Manager MUST NOT surface secret material into planning context.
- Clarification (Ch 02 §4) MUST NOT elicit secrets into the planner; a human is never asked to type a credential into a planning conversation. Credentials are acquired only through the Secret Broker's secure one-time channels (Book 11 §05).
- The planner's **output** may reference secrets only as opaque `SecretRef` values inside capability invocations (Book 04 §Ch05 §2.2); a `SecretRef` MUST NOT flow into a `Reasoning` node (Book 04 §Ch03 §3.2). Verification enforces this (Book 04 §Ch08 §2.4) — a plan that leaks a `SecretRef` into reasoning fails and never executes.

The result: from the platform's perspective, a planner is a component that has *never seen* a secret and *cannot*. Even a fully compromised or malicious planner has no secret to leak.

## 3. Planners hold no execution authority (P8)

A planner *plans*; it does not *act*. The capability model enforces this cleanly:

- At `init`, a planner receives **least-privilege planning grants** (§Ch03 §5): capability **descriptors** (signatures and declared effects) to *reason about what is possible* — **not** grants to *invoke* those capabilities. Reasoning about a capability and being authorized to execute it are different authorities (Book 03 §Ch06), and the planner holds only the former.
- A planner therefore **cannot cause any effect**. It cannot call the network, touch the filesystem, materialize a secret, or invoke an effectful capability. Its sole product is IR. Every real effect happens downstream, at execution, under grants held by the *runtime* (Book 06), never the planner.
- Because it holds no execution authority and no secrets, a planner compromise is contained to *bad IR* — which verification (Book 04 §Ch08) and policy (Book 05 §Ch04) catch — not to unauthorized actions or leaked credentials.

## 4. Isolation and containment

Layered on the two disciplines above:

- **Sandboxed execution** (Book 11 §10): the planner runs in an isolation boundary with resource limits from `init`; it cannot exceed them, reach other tenants' data, or contact other components except through the Kernel-mediated interface (P4).
- **Network egress control.** A model-driven planner typically calls an external model provider. That egress is itself a *granted, policy-governed capability* (P8/P9), scoped and audited — not ambient. A workspace MAY require an on-prem model or forbid external egress; the planner honors this because it can only reach the model through a grant.
- **Fault containment** (Book 03 §Ch13): a crashing/hanging planner is contained; its Intent surfaces a failure condition; the Kernel and other planners are unaffected.

## 5. The three-part safety argument

The safety of putting an untrusted, non-deterministic model at the top of the system rests on three independent properties, each specified elsewhere and *combined* here:

1. **Output always verified** (§Ch01 §3, Book 04 §Ch08) — bad IR cannot reach execution.
2. **No secrets in planning** (§2) — nothing to leak from reasoning context.
3. **No execution authority in planning** (§3) — nothing effectful the planner can do directly.

Each alone is insufficient; together they are decisive. A malicious planner can, at worst, produce IR that fails verification/policy, wastes some planning budget, and leaks *its own inputs* (which contain no secrets). It cannot execute an unauthorized effect, cannot leak a credential, and cannot corrupt a downstream deterministic execution. This is why Sankalpa can embrace the best model-driven planning of any era without betting its security on the model's good behavior.

## 6. Auditing planning

- Planning activity is observable (P10): Goal derivation, clarifications, plan emission, and verification outcomes emit secret-free Events (Book 03 §Ch03) and are recorded for Experience (Book 10). Recurring verification/policy failures from a planner are a signal of a systematic planning weakness (Book 05 §Ch07 §6).
- The planner's model egress is audited as a capability invocation (P10, Book 03 §Ch06 §4) — *that* it called a model, under which grant, with what non-secret context — without recording any secret (there are none) and per policy on retaining prompts.

## 7. Invariants (normative summary)

1. Planners never receive secret values: context is secret-free, no interface method returns a secret, Knowledge/clarification never surface secrets, and output carries only `SecretRef`s that verification bars from reasoning nodes (P7).
2. Planners hold no execution authority: they receive capability *descriptors* to reason with, not grants to invoke; they can cause no effect; all effects occur downstream under runtime grants (P8).
3. A planner compromise is contained to un-verifiable/forbidden IR — caught by verification and policy — never to unauthorized actions or leaked secrets.
4. Planners are sandboxed and resource-limited; model egress is a granted, policy-governed, audited capability, never ambient.
5. Safety rests on three combined properties — output-always-verified, no-secrets-in-planning, no-execution-authority — which together make an untrusted, non-deterministic planner safe.
6. Planning is observable and audited by reference; recurring failures feed Experience.
