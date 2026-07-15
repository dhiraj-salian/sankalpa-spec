# The Sankalpa Specification

*Status: Draft skeleton · This is the normative core of the project. Everything else in the repository exists to produce, govern, and explain what lives here.*

The specification is organized as **Books**, each a self-contained treatment of one part of the system, divided into numbered **Chapters**. Read top to bottom for the full narrative — from mission down to bytes — or jump to a Book by concern.

## Reading order

Books are ordered so each builds on the last. A newcomer should read Books 00–01 first (the *why* and the vocabulary), then 02–05 (the *what*: model, kernel, IR, compiler), then the rest by interest.

| Book | Title | Concern |
|------|-------|---------|
| [00](book-00-preamble/README.md) | Preamble | How to read the spec; conformance language; notation. |
| [01](book-01-foundations/README.md) | Foundations | Mission, philosophy, the layered architecture, the foundational principles. |
| [02](book-02-resource-model/README.md) | The Agent Resource Model (ARM) | *Everything is a Resource*: metadata/spec/status, lifecycle, versioning. |
| [03](book-03-kernel/README.md) | The Kernel | The microkernel, its Managers, the Kernel API, and the Event Bus. |
| [04](book-04-aos-ir/README.md) | AOS IR | The two-level intermediate representation; the only executable form. |
| [05](book-05-compiler/README.md) | The Compiler | Passes, optimization, policy validation, lowering. |
| [06](book-06-runtimes/README.md) | Runtimes | Executing lowered graphs; the runtime plugin contract. |
| [07](book-07-controllers/README.md) | Controllers & Reconciliation | The controller runtime and reconciliation model. |
| [08](book-08-planners/README.md) | Planners | Intent → Goals → High IR; the planner plugin contract. |
| [09](book-09-knowledge/README.md) | Knowledge | The vault ⇄ graph knowledge system. |
| [10](book-10-experience/README.md) | Experience | First-class execution records and the feedback loop. |
| [11](book-11-security/README.md) | Security | Trust boundaries, capabilities, the Secret Broker, policy, approvals. |
| [12](book-12-packages/README.md) | Packages & Marketplace | Distribution, signing, dependencies, discovery. |
| [13](book-13-interfaces/README.md) | Interfaces & Channels | Web Runtime, channels, and the API gateway. |
| [14](book-14-observability-governance/README.md) | Observability & Runtime Governance | Events, metrics, tracing, audit, policy at runtime. |
| [15](book-15-appendices/README.md) | Appendices | End-to-end walkthroughs, notation reference, worked examples. |

## Normative conventions

- The keywords **MUST, MUST NOT, SHOULD, SHOULD NOT, MAY** are used per RFC 2119 and appear **only** in sections explicitly marked *Normative*. Everything else is explanatory (*Informative*).
- Each chapter states, up front, whether it is Normative or Informative, and which RFCs/ADRs it reflects.
- No chapter introduces a capitalized term without a corresponding [Glossary](../GLOSSARY.md) entry.

## The through-line

Every Book serves one sentence:

> **Sankalpa transforms human intent into deterministic execution, and continuously converts repeated reasoning into reusable deterministic capabilities.**

If a chapter cannot trace its relevance back to that sentence, it does not belong in the spec.

## Status

**All 16 Books are Draft-complete** — every chapter is authored (no placeholders) and cross-consistent, ~127k words of normative and informative text. Each Book's `README` states its status; each chapter declares its Nature (Normative/Informative) and the principles/RFCs it reflects. The corpus is ready for the **Phase 2 hardening pass** ([`../ROADMAP.md`](../ROADMAP.md)): adversarial review for inconsistencies, missing failure modes, and unspecified lifecycles, iterating toward a stable **v1.0**. New capability still enters through the RFC process ([`../process/`](../process/README.md)) — an accepted RFC is reflected into these Books to become `Final`.
