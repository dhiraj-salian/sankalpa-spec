# Book 08 — Planners

*Status: Draft skeleton · Nature: Normative. · Reflects: RFC-0001, AEP-0001 (Planner interface); realizes P1, P8, P11.*

## Scope
The path from Intent to Goals to **High IR**, and the **planner plugin contract**. Planners are the *only* place non-deterministic reasoning lives, and they are strictly runtime-ignorant: a planner never knows about n8n, Docker, or Python.

## Chapters
1. **`01-role-of-planning.md`** *(Normative)* — Intent → Goals → Plan → High IR; the boundary where non-determinism is allowed and where it stops.
2. **`02-intent-and-goal-derivation.md`** *(Normative)* — Turning natural-language Intent into structured Goals (Resources), including clarification loops and human-in-the-loop.
3. **`03-planner-interface.md`** *(Normative)* — The AEP-0001 contract: input (Goals + Knowledge context), output (valid High IR), error/uncertainty reporting, version negotiation.
4. **`04-planner-isolation-and-secrets.md`** *(Normative)* — Planners run without secrets (P7) and with only granted capabilities (P8); what context they may and may not receive.
5. **`05-non-determinism-boundary.md`** *(Normative)* — How reasoning steps are represented as *typed* IR constructs so downstream stays deterministic; reproducibility of planning inputs.
6. **`06-reference-planners.md`** *(Informative)* — Adapter notes for LangGraph, LangChain Deep Agents, OpenAI Agents SDK, CrewAI, AutoGen, Semantic Kernel, and custom planners. All emit AOS IR.
7. **`07-conformance-suite.md`** *(Normative)* — Tests a planner must pass: always emits valid, well-typed, effect-annotated High IR; never leaks secrets; degrades gracefully under uncertainty.
