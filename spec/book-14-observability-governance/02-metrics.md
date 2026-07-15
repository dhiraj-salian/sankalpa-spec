# Book 14 · Chapter 02 — Metrics

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P7, P10, P13. Companion to Book 03 §Ch12 (Observability Manager), Book 10 §Ch07 (evaluation), Book 05 §Ch03 (cost model).*

> Metrics are the quantitative view of the system: how much, how fast, how often, how costly. This chapter specifies the standard metrics Sankalpa exposes, the SLI/SLO discipline, and how metrics feed both operators and the platform's own cost/learning loops — all secret-free.

## 1. Metrics are derived, not instrumented ad hoc

Like all observability, metrics are **derived uniformly** from the Event stream (§Ch01) and Manager instrumentation by the Observability Manager (Book 03 §Ch12 §1), not wired up per-component. This means every subsystem — including third-party plugins — becomes measurable by participating in the Event/Resource model, with no bespoke metrics plumbing (Book 03 §Ch12 §1). Consistency of metrics across the platform follows from consistency of the model.

## 2. Standard metrics

The platform exposes standard metrics across its layers (each derived from the corresponding Events, §Ch01):

| Domain | Representative metrics |
|--------|------------------------|
| **Resources** | count/create/update/delete rates per kind; reconciliation lag (`generation − observedGeneration`, Book 02 §Ch03); condition states. |
| **Planning** | Goal-derivation latency; clarification rate; plan verification pass/fail rate (Book 08). |
| **Compiler** | compilation latency; pass durations; cache hit rate; policy-rejection rate; determinization substitutions (Book 05 §Ch08). |
| **Runtimes** | execution latency (p50/p99); retry/timeout/compensation counts; per-runtime success rate; capacity/utilization (Book 06 §Ch03). |
| **Outcomes** | Goal-success rate (evaluated, Book 10 §Ch07); failure taxonomy by stage/diagnostic. |
| **Determinization** | fraction of reasoning determinized; determinized-Capability error/drift rate (Book 10 §Ch06 §5). |
| **Security** | auth/authorization denials; policy denials; approval rates; secret materialization counts (by reference). |
| **Kernel** | Kernel API latency/error rates; Event Bus throughput/lag; backpressure/shed events (Book 03 §Ch13). |

This is the platform's **self-knowledge**: where it is healthy, wasteful, or drifting. The set is extended additively as the system grows.

## 3. SLIs and SLOs

Standard metrics are organized into **Service Level Indicators** (measurable signals of health, e.g. execution p99 latency, Goal-success rate) and **Service Level Objectives** (targets on those SLIs). The Kernel and each subsystem SHOULD define SLIs/SLOs so that "is the system healthy?" has a defined, measurable answer — feeding operability (§Ch08) and degradation decisions (Book 03 §Ch13). SLOs make health objective rather than anecdotal.

## 4. Metrics feed the platform's own loops (P13)

Metrics are not only for operators; they are inputs to the platform's improvement loops:
- The **cost model** (Book 05 §Ch03 §3, Book 06 §Ch04 §3) consumes latency/cost/retry metrics to inform compiler optimization and runtime selection — always among already-equivalent, already-permitted options (Book 10 §Ch05 §3), so a bad metric affects efficiency, never correctness.
- **Determinization health** metrics (§2) are the drift signal the determinization engine watches to retire faulty Capabilities (Book 10 §Ch06 §5).
- **Evaluation** metrics (Book 10 §Ch07 §4) key the feedback loops on *evaluated success*, not mere completion.

Metrics are thus part of the machinery that makes the system get better, not just a dashboard.

## 5. Metrics are secret-free and tenancy-scoped (P7, tenancy)

- **Secret-free (P7).** Metrics carry quantities and non-secret labels; they MUST NOT carry secret values, and — a subtle point — metric **labels/dimensions** MUST NOT encode secrets or high-cardinality sensitive identifiers (a common leak vector). Metric labels reference by class/id, never by value.
- **Tenancy-scoped (Book 11 §08).** Metrics are attributable and scoped by workspace; a tenant sees its own metrics, and cross-tenant aggregate access is privileged and audited (Book 11 §09).

## 6. Backends are replaceable providers (P11)

The Observability Manager is core, but the metrics **backend/exporter** is a provider plugin (Book 03 §Ch12 §5): the Manager knows the exporter interface, the deployment chooses its metrics stack. The *standard metric set and their meanings* (§2) are normative; the store is an implementation choice (ADR-0001).

## 7. Invariants (normative summary)

1. Metrics are derived uniformly from the Event stream and Manager instrumentation; components (incl. plugins) become measurable by participating in the model, not via bespoke plumbing.
2. The platform exposes a standard, additively-extended metric set across resources, planning, compiler, runtimes, outcomes, determinization, security, and the Kernel — its self-knowledge.
3. Subsystems define SLIs/SLOs so health is objective and measurable, feeding operability and degradation decisions.
4. Metrics feed the cost model and determinization/evaluation loops (among equivalent options only), making metrics part of the improvement machinery, not just dashboards (P13).
5. Metrics — including labels/dimensions — are secret-free and tenancy-scoped (P7); cross-tenant aggregate access is privileged and audited.
6. The standard metric set is normative; the metrics backend is a replaceable provider plugin (P11).
