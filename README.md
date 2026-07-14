# Sankalpa Specification (`sankalpa-spec`)

> **Sankalpa (संकल्प)** — *intent · resolve · determination.*
> An open-source operating system for intelligent work that transforms human **intent** into **deterministic execution**.

This repository is **the specification**, and in this project the specification *is the product*. Implementation is one possible realization of the architecture defined here; it does not begin in earnest until this specification reaches a stable **v1.0**.

Sankalpa is **not** a chatbot, an agent framework, or a workflow engine. It is a **platform**: a microkernel, a resource model, an intermediate representation (IR), a compiler pipeline, and a governed ecosystem of pluggable planners, runtimes, and capabilities. LLMs are confined to *non-deterministic reasoning*; everything Sankalpa can make deterministic, it does — and it continuously converts repeated reasoning into reusable, deterministic capabilities.

---

## Why a specification-first repository

Systems built to last a decade — Linux, LLVM, Kubernetes, PostgreSQL, Git, Rust, Terraform — share a trait: their *conceptual integrity* is defined and defended in writing before, and independently of, any implementation. Code changes weekly; the architecture it serves must change slowly and deliberately. This repository exists so that:

1. **Architecture precedes implementation.** No feature is built until its design is written, reviewed, and accepted (see [`GOVERNANCE.md`](GOVERNANCE.md)).
2. **Decisions are durable and discoverable.** Every significant choice leaves an [ADR](adrs/README.md); every substantial design leaves an [RFC](rfcs/README.md); every extension point leaves an [AEP](aeps/README.md).
3. **The system is understandable by newcomers in an afternoon and by experts in depth.** The [`spec/`](spec/README.md) "books" form a coherent narrative from mission down to bytes.

## Repository map

| Path | Purpose |
|------|---------|
| [`spec/`](spec/README.md) | The normative specification, organized as **Books → Chapters**. The core deliverable. |
| [`rfcs/`](rfcs/README.md) | **Request for Comments** — substantial design proposals under review or accepted. |
| [`adrs/`](adrs/README.md) | **Architecture Decision Records** — short, immutable records of *why* a decision was made. |
| [`aeps/`](aeps/README.md) | **Architecture Extension Proposals** — the contract for third-party extension of Sankalpa. |
| [`process/`](process/README.md) | How this project governs itself: RFC/ADR/AEP lifecycles, review gates, stability guarantees. |
| [`templates/`](templates/) | Canonical templates for every artifact (RFC, ADR, AEP, implementation/test/security/perf reviews). |
| [`research/`](research/README.md) | **Phase 0** prior-art studies and pattern analyses. Input to the spec; not normative. |
| [`diagrams/`](diagrams/) | Source (Mermaid/PlantUML/D2) and rendered architecture diagrams referenced by the spec. |
| [`GLOSSARY.md`](GLOSSARY.md) | Single source of truth for every term. If a word is capitalized in the spec, it is defined here. |
| [`ROADMAP.md`](ROADMAP.md) | The phased plan from Phase 0 (research) to the ecosystem marketplace. |

## The layered architecture at a glance

```
Mission → Strategy → Intent → Goals → Planning
   → AOS High IR → Optimization → Policy Validation → Compilation → AOS Low IR
      → Runtime → Controllers → Execution → Events → Experience → Knowledge ↺
```

Each arrow is a **transformation** with a defined owner, contract, and failure mode. Natural language enters only at the top; it is *never executable*. The sole executable representation is **AOS IR**. The loop closes because **Experience** enriches **Knowledge**, which improves future **Planning** — driving the system toward ever-greater determinism over time.

See [Book 01 — Foundations](spec/book-01-foundations/README.md) for the full treatment.

## Status

**Pre-v0.1 — draft.** Nothing here is stable. Every document carries a status header (`Draft` / `Proposed` / `Accepted` / `Final` / `Superseded`). See [`process/versioning-and-stability.md`](process/versioning-and-stability.md).

## How to participate

- Read [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`GOVERNANCE.md`](GOVERNANCE.md).
- Propose a change through the appropriate artifact — do not open a PR that alters normative text without a corresponding RFC or ADR.
- All participation is governed by the [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).

## Licensing

The specification text is licensed under **CC BY 4.0**; templates and code samples under **Apache-2.0**. See [`LICENSE`](LICENSE). Rationale is recorded in [ADR-0003](adrs/README.md).
