# Book 01 · Chapter 01 — Mission and Non-Goals

*Nature: **Normative** (the non-goals bind as strongly as the mission). · Reflects: ADR-0001; grounds the whole specification.*

> This chapter states what Sankalpa *is* and — with equal force — what it is *not*. The non-goals are not disclaimers; they are architectural constraints. A proposal that drifts toward a non-goal is out of scope by definition, and this chapter is the authority for saying so.

## 1. The mission

**Sankalpa is an open-source operating system for intelligent work that transforms human intent into deterministic execution.**

Unpacking each load-bearing word:
- **Operating system** — a *platform*, not an application: a kernel, a resource model, an IR, a compiler, and a governed ecosystem of pluggable parts (Books 02–14). It provides the substrate on which intelligent work runs, the way an OS provides the substrate on which programs run.
- **Intelligent work** — work that today requires human reasoning to plan and adapt. Sankalpa's domain is the messy space between a vague human wish and a reliable outcome.
- **Human intent** — the input is *intent* expressed in natural language (Book 08), not code, not configuration, not a workflow diagram.
- **Deterministic execution** — the output is *reproducible, governed, auditable* execution (Book 06). The system's purpose is to make intelligent work as reliable as compiled software.
- **Transforms** — the whole platform is a *transformation pipeline* (§Ch03): intent → goals → IR → execution → experience → knowledge → better intent.

The one-sentence version, which every Book serves (spec/README): *transform human intent into deterministic execution, and continuously convert repeated reasoning into reusable deterministic capabilities.*

## 2. What Sankalpa is NOT (binding non-goals)

Each non-goal is an architectural boundary. Violating one is not a feature request; it is a category error.

- **NOT a chatbot.** Conversation is a *channel* (Book 13), a way to carry intent — not the product. The product is deterministic execution, not dialogue.
- **NOT an agent framework.** Sankalpa does not execute model output directly (P1). It *compiles* intent into governed IR (Book 04–05). Agent frameworks are, at most, *planner adapters* behind AEP-0001 (Book 08) — one replaceable component, not the architecture.
- **NOT a workflow engine.** Workflow engines are *runtimes* (Book 06) — lowering targets selected after planning. Sankalpa is the compiler and platform *above* any workflow engine, not one of them.
- **NOT tied to any LLM.** LLMs perform only non-deterministic reasoning (P1, §Ch05), confined to planning (Book 08), and the system actively converts that reasoning into deterministic capabilities over time (P13). A rule-based planner makes the whole pipeline deterministic (Book 08 §Ch06) — proof the platform does not *require* an LLM.
- **NOT tied to any runtime.** Runtimes are plugins; the runtime is chosen *after* planning (Book 06). The same plan runs on any conforming runtime.
- **NOT tied to any vendor.** Every extension point — planner, runtime, compiler, channel, provider, storage, knowledge backend — is a replaceable plugin (P11, Book 12). No vendor is load-bearing.

## 3. Why the non-goals are binding

Conceptual integrity (ADR-0001) is preserved by knowing what *not* to build. Each non-goal defends a principle:
- "Not an agent framework" defends P1 (natural language is never executable).
- "Not tied to any LLM/runtime/vendor" defends P11 (everything is replaceable).
- "Not a chatbot/workflow engine" defends the layering (§Ch03): channels and runtimes are *edges*, not the *center*.

Without these boundaries, Sankalpa would drift into being "another AI agent thing" — exactly the crowded, non-deterministic space it exists to transcend. The non-goals keep it a *platform*.

## 4. The measure of a proposal

Any RFC/ADR/AEP is measured against this chapter: does it advance the mission *and* respect every non-goal? A proposal that makes the system a better chatbot, welds it to one LLM, or turns it into a workflow engine fails — regardless of its other merits. This is the first question the architecture gate asks (process/review-gates).

## 5. Invariants (normative summary)

1. Sankalpa's mission is to transform human intent into deterministic execution, and to continuously convert repeated reasoning into reusable deterministic capabilities.
2. Sankalpa is a platform (kernel + resource model + IR + compiler + governed ecosystem), not an application.
3. The non-goals are binding architectural boundaries: not a chatbot, not an agent framework, not a workflow engine, not tied to any LLM, runtime, or vendor.
4. Each non-goal defends a foundational principle (P1, P11) or the layering; violating a non-goal is a category error, not a feature.
5. Every proposal is measured against advancing the mission while respecting every non-goal.
