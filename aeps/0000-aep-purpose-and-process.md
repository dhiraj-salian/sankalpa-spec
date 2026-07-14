# AEP-0000: AEP purpose and process

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **Stability** | — (meta) |
| **Authors** | Founding maintainers |
| **Extension class** | — |
| **Created** | 2026-07-14 |

## Purpose

This meta-AEP defines what an Architecture Extension Proposal *is* and why it is a distinct instrument from RFCs and ADRs. The operational process lives in [`../process/aep-process.md`](../process/aep-process.md).

An **AEP is a promise to the ecosystem.** Where an RFC decides how Sankalpa works internally and an ADR records a single decision, an AEP defines a **public interface that third parties implement or consume** and commits the project to a **stability level** for it. Because breaking an extension contract harms everyone who built on it — not just the core — AEPs carry stricter obligations than RFCs: an explicit stability level, a versioning-and-negotiation scheme, a conformance suite, and a security/capability model.

## Why separate from RFCs

1. **Different blast radius.** Internal designs affect the core; extension contracts affect the entire ecosystem.
2. **Different stability discipline.** AEPs graduate Experimental → Beta → Stable on evidence (real adopters, passing conformance tests), independent of the internal design's maturity.
3. **Different audience.** AEPs are read primarily by *external* authors and must be self-contained and precise about the wire/type contract.

## The extension surface

The extension classes governed by AEPs (each its own AEP when authored):

- **Planner** — Goals → High IR.
- **Runtime** — Low IR / lowered graph → Execution + Events.
- **Compiler backend** — Low IR → target lowering.
- **Package format** — manifest, layout, signing, dependency semantics.
- **Channel adapter** — transport in/out.
- **Policy / Knowledge / Secret providers** — pluggable backends for those subsystems.
- **Controller SDK** — third-party Controllers.

## Obligations of every AEP

Beyond the standard 15-section design shape: a precise interface definition, lifecycle & version negotiation, capability/isolation model (Book 11), the data contract in terms of ARM/IR/Events, a conformance suite, an explicit stability level with promotion criteria, and a reference implementation pointer. See [`../templates/aep-template.md`](../templates/aep-template.md).

## Relationship to the Package ecosystem

Extensions ship as Packages (Book 12) and are discovered through the Marketplace. An AEP defines the *interface*; the Package format (its own AEP) defines the *distribution*. Conformance to an AEP at a given stability level is a publishable, verifiable property of a Package.
