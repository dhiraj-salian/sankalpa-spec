# Book 15 · Chapter 04 — Mapping to Prior Art

*Nature: **Informative**. · Companion to Book 01 §Ch06 (influences), [`../../research/`](../../research/README.md).*

> This chapter is a quick side-by-side: for each major Sankalpa concept, its closest analog in a well-known system, and where it *diverges*. It helps a reader who knows those systems learn Sankalpa fast (start from the analog, note the adaptation) and marks what is genuinely new. Book 01 §Ch06 gives the narrative; this is the lookup table.

## 1. Concept ↔ analog ↔ divergence

| Sankalpa concept | Closest analog | Key divergence |
|------------------|----------------|----------------|
| **ARM Resource** (Book 02) | Kubernetes API object | Machine-authored from Intent, not hand-written YAML; workload is an IR execution, not a container. |
| **Kernel + Managers** (Book 03) | K8s API server + controllers; OS microkernel | Includes a *compiler* as a first-class Manager; capability-based, not ambient trust. |
| **Reconciliation** (Book 07) | K8s controllers; Terraform plan/apply | Same level-triggered/idempotent model; drives an *intent→execution pipeline*, not just infra. |
| **AOS IR** (Book 04) | LLVM IR / MLIR | Its *front-end is non-deterministic* (a planner); typed **effects** and **secret-references** are first-class. |
| **Compiler passes** (Book 05) | LLVM pass framework | Adds a mandatory **policy** pass and **determinization** passes (no LLVM analog). |
| **Runtime** (Book 06) | A workflow engine / codegen target | Runtime is a *plugin selected after planning*; the same IR is portable across runtimes. |
| **Planner** (Book 08) | An agent framework | Not the architecture — a *replaceable adapter* whose only output is verified IR. |
| **Content addressing** (Book 04 §Ch07) | Git object hashing | Applied to IR for caching, replay, and determinization. |
| **Execution semantics** (Book 06 §Ch03) | Temporal durable execution | Same durability/compensation; driven by IR `ExecPolicy`, runtime-agnostic. |
| **Capability security** (Book 11 §03) | KeyKOS / E object-capabilities | Unified with *behavior* (a Capability is authority + typed function). |
| **Secret Broker** (Book 11 §04) | A secrets manager (Vault, KMS) | Values *never* enter IR/planning/logs; materialized only at execution by reference. |
| **Packages / Marketplace** (Book 12) | Cargo / npm / PostgreSQL extensions | install ≠ privilege; capability grants explicit; reconciled install. |
| **Knowledge vault ⇄ graph** (Book 09) | Obsidian + a graph DB | Two synchronized *authoritative* representations of one Resource. |
| **Experience loop** (Book 10) | (ML feedback / observability) | First-class per-execution Resource driving *determinization*. |
| **Governance** (`process/`) | Rust RFCs / K8s KEPs / PEPs | Spec-first: an accepted RFC must be *reflected into normative text* to be Final. |

## 2. What has no direct prior art

Two things are Sankalpa's own (Book 01 §Ch06 §3):
- **The non-determinism boundary** (Book 08 §Ch05) — a compiler whose front-end is a *non-deterministic reasoner*, with non-determinism confined to typed `Reasoning` nodes. No compiler has this because no compiler's source is a language model.
- **Determinization** (Book 10 §Ch06 + Book 05 §Ch06) — converting repeated reasoning into deterministic Capabilities so the system *evolves toward determinism* (P13). This is the piece that makes Sankalpa more than "a compiler for agents," and it has no ancestor.

## 3. How to use this mapping

- **Know Kubernetes?** Start ARM/Kernel/Controllers (Books 02/03/07) from the K8s analog; the divergences are the whole adaptation.
- **Know LLVM?** Start IR/Compiler (Books 04/05) from LLVM; the new parts are the non-deterministic front-end, effects, and the policy/determinization passes.
- **Know agent frameworks?** Note the inversion: they are a *plugin* (Book 08), and their output is *compiled and governed*, never executed directly.

## 4. Summary (informative)

Most of Sankalpa is a faithful adaptation of proven systems (Kubernetes, LLVM, Git, Temporal, capability security, package ecosystems), and this table maps each concept to its ancestor and its divergence. The genuinely new parts — the non-determinism boundary and determinization — are exactly where the platform targets a problem none of its ancestors did.
