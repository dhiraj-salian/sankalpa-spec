# Book 03 — The Kernel

*Status: Draft skeleton · Nature: Normative. · Reflects: ADR-0002; realizes principles P4, P5, P11.*

## Scope
The microkernel: the small trusted core that owns orchestration and enforces the invariants. Defines the Managers, the Kernel API, and the Event Bus — the two (and only two) communication substrates.

## Chapters
1. **`01-microkernel-rationale.md`** — Why a microkernel (ADR-0002); the "small trusted core + replaceable plugins" contract; what belongs in the core and what never does.
2. **`02-kernel-api.md`** *(Normative)* — The synchronous request/response contract: resource CRUD, capability invocation, admission, and the error model. Versioning and stability of the API.
3. **`03-event-bus.md`** *(Normative)* — Publish/subscribe semantics: event schema, naming, delivery guarantees (at-least-once vs. exactly-once), ordering, partitioning, and replay.
4. **`04-managers-overview.md`** — Map of the Managers and their responsibilities and boundaries.
5. **`05-resource-manager.md`** *(Normative)* — Admission, validation, storage, versioning, and event emission for all Resources.
6. **`06-capability-manager.md`** *(Normative)* — Registry, granting, and invocation of Capabilities under P8 (no ambient authority).
7. **`07-scheduler-and-runtime-manager.md`** *(Normative)* — Scheduling of work and selection/lifecycle of Runtimes (runtime chosen *after* planning).
8. **`08-compiler-manager.md`** *(Normative)* — Orchestration of the compile pipeline (entry point to Book 05).
9. **`09-plugin-and-package-managers.md`** *(Normative)* — Loading, isolating, versioning, and replacing plugins; installing Packages (ties to Book 12).
10. **`10-lifecycle-and-controller-runtime.md`** *(Normative)* — Hosting and scheduling Controllers (detailed in Book 07).
11. **`11-security-knowledge-experience-managers.md`** — Pointers into Books 11/09/10 for the Managers specified there; the Secret Broker, Policy Engine, Approval, Session, and User Managers.
12. **`12-observability-manager.md`** — Pointer into Book 14.
13. **`13-failure-modes-and-degradation.md`** *(Normative)* — Kernel failure modes, degradation strategy, and recovery invariants.
