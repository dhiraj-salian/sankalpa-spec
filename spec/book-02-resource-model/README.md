# Book 02 — The Agent Resource Model (ARM)

*Status: **Draft-complete** (all chapters authored) · Nature: Normative. · Reflects: ADR-0002; realizes principles P2, P3, P5, P6.*

## Scope
The uniform model in which **everything is a Resource**. Defines the anatomy of a Resource, the desired/actual-state reconciliation contract, versioning, identity, relationships, and the catalog of core Resource kinds. Every other Book expresses its entities in ARM terms.

## Chapters
1. **`01-why-a-uniform-resource-model.md`** — The Kubernetes-derived insight: one model → uniform tooling, observability, governance. What we adopt and what we change for an agentic domain.
2. **`02-resource-anatomy.md`** *(Normative)* — Metadata, Spec, Status, Desired State, Actual State, Version, Lifecycle, Controller reference, Events. Required vs. optional fields; naming and identity.
3. **`03-desired-vs-actual-state.md`** *(Normative)* — The reconciliation contract: how controllers observe drift and converge; conditions and status semantics.
4. **`04-lifecycle-model.md`** *(Normative)* — Generic lifecycle phases (Pending → Active → …→ Terminated), transitions, finalizers, and deletion/garbage-collection semantics.
5. **`05-versioning-and-schema-evolution.md`** *(Normative)* — Resource versioning, schema evolution rules, and conversion between versions (ties to versioning-and-stability).
6. **`06-relationships-and-references.md`** *(Normative)* — Ownership, references (including secret references), and referential integrity across Resources.
7. **`07-core-resource-catalog.md`** *(Normative)* — The canonical kinds: Mission, Strategy, Intent, Goal, Capability, Workflow, Project, Workspace, Knowledge, Experience, Runtime, Compiler, Planner, Package, Plugin, Secret (reference), Policy, Execution, Conversation, Artifact, Service, IRModule, Compilation, RuntimeGraph — each with its Spec/Status shape and owning Book.
8. **`08-storage-and-consistency.md`** — Storage-model expectations, consistency guarantees, and event-sourcing considerations (mechanism deferred to implementation phase).
