# Security Review: <Feature / RFC-NNNN>

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Reviewer (Security Domain Lead)** | <handle> |
| **Subject** | RFC-NNNN / component |

> Passing the security gate ([`../process/review-gates.md`](../process/review-gates.md)) requires this review for T3 changes and any change touching trust boundaries, secrets, capabilities, or policy.

## 1. Assets & trust boundaries
*What is being protected (secrets, user data, execution authority)? Draw/link the trust-boundary diagram. What crosses each boundary?*

## 2. Invariant checklist
Confirm each holds, or explain the deviation and its compensating control:

- [ ] Natural language is never executable; only AOS IR executes.
- [ ] Secrets never enter planner context, prompts, logs, memory, vector stores, or IR.
- [ ] Secrets flow **by reference** via the Secret Broker only.
- [ ] No ambient authority; access is via granted capabilities.
- [ ] Policy is enforced before compilation **and** before execution.
- [ ] Every privileged action is attributable via Events/Experience.

## 3. Threat model (STRIDE)
*Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege — for the components this change introduces or alters.*

## 4. Capability analysis
*Exactly which capabilities are granted to which components/extensions, and why each is least-privilege.*

## 5. Secret handling
*Every point a secret is acquired, referenced, or used at execution. Confirm no value ever enters a reasoning/log/storage path.*

## 6. Abuse & failure cases
*What an attacker (or a buggy extension) could attempt, and how the design contains it.*

## 7. Residual risk & recommendation
*What risk remains after mitigations. Verdict: Approve / Approve-with-conditions / Block.*
