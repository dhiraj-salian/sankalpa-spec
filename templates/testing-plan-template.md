# Testing Plan: <Feature / RFC-NNNN>

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Covers** | RFC-NNNN / component |
| **Owner** | <handle> |

> Sankalpa's mission is *deterministic execution*. Testing is how we prove determinism, not merely absence of crashes. Non-deterministic components (planners/LLMs) are tested at their boundaries; everything downstream is tested for exact, repeatable behavior.

## 1. Test objectives & risk model
*What must be true for this to be correct? What are the highest-risk failure modes (from the RFC's Risks section)?*

## 2. Conformance tests
*Behavior mandated by the spec/AEP, expressed as executable assertions. For extension interfaces, this is the suite implementers must pass.*

## 3. Property-based & invariant tests
*Invariants that must hold for all inputs (e.g., "compiling then lowering preserves semantics", "no secret value appears in any Event/log"). State each property.*

## 4. Deterministic replay tests
*Given the same Intent → Goals → IR, execution must be reproducible. Golden-file/record-replay strategy for IR and execution graphs.*

## 5. Integration tests
*Cross-subsystem paths (e.g., Kernel ↔ Controller ↔ Event Bus). Which real vs. faked components.*

## 6. Failure-injection & chaos
*Faults injected (dropped events, controller crash, runtime timeout) and the expected recovery/reconciliation.*

## 7. Security tests
*Assertions tied to the invariants: secret non-leakage, capability enforcement, policy rejection of disallowed plans.*

## 8. Performance tests
*Benchmarks and their pass thresholds (link the performance review).*

## 9. Coverage & exit criteria
*What "sufficiently tested" means for this change. Gaps knowingly accepted, with rationale.*
