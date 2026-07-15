# Book 15 · Chapter 05 — FAQ and Anti-Patterns

*Nature: **Informative**. · Points to the Normative Books for the authoritative answers.*

> This chapter answers the questions newcomers most often ask and names the anti-patterns the architecture forbids. It is informative — each answer points to the Book that specifies it — but it is one of the most useful pages for building correct intuition, because it corrects the misunderstandings that lead people to fight the architecture.

## 1. FAQ

**"Isn't this just an agent framework?"**
No (Book 01 §Ch01 non-goals). An agent framework *executes model output*; Sankalpa *compiles intent into governed IR* and never executes model output (P1). An agent framework, at most, is a *replaceable planner adapter* here (Book 08). The whole IR/compiler/runtime apparatus exists precisely because Sankalpa is *not* an agent framework.

**"Isn't this just Kubernetes for AI?"**
It borrows Kubernetes' resource/reconciliation model (Book 02/07, Book 01 §Ch06) but adds a *compiler with a non-deterministic front-end* (Books 04/05/08) and a *determinization loop* (Book 10) that have no K8s analog. The substrate is K8s-like; the point is what runs on it.

**"Does it require an LLM?"**
No (Book 01 §Ch01). LLMs do only non-deterministic reasoning, confined to planning (Book 08). A rule-based planner makes the entire pipeline deterministic (Book 08 §Ch06). The platform actively converts LLM reasoning into deterministic capabilities over time (P13).

**"Where do secrets go?"**
Never into planning, IR, logs, Events, or Knowledge (P7). Only as references, materialized as values *only at execution, only in the runtime*, by the Secret Broker (Book 11 §04, Book 06 §Ch06). See SI-2 (Book 11 §Ch11).

**"How is it deterministic if it uses a model?"**
Determinism is *relative to declared inputs and effects* and is a property of the *machine below planning*, not the model (Book 01 §Ch05, Book 06 §Ch03 §7). The model's non-determinism is confined to typed `Reasoning` nodes whose outputs are captured; everything downstream is reproducible.

**"What makes it get better over time?"**
Determinization (P13, Book 10 §Ch06 + Book 05 §Ch06): repeated reasoning becomes deterministic Capabilities, and Knowledge (Book 09) improves planning. The system evolves toward determinism the more it is used (Book 01 §Ch05).

**"Can I add my own runtime / planner / channel?"**
Yes — implement the AEP interface (Books 06/08/13), pass the conformance suite, package it (Book 12), register it. No core change (P11).

## 2. Anti-patterns the architecture forbids

Each is something that *seems* convenient but violates an invariant; the architecture is shaped to make each impossible or caught.

| Anti-pattern | Why it's wrong | Caught by |
|--------------|----------------|-----------|
| **Executing model output directly** | Defeats determinism, policy, portability (P1). | Only verified IR executes (Book 04 §Ch08, Book 08 §Ch01). |
| **Putting a secret in a prompt / "just this once"** | The #1 leak surface (P7, Book 11 §01). | `SecretRef` opacity + verification (Book 04 §Ch05/08); secret-free planning (Book 08 §Ch04). |
| **Memory-as-context** (rolling conversation into prompts) | Unstructured, unversioned, leak-prone (Book 09 §Ch01). | Knowledge is structured/provenanced; Conversation isn't re-injected (Book 09/13). |
| **Ambient authority** ("it runs as admin") | Confused-deputy, unbounded blast radius (P8). | Capability-based security (Book 11 §03). |
| **install = privilege** | Supply-chain escalation (Book 12 §Ch01). | Install grants nothing; capabilities explicit (Book 12 §Ch05, P8). |
| **A channel that "decides"** | Forks behavior per transport, bypasses governance. | Channels carry, don't decide (Book 13 §Ch02). |
| **A planner that picks a runtime** | Breaks the non-determinism boundary and portability. | Runtime selected after planning (Book 06 §Ch04, IR-P7). |
| **Assuming two Resources change atomically** | No cross-Resource transactions (Book 02 §Ch08). | Reconciliation + conditions (Book 07 §Ch04). |
| **Retrying a non-idempotent write** | Doubles external effects (IR-P1). | Verification rejects it (Book 04 §Ch08 §2.5). |
| **Advisory-only policy** | Governance that documents but doesn't constrain. | Policy enforced pre-compile and pre-execute, fail-closed (Book 05 §Ch04, Book 14 §Ch06, P9). |
| **Trusting a plugin's output** | Untrusted code (Book 11 §01). | Output re-verified; conformance-tested (Book 11 §10 §4). |

## 3. The meta-lesson

Most anti-patterns share a root: **treating a shortcut as free when it costs an invariant.** The architecture is deliberately designed so those shortcuts are either impossible (a `SecretRef` cannot become a value in IR) or caught (bad IR fails verification, forbidden plans fail policy, unfaithful runtimes fail the fidelity gate). When something feels harder than "just doing it directly," that friction is usually an invariant being protected — the guidance is to work *with* the boundary (encode the secret as a reference, express the effect, add the capability), not around it.

## 4. Summary (informative)

Sankalpa is not an agent framework, not K8s-for-AI, not LLM-required; secrets are always references; determinism is a property of the machine below planning; the system improves via determinization. The forbidden anti-patterns are the shortcuts that cost invariants — and the architecture makes each impossible or caught. Building *with* the boundaries is how the system stays deterministic, secure, and improvable.
