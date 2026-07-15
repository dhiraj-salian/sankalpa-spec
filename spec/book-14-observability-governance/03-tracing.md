# Book 14 · Chapter 03 — Tracing

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P7, P10 and IR-P6. Companion to Book 15 §01 (end-to-end walkthrough), Book 04 §Ch07 (content addressing).*

> Tracing stitches the many steps of turning one intent into executed effects into a single, followable story. This chapter specifies end-to-end tracing across the intent→execution arc, the correlation model, the rule that traces reference IR by content hash (never contents), and the secret-freedom guarantee. Tracing is what makes the layered pipeline *debuggable* and the intent→experience chain *legible*.

## 1. One intent, one trace

A single Intent flows through many components — planner, compiler passes, scheduler, runtime, controllers — each emitting Events (§Ch01). Tracing correlates them into **one end-to-end trace** so an operator (or the Experience engine, Book 10) can follow the whole journey:

```
Intent → Goals → planning → High IR → [verify · optimize · policy · determinize · lower] → Low IR
   → runtime selection → RuntimeGraph → execution (per-instruction) → effects → outcome
```

Every span in this trace is linked by **correlation/trace context** carried on every Event (Book 03 §Ch03 §2) and by the **reference graph** (Book 02 §Ch06). This is the operational realization of the layered architecture (Book 01 §03): the trace *is* the pipeline, observable.

## 2. The correlation model

- Every Event carries `traceContext` (a trace id plus span/parent ids, Book 03 §Ch03 §2), propagated across component boundaries through the Kernel API and Event Bus (P4).
- Spans nest to mirror the pipeline: a compilation span contains pass spans; an execution span contains per-instruction spans (Book 06 §Ch03).
- Correlation is robust to the Event Bus's at-least-once, non-global-order delivery (Book 03 §Ch03 §4): trace assembly is idempotent and reconstructs structure from the reference graph and span parentage, not from arrival order.

## 3. Traces reference IR by content hash, never contents (IR-P6, P7)

A defining rule: trace spans reference IR (and other large/sensitive artifacts) **by content hash** (Book 04 §Ch07), **not** by embedding contents. A compilation span records the input High IR hash and output Low IR hash; an execution span records the executed Low IR hash. Rationale:
- **Efficiency & stability.** Hashes are small and stable; a trace need not carry (or duplicate) whole IR modules.
- **Reproducibility.** The hash ties the trace to the *exact* artifact, so a trace can be correlated with the exact Compilation/Execution and replayed (Book 04 §Ch07 §4).
- **Secret-freedom (P7).** Because IR is referenced by hash and the hash contains no secret (Book 04 §Ch07 §5), traces carry no secret via IR. More broadly, **no span carries a secret value** (§4).

## 4. Secret-freedom in traces (P7)

Traces are a high-fanout, retained surface, so the secret rule is explicit here as in logs (§Ch04) and Events (§Ch01):
- No span, attribute, or tag contains a secret value. A span about a `secret.materialized` operation references the `SecretRef`, grant, and execution — never the value (Book 11 §04 §6).
- Span attributes MUST NOT encode secrets or sensitive high-cardinality data; they reference by class/id.
- A trace that could carry a secret is a security defect (Book 11 §01), not a mere observability bug.

## 5. Tracing serves debugging and learning

- **Operators** use traces to debug: *why did this intent fail? which stage? which instruction? which policy denied it?* Because failures are staged and diagnostic-tagged (Book 05 §Ch07 §2), a trace pinpoints responsibility (planner vs. compiler vs. runtime vs. policy).
- **Experience** (Book 10) uses the same correlation to assemble the per-execution record (Book 10 §Ch03 §2) — trace context and the reference graph are exactly what capture uses to correlate Events into one Experience.
- The end-to-end trace is the backbone of the Appendices walkthrough (Book 15 §01), which follows one intent through every artifact.

## 6. Tenancy and attribution (P10, tenancy)

- Traces are **workspace-scoped** (Book 11 §08 §5): a tenant sees its own traces; cross-tenant trace access is privileged and audited.
- Every span is **attributed** (its Events name `source`/`subject`, Book 03 §Ch03 §2), so a trace shows not just *what* happened but *which identity* did each step — consistent with audit (Book 11 §09).
- Trace backends are replaceable providers (Book 03 §Ch12 §5); the correlation model and hash-reference rule are normative, the collector is an implementation choice.

## 7. Invariants (normative summary)

1. A single Intent yields one end-to-end trace across the whole pipeline, correlated by trace context on every Event and by the reference graph — the pipeline made observable.
2. Spans nest to mirror the pipeline; trace assembly is idempotent and robust to at-least-once, non-global-order delivery.
3. Traces reference IR (and large/sensitive artifacts) by content hash, never contents — for efficiency, reproducibility, and secret-freedom (IR-P6, P7).
4. No span/attribute/tag carries a secret value or sensitive high-cardinality data; violations are security defects (P7).
5. Traces serve both operator debugging (staged, diagnostic-pinpointed) and Experience capture (shared correlation), and back the end-to-end walkthrough.
6. Traces are workspace-scoped and attributed; the correlation model and hash-reference rule are normative, the trace backend replaceable (P10, P11).
