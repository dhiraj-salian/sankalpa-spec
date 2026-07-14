# Book 06 — Runtimes

*Status: **Draft-complete** (all chapters authored) · Nature: Normative. · Reflects: ADR-0002, RFC-0001, AEP-0002 (Runtime interface); realizes P11.*

## Scope
How lowered RuntimeGraphs actually execute, and the **runtime plugin contract** every backend implements. Runtimes are plugins; the runtime is selected *after* planning. No planner or IR construct is runtime-specific.

## Chapters
1. **`01-runtime-abstraction.md`** *(Normative)* — What a Runtime is; the boundary between the compiler's RuntimeGraph and runtime execution; the execution/event reporting contract.
2. **`02-runtime-interface.md`** *(Normative)* — The AEP-0002 interface: load, validate graph, execute, report Events, cancel, health. Version negotiation and capability grants (Book 11).
3. **`03-execution-semantics.md`** *(Normative)* — Determinism obligations, idempotency, retries, timeouts, partial-failure and compensation semantics.
4. **`04-runtime-selection.md`** *(Normative)* — How the Scheduler/Runtime Manager select a runtime from capabilities, policy, and cost — after planning, never before.
5. **`05-reference-runtimes.md`** *(Informative)* — Target profiles and mapping notes for n8n, Temporal, Python, Docker, Kubernetes, Trigger.dev, Windmill, Kestra, Bash, Cloudflare Workers. Each is a lowering target, not a dependency.
6. **`06-secret-materialization.md`** *(Normative)* — How a runtime resolves secret *references* via the Secret Broker at execution and never before (P7).
7. **`07-conformance-suite.md`** *(Normative)* — The tests a runtime must pass to claim conformance at a given stability level.
