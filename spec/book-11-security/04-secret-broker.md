# Book 11 · Chapter 04 — The Secret Broker

*Nature: **Normative**. · Reflects: SECURITY.md, RFC-0001; realizes principle P7 (and P8, P9, P10). Companion to Book 06 §Ch06 (materialization), Book 02 §Ch06 §4 (secret references).*

> The Secret Broker is the **sole custodian of secret values** in Sankalpa. It is the mechanism that makes P7 — *secrets never enter planner context, prompts, logs, memory, vector stores, or IR* — mechanically true. This chapter specifies the Broker: its separate protected store, the reference model, resolution, and the guarantees it upholds. It is the single most security-critical component in the platform.

## 1. The Broker's singular responsibility

The Broker exists so that exactly one component in the system ever holds secret *values*, and that component is small, hardened, audited, and separate from everything else. Every other component — planners, compilers, runtimes (except transiently, at execution), the ARM store, the Event Bus, logs — deals only in **references** (Book 02 §Ch06 §4). Centralizing custody means there is one place to reason about, protect, and audit secret exposure, rather than N components each inventing secret handling (Ch 01 §6, defense in depth notwithstanding — this is the depth's anchor).

## 2. Separate protected store (P7's physical anchor)

The Broker's secret store is **physically and logically separate from the ARM store** (Book 02 §Ch08 §6, Book 03 §Ch11 §1.3):
- Secret values live **only** here, under access controls distinct from and stricter than the Resource store's.
- A **full compromise of the ARM store exposes no secret** — because no secret value is present there (Book 02 §Ch08 §6). This is the concrete payoff of the reference model: the most-replicated, most-queried store in the system is secret-free by construction.
- The store SHOULD support encryption at rest, envelope encryption, and integration with external secret managers/HSMs as provider plugins (§7) — but the *contract* is the separation and the access model, not a specific technology (ADR-0001).

## 3. The reference model

A **`Secret` Resource** (Book 02 §Ch07 §5) carries only *reference metadata* — how the Broker can resolve it, its class/rotation policy, its provenance — and **never a value**. A **`SecretRef`** (Book 02 §Ch06 §4) is an opaque handle naming *which* secret, not *what* it is.

- References are safe to store, cache, log, diff, and display — they carry no secret (Book 04 §Ch07 §5).
- The type system makes `SecretRef` **opaque**: no IR construct can derive a value from it (Book 04 §Ch05 §2.2); verification rejects any attempt (Book 04 §Ch08 §2.4). P7 is thus enforced at the type level, not merely by convention.

## 4. Resolution — the only path to a value

The Broker resolves a reference to a value under a strict protocol (the materialization protocol, Book 06 §Ch06 §2), and this is the **only** path by which a secret becomes a value:

1. **Requester presents a reference** (a binding token), never a name-based "give me secret X" query, and never a bulk request.
2. **Capability check (P8).** The requester MUST hold the capability authorizing resolution of *this specific* secret (Ch 03; the `grantedBy` grant, Book 04 §Ch04 §5). The reference alone is insufficient.
3. **Policy/approval check (P9).** Any policy conditions on this secret's use are satisfied — e.g. a `payments`-class secret requires a present Approval decision (Ch 06, Ch 07).
4. **Context & timing check.** Resolution is permitted **only at execution**, **only** to the selected runtime's execution context (Book 06 §Ch06). The Broker MUST refuse resolution requested at plan time, compile time, or by a planner/compiler — those callers can never satisfy the context requirement.
5. **Scoped injection.** On success, the value is injected directly into the runtime's execution context for the specific instruction, and **audited by reference** (Ch 09). It is never returned up an API, placed in a Resource/IR/Event, or logged.

Any failure is **fail-closed** (Ch 01 §6, Book 03 §Ch13 §1): no value, the instruction fails, the execution reaches an explained terminal state.

## 5. The guarantees the Broker upholds (P7, enumerated)

The Broker, together with the reference model and isolation, guarantees a secret value **never** appears in:
- planner/reasoning context or prompts (Book 08 §04, Book 04 §Ch03 §3.2),
- IR of either level (Book 04 §Ch05 §2.2, §Ch08),
- the ARM store or any Resource (§2, Book 02 §Ch08 §6),
- the Event Bus (Book 03 §Ch03 §8),
- logs, traces, or metrics (Book 03 §Ch12 §3, Book 14 §04),
- vector stores / Knowledge (Book 09 §06),
- caches or persisted execution artifacts beyond transient use (Book 06 §Ch06 §5).

The value exists only transiently, in one isolated runtime, for one purpose, then is discarded (Book 06 §Ch06 §5). This enumerated list *is* P7, and each entry is enforced by the mechanism named beside it.

## 6. Audit without exposure (P10)

Every Broker operation — issuance of a reference, a resolution, a rotation, a revocation — is **audited by reference** (Ch 09): the record states *that* runtime R materialized secret S for execution E under grant G, with the policy/approval that permitted it — and **never** the value. Secret *use* is therefore fully traceable and attributable without secret *exposure*. This reconciles P7 (no exposure) with P10 (full attribution): you can always prove which secret was used where, without the audit trail itself becoming a leak.

## 7. Provider model and rotation

- The Broker is **core** (it enforces P7; it cannot be a third-party-replaceable component, Book 03 §Ch11 §4), but its **backends** are provider plugins (P11): an external secret manager, an HSM, a cloud KMS. The Broker knows the provider interface; the deployment chooses the backend.
- **Rotation** is first-class: a secret can be rotated (new value, same reference) so that references remain stable while values change (the reverse of the reference model's other benefit). Rotation is audited; consumers are unaffected because they hold references, not values.
- **Acquisition** (how a value first enters the Broker) is specified in Ch 05.

## 8. Invariants (normative summary)

1. The Secret Broker is the sole custodian of secret values, backed by a store physically separate from ARM; an ARM-store compromise exposes no secret (P7).
2. Secrets exist elsewhere only as opaque references; `SecretRef` is type-level opaque and verification bars deriving a value from it.
3. Resolution is the only path to a value and requires: a presented reference, the authorizing capability (P8), satisfied policy/approval (P9), execution-time context, and scoped injection into a runtime — failing any is fail-closed.
4. A secret value never appears in planning, IR, ARM, Events, logs/traces/metrics, Knowledge, or persistent caches; it exists only transiently in one isolated runtime and is then discarded.
5. Every Broker operation is audited by reference, never by value, reconciling no-exposure (P7) with full attribution (P10).
6. The Broker is core; its backends are provider plugins; rotation changes values while references stay stable, and consumers are unaffected.
