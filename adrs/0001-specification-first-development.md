# ADR-0001: Specification-first development

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **Date** | 2026-07-14 |
| **Deciders** | Founding maintainers |
| **Domain / Book** | Governance / Book 00 |
| **Related** | ADR-0000, ADR-0002, ROADMAP.md |

## Context

Sankalpa's ambition — an operating system for intelligent work that turns intent into deterministic execution — is a *systems* ambition, not a feature. Systems of this class succeed or fail on **conceptual integrity**: the degree to which the whole reflects one coherent set of ideas. Fred Brooks argued that conceptual integrity is the most important consideration in system design and is achieved by separating architecture from implementation. The projects Sankalpa aspires to join (Linux, LLVM, Kubernetes, PostgreSQL, Rust, Terraform) all maintain a specification/design layer that leads implementation.

The dominant failure mode for AI-era platforms is the opposite: implementation-first, where a demo becomes an architecture by accident, non-determinism leaks into every layer, and the "design" is reverse-engineered from whatever the code happens to do. That path cannot reach determinism.

## Decision

We will develop **specification-first**: architecture is authored, reviewed, and accepted in `sankalpa-spec` **before** the corresponding implementation begins. Implementation (Roadmap Phase 3+) does not start until the specification reaches a stable **v1.0** (Roadmap Phase 2 gate). No normative behavior exists in code that is not first described in the spec. Implementation is treated as *one realization* of the architecture, never as its source of truth.

## Consequences

**Positive**
- Conceptual integrity is protected: the whole is designed before its parts are built.
- Parallel implementations, alternative runtimes, and third-party extensions can target a stable written contract.
- Non-determinism is contained by design rather than discovered in production.

**Negative / costs**
- Slower to a first running demo; the payoff is deferred.
- Requires cultural discipline to resist "let's just build it and see."
- Risk of over-specifying things that only implementation would reveal — mitigated by design docs, spikes, and keeping `0.x` MINOR versions breakable.

**Neutral / follow-ups**
- Defines the entire Roadmap phase structure.
- Pairs with ADR-0000 (record decisions) and the review-gate process.

## Alternatives Considered

- **Implementation-first / "docs follow code."** Rejected: sacrifices conceptual integrity and lets non-determinism metastasize; contradicts the mission.
- **Parallel spec and implementation from day one.** Rejected for the core: the two would diverge and the spec would lose authority. (Permitted *after* v1.0, where implementation validates and feeds back into the spec via RFCs.)
- **Prototype-then-freeze.** Partially adopted via design docs and spikes for *exploration*, but exploration outputs must graduate into RFCs before becoming normative.

## Compliance / Invariants touched

Load-bearing for the whole project: "Implementation may never precede architecture," "Architecture is more important than implementation." Governs [`../ROADMAP.md`](../ROADMAP.md) and [`../GOVERNANCE.md`](../GOVERNANCE.md).
