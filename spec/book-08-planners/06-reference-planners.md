# Book 08 · Chapter 06 — Reference Planners

*Nature: **Informative**. · Reflects: RFC-0001; illustrates P1, P11, IR-P7. These are planner *adapters*, not dependencies of the core.*

> This chapter sketches how representative planning technologies map to the planner interface (§Ch03). It is illustrative: each is a *plugin adapter*, and the core names none of them (IR-P7, Book 03 §Ch04 §3). The point is to show that the "Goals in, verified High IR out" contract is realizable across very different planning paradigms — the evidence that Sankalpa is genuinely *not tied to any LLM or framework* (Book 01 §01).

## 1. How to read these mappings

For each technology the key question is: **how does it produce a valid, fully-typed, fully-effect-annotated, secret-safe, runtime-agnostic High IR module** (§Ch03 §3) from Goals + non-secret context? The adapter's job is to bridge the framework's native output to High IR and to honor the isolation/secret rules (§Ch04). Whatever the framework does internally, what *leaves* the adapter is verified IR (§Ch01 §3).

## 2. Graph/agent frameworks (e.g. LangGraph-class)

- **Paradigm:** a graph of nodes/agents with explicit state transitions — structurally close to Sankalpa's dataflow/control model.
- **Mapping:** the adapter runs the framework's graph to *construct a plan*, then emits it as High IR — framework nodes that call tools → `CapabilityInvocation`s; branching → `Branch`; model-decision nodes that must run at execution → `Reasoning` nodes (Book 04 §Ch03).
- **Roadmap role:** the **first** planner adapter (Roadmap Phase 11), because its graph model maps cleanly to High IR, letting the intent→execution loop be demonstrated end to end.

## 3. SDK-style agent runtimes (e.g. OpenAI-Agents-SDK-class, Semantic-Kernel-class)

- **Paradigm:** an agent loop with tool-calling and planning built in.
- **Mapping:** the adapter intercepts the agent's *intended tool calls and control flow* and expresses them as High IR rather than executing them directly. The critical adaptation: the SDK's instinct to *execute* tools is redirected to *emit IR*, so effects happen downstream under governance, not inside the planner (§Ch04 §3).
- **Consideration:** frameworks that eagerly execute must be wrapped so their tool calls become IR nodes; a framework that cannot be prevented from executing is unsuitable as a planner (it would violate P1/P8).

## 4. Multi-agent orchestrators (e.g. CrewAI-class, AutoGen-class)

- **Paradigm:** multiple cooperating agents decomposing and solving a task.
- **Mapping:** used for *plan construction* — the crew reasons about how to achieve the Goals — and the resulting plan is emitted as a single High IR module. Inter-agent negotiation is planner-internal and non-deterministic; only the agreed plan (as verified IR) leaves the adapter.
- **Consideration:** the richer the internal reasoning, the more important that the adapter marks residual `Reasoning` nodes and `determinize` eligibility honestly (§Ch05 §4).

## 5. Deterministic / rule-based planners

- **Paradigm:** a non-model planner — templates, rules, or a search algorithm — that maps well-understood Goal classes to plans deterministically.
- **Mapping:** direct construction of High IR from Goals with **no `Reasoning` nodes** at all. `describe` advertises the planner as deterministic (§Ch03 §4).
- **Significance:** this is the important existence proof that **Sankalpa is not tied to any LLM** (Book 01 §01). For Goal classes a rule engine handles, the *entire* pipeline is deterministic front-to-back. Model-driven planners are one option, not a requirement — and the platform actively converts model reasoning into this deterministic form over time (P13).

## 6. Custom and future planners

- Any planner — a domain-specific planner for a vertical, a novel architecture not yet invented, a hybrid of the above — is added by implementing AEP-0001 (§Ch03), passing the conformance suite (§Ch07), packaging it (Book 12), and registering a `Planner` Resource. No Book is edited to add a planner.
- Different planners MAY be selected per Goal class, per workspace, or per policy — a workspace can route sensitive planning to an on-prem deterministic planner and open-ended planning to a model-driven one, all behind the same interface.

## 7. What the spread demonstrates

These paradigms differ enormously — a graph framework, an eager agent SDK, a multi-agent crew, a pure rule engine — yet each, wrapped as an adapter, produces the **same** artifact: verified High IR that carries no secrets and names no runtime. That is the whole point (Book 01 §01, §05):

- **Not tied to any LLM or framework** (P11): the contract is the IR, not the planner; adopting a better planner is writing an adapter, never a core change.
- **Non-determinism stays boxed** (§Ch05): whatever wild reasoning happens inside an adapter, only verifiable IR escapes, with residual reasoning confined to typed nodes.
- **Determinism is reachable** (§5): a rule-based planner makes the entire pipeline deterministic for its Goal classes — the direction the whole system moves.

## 8. Note on adding a planner

A new planner is added by: implementing AEP-0001 (§Ch03), passing the planner conformance suite (§Ch07) at a declared stability level, packaging it (Book 12), and registering a `Planner` Resource. This chapter's list is illustrative and open-ended, exactly as P11 intends.
