# Book 06 · Chapter 04 — Runtime Selection

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P9, P11 and IR-P7, IR-P10. Companion to Book 03 §Ch07 (Scheduler & Runtime Manager).*

> Runtime selection is the decision *where* a plan runs. This chapter specifies when it happens (after planning, on runtime-agnostic Low IR), the inputs it considers, and the guarantees it must uphold — chiefly that selection never compromises fidelity, policy, or determinism. The mechanism lives in the Runtime Manager (Book 03 §Ch07); this chapter fixes its *contract*.

## 1. Selection comes last, and why

The runtime is chosen **after** planning and (normally) after Low IR exists (RFC-0001, Book 03 §Ch07 §1). Nothing upstream names a runtime (IR-P7). This ordering is load-bearing:

- Planners stay runtime-ignorant (P1, P11) — a planner never encodes "use engine X."
- Selection can use information available **only late**: which runtimes are healthy *now*, current capacity and cost, and the policy in force for this workspace.
- The **same Low IR remains eligible for multiple runtimes** (Book 05 §Ch05 §4), so selection is a real choice among faithful options, not a foregone conclusion baked in at planning.

## 2. Inputs to selection

The Runtime Manager selects from registered `Runtime` Resources (Book 02 §Ch07) using:

1. **Requirements of the Low IR** — the Capabilities it invokes, the effects it declares (Book 04 §Ch06), and the `ExecPolicy` features it needs (durable retry, compensation, timeouts — Book 04 §Ch04 §4).
2. **Runtime capability reports** — each candidate's `describe`/`supports` (Book 06 §Ch02 §2): does it honor every instruction, effect, and policy in this module?
3. **Live signals** — health/readiness (Book 03 §Ch09), current capacity, and cost profile (fed by Experience, Book 10).
4. **Policy** — control-plane and workspace policy (Book 11 §06) that may pin, forbid, or prefer runtimes (e.g. "PII workloads only on the on-prem runtime").

## 3. The selection contract

Selection MUST satisfy all of the following. Violating any is a defect, not a tuning choice:

### 3.1 Fidelity gate (IR-P10) — mandatory, non-negotiable
A runtime is a **candidate only if** it can execute the module observably-equivalently: `supports(lowModule)` returns true for **every** instruction, effect, and `ExecPolicy` (Book 06 §Ch02 §2, Book 05 §Ch05 §3). A runtime that cannot honor, say, a required compensation (Book 04 §Ch04 §4) is **not** a candidate — the Runtime Manager MUST NOT select it and hope. If **no** runtime is a candidate, selection **fails** with a typed, explainable error (§4); the execution does not proceed on an unfaithful engine.

### 3.2 Policy gate (P9)
Among fidelity-valid candidates, any that policy **forbids** for this workspace/effect-class are excluded. A policy that *pins* a runtime narrows the set to that runtime (and if it is not a valid candidate, selection fails — policy never forces an unfaithful choice).

### 3.3 Explainable and auditable (P10)
Selection is a **pure function of declared requirements + live signals** and MUST be **explainable**: the chosen runtime and the reason (which requirements matched, which signals decided) are recorded and emitted as an Event. An operator can always answer "why did this run there?"

### 3.4 Cost/health optimization — only among valid candidates
*After* the fidelity and policy gates, the Runtime Manager MAY optimize among the remaining valid candidates using cost and health (Book 10 cost model). This is where efficiency lives — but it operates strictly on a set every member of which is already faithful and permitted. Efficiency never overrides correctness or governance.

## 4. Unsatisfiable selection fails cleanly

If no runtime satisfies the fidelity and policy gates, selection MUST fail with a typed, explainable error (Book 03 §Ch02 §4, Book 05 §Ch07-style diagnostic) — e.g. *"No available runtime supports required ExecPolicy `Compensate` for instruction i3; install or enable a durable runtime."* It MUST NOT:
- silently drop the unsupported behavior (would break IR-P10), or
- pick the "closest" runtime and execute unfaithfully.

Fail-closed (Book 03 §Ch13 §1): the safe outcome of "nowhere valid to run" is *do not run*, with an actionable explanation.

## 5. Rescheduling and runtime failure

- If a selected runtime becomes unhealthy before/during execution, the Runtime Manager MAY re-select **iff** the Low IR's idempotency/compensation contract makes re-execution safe (Book 03 §Ch07 §6, Book 06 §Ch03 §2/§4). Where re-execution is unsafe, the execution fails and its compensation runs — it is not silently retried elsewhere.
- Re-selection decisions are Events (P5) and feed Experience (Book 10) so future selection is health- and cost-aware.

## 6. Portability in practice

Because selection only ever picks a **faithful** runtime (§3.1), the portability guarantee (Book 05 §Ch05 §4) holds operationally: whichever valid runtime is chosen, the observable outcome is equivalent. A deployment can add a cheaper or more capable runtime and existing plans transparently benefit (better cost/health choices) *without any change to planners or IR* — the concrete payoff of runtime-agnostic IR (Book 01 §05).

## 7. Invariants (normative summary)

1. Runtime selection occurs after planning on runtime-agnostic Low IR; nothing upstream names a runtime (IR-P7).
2. A runtime is a candidate only if it can execute the module observably-equivalently (fidelity gate); if none can, selection fails cleanly (fail-closed).
3. Policy excludes or pins candidates; policy never forces an unfaithful selection.
4. Selection is an explainable, auditable function of declared requirements + live signals; the "why" is recorded and emitted.
5. Cost/health optimization operates only among already-faithful, already-permitted candidates; efficiency never overrides fidelity or policy.
6. Re-selection on runtime failure occurs only where re-execution is safe; otherwise the execution fails with compensation; all such decisions feed Experience.
