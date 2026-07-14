# Book 10 · Chapter 02 — Experience Anatomy

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P2, P6, P7, P10. Companion to Book 02 §Ch07 (Experience Resource), Book 04 §Ch07 (content addressing).*

> This chapter defines what an `Experience` contains. Its shape is chosen so the three feedback paths (Knowledge, planning, determinization) can consume it deterministically, and so the whole intent→execution→outcome story is reconstructable from one Resource — by reference, without any secret.

## 1. The anatomy

An `Experience` (Book 02 §Ch07) records one Execution's full lifecycle and outcome:

```
Experience:
  spec:                          # (system-produced; assembled by the Experience Manager, Ch 03)
    intentRef        Ref<Intent>         # the originating intent
    goalRefs         list<Ref<Goal>>     # goals + their success criteria (Book 08 §02)
    plannerRef       Ref<Planner>        # which planner produced the plan
    highIRHash       Hash                # the High IR (by content hash, Book 04 §Ch07)
    compilationRef   Ref<Compilation>    # passes run, diagnostics, substitutions (Book 05 §Ch08)
    lowIRHash        Hash                # the executed Low IR
    runtimeRef       Ref<Runtime>        # where it ran (chosen after planning, Book 06 §Ch04)
    executionRef     Ref<Execution>
  status/record:
    outcome          Succeeded | Failed | Compensated | Cancelled
    metrics          ExecMetrics         # durations, retries, cost, resource use (§3)
    reasoningRecord  list<ReasoningTrace> # captured typed outputs of Reasoning nodes (§4)
    successEval      SuccessEvaluation   # actual vs. Goal success criteria (Ch 07)
    humanFeedback    list<Feedback>?     # explicit human signals (Ch 07)
    lessons          list<Lesson>        # extracted, secret-free lessons (§5)
    knowledgeUpdates list<Ref>           # Knowledge promoted from this (Ch 04)
    determinizationCandidates list<Ref>  # reasoning flagged for folding (Ch 06)
```

Everything is **by reference or by content hash** — the Experience points at the Intent, Goals, IR, Compilation, Runtime, and Execution rather than duplicating them. This keeps it compact, keeps identities consistent (Book 04 §Ch07), and — critically — keeps it **secret-free** (§6).

## 2. Why these fields

The anatomy is not a data dump; each field serves a consumer:

- **`intentRef`/`goalRefs`** — the *what was wanted*, so outcome can be judged against success criteria (Ch 07) and so lessons attach to the right objective.
- **`plannerRef`/`highIRHash`** — the *how it was planned*, so recurring planner weaknesses (verification/policy failures) are attributable (Book 05 §Ch07 §6) and so determinization can match reasoning by plan shape.
- **`compilationRef`/`lowIRHash`** — the *how it was compiled*, including which reasoning was already determinized (Book 05 §Ch08) and the exact executed artifact for replay (Book 04 §Ch07 §4).
- **`runtimeRef`/`metrics`** — the *where and how well it ran*, feeding runtime selection cost/health (Book 06 §Ch04, §Ch05).
- **`reasoningRecord`** — the captured non-determinism (§4), the raw material for determinization discovery (Ch 06).
- **`successEval`/`humanFeedback`** — the *did it actually work*, the ground truth for learning (Ch 07).
- **`lessons`/`knowledgeUpdates`/`determinizationCandidates`** — the *what we learned and what to do about it*, the outputs of the feedback paths.

## 3. Metrics

`metrics` (`ExecMetrics`) capture the quantitative outcome: per-instruction and total durations, retry counts, timeouts, compensation events, cost (model tokens, API/`Cost` effects — Book 04 §Ch06), and resource use. These are the inputs to the compiler's cost model (Book 05 §Ch03 §3) and to runtime selection (Book 06 §Ch04 §3). Metrics are quantitative and non-secret by nature; they are captured for *every* execution so the cost model is well-fed.

## 4. Reasoning records (the determinization substrate)

For each `Reasoning`/`CapturedReasoning` node that executed (Book 04 §Ch03 §3.2, §Ch04 §3), the Experience records a **`ReasoningTrace`**: the node's identity (by shape/hash), its **typed inputs** (by content hash) and its **typed output**, and its `determinize` eligibility. This is the exact data the determinization engine (Ch 06) mines: "the same reasoning shape, over equivalent typed inputs, produced these outputs — reliably?" Because inputs/outputs are typed and content-addressed (Book 04 §Ch07), equivalence is a *decidable* check, not a heuristic. The trace records the **typed output value**, never the model prompt or any secret (§6).

## 5. Lessons

A **`Lesson`** is an extracted, structured, secret-free insight: a recurring failure mode and its cause, an optimization opportunity, a stable reasoning worth determinizing, a success pattern worth reusing. Lessons are the bridge from raw record to actionable feedback — they are what gets promoted to Knowledge (Ch 04) or turned into determinization candidates (Ch 06). A Lesson references the facts that support it (by reference) so it is traceable and revisable (Ch 06 §reversibility).

## 6. Secret-freedom and content addressing (P7, P6)

Two structural properties make the anatomy safe and durable:

- **Secret-free (P7).** Because the Experience is assembled from the secret-free Event stream (§Ch03, Book 03 §Ch03 §8) and stores references/hashes rather than payloads, **no field ever contains a secret value** — including `reasoningRecord` (typed outputs and hashed inputs only) and `metrics`. Secrets appear, if at all, as `SecretRef`/class in a lesson (Book 11 §04 §6). This is why Experience can be freely retained, queried, and fed to planning without a leak surface.
- **Content-addressed (P6).** IR is referenced by hash (Book 04 §Ch07); an Experience is thus tied to the *exact* plan and execution it describes, and equivalence across Experiences (same High IR, same reasoning inputs) is a hash comparison — the basis for both determinization (Ch 06) and deduplicated learning.

## 7. Retention

An Experience is **long-lived** (Book 02 §Ch04 §5): it outlives its Execution and is retained per policy — long enough for the cost model, determinization discovery, audit correlation (Book 11 §09), and evaluation. Retention is tenant-scoped (Book 11 §08 §5) and secret-free by construction, so long retention carries no secret-exposure risk.

## 8. Invariants (normative summary)

1. An Experience records one Execution's full lifecycle by reference/hash: intent, goals+criteria, planner, High/Low IR hashes, compilation, runtime, execution, outcome, metrics, reasoning traces, evaluation, feedback, lessons, and feedback outputs.
2. Every field serves a specific feedback consumer (evaluation, cost model, determinization, planning); the anatomy is purpose-built, not a dump.
3. Reasoning traces record typed, content-addressed inputs and typed outputs (never prompts or secrets), making determinization equivalence a decidable check.
4. Lessons are extracted, structured, secret-free insights that reference their supporting facts and drive Knowledge/determinization feedback.
5. No field ever contains a secret value; secrets appear only as references/classes (P7); IR is referenced by content hash (P6).
6. Experience is long-lived, tenant-scoped, and retained per policy with no secret-exposure risk.
