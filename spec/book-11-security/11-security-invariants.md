# Book 11 · Chapter 11 — Consolidated Security Invariants

*Nature: **Normative**. · Reflects: SECURITY.md, all of Book 11; realizes P1, P7, P8, P9, P10. This is the checklist the security review gate enforces.*

> This chapter consolidates the security invariants scattered through the specification into one authoritative list — the checklist the [security review gate](../../process/review-gates.md) applies to every change, and the criteria against which any design is judged. A proposal that weakens any invariant here is a security issue by definition (SECURITY.md §Non-negotiable invariants). This chapter is normative and load-bearing: it is the single place a reviewer confirms a change is safe.

## 1. The non-negotiable invariants

These are **fail-closed** (Book 03 §Ch13 §1): if the mechanism enforcing one cannot operate, the guarded action is refused, never allowed.

### SI-1 — Natural language is never executable (P1)
Only AOS IR executes. No prompt, model output, or natural-language text is ever executed. *Enforced by:* the compiler pipeline as the sole path to execution (Book 05), verification (Book 04 §Ch08), planner output-always-verified (Book 08 §01).

### SI-2 — Secrets never enter reasoning, IR, or observability; by reference only (P7)
No secret value appears in planner/reasoning context, prompts, IR (either level), the ARM store, Events, logs, traces, metrics, Knowledge/vector stores, or persistent caches. Secrets flow only as references and are materialized only at execution. *Enforced by:* the Secret Broker + separate store (Ch 04), `SecretRef` type-opacity + verification (Book 04 §Ch05 §2.2, §Ch08), planner isolation (Book 08 §04), secret-free Events/logs (Book 03 §Ch03 §8, §Ch12 §3), out-of-band acquisition (Ch 05), materialization discipline (Book 06 §Ch06).

### SI-3 — No ambient authority (P8)
A subject can do only what its explicitly-held, unforgeable capabilities permit; identity alone confers nothing; there is no transitive escalation; delegation only attenuates. *Enforced by:* the capability model (Ch 03), the Capability Manager (Book 03 §Ch06), identity-is-not-authority (Ch 08 §1).

### SI-4 — Policy is enforced before compilation and before execution (P9)
Plans are policy-validated before lowering; actions are policy-checked at the control plane and again at runtime against live state; unresolved/un-evaluable policy denies. *Enforced by:* the Policy Engine's three checkpoints (Ch 06, Book 05 §Ch04, Book 14 §06).

### SI-5 — Every privileged action is attributable and audited (P10)
Every privileged action is attributed to an identity and durably, tamper-evidently recorded — by reference, never exposing secrets. *Enforced by:* audit derived from the immutable Event stream (Ch 09, Book 03 §Ch03), attribution intrinsic to Events.

## 2. Supporting invariants

These uphold the non-negotiable ones and are equally checked.

### SI-6 — Untrusted by default; verify output
Everything outside the Kernel core is untrusted; security/determinism-relevant plugin output is re-verified; self-claims are conformance-checked, never trusted. *Enforced by:* isolation (Ch 10), trust boundaries (Ch 02), verification (Book 04 §Ch08), conformance suites (Book 06 §Ch07, Book 08 §Ch07).

### SI-7 — Isolation and least privilege for all plugins
Every plugin is isolated, resource-limited, and granted the minimum attenuated capabilities its function needs. *Enforced by:* plugin isolation (Ch 10), Plugin Manager (Book 03 §Ch09), attenuation (Ch 03 §4).

### SI-8 — Tenancy isolation is default-deny
All access is workspace-scoped; cross-tenant flow requires an explicit, audited, capability-gated grant. *Enforced by:* workspace scoping (Book 02 §Ch08 §7), reference rules (Book 02 §Ch06 §5), identity (Ch 08 §5).

### SI-9 — High-consequence actions require human approval
Irreversible/sensitive actions are gated on non-repudiable human authorization, checked against live state, fail-closed. *Enforced by:* the Approval Engine (Ch 07), the runtime checkpoint (Book 14 §06).

### SI-10 — Credentials enter only out-of-band
Secrets are acquired only via secure one-time pages / OAuth directly into the Broker, never through channels/planning; OAuth preferred. *Enforced by:* credential acquisition (Ch 05).

### SI-11 — Fail-closed everywhere
Any failure of a security mechanism results in refusal, not permission; degradation removes capability, never relaxes an invariant. *Enforced by:* Kernel failure model (Book 03 §Ch13 §1), and each mechanism's fail-closed behavior above.

### SI-12 — Defense in depth
No single mechanism is load-bearing alone; each asset is protected by multiple independent layers. *Enforced by:* the layering of SI-2 (reference + Broker + isolation + audit) and SI-4 (three checkpoints), etc.

## 3. How this chapter is used

- **Security review gate.** Every change of tier T2+ (and any change touching secrets, capabilities, policy, trust boundaries, or isolation) is checked against SI-1…SI-12 via the [security-review template](../../templates/security-review-template.md) (Book 00 §review-gates). The reviewer confirms each relevant invariant holds or the change is blocked.
- **Design judgment.** Any RFC/ADR/AEP states which invariants it touches and how each is upheld (CONTRIBUTING §mandatory shape, §9 Security Impact). Weakening one requires an RFC, the security Domain Lead's sign-off, and — for a non-negotiable invariant — is presumptively rejected.
- **Conformance.** Extension conformance suites assert the invariants an extension must uphold (secret-safety at every level, no undeclared effects, isolation) — SI-2/SI-6/SI-7 are tested, not trusted.

## 4. The one-sentence security model

If the whole Book reduces to a sentence: **an untrusted, non-deterministic edge produces only verified, secret-free, effect-declared IR; a small trusted core enforces capabilities, policy, and audit at every boundary; secrets exist as values only transiently at execution; and every security failure is fail-closed.** Everything in Book 11 is the elaboration and enforcement of that sentence.

## 5. Invariants (normative summary)

1. SI-1…SI-5 are the non-negotiable, fail-closed invariants (P1, P7, P8, P9, P10); weakening any is a security issue by definition.
2. SI-6…SI-12 are the supporting invariants (untrusted-by-default, isolation, least privilege, tenancy, approval, out-of-band credentials, fail-closed, defense in depth).
3. The security review gate checks every applicable invariant on every qualifying change; unmet invariants block the change.
4. Extension conformance suites test SI-2/SI-6/SI-7 rather than trusting them.
5. The model in one sentence: untrusted edge → verified secret-free effect-declared IR; trusted core enforces capabilities/policy/audit at every boundary; secrets are values only transiently at execution; every failure is fail-closed.
