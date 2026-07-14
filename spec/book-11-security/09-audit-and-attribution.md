# Book 11 · Chapter 09 — Audit and Attribution

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P5, P7, P10. Companion to Book 03 §Ch03 (Event Bus), Book 14 §05 (audit), Book 10 (Experience).*

> Every privileged action in Sankalpa MUST be truthfully attributable and durably recorded. This chapter specifies audit and attribution: what must be captured, how the trail is made tamper-evident, and how it achieves full accountability **without** ever exposing a secret (reconciling P10 with P7).

## 1. Why attribution is a security asset

Attribution integrity is listed among the protected assets (Ch 01 §1) because the other protections are unenforceable without it. If actions could not be truthfully attributed:
- A capability misuse could not be traced to its holder.
- An approval could be repudiated.
- A policy bypass could not be detected or investigated.
- Tenancy violations could not be proven.

Audit is therefore not merely operational hygiene; it is the mechanism that makes every *other* security guarantee **verifiable after the fact** (P10).

## 2. What is attributed

Every **privileged action** MUST be attributed to an identity (Ch 08 — human, session, plugin, controller, service account) and recorded. Privileged actions include, at minimum:
- Every Kernel API mutation and capability `Invoke` (Book 03 §Ch02, §Ch06 §4).
- Every capability grant, attenuation, and revocation (Ch 03).
- Every secret acquisition, materialization, rotation, and revocation — **by reference** (Ch 04 §6, Ch 05 §4).
- Every policy decision (allow/guard/deny) and every approval decision (Ch 06, Ch 07).
- Every plugin load, and every privileged/forced operation (e.g. force-delete, Book 02 §Ch04 §4.3).

For each, the record captures *who* (attributed identity), *what* (the action, by reference), *when* (timestamp), *where* (workspace), *under what authority* (the grant/approval), and *outcome*.

## 3. The audit trail derives from the Event stream

Audit is not a separate best-effort log; it **derives from the immutable Event stream** (P5, Book 03 §Ch03), which itself derives from committed state (Book 03 §Ch03 §6). Consequences:
- **Completeness by construction.** Because every state change emits an Event, and audit derives from Events, there is no privileged action that escapes audit (Book 03 §Ch05 §5).
- **Consistency.** The audit trail cannot disagree with what actually happened — there is no dual-write gap (Book 03 §Ch03 §6). An action and its audit record stand or fall together.
- **Attribution built in.** Every Event names its `source` and `subject` (Book 03 §Ch03 §2); attribution is intrinsic, not reconstructed.

The Observability Manager (Book 03 §Ch12, Book 14 §05) exposes the audit view; this chapter fixes its security requirements.

## 4. Tamper-evidence

The audit trail MUST be **tamper-evident**: any alteration, deletion, or reordering of records is detectable. Because Events are immutable and append-only (Book 03 §Ch03 §2) and derive from a commit-consistent source, the trail supports integrity verification (e.g. hash-chaining / sequence integrity per subject). An adversary who compromised a component still cannot silently rewrite history without detection — corrections are new records, never edits (Book 03 §Ch03 §2). This is what makes the trail trustworthy *as evidence*.

## 5. Audit without exposure (P7 ∧ P10)

The central reconciliation of this Book: **full attribution with zero secret exposure.**
- Audit records are **secret-free** (Book 03 §Ch03 §8, §Ch12 §3): they reference secrets by `SecretRef`/class, never by value. A materialization record proves *that* runtime R used secret S for execution E under grant G — never *what* S was (Ch 04 §6).
- The same holds for sensitive data: records attribute and reference, applying redaction/classification so the audit trail is not itself a leak of the data it describes.
- This is why P7 and P10 do not conflict: attribution is about *references and identities*, which are safe to record; exposure is about *values*, which are never recorded. You can prove everything about who did what without the trail becoming a treasure trove.

## 6. Retention, tenancy, and access

- Audit records are **retained per policy** (Book 02 §Ch04 §5, Book 14) long enough for investigation, compliance, and Experience (Book 10) — and, for audit specifically, often longer than the Resources they describe.
- Audit is **workspace-scoped** (Ch 08 §5): a tenant sees its own audit trail; cross-tenant audit access is a privileged, itself-audited operation.
- **Access to audit is capability-gated** (P8) and — recursively — audited: reading the audit trail is itself a privileged action that leaves a record. There is no un-audited window into the audit.

## 7. Relationship to Experience

Audit (this chapter) and Experience (Book 10) both derive from the Event stream but serve different masters: audit answers *"who did what, provably"* (accountability, security); Experience answers *"how did it go, what should improve"* (learning). They share the stream and the secret-freedom guarantee (Book 03 §Ch12 §4). Audit is normative and security-critical; Experience is about the platform getting better. Keeping them distinct prevents learning concerns from diluting the integrity requirements of the audit trail.

## 8. Invariants (normative summary)

1. Every privileged action is attributed to an identity and recorded: who, what (by reference), when, where, under what authority, and outcome.
2. Audit derives from the immutable, commit-consistent Event stream, making it complete by construction and consistent with what actually happened.
3. The trail is tamper-evident; alteration/deletion/reordering is detectable; corrections are new records, never edits.
4. Audit records are secret-free and reference-only, achieving full attribution (P10) with zero secret exposure (P7).
5. Audit is retained per policy, workspace-scoped, capability-gated, and access to it is itself audited.
6. Audit (accountability) and Experience (learning) are distinct systems over the shared, secret-free Event stream.
