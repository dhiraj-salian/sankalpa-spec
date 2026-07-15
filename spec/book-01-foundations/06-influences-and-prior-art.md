# Book 01 · Chapter 06 — Influences and Prior Art

*Nature: **Informative**. · Companion to [`../../research/`](../../research/README.md) (the Phase 0 studies).*

> Sankalpa is not invented from nothing; it is a synthesis of hard-won lessons from the systems worth learning from, adapted to a new domain. This chapter names what it borrows and — as importantly — where it deliberately diverges. Honest attribution of influence is both intellectual integrity and a map for contributors: to understand a Sankalpa mechanism, look at its ancestor and note the adaptation.

## 1. What we adopt, and from where

| Source | Lesson adopted | Where it lives |
|--------|----------------|----------------|
| **Kubernetes** | Uniform resource model; declarative desired-state + reconciling controllers; API-server-as-single-front-door; alpha/beta/stable versioning. | ARM (Book 02), Kernel/API (Book 03), Controllers (Book 07) |
| **LLVM** | A well-defined IR between many front-ends and back-ends; a pass framework; target-independent middle-end + lowering. | AOS IR (Book 04), Compiler (Book 05) |
| **MLIR** | Progressive lowering; structural IR infrastructure. | Two-level IR (Book 04) |
| **Temporal** | Durable execution; determinism; retries and saga compensation. | Execution semantics (Book 06 §Ch03) |
| **Git** | Content addressing; immutable, append-only history. | IR content addressing (Book 04 §Ch07), Events (Book 03 §Ch03) |
| **Terraform** | Desired-state with plan/apply; a provider plugin model. | Reconciliation (Book 07), providers (Book 03 §Ch04) |
| **Docker / Cloudflare Workers** | Packaging and isolation of execution. | Runtimes (Book 06), isolation (Book 11 §10) |
| **PostgreSQL** | Extensibility with stability; a query planner/optimizer. | Compiler (Book 05), Packages (Book 12) |
| **Rust / Cargo** | The RFC process; a package ecosystem with real stability guarantees. | Governance (`process/`), Packages (Book 12) |
| **Microkernels; capability security (KeyKOS, E); VS Code** | Small trusted core + isolated plugins; unforgeable authority; an extension host. | Microkernel (ADR-0002), capabilities (Book 11 §03), Plugin Manager (Book 03 §Ch09) |

The [Phase 0 research](../../research/README.md) studies each of these in depth, with an explicit "what we adopt / reject / why" per system (the Kubernetes study, [`kubernetes.md`](../../research/prior-art/kubernetes.md), is the exemplar).

## 2. Where we deliberately diverge

Adoption is not imitation. The consequential divergences:

- **From Kubernetes:** Resources are largely **machine-authored** (from Intent), not hand-written YAML; the "workload" is an **AOS IR execution**, not a container; authority is **capability-based**, replacing ambient cluster trust (Book 11 §03); storage is a **property set**, not a mandated backend (Book 02 §Ch08).
- **From LLVM:** the front-end is **non-deterministic** (a planner/model, Book 08), which no compiler front-end is — so the whole *non-determinism boundary* discipline (Book 08 §Ch05) is a Sankalpa invention with no LLVM analog.
- **From agent frameworks:** Sankalpa **does not execute model output** (P1). Where agent frameworks *are* the architecture, in Sankalpa they are at most a **replaceable planner adapter** (Book 08). This is the sharpest divergence and the reason for the whole IR/compiler apparatus.
- **From workflow engines:** they are **lowering targets** (Book 06), selected after planning — not the platform. Sankalpa sits *above* any of them.

## 3. The synthesis is the novelty

No single borrowed idea is novel; the **synthesis** is. Sankalpa's distinctive claim is combining:
- a **compiler** (LLVM) whose **front-end is a non-deterministic planner** (agent frameworks, confined and governed), producing
- **content-addressed IR** (Git) that is **reconciled** (Kubernetes/Terraform) and **durably executed** (Temporal), under
- **capability security** (KeyKOS) and a **microkernel** (OS microkernels) with an **extension ecosystem** (Cargo/PostgreSQL/VS Code),
- all driving a **learning loop** that **converts reasoning into determinism over time** (P13) — which is Sankalpa's own contribution, present in none of the ancestors.

That last element — determinization (§Ch05) — is the piece with no direct prior art, and it is what makes Sankalpa more than "a compiler for agents."

## 4. Why attribution matters

Naming influences is not modesty; it is method:
- It lets contributors **learn a mechanism by studying its ancestor** (to understand ARM, read Kubernetes; to understand the IR, read LLVM).
- It keeps the design **honest about what is solved** — we do not re-invent reconciliation or content addressing; we adopt them and note the adaptation.
- It marks the **genuinely new** (the non-determinism boundary, determinization) so that innovation effort concentrates where there *is* no prior art, not where there is.

## 5. Summary (informative)

Sankalpa stands on Kubernetes' resource model, LLVM's IR and passes, Git's content addressing, Temporal's durable execution, Terraform's providers, capability security, and the microkernel + package-ecosystem tradition — adapted for a domain none of them targeted: turning non-deterministic human intent into deterministic execution, and learning to make more of it deterministic over time. The borrowed parts are proven; the synthesis and the determinization loop are the contribution.
