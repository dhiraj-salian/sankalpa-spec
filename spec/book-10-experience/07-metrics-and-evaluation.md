# Book 10 · Chapter 07 — Metrics and Evaluation

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P7, P10, P13. Companion to Book 08 §02 (Goal success criteria), Book 13 (channels/feedback), Book 14 (metrics).*

> Learning requires ground truth: to improve, the system must know *whether an execution actually succeeded* — not merely whether it ran. This chapter specifies evaluation (actual outcome vs. Goal success criteria), human feedback capture, and how experience quality is measured. It is what makes the feedback loops (§Ch04–§Ch06) trustworthy rather than self-reinforcing.

## 1. Ran ≠ succeeded

A common failure of learning systems is conflating *completion* with *success*: a run that "finished without error" may still have produced the wrong result. Sankalpa avoids this by evaluating every Experience against the **explicit success criteria** the Goals carried (Book 08 §02 §3). Because Goals are required to state checkable success criteria, "did it work?" is a *defined question with a defined answer* — not a guess. An execution that succeeded technically but failed its Goal criteria is recorded as a **failure of outcome**, and the feedback loops treat it as such.

## 2. Success evaluation

For each Experience, the Experience Manager produces a **`SuccessEvaluation`** (Book 10 §02 §1):
- It compares the execution's actual results (from the secret-free Event/outcome record, §Ch03) against each Goal's success criteria.
- The result is structured: which criteria were met, which were not, and the evidence (by reference) for each.
- Evaluation is **secret-free** (P7): it reasons over outcomes and references, never secret values.

Where success criteria are objective (a file produced, a record created with expected fields), evaluation is deterministic. Where criteria are inherently judgemental (a summary is "good"), evaluation MAY require a `Reasoning`-style judgement or human feedback (§3) — and, notably, evaluation *itself* can become a determinization candidate (a repeatedly-applied quality judgement can be folded into a deterministic evaluator, §Ch06).

## 3. Human feedback

Human feedback is a first-class evaluation input (Book 10 §02 §1):
- **Explicit signals** — approval/rejection, ratings, corrections captured through channels/web (Book 13) and attributed to the providing identity (P10, Book 11 §08).
- **Implicit signals** — a human re-running, editing, or discarding a result is itself a signal, captured as Events (P5).
- Human feedback is **weighted and provenanced** (Book 09 §06): a correction from an authorized human is strong ground truth and MAY override an automated evaluation, with the override recorded.

Feedback closes the gap where machine evaluation is insufficient — and, over time, teaches the system what "good" means for a workspace (feeding Knowledge, §Ch04, and evaluation-determinization, §2).

## 4. What is measured

Experience quality and system health are measured across dimensions, all fed by metrics (Book 10 §02 §3) and evaluations, surfaced through observability (Book 14 §02):
- **Outcome success rate** — per Goal class, Capability, planner, and runtime (did outcomes meet criteria?).
- **Efficiency** — latency, cost, retries, resource use (the cost-model inputs, Book 05 §Ch03 §3).
- **Determinization health** — how much reasoning has been determinized, and the error/drift rate of determinized Capabilities (the safety signal for §Ch06 §5).
- **Failure taxonomy** — recurring failure modes by stage/diagnostic (Book 05 §Ch07 §6), driving anti-pattern Knowledge (§Ch04 §2).

These measures are the platform's self-knowledge: they tell operators and the feedback loops where the system is doing well, where it is wasteful, and where it is drifting.

## 5. Evaluation guards the feedback loops

Evaluation is not just reporting — it is the **integrity check** on learning:
- **Determinization** (§Ch06) synthesizes only from Experiences whose outcomes were *evaluated successful* with stable quality; folding in a reasoning that "ran" but produced poor outcomes would bake a bad answer into a Capability. Evaluation is what makes the determinization evidence trustworthy.
- **Knowledge promotion** (§Ch04) weights lessons by evaluated success; an approach that completed but failed its criteria becomes an *anti-pattern*, not a recommended pattern.
- **Cost feedback** (§Ch05) is tied to evaluated outcomes so the cost model does not optimize toward cheap-but-wrong executions.

Without rigorous evaluation, the feedback loops would risk reinforcing whatever *ran*, not what *worked* — a self-deceiving system. Evaluation is the ground truth that keeps learning honest.

## 6. Secrecy and attribution (P7, P10)

- Evaluation and metrics are **secret-free** (inherited from the Event stream, §Ch03 §4): a `SuccessEvaluation` references outcomes and criteria, never secret values.
- Every evaluation and feedback item is **attributed** (P10): automated evaluations to the Experience Manager, human feedback to the providing identity — so the basis of every learning decision is accountable (Book 11 §09).

## 7. Invariants (normative summary)

1. Every Experience is evaluated against its Goals' explicit success criteria; completion is never treated as success; an outcome-failure is recorded as failure even if the run finished cleanly.
2. `SuccessEvaluation` is structured, evidence-referenced, and secret-free; objective criteria evaluate deterministically, and repeated judgemental evaluation can itself be determinized.
3. Human feedback (explicit and implicit) is first-class, weighted, provenanced, and may override automated evaluation with the override recorded.
4. The platform measures outcome success, efficiency, determinization health/drift, and failure taxonomy, surfaced through observability.
5. Evaluation guards the feedback loops: determinization, Knowledge promotion, and cost feedback all key on *evaluated success*, so learning reinforces what worked, not merely what ran.
6. Evaluation and feedback are secret-free and fully attributed (P7, P10).
