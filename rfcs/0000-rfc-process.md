# RFC-0000: The RFC process

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **Authors** | Founding maintainers |
| **Domain / Book** | Governance / Book 00 |
| **Shepherd** | Steering Council |
| **Created** | 2026-07-14 |

## 1. Executive Summary
This meta-RFC bootstraps the RFC process itself: it establishes that substantial designs in Sankalpa are proposed, reviewed, and accepted as numbered RFCs, and it points to the authoritative process definition in [`../process/rfc-process.md`](../process/rfc-process.md).

## 2. Problem Statement
A specification-first project (ADR-0001) needs a disciplined way to make substantial, cross-cutting design decisions in the open, with durable rationale and a clear consensus bar. Ad-hoc PRs cannot carry that weight: they lack a defined review, a comment period, and a status lifecycle.

## 3. Alternatives Considered
- **PR-only design.** Rejected: no comment period, no status, poor discoverability of design history.
- **Issue-thread design.** Rejected: discussion is ephemeral and unversioned; no canonical accepted artifact.
- **Adopt PEP/KEP/Rust-RFC wholesale.** Adopted *in spirit*; adapted to a spec-first project where an accepted RFC must be *reflected into the normative spec* to become Final.

## 4. Proposed Design
The full mechanism lives in [`../process/rfc-process.md`](../process/rfc-process.md): when an RFC is required, the status lifecycle (`Draft → Proposed → Accepted → Final`, plus `Deferred/Rejected/Withdrawn/Superseded`), the 10-working-day FCP, the consensus test, numbering, and the requirement that acceptance be followed by reflection into `spec/`. RFCs use [`../templates/rfc-template.md`](../templates/rfc-template.md) and clear the [review gates](../process/review-gates.md).

## 5. Tradeoffs
More process than a raw PR, in exchange for durable, reviewable, discoverable architecture. We accept slower merges for substantial designs to protect conceptual integrity.

## 6. API Changes
None (process only).

## 7. Resource Changes
None.

## 8. Event Changes
None.

## 9. Security Impact
Positive: forces every substantial design through the security gate before acceptance.

## 10. Performance Impact
None on the system; adds human latency to design acceptance, deliberately.

## 11. Testing Strategy
CI enforces the mechanical parts (template completeness, numbering, link integrity). Human review enforces the rest.

## 12. Documentation Changes
Establishes `rfcs/` and its README; referenced from GOVERNANCE and CONTRIBUTING.

## 13. Migration Strategy
N/A — this is the initial process.

## 14. Risks
Process-heaviness deterring contributors (mitigated by ADRs for smaller decisions and editorial fast-paths); FCP gaming (mitigated by requiring blocking objections to cite concrete harm).

## 15. Future Improvements
Automated FCP bots, RFC dashboards, and metrics on time-to-decision once the community scales.
