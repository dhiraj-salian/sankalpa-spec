# Book 14 · Chapter 07 — Experience Integration

*Nature: **Informative** (the normative specifications are Book 10 and §Ch01–§Ch05). · Reflects: realizes P5, P10, P13. Companion to Book 10 (Experience), Book 11 §09 (audit).*

> Observability and Experience are close cousins — both consume the Event stream — and this chapter clarifies how they relate, so the boundary is unambiguous. It is informative: the normative rules live in Book 10 (Experience) and in this Book's earlier chapters (the Event model and views). The purpose here is to place Experience within the observability picture and prevent the two from being conflated or duplicated.

## 1. Three consumers of one stream

The Event stream (§Ch01) has three principal consumers, each producing a distinct output from the same source:

| Consumer | Output | Purpose | Normative home |
|----------|--------|---------|----------------|
| **Observability** (this Book) | metrics, traces, logs | operate the system | Book 14 |
| **Audit** (Book 11 §09, exposed §Ch05) | tamper-evident trail | accountability | Book 11 §09 |
| **Experience** (Book 10) | structured learning records | improve the system | Book 10 |

All three read the *same* commit-consistent, secret-free stream (§Ch01 §5–6), so they can never disagree about what happened, and all three inherit secret-freedom (Book 03 §Ch12 §4). Their difference is entirely one of *purpose and output shape*.

## 2. Observability feeds Experience its raw signals

Experience capture (Book 10 §Ch03) and observability draw on overlapping signals:
- **Metrics** (§Ch02) supply the quantitative `ExecMetrics` an Experience records (Book 10 §Ch02 §3) and the cost/determinization-health signals the loops consume (Book 10 §Ch05–§Ch06).
- **Traces** (§Ch03) supply the correlation that lets capture assemble one Experience from many Events (Book 10 §Ch03 §2) — the trace context and reference graph are the same backbone.
- **Logs/diagnostics** (§Ch04, Book 05 §Ch07) supply the failure detail that becomes lessons and anti-patterns (Book 10 §Ch04 §2).

So observability is not just for humans watching dashboards; it is the sensory apparatus whose signals Experience distills into learning. This is the operational face of the toward-determinism mission (P13): the system observes itself in order to improve.

## 3. Where they diverge

Despite the shared source, observability and Experience are **distinct subsystems** (Book 03 §Ch12 §4) and MUST NOT be conflated:
- **Observability** produces *operator/auditor* views — is it healthy, what happened, who did it — largely aggregate or per-request, consumed live.
- **Experience** produces a *structured, analyzable Resource per execution* (Book 10 §Ch01 §2), evaluated against Goal criteria (Book 10 §Ch07), consumed by the learning loops.

Duplicating Experience's analysis inside observability (or vice versa) would blur ownership and risk inconsistency. The clean split — observability *senses*, Experience *learns* — keeps each focused.

## 4. Shared guarantees

Because both derive from the Event stream, they share the stream's guarantees:
- **Secret-free** (§Ch01 §6): neither can surface a secret, inherited not scrubbed.
- **Attributed** (P10): both trace to acting identities.
- **Tenant-scoped** (Book 11 §08): both are partitioned by workspace.
- **Complete & consistent** (§Ch01 §5): both see every state change and cannot describe one that did not happen.

These shared guarantees are why it is safe to build learning (which feeds planning) on the same foundation as audit (which must be trustworthy) — the foundation is designed to satisfy the strictest consumer, and the others inherit it.

## 5. Summary (informative)

Observability and Experience are two purposes served by one secret-free, commit-consistent, attributed Event stream: observability turns it into operational sight (metrics/traces/logs) and, with audit, accountability; Experience turns it into learning. The Event model (§Ch01) is the shared substrate; Book 10 owns the learning normative rules; this Book owns the operational ones. Together they let the system be *seen*, *held accountable*, and *improved* — from one source of truth.
