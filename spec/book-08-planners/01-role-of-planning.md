# Book 08 · Chapter 01 — The Role of Planning

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P8, P11 and IR-P1. Companion to Book 04 §Ch03 (High IR), Book 02 (Intent/Goal), Book 05 (Compiler).*

> Planning is the **front-end** of Sankalpa's compiler: it turns natural-language Intent into a structured plan expressed as **High IR**. It is also the platform's **one sanctioned home for non-determinism** (P1). This chapter fixes what planning is responsible for, where its authority begins and ends, and the invariant that everything downstream depends on: *a planner's only output is verified High IR.*

## 1. Planning in the transformation chain

```
Intent (natural language) → Goals (structured) → Plan → HIGH IR
     └──────────────── the Planner's territory (Book 08) ────────────────┘
                                    │  verified (Book 04 §Ch08)
                                    ▼
                        Compiler (Book 05) → Low IR → Runtime (Book 06)
```

Planning occupies the top of the chain (Book 01 §03). Above it, humans express Intent; below it, everything is deterministic. The planner is the hinge: it consumes ambiguity and emits structure. Its output is the single input the entire deterministic machine (Books 04–06) assumes but does not itself produce.

## 2. What a planner is responsible for

A **Planner** (a plugin, §Ch03) is responsible for exactly this transformation and nothing more:

1. **Derive Goals from Intent** — turn an `Intent` Resource's natural-language utterance into structured `Goal` Resources with explicit success criteria (Book 02 §Ch07, §Ch02 §2). Includes clarification loops where Intent is underspecified (§Ch02).
2. **Plan** — decide the steps, data flow, capability invocations, and control flow that would achieve the Goals.
3. **Emit High IR** — express that plan as a valid, fully-typed, fully-effect-annotated High IR module (Book 04 §Ch03), wrapped in an `IRModule` Resource (`level: High`).

A planner is **not** responsible for — and MUST NOT attempt — optimization, policy decisions, lowering, runtime selection, or execution. Those are downstream, deterministic, and owned by other Books. A planner that tried to, say, pick a runtime would violate the layering (P1, P11) and its output would be rejected.

## 3. The defining invariant: output is verified High IR, always

The single most important rule in this Book: **a planner's only externally-consumed output is High IR that passes verification** (Book 04 §Ch08). Consequences:

- The planner may reason however it likes internally — call a model many times, use any framework, backtrack — but what *leaves* the planner is IR, never natural language, never a prompt, never model output as text (P1).
- That IR is **verified before anything downstream touches it** (Book 04 §Ch08 §1, Book 05 §Ch08). A planner that emits ill-formed, ill-typed, or under-annotated IR fails verification and the plan does not proceed — non-determinism is *stopped at the planner boundary*.
- Therefore the correctness of the entire deterministic arc does **not** depend on the planner being correct — only on the planner's *output being verifiable*. A buggy or adversarial planner cannot corrupt execution; at worst it produces IR that fails verification or policy. This is what makes it safe to run untrusted, third-party, model-driven planners (§Ch03, §Ch04).

## 4. Why planning is the *only* non-deterministic stage

Sankalpa confines non-determinism to planning deliberately (Book 01 §05, P1):

- **Determinism is the product.** Everything the platform can make reproducible, it does. The LLM's value is *reasoning through ambiguity* — deriving structure from a vague request — which is inherently non-deterministic. That reasoning belongs here and only here.
- **Downstream stays pure.** Because planning emits typed, effect-annotated IR, the compiler and runtime can be fully deterministic (Books 05–06). If planning leaked prompts or model output into execution, determinism, policy validation, and portability would all collapse (Book 04 §Ch01 §2).
- **Non-determinism is captured, not banished.** Within the plan, genuinely non-deterministic decisions are represented as **`Reasoning` nodes** (Book 04 §Ch03 §3.2) — typed, effect-declared, and eligible for later determinization (Book 05 §Ch06). So even the reasoning the planner *couldn't* resolve at plan time is expressed in a form the deterministic machine can govern and, over time, eliminate (P13).

## 5. Planners are plugins (P11)

No planner is privileged. LangGraph, an OpenAI-Agents-SDK adapter, CrewAI, a custom planner — all are plugins behind the **planner interface** (AEP-0001, §Ch03). The core names none of them (Book 03 §Ch04 §3). This is what lets Sankalpa adopt the best planning technology of each era without changing anything downstream: the contract is "Goals in, verified High IR out," and any planner that honors it is interchangeable (§Ch07 conformance).

## 6. The planner is untrusted

Because a planner is a model-driven plugin operating on user intent, it is treated as **untrusted** (Book 03 §Ch09 §2) and, critically, **secret-free** (§Ch04): it never receives secret values, and it holds only the least-privilege capabilities it needs to *plan* (not to *execute*, P8). The combination — untrusted, secret-free, output-always-verified — is precisely what makes it safe to put a non-deterministic model at the top of a deterministic platform.

## 7. Invariants (normative summary)

1. Planning transforms Intent → Goals → Plan → High IR and nothing more; it does not optimize, decide policy, lower, select runtimes, or execute.
2. A planner's only consumed output is High IR that passes verification (Book 04 §Ch08); natural language, prompts, and model text never leave the planner as executable output (P1).
3. Downstream correctness depends only on the planner's output being verifiable, not on the planner being correct — so untrusted, model-driven planners are safe.
4. Planning is the sole home of non-determinism; residual non-determinism is captured as typed `Reasoning` nodes, governable and later determinizable (P13).
5. Planners are interchangeable plugins behind AEP-0001; the core names none.
6. Planners are untrusted, secret-free, and hold only least-privilege planning capabilities (P8).
