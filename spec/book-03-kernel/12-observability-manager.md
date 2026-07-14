# Book 03 · Chapter 12 — Observability Manager

*Nature: **Normative** (Kernel-integration contract) with pointer to Book 14. · Reflects: ADR-0002; realizes principles P5, P7, P10.*

> "Everything is observable" (P10) is only true if something makes it so uniformly. The Observability Manager is that core component: it turns the Event stream (§Ch03) and Manager instrumentation into metrics, traces, structured logs, and the audit trail. Book 14 specifies the full observability and runtime-governance model; this chapter fixes how the Manager plugs into the Kernel.

## 1. Position and principle

Because every state change emits an Event (P5) and every Event is attributable and secret-free (§Ch03 §2, §8), the raw material for observability already exists on the bus. The Observability Manager **derives** the operator- and auditor-facing views from it, rather than requiring each component to wire up its own telemetry. This keeps observability uniform across all kinds and plugins — including third-party ones, which become observable simply by participating in the Event/Resource model.

## 2. Responsibilities (Kernel contract)

1. **Metrics** — derive standard metrics for Resources, Executions, the compiler, and runtimes (Book 14 §02), including SLIs for the Kernel itself.
2. **Tracing** — stitch `traceContext` (§Ch03 §2) across the Intent → Goals → Compilation → Execution path into end-to-end traces (Book 14 §03), referencing IR by content hash (Book 04 §Ch07) rather than by contents.
3. **Logging** — provide structured logging with a **hard secret-freedom guarantee** (P7): no secret value is ever logged, mirroring the Event constraint (§Ch03 §8). This is enforced, not merely intended (Book 14 §04).
4. **Audit** — expose the tamper-evident audit trail derived from the immutable Event stream (Book 14 §05); because Events derive from committed state (§Ch03 §6), the audit is complete and consistent by construction.
5. **Health signals** — surface Kernel and plugin health/readiness (fed by the Plugin Manager §Ch09, Runtime Manager §Ch07) for operators and for degradation decisions (§Ch13).

## 3. Secret-freedom as an observability invariant (P7)

The observability surfaces — logs, traces, metrics labels, audit records — are high-fanout and long-retained, exactly where a leaked secret does the most damage. Therefore:
- The Observability Manager MUST NOT accept or emit secret values anywhere; it operates on the already-secret-free Event stream and on instrumentation held to the same rule.
- Reports about secret-referencing operations name the `SecretRef` and the action, never a value (mirrors §Ch02 §6, Book 04 §Ch08 §4).
- A telemetry path that could carry a secret is a security defect (Book 11), treated with the same seriousness as a leak in IR or Events.

## 4. Relationship to Experience (Book 10)

The Observability Manager and the Experience Manager (§Ch11 §3) both consume the Event stream, with different purposes: observability produces *operator/auditor* views (is the system healthy, what happened, who did it); Experience produces *learning* records (how did this execution go, what should improve). They share the stream and the secret-freedom guarantee but are distinct Managers with distinct outputs (Book 14 §07).

## 5. Core, with pluggable backends

The Observability Manager is **core** (P10 is a system invariant), but its exporters/backends (metrics store, trace collector, log sink) are **provider plugins** (P11) behind a provider interface — the Manager knows the interface, not a specific vendor (§Ch04 §1). This lets deployments target their own observability stacks without touching the core.

## 6. Invariants (normative summary)

1. Observability is derived uniformly from the Event stream and Manager instrumentation; components (including plugins) become observable by participating in the Resource/Event model (P5, P10).
2. Traces reference IR by content hash, not contents; audit derives from the immutable, commit-consistent Event stream and is complete by construction.
3. No observability surface (logs, traces, metrics, audit) ever carries a secret value; violations are security defects (P7).
4. Observability and Experience are distinct Managers sharing the stream and the secret-freedom guarantee.
5. The Manager is core; its exporters/backends are replaceable provider plugins.
