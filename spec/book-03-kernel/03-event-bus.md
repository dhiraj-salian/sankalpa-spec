# Book 03 · Chapter 03 — The Event Bus

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P4, P5, P10. Companion to Book 14 (Observability).*

> The Event Bus is the **asynchronous** communication substrate (P4) and the mechanism behind "everything emits events" (P5). Where the Kernel API is request/response, the Event Bus is publish/subscribe: it is how state changes are announced, how controllers are woken, and how Experience and audit are built. This chapter defines its schema, delivery semantics, and ordering. Book 14 §01 defines the full Event taxonomy.

## 1. Purpose and position

Every meaningful state change in Sankalpa emits an Event (P5). The Event Bus carries these Events from producers to any number of subscribers without the producer knowing who listens (P4 — no direct component-to-component coupling). It serves four jobs at once:

1. **Reconciliation triggering** — controllers subscribe to their kinds and wake on change (Book 07); but note delivery is a *hint*, not a command (§4).
2. **Observability (P10)** — dashboards, metrics, and traces derive from the stream (Book 14).
3. **Experience capture** — the Experience engine assembles Events into records (Book 10 §03).
4. **Audit** — the stream is the tamper-evident history of what happened (Book 14 §05).

## 2. Event anatomy

An Event is an immutable, typed, append-only record:

```
Event:
  id            ULID          # unique, monotonic per producer
  type          string        # dotted name, e.g. "resource.intent.created" (Book 14 §01)
  source        ComponentRef  # the emitting Manager/Controller (attribution, P10)
  subject       Ref           # the Resource this Event concerns (by id, Book 02 §Ch06)
  time          Timestamp
  workspace     Ref?          # tenancy scope (Book 02 §Ch02)
  sequence      per-subject monotonic ordinal (§5)
  data          object        # typed payload for this event type
  traceContext  object        # correlation ids for tracing (Book 14 §03)
```

Normative rules:
- Events are **immutable**: once emitted, never edited or deleted (audit integrity). Corrections are new Events, never rewrites.
- Event `data` **MUST NOT** contain secret values (P7); it carries references and non-secret facts only. This is checked at emission.
- Every Event names its `source` and `subject`, so every state change is attributable (P10).
- Event `type` is a versioned schema (Book 14 §01); payloads evolve additively (P6).

## 3. Publish/subscribe model

- **Producers** are Managers and Controllers (plugins never publish directly to the bus; they cause Events by making Kernel API calls, which the responsible Manager reflects as Events — preserving P4).
- **Subscribers** register interest by `type` pattern, `subject` kind, and/or `workspace`, and receive matching Events.
- Subscription is **capability-gated** (P8) and **workspace-scoped**: a subscriber receives only Events for workspaces it is authorized to observe (Book 02 §Ch08 §7). A buggy or hostile subscriber cannot eavesdrop across tenants.

## 4. Delivery semantics — at-least-once, level-triggered consumers

The bus provides **at-least-once** delivery. It does **not** promise exactly-once. This is a deliberate, load-bearing choice:

- Consumers **MUST** be idempotent and **level-triggered**: on receiving an Event, a controller reconciles from the *current observed state* of the subject, not from the Event's contents as a command (Book 02 §Ch03 §3.1, Book 07 §01). Duplicate or out-of-order delivery therefore cannot cause incorrect convergence.
- Rationale: exactly-once delivery across a distributed system is either impossible or ruinously expensive; the correct engineering answer is at-least-once delivery plus idempotent, level-triggered consumers — the Kubernetes lesson ([study](../../research/prior-art/kubernetes.md)). Correctness lives in the consumer's idempotence, not in heroic delivery guarantees.
- The bus **MUST** support **resync**: a consumer that was disconnected can recover current state (via `List` + resume-from-`resourceVersion`, Book 02 §Ch08 §3) without silently missing the latest state. An Event is a wake-up; the Resource is the truth.

## 5. Ordering

- **Per-subject ordering is guaranteed.** Events concerning the same `subject` (same Resource `id`) are delivered to a given subscriber in their `sequence` order. This is what a controller needs to reason about one Resource's progression.
- **Cross-subject ordering is NOT guaranteed.** Events about different Resources may be delivered in any relative order. Consumers **MUST NOT** assume a global order. Where an ordering constraint spans Resources, it is modeled as a dependency in ARM (Book 02 §Ch06) and resolved by reconciliation, not by bus ordering.
- Rationale: total ordering across all Events is a scalability bottleneck (it serializes the whole system). Per-subject ordering is both sufficient for correctness and horizontally scalable.

## 6. Relationship to storage (no dual-write hazard)

Events and committed state must never disagree (Book 02 §Ch08 §3/§5): an Event **MUST NOT** be observable for an uncommitted write, and every committed mutation **MUST** eventually produce its Event. A conforming implementation therefore derives Events from committed state (e.g. a transactional outbox or a log-as-source-of-truth), never by a best-effort "write DB then publish" that can lose or duplicate against the commit. This closes the classic dual-write gap and is what makes the stream trustworthy for audit and Experience.

## 7. Retention and replay

- Events are retained per policy (Book 02 §Ch04 §5, Book 14) — long enough for audit, Experience assembly, and consumer resync.
- The stream is **replayable**: a new or recovering consumer can replay from a checkpoint to rebuild its view (event-sourced projection). Replay is idempotent for level-triggered consumers (§4).
- Retention respects tenancy and secret-freedom: retained Events carry no secrets (P7) and are scoped by workspace.

## 8. Secrets, again (P7)

The Event Bus is one of the highest-fanout, most-retained, most-observed surfaces in the system — precisely where a leaked secret would spread widest. Therefore the "no secret value in an Event" rule (§2) is enforced at emission and is a verification target: an Event carrying secret material is a security defect (Book 11), not a mere bug.

## 9. Invariants (normative summary)

1. Every meaningful state change emits an immutable, attributable, secret-free Event (P5, P7, P10).
2. Plugins do not publish directly; Managers/Controllers emit Events reflecting Kernel-mediated changes (P4).
3. Delivery is at-least-once; consumers are idempotent and level-triggered; the Resource is the source of truth, the Event a wake-up.
4. Per-subject ordering is guaranteed; cross-subject ordering is not; cross-Resource ordering is modeled in ARM, not assumed on the bus.
5. Events derive from committed state — no dual-write; committed writes always produce their Events and never before commit.
6. Subscriptions are capability-gated and workspace-scoped; the stream is retained and replayable within tenancy and secret-freedom constraints.
