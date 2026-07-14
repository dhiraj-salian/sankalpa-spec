# Performance Review: <Feature / RFC-NNNN>

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Reviewer** | <handle> |
| **Subject** | RFC-NNNN / component |

## 1. Performance-relevant behavior
*What operations does this change add or alter on the hot path? Where does it sit relative to planning (rare, expensive) vs. execution (frequent, must be cheap)?*

## 2. Cost model
*Analytical model of time/space cost as a function of the inputs that matter (number of Resources, IR nodes, events/sec, concurrent executions). Big-O and constants that matter.*

| Operation | Time | Space | Scales with |
|-----------|------|-------|-------------|
| … | | | |

## 3. Targets & budgets
*Explicit budgets: p50/p99 latency, throughput, memory ceiling, event-bus load. Tie to user-visible experience.*

## 4. Benchmarks
*What will be measured, how, and the pass thresholds. Reference the testing plan's performance section.*

## 5. Scaling & backpressure
*Behavior under load: what saturates first, how backpressure propagates, degradation mode (fail-fast vs. shed vs. queue).*

## 6. Caching & determinization opportunities
*Where results can be cached or repeated reasoning converted to deterministic Capabilities (the core mission lever).*

## 7. Risks & recommendation
*Performance risks and verdict: Approve / Approve-with-conditions / Block.*
