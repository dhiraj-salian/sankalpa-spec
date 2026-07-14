# Book 02 · Chapter 01 — Why a Uniform Resource Model

*Nature: **Informative**. · Reflects: ADR-0002; realizes principles P2, P3, P5, P6, P10.*

## 1. The claim

Sankalpa asserts, as principle **P2**, that *everything is a Resource*. This chapter justifies that claim before the following chapters make it precise. The claim is strong: a Mission and a Secret reference, a Planner plugin and a single Execution, a Policy and a Conversation — all are modeled in one uniform envelope with the same anatomy, the same lifecycle machinery, the same versioning rules, and the same event semantics.

Uniformity of this kind is not an aesthetic preference. It is the mechanism by which a system of this scope stays comprehensible, governable, and extensible over a decade.

## 2. What uniformity buys

Every capability below is written **once**, against the abstract Resource, and is then inherited by every concrete kind — including kinds invented years from now by third parties who never coordinated with the core team:

- **Uniform tooling.** One way to create, read, update, watch, and delete anything. The Kernel API (Book 03) has a small, stable surface because it speaks *Resources*, not two dozen bespoke object types.
- **Uniform observability (P10).** Because every Resource has `status`, `conditions`, and emits `Events` (P5), one dashboard, one query language, and one audit trail cover the whole system.
- **Uniform governance (P9).** Policy can be written against Resource shapes and transitions without special-casing each kind.
- **Uniform lifecycle (P3).** Reconciliation, finalizers, and garbage collection are implemented in the Controller Runtime once, not per kind.
- **Uniform versioning (P6).** Schema evolution and migration follow one set of rules everywhere.
- **Uniform extensibility (P11).** A new kind is *data* (a registered schema + a controller), not a change to the Kernel. The system grows without the core growing.

The alternative — bespoke models per concept — produces a system where every new feature touches the core, every tool handles a growing zoo of shapes, and governance is a patchwork. That system cannot be maintained for ten years by a rotating community.

## 3. What we adopt from prior art — and what we change

The uniform-resource idea is proven at scale by Kubernetes (see the [prior-art study](../../research/prior-art/kubernetes.md)) and, in a different form, by Terraform's desired-state provider model and by REST's uniform interface constraint. Sankalpa adopts the shape but adapts it to an *agentic* domain:

| Kubernetes / Terraform | Sankalpa (ARM) |
|------------------------|----------------|
| Objects are primarily **authored by humans** as YAML/HCL. | Most Resources are **derived by the system** from Intent; humans express intent in natural language, not manifests. |
| Workload = container/pod/cloud resource. | Workload = an **AOS IR execution** (Books 04–06); the runtime is pluggable. |
| Ambient cluster/provider trust is common. | **No ambient authority** (P8); every Resource operation is capability-gated (Book 11). |
| Storage is a mandated backend (etcd/state file). | We mandate storage *properties*, not a backend (Ch 08). |
| Secrets are (weakly) Resources holding values. | A Secret is a **reference-only** Resource; values never live in ARM (P7). |

The deepest divergence is philosophical: in Kubernetes a human writes the desired state directly; in Sankalpa the desired state is usually the *output of planning*. ARM must therefore be excellent at being **machine-authored and machine-reconciled**, with human authoring as a supported-but-secondary path.

## 4. The cost, stated honestly

Uniformity has a price, and this Book pays it deliberately:

- **Abstraction pressure.** Forcing genuinely different things into one envelope risks a lowest-common-denominator model. We mitigate this by keeping the *envelope* uniform while allowing each kind's `spec`/`status` to be arbitrarily rich (Ch 02).
- **Indirection.** Everything going through the Resource Manager (P4) adds a hop versus direct manipulation. We accept this because it is the single enforcement point for policy, audit, and versioning.
- **Ceremony for ephemeral things.** A one-shot Execution carries the same machinery as a long-lived Service. Ch 04 and Ch 08 address this with lightweight lifecycles and retention policies rather than by exempting kinds from the model.

## 5. The boundary of the claim

"Everything is a Resource" governs everything the Kernel *manages and reasons about*. It does **not** mean every byte in the system is a Resource: the *contents* of a rendered artifact, the *value* of a secret, and the *interior* of a runtime's execution are referenced by Resources but are not themselves ARM objects. The rule is: **if the system creates it, governs it, versions it, or reconciles it, it is a Resource; if the system merely points at it, it is a referent.** Chapter 06 makes the reference model precise.

## 6. What the rest of Book 02 specifies

The remaining chapters turn this justification into a normative contract: the exact [anatomy](02-resource-anatomy.md) of a Resource; the [desired-vs-actual](03-desired-vs-actual-state.md) reconciliation contract; the [lifecycle](04-lifecycle-model.md); [versioning and schema evolution](05-versioning-and-schema-evolution.md); [relationships and references](06-relationships-and-references.md); the [catalog of core kinds](07-core-resource-catalog.md); and [storage and consistency](08-storage-and-consistency.md).
