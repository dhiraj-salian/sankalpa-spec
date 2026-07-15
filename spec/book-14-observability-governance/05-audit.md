# Book 14 · Chapter 05 — Audit

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P5, P7, P10. Companion to Book 11 §09 (audit & attribution — the security specification).*

> Audit is specified normatively in Book 11 §09 as a *security* concern (attribution integrity is a protected asset). This chapter specifies audit as an *observability* concern: how the Observability Manager exposes the audit trail derived from the Event stream, and how it relates to metrics, traces, and logs. Book 11 §09 owns the security requirements; this chapter owns the operational exposure. The two are consistent by construction because both derive from the one Event stream.

## 1. Audit is the tamper-evident view of the Event stream

The audit trail is the **tamper-evident, attribution-complete view** of the Event stream (Book 11 §09 §3). The Observability Manager (Book 03 §Ch12 §2 item 4) exposes it. Because it derives from the immutable, commit-consistent Event stream (§Ch01 §5, Book 03 §Ch03 §6):
- it is **complete by construction** — no privileged action escapes it (Book 11 §09 §3);
- it is **consistent** with what actually happened — no dual-write gap;
- it is **attributed** — every entry names the acting identity (Book 03 §Ch03 §2, Book 11 §08 §6).

This chapter does not restate Book 11 §09's requirements (what must be audited, tamper-evidence, retention, access) — those are normative there. It specifies how audit sits *within observability*.

## 2. Audit vs. the other observability views

Audit, metrics, traces, and logs are four views of one Event stream, distinguished by purpose:

| View | Question | Emphasis |
|------|----------|----------|
| **Metrics** (§Ch02) | how much/fast/often? | aggregate quantities |
| **Traces** (§Ch03) | how did this one flow? | per-request causal structure |
| **Logs** (§Ch04) | what did components report? | per-event detail |
| **Audit** (this chapter, Book 11 §09) | who did what, provably? | integrity + attribution |

Audit is distinguished by its **integrity requirement**: it must be *trustworthy as evidence* (Book 11 §09 §4), which the others need not be. This is why audit is specified in the Security Book (its requirements are security-critical) but exposed through observability (it is operationally consumed). Keeping audit's integrity requirements in Book 11 prevents operational convenience from diluting them.

## 3. Audit without secret exposure (P7 ∧ P10)

The reconciliation of full attribution with zero secret exposure (Book 11 §09 §5) holds here because audit derives from the **secret-free** Event stream (§Ch01 §6). An audit view of a `secret.materialized` operation shows *that* runtime R materialized secret S for execution E under grant G — never the value. The observability layer cannot expose in audit what is not in the stream; secret-freedom is inherited, not added (matching logs §Ch04 §2 and Experience Book 10 §Ch03 §4).

## 4. Audit and Experience share a source, differ in purpose

Audit (accountability) and Experience (learning) both derive from the Event stream (Book 11 §09 §7, Book 10 §Ch03). The observability layer surfaces both without conflating them: audit is integrity-focused and security-normative; Experience is analysis-focused and improvement-oriented. Sharing the stream guarantees they cannot disagree about what happened, while their distinct purposes keep audit's evidentiary integrity uncompromised by learning concerns.

## 5. Operational access to audit

- The Observability Manager exposes audit query/retrieval **capability-gated** (P8) and **workspace-scoped** (Book 11 §08 §5); access to audit is itself audited (Book 11 §09 §6) — there is no un-audited window into the audit.
- Audit exposure uses replaceable provider backends (Book 03 §Ch12 §5) for storage/query, but the **integrity properties** (tamper-evidence, completeness) are normative (Book 11 §09) and MUST be preserved by any backend — a backend that could silently alter records is non-conforming.

## 6. Invariants (normative summary)

1. Audit is the tamper-evident, attribution-complete view of the Event stream, exposed by the Observability Manager; its security requirements are normative in Book 11 §09 and are not diluted here.
2. Audit is one of four Event-stream views (with metrics, traces, logs), distinguished by its integrity/evidence requirement.
3. Audit inherits secret-freedom from the Event stream, achieving full attribution with zero secret exposure (P7 ∧ P10).
4. Audit and Experience share the Event-stream source but differ in purpose; sharing guarantees consistency, separation preserves audit's evidentiary integrity.
5. Operational audit access is capability-gated, workspace-scoped, and itself audited; backends are replaceable but MUST preserve the normative integrity properties.
