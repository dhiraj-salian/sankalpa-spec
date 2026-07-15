# Book 14 · Chapter 08 — Operability

*Nature: **Informative** (operational guidance; the referenced mechanisms are normative in their home Books). · Reflects: realizes P10. Companion to Book 03 §Ch13 (failure & degradation), §Ch02 (metrics), §Ch01 (events).*

> Operability is the degree to which the running system can be understood, kept healthy, and safely intervened upon by operators. This chapter assembles the operational surface — health, readiness, backpressure, and degradation signals — from the mechanisms specified across the spec, so operators have one place that describes "how do I run and reason about this system?" It is informative: each mechanism is normative in its home Book.

## 1. What operability requires

A decade-scale platform must be *operable*: an operator (or automated control) must be able to answer, at any moment, "is it healthy? if not, where and why? what will it do under stress? how do I intervene safely?" Sankalpa answers these through mechanisms already specified — this chapter shows how they compose into an operational picture.

## 2. Health and readiness

- **Component health.** Every Manager and every plugin exposes health/readiness (Book 03 §Ch09 §4, §Ch12 §2, Book 06 §Ch02 §3, Book 08 §Ch03 §5). The Observability Manager surfaces these as health signals; an unhealthy component is routed around (a runtime by selection, Book 06 §Ch04; a plugin by the Plugin Manager, Book 03 §Ch09).
- **SLIs/SLOs** (§Ch02 §3) turn health into objective targets, so "healthy" is measured, not guessed.
- **Reconciliation lag** (`generation − observedGeneration`, §Ch02) reveals whether controllers are keeping up — a leading indicator of trouble.

## 3. Backpressure and load

The system degrades under load in a defined order (Book 03 §Ch13 §2), and operability requires that this be *visible*:
- **Backpressure/shed** events (`scheduling.shed`, `Busy`/`Unavailable` responses, §Ch01, Book 03 §Ch02 §4, §Ch07) are surfaced as metrics/Events so operators see the system shedding *before* it topples.
- **Saturation signals** (queue depths, capacity utilization, Event Bus lag, §Ch02) show what will saturate first — the operator's early warning.
- Because degradation is *ordered* (backpressure → domain isolation → plugin containment → read-mostly survival, Book 03 §Ch13 §2), operators can reason about *what the system will do next* under continued stress, rather than being surprised.

## 4. Degradation and failure visibility

- **Fail-closed invariants** (Book 03 §Ch13 §1, Book 11 §01 §6): operators can rely on the fact that under failure the system *removes capability* rather than *relaxing an invariant* — a denied action under stress is the system behaving correctly, not malfunctioning. Observability distinguishes "denied by fail-closed safety" from "broken."
- **Explained terminal states** (Book 03 §Ch13 §3, Book 06 §Ch03 §4): a failed execution says what completed, compensated, and failed — so operators debug outcomes, not mysteries.
- **Staged diagnostics** (Book 05 §Ch07 §2): a failure names its responsible stage, so operators (and traces, §Ch03) pinpoint where to look.

## 5. Safe intervention

Operator actions are themselves governed — operability does not mean an escape hatch around the invariants:
- Privileged operations (e.g. force-delete past a stuck finalizer, Book 02 §Ch04 §4.3; forced grant revocation) are **capability-gated and audited** (Book 11 §03, §09) — an operator acts as an attributed, authorized identity, never anonymously or with ambient power (P8, Book 11 §08 §6).
- Interventions leave the same audit trail as any action (Book 11 §09), so "what did the operator do?" is always answerable.
- Recovery is **deterministic from committed state** (Book 03 §Ch13 §4): operators can trust that restarting components converges to the same place, because state is authoritative and controllers re-reconcile idempotently.

## 6. The operational picture, assembled

Putting it together, an operator has:
- **Metrics + SLOs** (§Ch02) for aggregate health,
- **Traces** (§Ch03) to follow any single intent,
- **Logs + staged diagnostics** (§Ch04, Book 05 §Ch07) for detail,
- **Audit** (§Ch05, Book 11 §09) for accountability,
- **Health/backpressure/degradation signals** (§2–§4) for load and failure behavior,
- **Governed, audited intervention** (§5) for safe action,
- all **secret-free** (§Ch01 §6) and **tenant-scoped** (Book 11 §08).

This is what makes Sankalpa operable at a decade scale: not a bolted-on ops dashboard, but an operational picture that *falls out of* the same Event/Resource model and fail-closed discipline that make the system correct.

## 7. Summary (informative)

Operability is emergent, not additive: because everything is a Resource with a lifecycle and Events (P2/P3/P5), because failure is ordered and fail-closed (Book 03 §Ch13), because diagnostics are staged (Book 05 §Ch07), and because intervention is governed and audited (Book 11) — the system is understandable, healthy-or-visibly-not, predictable under stress, and safely operable, from one secret-free source of truth.
