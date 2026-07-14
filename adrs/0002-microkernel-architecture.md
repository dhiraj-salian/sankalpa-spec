# ADR-0002: Microkernel architecture for the Kernel

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **Date** | 2026-07-14 |
| **Deciders** | Founding maintainers |
| **Domain / Book** | Kernel / Book 03 |
| **Related** | ADR-0001, RFC-0001, Book 03, Book 12 (Packages) |

## Context

Sankalpa must support a large, evolving set of planners, runtimes, compilers, channels, policies, and knowledge providers — most of them written by third parties and none of them known in advance. It must also uphold hard invariants (secret non-leakage, policy enforcement, capability-based authority, auditability) *regardless* of which extensions are installed. These two forces — radical extensibility and non-negotiable invariants — are in tension. A monolith cannot safely admit untrusted extensions; a fully decentralized mesh cannot enforce global invariants.

The microkernel (a.k.a. plug-in) architecture resolves this tension and is proven at scale: it is how OS microkernels, LLVM's pass/target plugins, VS Code's extension host, and Kubernetes' API-server-plus-controllers all achieve "small trusted core + many replaceable parts."

## Decision

We will structure the Sankalpa **Kernel** as a **microkernel**: a small, trusted core that owns orchestration and the invariants, surrounded by **plugins** (planners, runtimes, compilers, channels, providers, controllers) loaded through AEP-defined interfaces. Nothing communicates directly; all interaction flows through the **Kernel API** (synchronous) or the **Event Bus** (asynchronous). The Kernel hosts its Managers (Resource, Capability, Scheduler, Runtime, Compiler, Plugin, Package, Security, Knowledge, Experience, Policy, Lifecycle, Controller Runtime, Observability, User, Session, Approval, Secret Broker) as the trusted core; everything else is replaceable.

## Consequences

**Positive**
- Extensibility without compromising invariants: plugins get only granted capabilities and never bypass the core.
- Replaceability: any runtime/planner/compiler can be swapped without touching the core.
- Testability & fault isolation: plugins can be sandboxed, mocked, and crash-contained.
- Clear ownership: the core team owns a small surface; ecosystems own the edges.

**Negative / costs**
- Indirection cost: mediated communication adds latency and design complexity vs. direct calls.
- The Kernel API and Event schema become critical, hard-to-change contracts (governed as Stable interfaces).
- Risk of a "distributed monolith" if the core grows to know too much about plugins — actively resisted; the core must remain policy/mechanism, not plugin-specific logic.

**Neutral / follow-ups**
- Drives the Plugin Manager, Package Manager, and the AEP process.
- Necessitates precise Event Bus delivery/ordering semantics (Book 03/14).

## Alternatives Considered

- **Monolith with compile-time modules.** Rejected: cannot admit third-party/untrusted extensions safely; every new runtime forks the core.
- **Fully decentralized actor mesh (no privileged core).** Rejected: no place to enforce global invariants (secrets, policy, audit); determinism and governance become impossible.
- **Layered architecture without a plugin host.** Rejected: layering organizes code but does not provide the load/isolate/replace machinery extensibility demands.

## Compliance / Invariants touched

Realizes "The Kernel owns orchestration," "Nothing communicates directly," "Everything is replaceable," and is the enforcement point for every security invariant in [`../SECURITY.md`](../SECURITY.md).
