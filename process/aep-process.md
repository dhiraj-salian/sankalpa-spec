# The AEP Process

*Status: Draft · Instrument owner: Steering Council · Template: [`../templates/aep-template.md`](../templates/aep-template.md)*

An **AEP (Architecture Extension Proposal)** defines or evolves a **public extension contract** — the stable interface a third party codes against to extend Sankalpa without forking it. AEPs are to Sankalpa's *extensibility* what RFCs are to its *internals*: a promise to the ecosystem.

## What needs an AEP

Anything that ships an interface an external author implements or consumes:

- **Planner interface** — how a Planner receives Goals and returns High IR.
- **Runtime interface** — how a Runtime executes runtime-specific graphs and reports Events.
- **Compiler/lowering interface** — how a backend lowers Low IR to a target.
- **Package format** — the manifest, layout, signing, and dependency semantics of a Package.
- **Channel adapter interface** — how a transport delivers/receives messages.
- **Policy provider, Knowledge provider, Secret provider** interfaces.
- **Controller SDK** surface for third-party Controllers.

## Why AEPs are separate from RFCs

Extension contracts have **stricter stability obligations** than internal designs: once external authors depend on them, breaking changes cost the whole ecosystem, not just the core. AEPs therefore carry an explicit **stability level** and a **compatibility guarantee**, and their FCP is longer.

## Stability levels

| Level | Guarantee |
|-------|-----------|
| **Experimental** | May change or vanish in any release. Opt-in; loud warnings. |
| **Beta** | Breaking changes only across minor versions, with a deprecation cycle. |
| **Stable** | Backward-compatible within a major version. Removal requires a major bump + migration path. |

## Lifecycle

```
Draft ──► Proposed ──► FCP(15 working days) ──► Accepted@Experimental
      ──► (adoption + feedback) ──► Beta ──► Stable ──► (Deprecated ──► Removed)
```

Promotion between stability levels is itself an AEP amendment requiring evidence: real adopters, conformance tests, and a stable test suite.

## Mandatory contents

Beyond the standard 15-section design shape, every AEP MUST specify:

- The **exact interface** (types, methods, wire format, error model).
- **Versioning & negotiation** — how a host and an extension agree on a version.
- **Conformance tests** — the suite an implementation must pass to claim compliance.
- **Security model** — the capabilities the extension is granted and denied (ties to Book 11).
- **Reference implementation** pointer (once implementation begins).

## Numbering & location

- File: `aeps/NNNN-short-slug.md`.
- `aeps/0000-aep-purpose-and-process.md` defines the AEP concept itself.
