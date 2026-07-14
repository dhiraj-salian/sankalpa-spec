# Book 03 · Chapter 05 — Resource Manager

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P2, P4, P5, P6, P8, P10. Companion to Book 02 (ARM).*

> The Resource Manager is the heart of the Kernel: it is the single component that admits, validates, stores, versions, and announces every ARM Resource. Every other Manager persists its domain state through it. If Book 02 defines *what* a Resource is, this chapter defines the *component that owns them*.

## 1. Responsibilities

The Resource Manager:
1. **Admits** Resources via the Kernel API CRUD verbs (§Ch02 §2), applying the full request pipeline (§Ch02 §3).
2. **Validates** each Resource against the schema for its `apiVersion`+`kind` and against referential-integrity rules (Book 02 §Ch02, §Ch06).
3. **Assigns identity** — the immutable `metadata.id` (Book 02 §Ch02 §2) — and manages `generation`/`resourceVersion`.
4. **Stores** Resources satisfying the consistency properties of Book 02 §Ch08.
5. **Emits Events** for every admitted mutation (P5) via the Event Bus (§Ch03), derived from committed state (no dual-write).
6. **Serves reads/watches**, including multi-version conversion (Book 02 §Ch05 §3).
7. **Enforces tenancy** — every operation is workspace-scoped and capability-checked (P8).

It does **not** reconcile Resources (that is Controllers, §Ch10/Book 07) and does **not** interpret `spec` semantics beyond validation — it is the custodian of the model, not the actor upon it.

## 2. Admission pipeline

On a create/update, the Resource Manager performs, atomically per Resource:

1. **Schema validation** — `spec`/`status` conform to the kind's schema; unknown fields rejected per the kind's policy; required fields present.
2. **Identity & versioning** — assign `id` on create; bump `generation` iff `spec` changed; set `resourceVersion`; reject on `resourceVersion` mismatch (optimistic concurrency, Book 02 §Ch05 §1).
3. **Write-path separation** — reject a `status` write from a non-owner and a `spec` write to a status subresource (Book 02 §Ch02 §4); enforce the separate capabilities (P8).
4. **Referential checks** — `Ref`s resolve by `id`; `ownerRefs` form no cycle (Book 02 §Ch06 §2.1); cross-workspace refs are gated (Book 02 §Ch06 §5).
5. **Secret-freedom check** — reject any Resource carrying secret material in `metadata`/`spec`/`status` (P7); `Secret` kind is constrained to reference metadata only (Book 02 §Ch07 §5).
6. **Kind-specific admission** — optional per-kind admission hooks (validating/defaulting) registered by the owning Book/Package.
7. **Commit + emit** — commit to storage, then the corresponding Event becomes observable (never before commit, §Ch03 §6).

A failure at any step rejects the whole operation with a typed error (§Ch02 §4) and an audit Event; partial writes never occur.

## 3. Storage contract

The Resource Manager depends on a store providing the properties of Book 02 §Ch08 (linearizable single-Resource ops, optimistic concurrency, watch with resync, no cross-Resource transactions). The store is an implementation choice, not a mandated product (ADR-0001). The Resource Manager MUST NOT expose store-specific semantics through the Kernel API; callers see only the specified contract.

Domain Managers (Capability, Runtime, Policy, …) persist their state as Resources through the Resource Manager rather than owning private databases, unless a domain has a specialized store requirement (e.g. the Secret Broker's protected store, Book 11 §04, which is deliberately *not* in the ARM store — P7). This keeps one uniform, observable, versioned substrate for almost everything.

## 4. Versioning and conversion

- The Resource Manager is the enforcement point for schema versioning (Book 02 §Ch05): it stores each kind in its designated storage version and losslessly converts to any served version on read/write, preserving unknown-to-old-version fields (Book 02 §Ch05 §3).
- It runs (or hosts the controller that runs) kind migrations as a reconciled process (Book 02 §Ch05 §4), never as an unversioned batch mutation.

## 5. Events and observability (P5, P10)

- Every admitted create/update/delete and every significant status/condition transition emits a typed, secret-free Event with `source = ResourceManager` (or the writing Controller) and `subject` = the Resource (§Ch03 §2).
- Events derive from committed state via a transactional-outbox-style mechanism (§Ch03 §6); the Resource Manager MUST NOT "write then best-effort publish."
- The Resource Manager exposes `Watch` for level-triggered consumers and resync (§Ch03 §4).

## 6. Security and tenancy (P8)

- Every operation is authenticated and capability-checked at the Kernel API before reaching the Resource Manager (§Ch02 §3); the Resource Manager additionally enforces **workspace scoping** on every read/write/watch so no caller can touch or observe another tenant's Resources (Book 02 §Ch08 §7).
- The Resource Manager is a primary audit source: because all state changes flow through it and emit Events, the audit trail (Book 14 §05) is complete by construction.

## 7. Failure behavior

- On store unavailability, the Resource Manager fails reads/writes with `Unavailable` (§Ch02 §4) rather than serving stale-uncommitted or dropping Events; it never acknowledges a write it did not commit.
- It applies backpressure under load (§Ch13), shedding with `Busy` rather than growing unbounded queues.
- Recovery is clean because state is the source of truth and Events derive from it: after a restart, consumers resync from `resourceVersion` (§Ch03 §4).

## 8. Invariants (normative summary)

1. All Resources are admitted, validated, versioned, stored, and announced through the Resource Manager; nothing bypasses it (P2, P4).
2. Admission is atomic per Resource: validate → identity/version → write-path separation → referential checks → secret-freedom → kind admission → commit → emit; failures reject wholly.
3. `generation`/`resourceVersion` and optimistic concurrency are enforced here; `spec` and `status` are separately authorized.
4. No Resource carries secret material; the Secret Broker's store is separate (P7).
5. Events derive from committed state (no dual-write) and make the audit trail complete (P5, P10).
6. Every operation is workspace-scoped and capability-checked (P8); the store contract is a property set, not a product.
