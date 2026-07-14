# RFC-NNNN: <Title>

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Authors** | <name/handle> |
| **Domain / Book** | <e.g., Compiler / Book 05> |
| **Shepherd (Domain Lead)** | <handle> |
| **Created** | YYYY-MM-DD |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | <link> |

> Delete these italic hints as you write. Every section below is mandatory; mark a truly inapplicable section **"N/A — because …"**. Do not merge with blank or "TODO" sections.

## 1. Executive Summary
*One paragraph a busy maintainer can read to understand what this proposes and why it matters.*

## 2. Problem Statement
*What is broken, missing, or unclear today? Who is affected? What is the cost of doing nothing? Ground it in the layered architecture and the determinism mission.*

## 3. Alternatives Considered
*At least two, including "do nothing." For each: sketch, pros, cons, and why it was not chosen. Cite prior art (Linux/LLVM/K8s/Temporal/Git/…) where relevant.*

## 4. Proposed Design
*The heart of the RFC. Concepts, structures, protocols, sequence of operations. Include diagrams (link to `diagrams/src/`). Define every new term and add it to the Glossary. State normative requirements with MUST/SHOULD/MAY.*

## 5. Tradeoffs
*What we gain and what we give up. Be honest about the costs of the chosen design.*

## 6. API Changes
*Kernel API, extension APIs, wire formats. Show before/after signatures.*

## 7. Resource Changes
*New/changed ARM Resources: Metadata, Spec, Status, Desired/Actual State, Lifecycle, Controller.*

## 8. Event Changes
*New/changed Events on the Event Bus: name, schema, producers, consumers, ordering/delivery semantics.*

## 9. Security Impact
*Trust boundaries, capabilities, secret flows, policy implications. Confirm the invariants in SECURITY.md hold. Attach a security review for T3.*

## 10. Performance Impact
*Latency, throughput, memory, scaling. A cost model or benchmark plan. Attach a performance review for T3.*

## 11. Testing Strategy
*How correctness is verified: conformance, property-based, integration, failure-injection. Attach a testing plan for T3.*

## 12. Documentation Changes
*Which Books/chapters, glossary entries, and diagrams change.*

## 13. Migration Strategy
*How existing Resources/IR/packages move to the new world. Tooling, phases, dual-run windows.*

## 14. Risks
*What could go wrong, likelihood, and mitigation. Include "unknown unknowns" you are watching.*

## 15. Future Improvements
*Deliberately out of scope now; likely follow-ups. Prevents scope creep without losing ideas.*

---
### Resolved questions
*Moved here from review as they are settled, with the resolution.*

### Unresolved questions
*Open items that must be closed before FCP can conclude.*
