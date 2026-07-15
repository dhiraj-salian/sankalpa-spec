# Book 15 · Chapter 01 — End-to-End Walkthrough

*Nature: **Informative** (the capstone; illustrates every Book working together). · Where this and a normative Book differ, the Book wins.*

> This is the specification's capstone: one intent followed the entire way — Intent → Goals → Planning → High IR → Optimize → Policy → Low IR → Runtime selection → Execution → Events → Experience → Knowledge — naming the artifact and the responsible Book at every step. It demonstrates that the Books compose into one coherent system. Read it after Books 01–14 to see the whole in motion; read it first for a map of what those Books detail.

## The scenario

A user types into Slack: *"Every Monday, summarize last week's sales and email it to the team."* We follow it to executed, governed, reproducible action — and back into durable learning.

## 1. Ingress — the intent arrives (Book 13, Book 11)

The Slack **channel** (Book 13 §Ch02) *carries* the message; it does not interpret it. At the ingress boundary the Kernel API authenticates it to a **Session** (Book 11 §08, Book 13 §Ch05) and admits it. An **`Intent`** Resource (Book 02 §Ch07) is created: `utterance = "Every Monday, …"`, `channel = slack://…`, `submittedBy = user`. Natural language is captured but **not executable** (P1). An `resource.intent.created` Event is emitted (Book 03 §Ch03, Book 14 §Ch01).

## 2. Goal derivation — what is wanted (Book 08)

The Intent controller (Book 07) reconciles the Intent by invoking a **Planner** (Book 08 §Ch03). The planner derives **`Goal`** Resources with explicit **success criteria** (Book 08 §Ch02): *G1 — obtain last week's sales data; G2 — produce and email a summary to the team.* The recurring "every Monday" is a property of the long-lived Intent that spawns per-run Goals (Book 02 §Ch07). The planner retrieves non-secret, provenanced **Knowledge** (Book 09 §Ch06) — who "the team" is, prior report conventions — as context; it receives **no secrets** (Book 08 §Ch04, P7).

## 3. Planning — the plan as High IR (Book 08, Book 04)

The planner emits a **High IR** module (Book 04 §Ch03) — a typed dataflow graph:
- `n1 = CapabilityInvocation crm.sales.query` → `effects: Network(read,"crm"), SecretUse(crm-token), Cost`
- `n2 = Reasoning "write an executive summary"` → `effects: Reason` (the *only* non-determinism, Book 08 §Ch05; receives no secret)
- `n3 = CapabilityInvocation email.send` → `effects: Network(write,"email"), SecretUse(smtp-cred), StateWrite`

Secrets appear only as `SecretRef` (Book 04 §Ch05 §2.2). The planner's output is **verified** (Book 04 §Ch08): well-formed, typed, fully effect-annotated. A bad plan would fail here and stop — non-determinism is halted at the planner boundary (Book 08 §Ch01 §3).

## 4. Compilation — optimize, govern, lower (Book 05, Book 11)

The **Compiler Manager** (Book 03 §Ch08) creates a **`Compilation`** Resource (Book 05 §Ch08) and runs the pipeline (Book 05 §Ch01):
- **Verify** the High IR (Book 04 §Ch08). **Optimize** (Book 05 §Ch03) — semantics-preserving.
- **Policy validation** (Book 05 §Ch04, P9) on the human-meaningful High IR: policy `P-email-approval` says external email requires an **Approval** — so the pass *annotates* that a guard must be inserted (allow-with-guard), and confirms `Cost` is within budget. (A policy *forbidding* external email would **deny** here with an explainable diagnostic.)
- **Determinization** (Book 05 §Ch06): `n2` is volatile, not folded; had it been stable-and-repeated, it would become a deterministic Capability (P13).
- **Lower** to **Low IR** (Book 04 §Ch04): fix the schedule, attach `ExecPolicy` (the email `IdempotentWithKey` + `Compensate(email.retract)`; the reasoning gets **no retry** — non-idempotent, Book 04 §Ch08 §2.5), materialize the **approval instruction** from the policy annotation, and turn `SecretRef`s into execution-time **binding sites** (Book 04 §Ch04 §5). Low IR is **re-verified**. Still **no runtime named** (IR-P7).

## 5. Runtime selection & lowering (Book 06, Book 03)

Only now is a **Runtime** chosen (Book 06 §Ch04) — *after* planning. The **fidelity gate** requires a runtime that can honor the `Compensate` ExecPolicy; a durable-workflow-class runtime qualifies (Book 06 §Ch05). The **policy gate** confirms it is permitted; cost/health pick among valid candidates. The runtime's backend **lowers Low IR → RuntimeGraph** (Book 05 §Ch05), conformance-checked (Book 06 §Ch07). An **`Execution`** Resource (Book 02 §Ch07) is scheduled (Book 03 §Ch07).

## 6. Execution — governed, reproducible effects (Book 06, Book 14, Book 11)

Execution runs (Book 06 §Ch03):
- `i1` queries the CRM: at the `SecretUse` point, the runtime presents the binding token; the **Secret Broker** materializes `crm-token` **only now, only into the runtime**, under capability + policy (Book 11 §04, Book 06 §Ch06). The value never entered IR, planning, logs, or Events (P7).
- The **runtime policy checkpoint** (Book 14 §06) re-checks live state before the email: it reaches the **approval** instruction → the **Approval Engine** (Book 11 §07) renders a Web Runtime page (Book 13 §Ch06) showing the concrete consequence (sending to the team, cost) — no secret shown. The user approves; the decision is non-repudiable and audited (Book 11 §09).
- `i3` sends the email (materializing `smtp-cred` at that instant), keyed-idempotent with compensation ready. The `Execution` reaches `Succeeded`. Every step emitted secret-free **Events** (Book 03 §Ch03, Book 14 §Ch01), correlated into one **trace** (Book 14 §Ch03).

## 7. Experience & Knowledge — the loop closes (Book 10, Book 09)

The **Experience Manager** (Book 10 §Ch03) assembles an **`Experience`** from the Event stream (secret-free, Book 10 §Ch02): the intent, goals+criteria, IR hashes, runtime, metrics, reasoning trace, and outcome. It is **evaluated** against G1/G2's success criteria (Book 10 §Ch07 — *ran ≠ succeeded*). Metrics feed the cost model (Book 05 §Ch03). Lessons are promoted to **Knowledge** with provenance (Book 09 §Ch04) — improving next Monday's planning. If this summary reasoning proves stable over many weeks, it becomes a determinization candidate (Book 10 §Ch06), and a future run's `n2` folds into a deterministic Capability — **the boundary recedes** (P13, Book 01 §Ch05).

## 8. What the walkthrough demonstrates

In one intent, every principle appeared and every Book contributed:
- **P1** — natural language captured, never executed; only IR ran.
- **P2/P3/P5** — Intent, Goal, Compilation, Execution, Experience were all Resources, reconciled, emitting Events.
- **P7** — secrets were references throughout, values only transiently in the runtime.
- **P8** — every action capability-gated; **P9** — policy at compile-time and runtime, approval enforced.
- **P10** — the whole chain traceable and audited.
- **P11** — a Slack channel, a planner, a runtime, all replaceable plugins.
- **P13** — the execution became Experience became Knowledge, and a candidate for determinization.

The next Monday runs cheaper and better-informed than this one — because the loop closed. That is Sankalpa: **human intent → deterministic execution, evolving toward more determinism over time** (Book 01 §Ch01, §Ch05).
