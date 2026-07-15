# Book 14 · Chapter 04 — Logging

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P7, P10. Companion to Book 03 §Ch12 (Observability Manager), Book 11 §04 (Secret Broker).*

> Logs are the most notorious secret-leak surface in real systems: a well-meaning debug line prints a token and it lives forever in a log aggregator. This chapter specifies structured logging and its **hard secret-freedom guarantee** — the constraint that a secret value is *never* logged, enforced by construction rather than by reviewer vigilance.

## 1. Structured logging, derived from the model

Sankalpa logging is **structured** (key/value records, not free-form strings) and, like all observability, integrates with the Event/Resource model (Book 03 §Ch12). Structured logs correlate with traces (§Ch03) via trace context and reference Resources/IR by id/hash (§Ch03 §3). Structure is what makes logs queryable, correlatable, and — critically — *checkable* for the secret-freedom guarantee (§2).

## 2. The hard secret-freedom guarantee (P7)

The central rule, stated as strongly as the spec states anything: **no secret value is ever logged — anywhere, at any level, by any component.** This is a *hard* guarantee (Book 03 §Ch12 §3), not a best-effort redaction:

- Secrets exist as values only transiently inside a runtime at execution (Book 11 §04, Book 06 §Ch06). Logging operates on the Event stream and structured instrumentation, both of which are secret-free by construction (§Ch01 §6) — so there is *nothing secret to log* on those paths.
- A runtime handling a materialized secret (the one place a value exists) MUST NOT log it (Book 06 §Ch06 §3): materialized secrets are used for the effect and never surfaced, including to logs. This is part of the runtime conformance suite's secret-safety group (Book 06 §Ch07 §2.4) — tested with canary secrets that must never appear in any log.
- A log line that could contain a secret is a **security defect** (Book 11 §01), treated with the gravity of a leak in IR — not a formatting nit.

Because it is enforced by *there being no secret on the logging path* (plus runtime discipline + conformance testing), the guarantee does not depend on every developer remembering to redact — which is exactly why redaction-based approaches fail and this one does not.

## 3. What logs may contain

Logs carry **references, ids, hashes, classes, and non-secret facts**: which Resource, which IR (by hash), which capability/grant, which policy decision, which error code (Book 05 §Ch07). A log about a secret operation names the `SecretRef`/class and the outcome — never the value (Book 11 §04 §6). Sensitive non-secret data (e.g. PII) is classified and MUST be handled per policy (§5), minimized and referenced rather than dumped.

## 4. Levels, correlation, and usefulness

- Logs carry standard severity levels and correlate with traces/spans (§Ch03) and Resources, so a log line is never orphaned — it ties to the intent/execution it concerns.
- Because failures are staged and diagnostic-coded (Book 05 §Ch07), logs are *actionable*: they point at the responsible stage and carry the structured diagnostic, not an opaque stack trace.
- The secret-freedom guarantee does **not** make logs useless: references, hashes, classes, and diagnostics are more than enough to debug, precisely because the whole system is designed to be reasoned about by reference (Book 04 §Ch07, Book 11 §04).

## 5. Tenancy, retention, access (P10, tenancy)

- Logs are **workspace-scoped** (Book 11 §08 §5) and **attributed** (P10): a tenant sees its own logs; each line ties to a source identity.
- Logs are retained per policy (Book 02 §Ch04 §5); because they are secret-free, long retention carries no secret-exposure risk (the same property that makes Events/Experience safe to retain, Book 10 §Ch02 §7).
- Access to logs is capability-gated and, for cross-tenant/aggregate access, privileged and itself audited (Book 11 §09 §6).
- Log **backends/sinks are replaceable providers** (Book 03 §Ch12 §5); the secret-freedom guarantee and structure are normative, the sink is an implementation choice — and the guarantee holds regardless of sink, since no secret is on the path.

## 6. Invariants (normative summary)

1. Logging is structured, correlated with traces and Resources, and referencing IR by hash — queryable and checkable.
2. No secret value is ever logged, at any level, by any component — a hard guarantee enforced by there being no secret on the logging path, plus runtime discipline and conformance testing, not by redaction (P7).
3. A materialized secret (the one place a value exists) is never surfaced to logs; canary secrets must never appear in any log (tested in runtime conformance).
4. Logs carry references, ids, hashes, classes, non-secret facts, and structured diagnostics; sensitive non-secret data is classified, minimized, and policy-governed.
5. Logs remain actionable via references/hashes/diagnostics despite carrying no secrets; a loggable secret is a security defect, not a formatting issue.
6. Logs are workspace-scoped, attributed, retained per policy without secret-exposure risk, capability-gated, and written to replaceable provider sinks.
