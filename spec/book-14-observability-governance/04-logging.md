# Book 14 · Chapter 04 — Logging

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0010 (observability-egress verification); realizes principles P7, P10. Companion to Book 03 §Ch12 (Observability Manager), Book 11 §04 (Secret Broker).*

> Logs are the most notorious secret-leak surface in real systems: a well-meaning debug line prints a token and it lives forever in a log aggregator. This chapter specifies structured logging and its **hard secret-freedom guarantee** — the constraint that a secret value is *never* logged, enforced by construction on the Kernel's own paths and by a boundary check on the one path where a plaintext secret exists, rather than by reviewer vigilance.

## 1. Structured logging, derived from the model

Sankalpa logging is **structured** (key/value records, not free-form strings) and, like all observability, integrates with the Event/Resource model (Book 03 §Ch12). Structured logs correlate with traces (§Ch03) via trace context and reference Resources/IR by id/hash (§Ch03 §3). Structure is what makes logs queryable, correlatable, and — critically — *checkable* for the secret-freedom guarantee (§2).

## 2. The hard secret-freedom guarantee (P7)

The central rule, stated as strongly as the spec states anything: **no secret value is ever admitted to the Event stream, the reasoning ledger, logs, traces, Experience, or audit.** This is a *hard* guarantee (Book 03 §Ch12 §3), not a best-effort redaction — but it is guaranteed by two different mechanisms on two different paths, and the distinction is load-bearing:

- **On Kernel-internal paths, by construction.** Logging operates on the Event stream and structured instrumentation, which are secret-free by construction (§Ch01 §6) — there is genuinely *nothing secret to log* there.
- **On the one path where a plaintext secret exists, by a boundary check.** The runtime is simultaneously the **sole holder of a materialized secret** (Book 06 §Ch06 §1) and an **untrusted plugin** with no trusted tier (Book 11 §10 §1). The Event stream is *not* "already secret-free" upstream of it: it is populated **from that runtime's own output** — `events()` returns `RuntimeEvent`s the Runtime Manager reflects onto the bus and into the reasoning ledger (Book 06 §Ch02 §5, §Ch03 §1). So on this path the guarantee cannot rest on "nothing secret is here," because something secret *is* here. It rests instead on **observability-egress verification** (§2.1): the Manager checks the runtime's output against the execution's materialized-value digest set before any sink sees it.
- A log line that could contain a secret is a **security defect** (Book 11 §01), treated with the gravity of a leak in IR — not a formatting nit.

Stating this precisely matters more than stating it strongly. The earlier formulation — that the guarantee holds because *there is nothing secret on the logging path*, plus runtime discipline and conformance testing — was **false at exactly the point that matters**: a runtime *does* hold the value, "discipline" is an unverified `MUST` on untrusted code, and conformance is a **pre-admission** test that a malicious runtime passes and then ignores in production. A test is not an enforcement boundary. §2.1 supplies the boundary the claim always presupposed.

### 2.1 Observability-egress verification

Before reflecting any runtime-emitted `RuntimeEvent`, log record, trace span, or metric into **any** durable or shared sink — Event Bus, reasoning ledger, Experience, logs, audit — the Runtime Manager **MUST** verify the payload carries no value materialized into that Execution:

- The Secret Broker maintains, per Execution, a **materialized-value digest set**: for each value materialized (a small, stable set, Book 06 §Ch06 §2.1), a keyed digest `HMAC(k_E, value)` under a **per-execution key held by the Broker/Manager and never given to the runtime**. A keyed digest — not a bare hash — keeps the set from becoming an offline-guessable oracle, and the per-execution key bounds its lifetime.
- The Manager checks each egress payload's field values against the set in constant time. The set covers each value **verbatim** plus a **fixed, closed set of common encodings** (base64, hex, URL-encoding): these are the *accidental*-leak vectors (a debug dump that base64s an auth header), and covering them is decidable and cheap.
- The check is **structural, not heuristic**: `RuntimeEvent`s are already typed (Book 06 §Ch02 §2), and the Manager **MUST** reject payloads with free-form/opaque fields it cannot decompose into checkable typed values.
- **On a match** the Manager **MUST NOT** persist or forward the payload — it is dropped at the boundary — and **MUST** fail the Execution with a `SecretEgressViolation` condition (Book 02 §Ch03 §3.5), emit a high-severity **secret-free** Event and audit record naming runtime, `SecretRef`/class, and execution (never the value, §Ch05, Book 11 §09), and surface it for operator action. It **MUST NOT** auto-revoke the runtime's grants: containment is already achieved on the data path (the payload never lands), and auto-revocation would be a self-inflicted-DoS lever on a single false positive. Fail closed on the **data**; human-in-the-loop on the **component**, using the existing revocation lever (Book 11 §03 §5) at an operator's decision.

**Scope, stated honestly.** This catches emission of a materialized value **verbatim or under the declared encodings**. It does **not** catch a runtime that transforms the value outside that closed set (encrypts it, splits it across fields) or exfiltrates it directly across **B4** to its own endpoint (Book 11 §02). Those remain governed by the primary defenses — effect declaration and policy (Book 04 §Ch06, §Ch06 of this Book), egress capability grants, and isolation (Book 11 §10). The precise claim: **the platform's own durable stores are protected by an enforced boundary rather than by an untrusted component's promise**, and accidental verbatim leakage — the dominant real-world failure this chapter opens by describing — becomes *impossible* rather than merely *prohibited*. Claiming more would repeat the error §2 just corrected.

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
- Log **backends/sinks are replaceable providers** (Book 03 §Ch12 §5); the secret-freedom guarantee and structure are normative, the sink is an implementation choice — and the guarantee holds regardless of sink because egress verification (§2.1) sits **before** the hand-off to any sink. It is a Kernel-side check, never delegated into a replaceable provider: a guarantee enforced inside a swappable component would be no guarantee at all.

## 6. Invariants (normative summary)

1. Logging is structured, correlated with traces and Resources, and referencing IR by hash — queryable and checkable.
2. No secret value is ever admitted to the Event stream, ledger, logs, traces, Experience, or audit — a hard guarantee held by construction on Kernel-internal paths (nothing secret is there) and by **observability-egress verification at the boundary** on the one path where a plaintext secret exists: the untrusted runtime's own output (§2.1). Never by redaction (P7).
3. A runtime's observability output is **verified, not trusted**: checked against the execution's materialized-value digest set before any sink, dropped and failed `SecretEgressViolation` on a match, with escalation and attribution but no automatic revocation. The check covers verbatim emission plus a closed set of common encodings; it does not bound a runtime that transforms the value or exfiltrates across B4, which isolation, effect declaration, and egress policy govern (§2.1).
4. Logs carry references, ids, hashes, classes, non-secret facts, and structured diagnostics; sensitive non-secret data is classified, minimized, and policy-governed.
5. Logs remain actionable via references/hashes/diagnostics despite carrying no secrets; a loggable secret is a security defect, not a formatting issue.
6. Logs are workspace-scoped, attributed, retained per policy without secret-exposure risk, capability-gated, and written to replaceable provider sinks.
