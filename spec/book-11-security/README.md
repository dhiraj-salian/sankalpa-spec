# Book 11 — Security

*Status: Draft skeleton · Nature: Normative. · Reflects: SECURITY.md; realizes P1, P7, P8, P9. This Book is load-bearing across the whole system.*

## Scope
The complete security architecture: trust boundaries, the capability-based authority model, the Secret Broker, credential acquisition, the Policy Engine, the Approval Engine, identity/sessions, and audit. Every mechanism here exists to make a foundational principle mechanically true.

## Chapters
1. **`01-threat-model.md`** *(Normative)* — Assets, adversaries, and the top risk: leakage through reasoning context. STRIDE per subsystem.
2. **`02-trust-boundaries.md`** *(Normative)* — Every boundary in the system and what crosses it; the untrusted-plugin boundary.
3. **`03-capability-based-security.md`** *(Normative)* — No ambient authority (P8); capabilities as unforgeable references; granting, attenuation, and revocation.
4. **`04-secret-broker.md`** *(Normative)* — The custodian of secrets: reference issuance, resolution at execution only, and the guarantee that values never enter planner context, prompts, logs, memory, vector stores, or IR (P7).
5. **`05-credential-acquisition.md`** *(Normative)* — Secure one-time HTTPS pages; OAuth preferred over passwords; storage and rotation.
6. **`06-policy-engine.md`** *(Normative)* — Machine-checkable policy; validation before compilation and before execution (P9); policy as Resources and Packages.
7. **`07-approval-engine.md`** *(Normative)* — Human authorization gates; approval pages (Book 13); non-repudiation.
8. **`08-identity-users-sessions.md`** *(Normative)* — User Manager, Session Manager, authn/authz, and multi-tenant isolation.
9. **`09-audit-and-attribution.md`** *(Normative)* — Every privileged action attributable via Events/Experience; tamper-evidence.
10. **`10-plugin-isolation.md`** *(Normative)* — Sandboxing planners/runtimes/compilers; resource limits; blast-radius containment.
11. **`11-security-invariants.md`** *(Normative)* — The consolidated invariant list all designs are checked against (the security gate's checklist).
