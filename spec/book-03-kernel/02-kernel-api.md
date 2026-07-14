# Book 03 · Chapter 02 — The Kernel API

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P4, P6, P8, P9, P10. Companion to Book 02 (ARM).*

> The Kernel API is one of the two communication substrates (P4) and the **synchronous** one. It is the single front door for every request into the Kernel. This chapter defines its shape, semantics, guarantees, and stability. It is a Stable interface governed like a public contract.

## 1. Purpose and position

The Kernel API is how any actor — a channel adapter, a controller, a plugin, an operator, the web runtime — asks the Kernel to *do* or *observe* something and gets an answer. Its counterpart, the Event Bus (§Ch03), is asynchronous notification. Together they are the only ways anything interacts with the Kernel or, transitively, with anything else (P4).

Because it is the single front door, the Kernel API is also the single point where **authentication, capability checks (P8), admission, policy (P9), and audit (P5/P10)** are applied uniformly. There is no side door that bypasses them.

## 2. Resource-oriented design

The API is **resource-oriented**: its primary vocabulary is CRUD-plus-watch over ARM Resources (Book 02), not an open-ended set of bespoke RPCs. This mirrors the Kubernetes lesson — a small, uniform verb set over a large, uniform noun set scales better than many special verbs.

### 2.1 Core verbs
For any Resource kind the caller is authorized for:

| Verb | Semantics |
|------|-----------|
| `Create` | Admit a new Resource; assign `id`; validate; emit `created` Event. |
| `Get` | Read current state by `id` (or by `name` within a workspace). |
| `List` | Query by workspace + label selector (Book 02 §Ch02); paginated. |
| `Update` | Compare-and-swap on `resourceVersion` (Book 02 §Ch05 §1); mutate `spec`. |
| `UpdateStatus` | Status subresource write; only the owning Controller is authorized (Book 02 §Ch02 §4). |
| `Delete` | Request deletion (sets `deletionTimestamp`, runs finalizers — Book 02 §Ch04). |
| `Watch` | Subscribe to changes for a kind/selector; supports level-based resync (Book 02 §Ch08 §3). |

`spec` and `status` are **separate write paths** with **separate capabilities** (P8): a component that may observe (write `status`) is not thereby able to redirect intent (write `spec`).

### 2.2 Capability invocation
Beyond CRUD, the API exposes one controlled action verb:

| Verb | Semantics |
|------|-----------|
| `Invoke` | Invoke a granted **Capability** (Book 02 §Ch07) by reference, with typed inputs, returning typed outputs or a typed error. |

`Invoke` is the *only* way to cause an effect through the Kernel, and it is fully capability-gated (§Ch06). There is no generic "run this" verb — everything effectful is a typed Capability so it can be governed, audited, and (P13) determinized.

## 3. Request lifecycle and cross-cutting checks

Every Kernel API request passes, in order, through the same pipeline (a request that fails any stage is rejected with a typed error and an audit Event):

1. **Authentication** — establish the caller's identity/session (§Ch11, Book 11 §08).
2. **Authorization / capability check** — the caller must hold a capability permitting this verb on this Resource/Capability (P8, §Ch06). No ambient authority.
3. **Admission & validation** — schema validation against `apiVersion`+`kind` (Book 02 §Ch02); referential checks (Book 02 §Ch06); kind-specific admission.
4. **Policy** — policy that applies to control-plane operations (Book 11 §06) — distinct from IR policy validation, which is a compiler pass (Book 05 §04).
5. **Execute** — the Manager performs the operation against storage (Book 02 §Ch08).
6. **Audit & emit** — a secret-free Event is emitted (P5/P7) and the operation is auditable (Book 14 §05).

This uniform pipeline is *why* the single-front-door design is worth its indirection cost (§Ch01 §5): the invariants are enforced in exactly one place, for every request, by construction.

## 4. Semantics and guarantees

- **Consistency.** Single-Resource operations are linearizable; concurrency is optimistic via `resourceVersion` (Book 02 §Ch08 §2). The API surfaces conflicts as a typed `Conflict` error the caller retries.
- **Idempotency.** `Create` supports a caller-supplied idempotency key so retries do not double-create. `Delete`, `UpdateStatus`, and `Invoke` of an idempotent Capability are safe to retry (Book 04 §Ch04 §4).
- **No cross-Resource transactions.** The API offers no multi-Resource atomic write (Book 02 §Ch08 §4); multi-Resource consistency is achieved by reconciliation. Callers MUST NOT assume two writes are atomic.
- **Typed errors.** Every failure is a typed, stable error (`NotFound`, `Conflict`, `Unauthorized`, `PolicyDenied`, `ValidationFailed`, `Unavailable`, …) with a machine reason and a secret-free message.
- **Deadlines & backpressure.** Requests carry deadlines; under load the Kernel sheds with `Unavailable`/`Busy` rather than queueing unboundedly (§Ch13).

## 5. Transport independence

The Kernel API is specified **semantically**, independent of wire protocol. A conforming implementation MAY expose it over gRPC, HTTP/JSON, or in-process calls; all MUST provide identical semantics, verbs, error model, and checks. Rationale (ADR-0001): the *contract* is normative; the *transport* is an implementation choice that must be free to evolve. The external, internet-facing surface (the API Gateway, Book 13 §04) is a channel *in front of* the Kernel API, not a different API.

## 6. Secrets and the API (P7)

- No Kernel API request or response ever carries a secret **value**; secrets travel only as `SecretRef`s (Book 02 §Ch06 §4). `Invoke` may pass a `SecretRef` input; the value is materialized only at execution by the Secret Broker (§Ch11, Book 11 §04), never in the API layer.
- Audit records and error messages are secret-free even when reporting on secret-referencing operations.

## 7. Versioning and stability

- The Kernel API is a **Stable** interface (versioning-and-stability). Its verb set and error model change only additively within a major version; breaking changes require an RFC and a major bump with a migration path.
- Resource *kinds* served by the API are versioned independently (Book 02 §Ch05); the API multiplexes served versions and converts losslessly.
- Because so much depends on it, the Kernel API has a conformance suite: two conforming implementations MUST agree on verb semantics, checks, and error behavior for the suite.

## 8. Invariants (normative summary)

1. The Kernel API is the single synchronous front door; every request passes the same auth → authorize → admit → policy → execute → audit pipeline.
2. It is resource-oriented (CRUD + Watch) plus a single capability-gated `Invoke`; `spec` and `status` writes are separately authorized.
3. Every effectful action is a typed, granted Capability; there is no generic "run" verb and no ambient authority (P8).
4. Single-Resource ops are linearizable with optimistic concurrency; there are no cross-Resource transactions; errors are typed and stable.
5. No request/response/audit ever carries a secret value; secrets are references resolved only at execution (P7).
6. The API is transport-independent, versioned as a Stable interface, and conformance-tested.
