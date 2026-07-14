# Book 10 · Chapter 03 — The Capture Pipeline

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P5, P7, P10. Companion to Book 03 §Ch03 (Event Bus), §Ch11 §3 (Experience Manager), Book 11 §09 (audit).*

> This chapter specifies how an `Experience` is assembled: the Experience Manager subscribes to the Event stream, correlates the Events of one execution into a coherent record, redacts nothing-because-there-is-nothing-to-redact (secrets never arrive), and produces the first-class Experience Resource. Capture is derivation from the commit-consistent Event stream, not a parallel logging path.

## 1. Capture derives from Events, not a side channel

The Experience Manager (Book 03 §Ch11 §3) builds Experiences by **subscribing to the Event Bus** (Book 03 §Ch03), exactly as the audit and observability paths do (Book 11 §09 §3, Book 03 §Ch12). It does not receive a private stream and does not ask components to "also send experience data." Consequences:

- **Completeness (P5).** Because every meaningful state change emits an Event, and capture consumes Events, an execution's full lifecycle is available to assemble — nothing that happened is invisible to capture.
- **Consistency.** Events derive from committed state (Book 03 §Ch03 §6); an Experience therefore cannot describe something that did not commit, and cannot miss a committed step.
- **No new leak surface.** Capture introduces no new data path that could carry a secret; it reads the *already secret-free* stream (§4).

## 2. Correlation: many Events → one Experience

An execution emits many Events across Managers and the runtime (compilation stages, scheduling, runtime progress, effects, terminal outcome). Capture **correlates** them into one Experience using:
- the **trace/correlation context** every Event carries (Book 03 §Ch03 §2, Book 14 §03), and
- the **reference graph** (Book 02 §Ch06): Intent → Goals → Compilation → RuntimeGraph → Execution, all linked by id/hash.

Correlation MUST be robust to the Event Bus's delivery semantics (Book 03 §Ch03 §4): at-least-once and per-subject-ordered but not globally ordered. Capture is therefore **idempotent** (a duplicate Event does not double-count) and **level-oriented** (it assembles from the current committed state of the referenced Resources, not solely from Event payloads). A late or duplicated Event never corrupts the assembled Experience.

## 3. Lifecycle of capture

```
Execution Events stream in (Book 03 §Ch03)
   │  Experience Manager correlates by trace context + reference graph
   ▼
Assemble Experience.spec (refs/hashes) + record (metrics, reasoning traces)
   │  on Execution terminal (Succeeded/Failed/Compensated/Cancelled, Book 06 §Ch03 §5)
   ▼
Finalize: successEval vs. Goal criteria (Ch 07), extract lessons (Ch 02 §5)
   │
   ▼
Emit feedback signals: Knowledge candidates (Ch 04), planning/cost updates (Ch 05),
                        determinization candidates (Ch 06)
```

- Capture MAY produce a **partial** Experience while the Execution progresses (for live observability) but **finalizes** it when the Execution reaches a terminal state (Book 02 §Ch03 §6). A terminal Experience is immutable in its factual record (like the Execution it describes); later analysis adds derived artifacts (lessons, candidates) as related Resources, not by rewriting facts.
- Because the Execution's terminal state is *explained* (Book 06 §Ch03 §4, Book 03 §Ch13 §3), the Experience captures not just success/failure but *how* — what completed, what compensated, what failed and why.

## 4. Secret-freedom is inherited, not added (P7)

A subtle but important point: capture does **not** need to *scrub* secrets, because **no secret ever reaches the stream it reads.** The Event Bus is secret-free by construction (Book 03 §Ch03 §8); IR is referenced by hash and carries no secret (Book 04 §Ch07 §5); metrics and reasoning traces are typed/hashed values (Book 10 §02 §4). So the Experience is secret-free *because its inputs are*, not because a redaction step removed secrets after the fact.

This is stronger than redaction: a redaction step can miss something; an input that never contains secrets cannot leak one. Capture MUST NOT introduce any path that could pull a secret value in (e.g. it MUST NOT resolve a `SecretRef` — it has neither the capability nor any reason to, Book 11 §04 §4). If capture ever *could* obtain a secret, that would itself be a security defect (Book 11 §01).

## 5. Attribution and tenancy (P10, tenancy)

- Every Experience is **attributed** by inheritance: its constituent Events name their `source`/`subject` (Book 03 §Ch03 §2), so the Experience traces to the identities that produced it (Book 11 §09). The learning trail is as accountable as the audit trail because it derives from the same attributed source.
- Capture is **workspace-scoped** (Book 11 §08 §5): an Experience belongs to the workspace of its execution, and cross-tenant experience access is a privileged, audited operation. One tenant's Experience never informs another's planning without an explicit grant (Book 02 §Ch06 §5).

## 6. Relationship to observability and audit

Capture, audit (Book 11 §09), and observability (Book 14) are three consumers of one Event stream, each producing a different view. Capture is distinguished by *purpose* (learning) and by producing a *structured, analyzable Resource* rather than metrics/traces or an integrity-focused trail. Sharing the stream keeps the three mutually consistent (they cannot disagree about what happened) and keeps the secret-freedom guarantee uniform across all of them (Book 03 §Ch12 §4).

## 7. Invariants (normative summary)

1. Experience is captured by deriving from the commit-consistent Event stream — not via a private side channel — making it complete and consistent with what actually happened (P5).
2. Capture correlates many Events into one Experience via trace context and the reference graph, idempotently and level-orientedly, robust to at-least-once/non-global-order delivery.
3. Capture may build partial Experiences during execution but finalizes an immutable factual record at terminal state; derived artifacts are added as related Resources, not by rewriting facts.
4. Experience is secret-free because its inputs (Events, hashed IR, typed traces) are secret-free — inherited, not scrubbed; capture may never obtain a secret value (P7).
5. Experience is attributed by inheritance from its Events and is workspace-scoped; cross-tenant access is privileged and audited (P10, tenancy).
6. Capture, audit, and observability are distinct consumers of one shared, secret-free, commit-consistent Event stream.
