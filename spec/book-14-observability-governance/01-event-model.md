# Book 14 · Chapter 01 — The Event Model

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P5, P6, P7, P10. Companion to Book 03 §Ch03 (Event Bus), §Ch12 (Observability Manager).*

> Everything emits Events (P5), and Events are the raw material of observability, audit, and Experience. Book 03 §Ch03 specified the Event *Bus* (delivery, ordering); this chapter specifies the Event *model* — the complete taxonomy, the naming scheme, schema evolution, and the guarantees that make Events trustworthy as the system's single source of what-happened. It is the foundation the rest of Book 14 builds on.

## 1. Events are the system's shared truth

Three subsystems derive their views from the one Event stream: **observability** (this Book), **audit** (Book 11 §09), and **Experience** (Book 10). They can never disagree about what happened because they read the same stream, and that stream derives from committed state (Book 03 §Ch03 §6). This chapter fixes the model so all three consume a consistent, complete, secret-free, versioned taxonomy.

## 2. Event anatomy (recap and completion)

The Event envelope is specified in Book 03 §Ch03 §2; this Book completes the **`type`** taxonomy and **`data`** schemas. Recall every Event is immutable, attributed (`source`/`subject`), timestamped, workspace-scoped, per-subject-ordered, trace-correlated, and **secret-free**.

## 3. The naming scheme

Event `type` is a dotted, hierarchical name: **`<domain>.<subject-kind>.<action>`** (optionally further qualified). Examples:

```
resource.intent.created
resource.goal.updated
ir.high.produced          ir.optimized          ir.policy.validated / ir.policy.rejected
ir.lowered                runtime.graph.produced
compilation.started       compilation.succeeded / compilation.failed
scheduling.admitted       scheduling.shed
execution.started         execution.instruction.completed
execution.succeeded / execution.failed / execution.compensated / execution.cancelled
capability.granted        capability.attenuated       capability.revoked
capability.invoked
secret.reference.issued   secret.materialized        secret.rotated       secret.revoked
policy.evaluated          policy.denied
approval.requested        approval.granted / approval.denied / approval.expired
package.installed         plugin.loaded / plugin.unhealthy
knowledge.updated         knowledge.synced
experience.captured       determinization.synthesized / determinization.retired
```

Rules:
- The scheme is **hierarchical and prefix-queryable**: a subscriber can match `execution.*`, `ir.*`, `secret.*` (Book 03 §Ch03 §3). This is why the taxonomy is structured, not flat.
- Names are **stable identifiers** (P6): a type's meaning does not silently change; a new action is a new type, not a redefinition.
- Every state-changing operation across the spec has a corresponding type here; this taxonomy is the authoritative registry, extended additively as the system grows.

## 4. Event schemas evolve additively (P6)

Each `type` has a versioned **`data` schema**:
- Schemas evolve **additively by default** (P6, mirroring ARM Book 02 §Ch05 and IR Book 04 §Ch09): new optional fields may be added within a version; removing/retyping a field or changing its meaning is a breaking change requiring a new schema version and an RFC (P12).
- Consumers **tolerate unknown fields** (forward compatibility) but MUST NOT silently drop a field they *should* understand for a security decision — where an Event feeds audit or a security-relevant view, an unrecognized *type* is surfaced, not ignored (mirroring Book 04 §Ch09 §3 deny-by-default for unknowns).
- Because observability/audit/Experience all consume these schemas, schema stability is a cross-cutting contract, governed like a public interface.

## 5. Events derive from committed state (no dual-write)

Restated because the whole Book depends on it (Book 03 §Ch03 §6): an Event is observable **only** for a committed state change, and **every** committed change eventually produces its Event. There is no "log then maybe commit" or "commit then maybe log" gap. Consequences for this Book:
- **Completeness (P5).** No state change escapes the stream; therefore no state change escapes observability, audit, or Experience.
- **Consistency.** Metrics, traces, audit, and Experience cannot describe something that did not happen or miss something that did.
- This is what lets audit be *trustworthy as evidence* (Book 11 §09 §4) and Experience be *complete* (Book 10 §Ch03 §1).

## 6. Secret-freedom is a model-level invariant (P7)

Every Event, of every type, is **secret-free** (Book 03 §Ch03 §8). This is not per-emitter discipline but a property of the model: Event `data` schemas are defined to carry references, ids, hashes, and non-secret facts — never secret values. A `secret.materialized` Event, for instance, carries the `SecretRef`, the grant, the execution, and the runtime — never the value (Book 11 §04 §6). Because Events fan out to the highest-retained, most-queried surfaces (observability, audit), a secret in an Event would be the widest possible leak — so secret-freedom is enforced at emission and is a schema-design constraint, not a hope.

## 7. Invariants (normative summary)

1. Events are the single shared source of what-happened for observability, audit, and Experience, derived from the one committed-state stream so the three views cannot disagree.
2. Event `type` follows a hierarchical, prefix-queryable `<domain>.<subject>.<action>` scheme; this taxonomy is the authoritative, additively-extended registry with stable type meanings (P6).
3. Every state-changing operation across the spec has a corresponding Event type; the taxonomy is complete by construction.
4. Event `data` schemas are versioned and evolve additively; consumers tolerate unknown fields but never silently drop security-relevant unknown types.
5. Events derive from committed state (no dual-write): complete and consistent by construction (P5).
6. Every Event of every type is secret-free by schema design, carrying references/ids/hashes/facts, never secret values (P7).
