# Book 01 · Chapter 03 — The Layered Architecture

*Nature: **Normative** (the layering and its ownership are binding). · Reflects: ADR-0002, RFC-0001; grounds the whole specification. See [`../../diagrams/src/layered-architecture.mmd`](../../diagrams/src/layered-architecture.mmd).*

> This chapter specifies the full stack — every layer from Mission down to Knowledge — with each layer's owner, contract, and failure mode. It is the map that every other Book fills in. The layering is normative: a transformation must happen at its layer, owned by its Book, or the architecture is violated.

## 1. The stack

```
Mission          — the enduring purpose (Book 01, Book 02 Mission)
  ↓ Strategy      — how the mission is pursued (Book 02 Strategy)
  ↓ Intent        — a human wish, natural language (Book 08, Book 02 Intent)   ┐ non-deterministic
  ↓ Goals         — structured objectives + success criteria (Book 08)         │ (reasoning allowed,
  ↓ Planning      — construct a plan (Book 08)                                  ┘  runtime-agnostic)
  ↓ AOS High IR   — the plan as typed, effect-annotated IR (Book 04)          ┐
  ↓ Optimization  — semantics-preserving improvement (Book 05)                 │
  ↓ Policy Valid. — governance before execution (Book 05, Book 11)             │ deterministic
  ↓ Compilation   — lower to Low IR (Book 05)                                  │
  ↓ AOS Low IR    — execution-ready, runtime-agnostic IR (Book 04)             │
  ↓ Runtime       — selected after planning; lower to RuntimeGraph (Book 06)   │
  ↓ Controllers   — reconcile toward desired state (Book 07)                   │
  ↓ Execution     — runtime-specific effects (Book 06)                         ┘
  ↓ Events        — every state change announced (Book 03, Book 14)
  ↓ Experience    — the record of one execution (Book 10)
  ↓ Knowledge     — durable understanding (Book 09)
  ↺ improves future Planning
```

The **line between Planning and High IR is the non-determinism boundary** (Book 08 §Ch05): above it, reasoning; below it, deterministic machinery. The **loop closes** at the bottom: Knowledge improves Planning (Book 09 §Ch06, Book 10).

## 2. Each layer has an owner, a contract, and a failure mode

The layering is meaningful only because each transformation is *owned* and *contracted*. Representative layers (each Book is authoritative for its own):

| Transformation | Owner | Contract | Failure mode |
|----------------|-------|----------|--------------|
| Intent → Goals | Planner (Book 08) | structured Goals with success criteria | clarification / blocked with condition |
| Goals → High IR | Planner (Book 08) | verified, typed, effect-annotated IR | verification failure (stops here) |
| High IR → (optimized) | Compiler (Book 05) | semantics-preserving | pass rejected, re-verified |
| High IR → policy-checked | Policy Engine (Book 05/11) | permitted or explained denial | policy denial (fail-closed) |
| → Low IR | Compiler (Book 05) | execution-ready, runtime-agnostic | internal defect (re-verified) |
| Low IR → runtime | Runtime Mgr (Book 06) | fidelity + policy gate | unsatisfiable → fail-closed |
| → Execution | Runtime (Book 06) | observably-equivalent, honored ExecPolicy | explained terminal + compensation |
| → Experience | Experience Mgr (Book 10) | secret-free structured record | (capture is derivation, robust) |
| → Knowledge | Knowledge Mgr (Book 09) | provenanced, revisable, secret-free | conflict surfaced to human |

Every arrow is a contract; every contract has a defined failure that is *explained*, never silent (Book 02 §Ch03 §3.5, Book 03 §Ch13).

## 3. Communication crosses layers only through the Kernel (P4)

The layers do not call each other directly. All transformations are mediated by the **Kernel** (Book 03) — its Managers own the transformations, and communication is via the **Kernel API** (synchronous) or **Event Bus** (asynchronous). A layer is realized as a Manager and/or a reconciling controller (Book 07); the pipeline is a chain of reconciliations (Book 07 §Ch01 §6), each layer reacting to the state the previous produced. This is why the layering and the microkernel (ADR-0002) are the same fact seen twice: the layers are *what* transforms; the Kernel is *how* they are mediated and governed.

## 4. Determinism gradient

Read top to bottom, the stack goes from **non-deterministic** (Mission/Intent/Planning — human and model) to **deterministic** (IR/Compiler/Runtime/Execution — the machine), and the determinism gradient is the whole point (§Ch02 §2, §Ch05):
- Above the High-IR line: exactly enough non-determinism to resolve genuine ambiguity, confined to Planning.
- Below it: pure determinism, so execution is reproducible, governable, and improvable.
- Over time (P13), the boundary *descends* — more of what was reasoning becomes deterministic capability (Book 05 §Ch06, Book 10 §Ch06). The gradient is not fixed; the deterministic region grows.

## 5. Every layer upholds the invariants

The foundational principles (§Ch04) thread through every layer: everything is a Resource (P2) at every layer; every transformation emits Events (P5); secrets are references at every layer and a value only at Execution (P7); policy gates before compilation and execution (P9); nothing is ambient authority (P8). The layering does not *contain* the invariants in one place — it *upholds* them everywhere. This is what makes the stack coherent rather than a pipeline of disconnected stages.

## 6. Invariants (normative summary)

1. The architecture is a layered transformation from Mission to Knowledge, closing as a loop where Knowledge improves Planning.
2. The boundary between Planning and High IR is the non-determinism boundary: reasoning above, deterministic machinery below.
3. Every transformation has a defined owner (a Book), a contract, and an explained (never silent) failure mode.
4. Layers communicate only through the Kernel API or Event Bus (P4); the layering and the microkernel are the same fact — layers transform, the Kernel mediates and governs.
5. The stack is a determinism gradient (non-deterministic top, deterministic bottom) whose boundary descends over time as reasoning becomes deterministic capability (P13).
6. The foundational principles are upheld at every layer, not localized to one — the layering upholds the invariants everywhere.
