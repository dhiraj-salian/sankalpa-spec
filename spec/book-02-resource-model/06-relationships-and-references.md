# Book 02 · Chapter 06 — Relationships and References

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P2, P6, P7, P8.*

> Resources do not exist in isolation: an Intent yields Goals, a Goal is planned into an IRModule, an Execution consumes secret references. This chapter defines how Resources refer to one another, how ownership drives lifecycle, and — critically — how **secret references** work so that no secret value ever enters ARM (P7).

## 1. References resolve by identity

A reference to another Resource **MUST** resolve by immutable identity, not by mutable name:

```
Ref:
  kind         string   # e.g. "Goal"
  id           string   # target metadata.id (immutable)
  apiVersion   string?  # optional; defaults to the currently served version
  workspace    ref?     # required when crossing workspace boundaries (if permitted)
```

Rationale: `name` may change (Ch 02 §2); resolving by `name` would silently re-target a reference. Resolving by `id` makes references stable for the life of the target. Human-facing tooling **MAY** display `name`, but the stored, authoritative reference is `id`.

Dangling references (target `id` absent) **MUST** be detectable: the referring Controller **MUST** surface a condition (`reason=ReferentMissing`) rather than failing opaquely, and **MUST NOT** treat a missing referent as success.

## 2. Kinds of relationship

ARM distinguishes three relationship semantics; a kind's schema **MUST** state which it uses for each reference field.

| Relationship | Meaning | Lifecycle effect |
|--------------|---------|------------------|
| **Ownership** (`ownerRefs`) | "This Resource is a part/product of that one." | Cascading deletion (Ch 04 §5). |
| **Dependency** (`spec` refs) | "This Resource needs that one to function." | No deletion cascade; drives readiness and ordering. |
| **Association** (`labels`/selectors) | "These Resources are grouped by a shared property." | Loose; no per-instance edge. |

### 2.1 Ownership
`metadata.ownerRefs` encodes the ownership graph. Each entry:

```
OwnerRef:
  kind, id, apiVersion
  controller   bool   # is the owner's controller the managing controller?
  deletionPolicy enum # "Cascade" | "Orphan"
```

Rules:
- Ownership **MUST** form a **DAG** — no cycles. The Resource Manager **MUST** reject an `ownerRef` that would create a cycle.
- Deleting an owner triggers GC of `Cascade` children through the normal deletion/finalizer path (Ch 04 §4–5).
- At most one `ownerRef` **SHOULD** have `controller: true`, identifying the managing Controller for provenance.

Example: an `Intent` owns its derived `Goal`s (`Cascade`); deleting the Intent reclaims its Goals. A `Goal` *depends on* (does not own) the `Capability` it invokes; deleting the Goal does not delete the shared Capability.

### 2.2 Dependency
Dependencies live in `spec` as typed `Ref` fields. They drive:
- **Readiness gating:** a Controller **MUST NOT** report `Ready=True` while a required dependency is absent or itself not `Ready`, unless the kind explicitly tolerates it.
- **Ordering:** reconciliation order follows dependencies; a compiler will not run until its input `IRModule` exists (Book 05).

Dependencies **MUST NOT** create deletion cascades — that is ownership's job. Conflating the two is a frequent modeling error and is prohibited.

### 2.3 Association
Associations use `labels` + selectors (Ch 02 §2). They express membership ("all Resources in project X") without materializing an edge per member, and are resolved by query, not by stored reference.

## 3. The core relationship graph

The primary ownership/dependency edges across the system (informative overview; each kind's Book is authoritative):

```
Mission ─owns→ Strategy ─owns→ Intent ─owns→ Goal
Goal ─dep→ Capability
Goal ─produces(owns)→ IRModule(High) ─dep→ Planner
IRModule(High) ─input-to→ Compilation ─produces(owns)→ IRModule(Low) ─produces(owns)→ RuntimeGraph
RuntimeGraph ─dep→ Runtime
Execution ─dep→ RuntimeGraph
Execution ─dep→ SecretRef (by reference only, §4)
Execution ─produces(owns)→ Experience ─updates→ Knowledge
```

Referential integrity across this graph is a system invariant: the Resource Manager and the owning Controllers **MUST** keep it consistent (§1 dangling-reference detection, §2.1 no cycles).

## 4. Secret references (P7) — the critical case

Secrets are the one place where the reference model is load-bearing for security. A **Secret** in ARM is a *reference*, never a value.

```
SecretRef:
  kind: "Secret"
  id:   <secret Resource id>          # points at a Secret Resource
  # NOTE: the Secret Resource's spec/status contain NO secret material —
  #       only metadata about how the Secret Broker (Book 11) can resolve it.
```

Normative requirements:
- A secret **value** **MUST NOT** appear in any Resource's `metadata`, `spec`, `status`, `ownerRefs`, `labels`, `annotations`, or in any emitted Event (P7).
- Resources that need a secret carry a `SecretRef` in `spec`. The reference is opaque; it names *which* secret, not *what* it is.
- The value is materialized **only** at execution, **only** by the Secret Broker, **only** to the runtime, and **only** by reference resolution (Book 06 §06, Book 11 §04). Planners and compilers see the reference, never the value (P7, P8).
- Access to resolve a `SecretRef` is capability-gated (P8): holding the reference is necessary but not sufficient; the resolver must also hold the capability to materialize it.

This is why "everything is a Resource" (P2) and "no secret value in ARM" (P7) coexist without contradiction: the *fact of* a secret is a Resource; the *content of* a secret is a referent that never enters the model.

## 5. Cross-workspace references

`workspace` is a tenancy boundary (Ch 02 §2). By default, references **MUST NOT** cross workspaces. A kind that permits cross-workspace references **MUST** require an explicit, capability-gated grant (P8) and **MUST** record it for audit (Book 14). Silent cross-tenant references are prohibited.

## 6. Invariants (normative summary)

1. References resolve by immutable `id`, not by `name`; dangling references are detected and surfaced, never treated as success.
2. Ownership forms a DAG and drives cascading deletion; dependency drives readiness/ordering with no cascade; the two are never conflated.
3. Referential integrity across the core graph is maintained by the Resource Manager and owning Controllers.
4. A Secret is a reference-only Resource; no secret value ever enters ARM or any Event; values are materialized only at execution by the Secret Broker, capability-gated.
5. References do not cross workspace boundaries without an explicit, audited, capability-gated grant.
