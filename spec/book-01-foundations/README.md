# Book 01 — Foundations

*Status: **Draft-complete** (all chapters authored) · Nature: mixed; §Principles is Normative.*
*Reflects: ADR-0001, ADR-0002, RFC-0001.*

## Scope
The *why* of Sankalpa and the vocabulary of ideas every other Book assumes: the mission, the core philosophy, the layered transformation from Mission to Knowledge, and the foundational principles that constrain all subsequent design. This is the constitution; later Books are legislation under it.

## Chapters
1. **`01-mission-and-non-goals.md`** — What Sankalpa is (an OS for intelligent work that makes intent deterministic) and, emphatically, what it is not (a chatbot, an agent framework, a workflow engine, or a vendor/LLM/runtime-bound product). Non-goals are as binding as goals.
2. **`02-core-philosophy.md`** — Intent → Goals → Plans → IR → Execution → Experience → Knowledge, and why the loop drives the system toward determinism. The doctrine that *LLMs perform only non-deterministic reasoning* and everything else must become deterministic.
3. **`03-the-layered-architecture.md`** — The full stack (Mission → Strategy → Intent → Goals → Planning → High IR → Optimization → Policy Validation → Compilation → Low IR → Runtime → Controllers → Execution → Events → Experience → Knowledge). Each layer's owner, contract, and failure mode. Includes the canonical architecture diagram.
4. **`04-foundational-principles.md`** *(Normative)* — See [`04-foundational-principles.md`](04-foundational-principles.md). The invariant principles ("everything is a Resource," "natural language is never executable," etc.) that every design MUST uphold.
5. **`05-determinism-and-determinization.md`** — What "deterministic execution" means precisely (reproducibility given fixed inputs), where non-determinism is *permitted* (typed reasoning steps), and how the system converts repeated reasoning into Capabilities.
6. **`06-influences-and-prior-art.md`** — What Sankalpa borrows from Linux, LLVM, Kubernetes, Temporal, Git, Terraform, Docker, PostgreSQL, and Rust — and where it deliberately differs. Pointers to [`../../research/`](../../research/README.md).
