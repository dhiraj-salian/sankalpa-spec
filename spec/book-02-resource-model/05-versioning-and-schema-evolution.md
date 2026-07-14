# Book 02 · Chapter 05 — Versioning and Schema Evolution

*Nature: **Normative**. · Reflects: ADR-0001; realizes principle P6. Companion to [`../../process/versioning-and-stability.md`](../../process/versioning-and-stability.md).*

> Two different things are versioned in ARM and this chapter keeps them distinct: (a) the **instance** of a Resource as it changes over time (`generation`, `resourceVersion`), and (b) the **schema** that governs a kind (`apiVersion`). Confusing the two is a common and costly error.

## 1. Instance versioning

An individual Resource changes over its life. Three fields track this (defined in Ch 02):

- `metadata.generation` — the **desired-state version**. Increments iff `spec` changes. Controllers compare it to `status.observedGeneration` to detect pending work (Ch 03).
- `metadata.resourceVersion` — the **storage version / concurrency token**. Opaque; changes on *any* write (spec or status). Used for optimistic concurrency (Ch 08).
- `metadata.id` — the **immutable identity**. Constant across all generations.

Rules:
- Clients performing a compare-and-swap update **MUST** send the `resourceVersion` they read; the Resource Manager **MUST** reject the write if it no longer matches (Ch 08 §concurrency).
- `generation` is monotonic and never decreases, even across `spec` rollbacks (a rollback is a *new* generation with old content).
- Instance history (who changed what, when) lives in the **Event stream** (P5), not inside the Resource; the Resource holds only current state.

## 2. Schema versioning

A **kind**'s schema (the shape of its `spec`/`status`) evolves over the project's life. This is versioned in `apiVersion` as `<group>/<version>`, e.g. `core.sankalpa.dev/v1alpha1`.

### 2.1 Stability levels in `apiVersion`
Versions carry a stability level, mirroring the project-wide policy and the Kubernetes convention:

| Form | Meaning | Guarantee |
|------|---------|-----------|
| `v1alpha1` | Experimental | May change or be removed in any release. Opt-in. |
| `v1beta1` | Beta | Breaking changes only across versions, with deprecation. |
| `v1` | Stable | Backward-compatible within the major; removal needs a major bump + migration. |

These levels are the ARM-instance expression of [`versioning-and-stability.md`](../../process/versioning-and-stability.md); a kind graduates `alpha → beta → stable` on the same evidence an AEP does.

### 2.2 What is a breaking schema change
A change to a kind's schema is **breaking** if it would invalidate a previously valid Resource or a conforming controller/consumer: removing or renaming a field, tightening a type or constraint, changing required-ness, or altering the meaning of an existing field. Breaking changes **MUST** advance the schema version and **MUST** be justified by an RFC (P12).

### 2.3 What is a compatible schema change
Adding an *optional* field with a safe default, relaxing a constraint, or adding a new `condition.type` is **compatible** and **MAY** be made within a version. Compatible-by-default is the rule (P11): grow schemas additively.

## 3. Multiple served versions and conversion

A kind **MAY** be served at more than one `apiVersion` simultaneously during a migration (e.g. `v1beta1` and `v1`). When it is:

- Exactly **one** version is the **storage version**; all instances are persisted in it.
- The Resource Manager **MUST** losslessly **convert** between served versions on read/write. Conversion **MUST** round-trip: converting `A → B → A` yields the original for any served pair, or the change is not compatible and must not be offered.
- Fields that exist only in a newer version are preserved across a round-trip through an older version by a defined preservation mechanism (annotation-based preservation is the default) so that a client using the old version cannot silently drop data it does not understand.

This is how a kind evolves without a flag day: producers and consumers migrate independently while the Resource Manager bridges versions.

## 4. Migration

When a kind must move to a new storage version, migration is itself a **reconciled** process, not a batch script bolted on:

1. Introduce the new version as *served, non-storage*; both versions validate.
2. Flip the storage version; a migration controller rewrites existing instances into the new storage version (idempotently, Ch 03 §3.2), emitting Events.
3. Once all instances are migrated and no consumer uses the old version, deprecate then remove it (never before its deprecation window elapses).

Migration tooling and the per-kind migration plan are captured in an [implementation plan](../../templates/implementation-plan-template.md) and gated by the [migration review gate](../../process/review-gates.md).

## 5. Relationship to AOS IR and Package versions

ARM schema versioning is one of several coordinated version spaces:

- **AOS IR** has its own versioning (Book 04 §09) — an `IRModule` is an ARM Resource whose *body* carries an IR schema version distinct from the `IRModule` kind's `apiVersion`.
- **Packages** (Book 12) declare compatibility against spec/AEP versions.

These spaces are independent but must be *jointly consistent*: a Package that ships a controller for kind `Foo/v1` must declare that dependency, and the Package Manager (Book 12 §07) enforces it. No version space may be advanced in a way that silently breaks another (P6, P11).

## 6. Invariants (normative summary)

1. Instance versioning (`generation`, `resourceVersion`, immutable `id`) and schema versioning (`apiVersion`) are distinct and never conflated.
2. `generation` bumps only on `spec` change; `resourceVersion` bumps on any write; `id` never changes.
3. Optimistic concurrency uses `resourceVersion`; stale writes are rejected.
4. Breaking schema changes advance the schema version, require an RFC, and preserve a migration path; compatible changes are additive and default-safe.
5. Multiple served versions convert losslessly and round-trip; exactly one is the storage version.
6. Kind migration is reconciled, idempotent, event-emitting, and gated; old versions are removed only after their deprecation window.
