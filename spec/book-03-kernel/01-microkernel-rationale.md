# Book 03 · Chapter 01 — Microkernel Rationale

*Nature: **Informative**. · Reflects: ADR-0002; realizes principles P4, P8, P9, P11.*

## 1. The two forces the Kernel must reconcile

Sankalpa faces a tension that shapes its entire core:

- **Radical extensibility.** Planners, runtimes, compilers, channels, policies, knowledge/secret providers, and controllers — most written by third parties, none known in advance — must be addable without touching the core.
- **Non-negotiable invariants.** Secret non-leakage (P7), policy enforcement (P9), capability-based authority (P8), determinism (P1), and auditability (P5/P10) must hold *regardless of which extensions are installed* — including buggy or hostile ones.

A monolith can enforce invariants but cannot safely admit untrusted extensions. A fully decentralized mesh can admit anything but has no place to enforce global invariants. The **microkernel** resolves this (ADR-0002): a small, trusted core owns the invariants and mediates everything; the interesting-but-untrusted parts live outside as plugins.

## 2. What "microkernel" means here

Concretely, the Sankalpa Kernel is:

- A **small trusted core** — the Managers (§Ch04) — that owns orchestration, the resource model (Book 02), the compile pipeline entry (Book 05), and every invariant.
- Surrounded by **plugins** loaded through AEP-defined interfaces (Book 12, the AEP process): planners, runtimes, compiler backends, channels, providers, controllers.
- With **exactly two communication substrates** and no third: the **Kernel API** (synchronous request/response, §Ch02) and the **Event Bus** (asynchronous publish/subscribe, §Ch03).

Principle **P4** is the load-bearing rule: *nothing communicates directly.* A plugin never calls another plugin, never reaches storage, never touches a secret store. It calls the Kernel API or reacts to Events. This single chokepoint is the only place invariants *can* be enforced, and making it the *only* path is what makes enforcement sound rather than best-effort.

## 3. Why this specific shape — lessons borrowed

The design is not novel in isolation; it is the convergence of several proven architectures (see [research](../../research/README.md)):

| Source | Lesson adopted |
|--------|----------------|
| OS microkernels (Mach, L4, seL4) | Small trusted core + user-space servers; minimize what must be trusted. |
| **Kubernetes** ([study](../../research/prior-art/kubernetes.md)) | API-server-as-single-front-door + reconciling controllers over a uniform resource model. |
| **LLVM** | A core pipeline with pluggable passes/targets behind stable interfaces. |
| **VS Code** | An extension *host* that isolates extensions from the core and from each other. |
| Capability security (KeyKOS, E) | Authority as unforgeable references, not ambient permission (P8). |

Sankalpa's core is closest to Kubernetes-plus-a-compiler: a control plane of Managers and controllers around ARM, with the compiler pipeline (Books 04–05) as a first-class Manager the way scheduling is in an OS.

## 4. What belongs in the core — and what never does

The boundary is a discipline, not a suggestion. Getting it wrong produces a "distributed monolith" (ADR-0002 §risk).

**In the trusted core (Managers):** the resource model and its storage contract; admission and validation; the Kernel API and Event Bus; capability granting and checking; policy enforcement points; the compile pipeline orchestration; plugin/package loading and isolation; lifecycle and the controller runtime; secret brokering; identity, sessions, approvals; observability.

**Never in the core:** any specific planner, runtime, compiler backend, channel, or provider; any vendor or LLM knowledge; any business logic about a particular task. The core knows *mechanism and policy*, never *which tool*. If a proposed core change encodes knowledge of a specific plugin, it is wrong (conceptual integrity, ADR-0001).

The litmus test: **could this be removed and replaced by a third party without changing the core?** If yes, it is a plugin. If no — because it enforces an invariant or defines the model — it is core.

## 5. The cost, stated honestly

Mediation is not free (ADR-0002 §Negative):

- **Indirection latency.** Every interaction is a Kernel API call or an Event, not a direct function call. We accept this because it is the price of a single enforcement point; §Ch13 and the performance reviews bound it, and the hot path (execution) runs *inside* a runtime, not across the Kernel per step.
- **Contract gravity.** The Kernel API and Event schema become critical, hard-to-change Stable interfaces. This is intended: they are governed like public APIs (versioning-and-stability), and their stability is a feature, not a burden.
- **Core-growth pressure.** There is constant temptation to "just add one plugin-aware thing" to the core. This is resisted structurally: the core depends only on AEP interfaces, never on a concrete plugin.

## 6. How the rest of Book 03 proceeds

The following chapters specify the two substrates — the [Kernel API](02-kernel-api.md) and the [Event Bus](03-event-bus.md) — then the [Managers](04-managers-overview.md) individually, then the [failure and degradation](13-failure-modes-and-degradation.md) model that keeps the invariants true even when the core itself is partially broken. Throughout, the question for any addition is the litmus test of §4.
