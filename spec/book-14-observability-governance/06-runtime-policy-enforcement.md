# Book 14 · Chapter 06 — Runtime Policy Enforcement

*Nature: **Normative**. · Reflects: RFC-0001, ADR-0002; realizes principles P9 (and P7, P8). Companion to Book 11 §06 (Policy Engine), Book 05 §Ch04 (compile-time pass), Book 11 §07 (Approval).*

> This chapter specifies the **third and final policy checkpoint**: enforcement immediately before execution, against *live* conditions. It is the last gate before an irreversible effect, and it exists because some governance facts are knowable only at execution — current budget, a live approval decision, runtime health, time-of-day. Together with the control-plane and compile-time checkpoints (Book 11 §06 §2), it completes P9's defense in depth.

## 1. Why a runtime checkpoint is necessary

The compile-time policy pass (Book 05 §Ch04) validates a plan's declared effects on the human-meaningful High IR — but it is **static**: it cannot see what will be true at execution time. Several governance conditions are inherently dynamic:
- **Budget/quota** — is the workspace within its `Cost` budget *now* (Book 04 §Ch06)?
- **Approval state** — has the required human `Approval` actually been granted, and is it still valid (Book 11 §07)?
- **Runtime health/context** — is the selected runtime healthy; is it the right time/place for this effect?
- **Revocation** — has a capability or grant been revoked since compilation (Book 11 §03 §5)?

A plan that was compliant at compile time may be non-compliant at execution time. The runtime checkpoint re-evaluates policy against these live facts **immediately before execution**, so governance reflects reality, not a stale snapshot.

## 2. Position: the last gate before effect

```
control-plane policy (Kernel API, Book 11 §06 §2)     — governs operations
   → compile-time policy pass (Book 05 §Ch04)          — governs the plan, statically, pre-lowering
      → RUNTIME CHECKPOINT (this chapter)              — governs the action, live, pre-execution
         → effect happens (Book 06 §Ch03)
```

The runtime checkpoint sits between a fully-compiled, runtime-selected plan and its actual execution (Book 06 §Ch03). It is the **last** point at which an effect can be stopped before it happens — which is exactly why it must re-check the conditions that only-now are knowable.

## 3. What the checkpoint enforces

Immediately before executing an instruction with a governed effect, the runtime checkpoint (invoked by the Runtime Manager / Kernel, consulting the Policy Engine, Book 11 §06) verifies against live state:
- **Budget/quota** — the `Cost` effect is within the current, live budget.
- **Approval** — any required `Approval` is present, affirmative, and unexpired (Book 11 §07 §6); a missing/denied/expired approval **stops** the action.
- **Capability validity** — the authorizing grant has not been revoked since compilation (Book 11 §03 §5).
- **Secret materialization conditions** — the policy/approval conditions the Secret Broker requires are satisfied *now* (Book 11 §04 §4), so a secret is materialized only under live-valid conditions.
- **Contextual policy** — any time/place/health conditions the policy attaches.

## 4. Fail-closed (P9, Book 03 §Ch13)

The checkpoint is **fail-closed**, like every security mechanism (Book 03 §Ch13 §1, Book 11 §01 §6):
- If policy cannot be evaluated, a required approval is absent, a grant was revoked, or a budget is exceeded — the action is **refused**, not admitted. The execution reaches an explained terminal state (Book 06 §Ch03 §4), and any completed non-idempotent effects are compensated (Book 04 §Ch04 §4).
- "We could not confirm this is allowed *right now*" is treated as "not allowed." Governance never proceeds on stale or unverifiable permission at the very last gate.

## 5. Defense in depth, completed

The three checkpoints are deliberately redundant (Book 11 §06 §2), each covering the others' blind spots:
- The **control-plane** checkpoint governs *operations* (who may do what to Resources) but cannot see plan internals.
- The **compile-time** pass governs the *plan's declared effects* on a clean artifact but cannot see live state.
- The **runtime** checkpoint governs the *actual action against live state* but operates on an already-validated plan (it re-checks conditions, it is not the place to first discover a plan is fundamentally forbidden).

No single checkpoint is load-bearing alone (Book 11 §01 §6). A plan must pass all applicable ones; any can stop it; the runtime checkpoint is the final, live-state backstop before the world is changed.

## 6. Secret-free and attributed (P7, P10)

- The checkpoint reasons over live facts — budget numbers, approval decisions (by reference), grant validity, health — **never over secret values** (P7). It governs secret *use* by class/reference (Book 11 §04 §6), consistent with the Policy Engine (Book 11 §06 §3).
- Every checkpoint decision emits a `policy.evaluated`/`policy.denied` Event (§Ch01) and is audited (Book 11 §09): the live-state governance of every effect is accountable.

## 7. Invariants (normative summary)

1. The runtime checkpoint is the third policy checkpoint: enforcement against live conditions immediately before execution, the last gate before an irreversible effect (P9).
2. It re-verifies budget/quota, approval presence/validity, capability non-revocation, secret-materialization conditions, and contextual policy against current state.
3. It is fail-closed: un-evaluable policy, missing/denied/expired approval, revoked grant, or exceeded budget refuses the action, which reaches an explained terminal state with compensation.
4. It completes defense in depth with the control-plane and compile-time checkpoints; no checkpoint is load-bearing alone, and any can stop an action.
5. It reasons over live non-secret facts and secret *use* by reference — never secret values — and every decision is Evented and audited (P7, P10).
