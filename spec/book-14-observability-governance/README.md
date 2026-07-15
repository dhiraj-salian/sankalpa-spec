# Book 14 — Observability & Runtime Governance

*Status: **Draft-complete** (all chapters authored) · Nature: Normative. · Reflects: realizes P5, P9, P10.*

## Scope
How the running system is seen and governed: the Event model in full, metrics, tracing, logging (secret-free by construction), audit, and the runtime enforcement of Policy. Complements Book 11 (security-time) with operate-time concerns.

## Chapters
1. **`01-event-model.md`** *(Normative)* — The complete Event taxonomy, naming scheme, schema evolution, and delivery/ordering guarantees (with Book 03).
2. **`02-metrics.md`** *(Normative)* — Standard metrics for Resources, Executions, compiler, and runtimes; SLIs/SLOs.
3. **`03-tracing.md`** *(Normative)* — End-to-end traces from Intent through Execution; correlation identifiers; content-addressed IR references in spans.
4. **`04-logging.md`** *(Normative)* — Structured logging with a hard guarantee that no secret value is ever logged (P7).
5. **`05-audit.md`** *(Normative)* — The audit trail; attribution and tamper-evidence (with Book 11 §09).
6. **`06-runtime-policy-enforcement.md`** *(Normative)* — The pre-execution policy checkpoint (P9) and runtime guardrails.
7. **`07-experience-integration.md`** — How observability feeds the Experience capture pipeline (Book 10).
8. **`08-operability.md`** *(Informative)* — Health, readiness, backpressure, and degradation signals for operators.
