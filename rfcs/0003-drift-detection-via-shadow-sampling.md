# RFC-0003: Drift detection for determinized Capabilities via shadow sampling

| Field | Value |
|-------|-------|
| **Status** | Proposed |
| **Authors** | Dhiraj Salian (Phase 2 hardening review) |
| **Domain / Book** | Experience & Compiler / Books 10, 05 |
| **Shepherd (Domain Lead)** | Experience Domain Lead |
| **Created** | 2026-07-15 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | TBD |

> Raised by the Phase 2 hardening pass. Number 0003 reserved; open for review by the Experience Domain Lead and Reviewers. Depends on RFC-0002 (reasoning ledger) for its data substrate — do not advance to Accepted ahead of 0002.

## 1. Executive Summary
Determinization is declared "safe to attempt" because it is reversible: a determinized `Capability` that drifts is retired and compilation falls back to reasoning ([Book 10 §Ch06 §5](../spec/book-10-experience/06-determinization-engine.md), [Book 05 §Ch06 §4](../spec/book-05-compiler/06-determinization-passes.md)). Drift is defined as the Capability's outputs "diverging **from fresh reasoning**." But once the compiler substitutes the `Reasoning` node with a `CapabilityInvocation` ([Book 05 §Ch06 §2](../spec/book-05-compiler/06-determinization-passes.md)), the model is *no longer called* — so no fresh reasoning is produced to diverge from. The safety net has no data source, and the reversibility guarantee is inert as specified. This RFC introduces **shadow sampling**: a policy-governed fraction of executions that still run the original reasoning alongside the substituted Capability, compare outputs, and feed drift detection — turning reversibility from an assertion into a mechanism.

## 2. Problem Statement
The entire safety argument for determinization ([Book 01 §Ch05 §3](../spec/book-01-foundations/05-determinism-and-determinization.md): "the worst case is a return to reasoning, never a silently-wrong deterministic answer") rests on retire-on-drift. Retire-on-drift rests on observing divergence between the deterministic Capability and current reasoning. After successful substitution the reasoning is gone from the executed plan, so:
- There is **no fresh reasoning output** to compare the Capability against; the world can change and the Capability keeps returning its stale answer undetected.
- [Book 10 §Ch06 §5](../spec/book-10-experience/06-determinization-engine.md) says the engine "continuously watches new Experience for drift", but new Experience for a fully-substituted node contains only the Capability's own outputs — a closed loop that can never surface divergence.

Who is affected: every workspace relying on determinization for correctness. Cost of doing nothing: determinization becomes a one-way ratchet toward silently-wrong deterministic answers — the exact outcome the spec claims is impossible.

## 3. Alternatives Considered
- **Do nothing.** Rejected: leaves the flagship safety property unenforceable.
- **Retire on a fixed TTL / periodic forced re-evaluation.** Partial: bounds staleness but is blunt (retires healthy Capabilities on a timer, wasting the determinization gain) and still needs a comparison run to decide. Kept as a *complementary* coarse backstop, not the primary mechanism.
- **Retire only on downstream failure signals (error rate).** Insufficient: many drifts produce a plausible-but-wrong output that never raises an error (misclassification, stale routing) — no failure signal is emitted.
- **Shadow sampling (chosen).** Run the original reasoning on a sampled fraction of executions in parallel with the substituted Capability, compare via the content-addressed reasoning ledger (RFC-0002), and drive retirement from measured divergence. Mirrors canary/shadow-traffic practice in production ML serving.

## 4. Proposed Design
**4.1 Shadow-eligible substitution (normative).** When the determinization pass substitutes a `Reasoning` node ([Book 05 §Ch06 §2](../spec/book-05-compiler/06-determinization-passes.md)), it **MUST** mark the resulting `CapabilityInvocation` with the identity of the reasoning it replaced (`shadowSource`), so the runtime can reconstruct the original reasoning call.

**4.2 Sampling decision (normative, policy-governed).** For each execution of a shadow-marked instruction the runtime consults a **shadow-sampling policy** ([Book 11 §Ch06](../spec/book-11-security/06-policy-engine.md)) yielding a sampling rate per Capability/effect-class. On a sampled execution the runtime:
1. Executes the deterministic Capability (the authoritative result used by the plan).
2. **Additionally** runs the original reasoning as a side computation whose output is recorded to the reasoning ledger (RFC-0002) but **MUST NOT** affect the plan's observable behavior (no external effects beyond `Reason`; the shadow run's non-`Reason` effects, if any, are suppressed or it is disqualified from shadowing).
3. Emits a `determinization.shadow_observed` Event with both output hashes and the shadow source's version (§4.5).

The shadow run is **non-blocking**: the execution completes without awaiting it, so sampled executions retain determinized latency. Its reasoning `Cost` is attributed to a workspace-level drift-monitoring budget, **not** the plan's declared cost — sampling **MUST NOT** cause a plan to exceed its own cost bounds or alter its cost accounting.

**4.3 Divergence and retirement (normative).** The determinization engine ([Book 10 §Ch06 §5](../spec/book-10-experience/06-determinization-engine.md)) aggregates shadow observations. Divergence **MUST** be computed under the *same* canonicalization/equivalence relation used at synthesis ([Book 10 §Ch06 §3](../spec/book-10-experience/06-determinization-engine.md)) — raw hash inequality over a non-deterministic process has a nonzero baseline from model variance alone, and a stricter relation than synthesis used would retire healthy Capabilities on noise. When measured divergence over a policy window exceeds the stability bound used at synthesis, the engine **MUST** retire the Capability via the Capability Manager's prompt-and-complete revocation ([Book 03 §Ch06 §7](../spec/book-03-kernel/06-capability-manager.md)); future compilations then fail the freshness gate ([Book 05 §Ch06 §3.4](../spec/book-05-compiler/06-determinization-passes.md)) and fall back to reasoning.

**4.4 Cost and confidence coupling.** Sampling rate SHOULD scale inversely with accrued confidence and with the Capability's `Cost`/consequence class: high-consequence determinizations are shadowed more; long-stable low-consequence ones decay toward a floor rate. For effect classes designated high-consequence, the minimum floor is a **hard normative floor** (MUST), not a policy default. A rate of zero is permitted only for effect classes a workspace explicitly designates as not requiring drift monitoring (accepting the staleness risk, audited). Shadow-ineligible reasoning (non-suppressible effects, §3) **MUST** instead be covered by TTL-based forced re-evaluation — no determinized Capability may be entirely unmonitored.

**4.5 Version-bucketed observations (normative).** Every shadow observation records the shadow source's version (model, prompt/config, planner revision). Retirement decisions **MUST** aggregate observations only within a single version bucket. When the shadow source's version changes — e.g. a planner model upgrade — cross-version divergence **MUST NOT** trigger retirement; instead the engine opens a **re-validation window** in which confidence re-accrues against the new version, retiring only if divergence *within the new bucket* exceeds the bound over a full policy window. Without this, a routine model upgrade could mass-retire healthy Capabilities in one window and silently destroy the workspace's accrued determinization asset.

## 5. Tradeoffs
**Gain:** reversibility becomes a real, measured mechanism; silent drift is bounded by the sampling rate.
**Give up:** shadow runs re-incur reasoning `Cost` on the sampled fraction, partially eroding determinization's savings; runtimes must support a side-effect-suppressed shadow execution path.

## 6. API Changes
- Low IR: `CapabilityInvocation` gains optional `shadowSource` (additive). — [Book 04 §Ch04](../spec/book-04-aos-ir/04-low-ir.md).
- Runtime interface: a shadow-execution capability that runs a reasoning node with non-`Reason` effects suppressed — [Book 06 §Ch02](../spec/book-06-runtimes/02-runtime-interface.md).

## 7. Resource Changes
`Capability` (determinized) status gains drift telemetry (sample count, divergence rate, last-shadow time) — additive, feeds retirement. No new kind.

## 8. Event Changes
Additive: `determinization.shadow_observed` (both output hashes plus shadow-source version, secret-free) and `determinization.retired` (already implied by [Book 10 §Ch06 §7](../spec/book-10-experience/06-determinization-engine.md); made explicit). Per-subject ordering on the Capability subject suffices.

## 9. Security Impact
The shadow reasoning run **MUST** obey the same secret-freedom and no-secret-in-reasoning rules ([Book 08 §Ch04](../spec/book-08-planners/04-planner-isolation-and-secrets.md), P7) as any reasoning. Because the shadow output is discarded from the plan, it **MUST NOT** be able to introduce effects into the execution (§4.2). Determinization remains tenant-scoped ([Book 10 §Ch06 §7](../spec/book-10-experience/06-determinization-engine.md)); shadow observations are workspace-scoped. Security review required (drift monitoring is part of the P13/P1 safety argument).

## 10. Performance Impact
Adds reasoning cost on the sampled fraction only; at a 1% floor the erosion of determinization savings is ~1%. The engine's aggregation is off the hot path (mines Experience). A cost model comparing sampling rate vs. staleness-risk is required for T3.

## 11. Testing Strategy
- Failure-injection: force a determinized Capability to drift (swap its backing data) and assert the engine retires it within the policy window at the configured sampling rate.
- Property: with sampling rate `r` and true divergence `d`, expected detection latency is bounded — verify empirically.
- Conformance: shadow runs never alter observable plan behavior.

## 12. Documentation Changes
Extend [Book 10 §Ch06 §5](../spec/book-10-experience/06-determinization-engine.md) with the shadow-sampling mechanism (replacing the under-specified "diverging from fresh reasoning" language); note `shadowSource` in [Book 05 §Ch06](../spec/book-05-compiler/06-determinization-passes.md) and [Book 04 §Ch04](../spec/book-04-aos-ir/04-low-ir.md). Glossary: *Shadow sampling*, *Drift*, *Shadow source*, *Version bucket*, *Re-validation window*.

## 13. Migration Strategy
Additive and pre-1.0. Existing determinized Capabilities gain a default sampling rate on next compilation; none are invalidated.

## 14. Risks
- **Cost erosion** if sampling rate is set too high — mitigated by confidence-decaying rates (§4.4).
- **Retirement storm on model upgrade** — a new planner model diverging stylistically from every synthesized Capability — mitigated by version-bucketed aggregation and re-validation windows (§4.5).
- **Shadow effect leakage** if a reasoning node's non-`Reason` effects aren't fully suppressible — mitigated by disqualifying such nodes from shadowing (they fall back to mandatory TTL re-evaluation, §4.4).
- **Unknown unknown:** reasoning that is non-deterministic *and* drift-prone but rarely sampled could still slip; watched via minimum-rate floors per consequence class.

## 15. Future Improvements
Adaptive sampling driven by observed variance; coordinated re-evaluation batching to amortize model cost; using downstream Experience outcomes (not just output divergence) as a second drift signal.

---
### Resolved questions
- **Sampling floor: policy default vs. hard normative floor?** Hard normative floor (MUST) for designated high-consequence effect classes; policy default elsewhere (§4.4).
- **Is TTL forced re-evaluation required for shadow-ineligible reasoning?** Yes — required (§4.4); otherwise effectful reasoning sits in exactly the unmonitored hole this RFC exists to close.
- **What happens on a shadow-source model upgrade?** Re-validation, never direct retirement: observations are version-bucketed and retirement uses only within-bucket divergence (§4.5).

### Unresolved questions
- Concrete numeric defaults for floors, policy windows, and re-validation window length (needs the T3 cost model, §10).
