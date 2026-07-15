# Book 12 · Chapter 01 — Everything Is Installable

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P6, P8, P11. Companion to Book 03 §Ch09 (Plugin/Package Managers), the AEP process.*

> The Package is Sankalpa's **unit of distribution**: the versioned, signed bundle through which every extension — capabilities, compilers, policies, knowledge packs, runtimes, planners, channels, controllers, UI, and more — enters the platform. This chapter establishes the "everything is installable" principle and what a Package may contain. It is the foundation of the ecosystem that makes P11 (everything is extensible/replaceable) a lived reality rather than an aspiration.

## 1. The principle

**Everything that extends Sankalpa ships as a Package.** There is no privileged back door by which an extension is added except installing a Package (or, for local development, an equivalent governed path). This uniformity is what makes the ecosystem governable: because every extension arrives the same way, every extension is subject to the same signing, provenance, dependency, capability-grant, and lifecycle discipline (§Ch02–§Ch07). An un-packaged extension would be an ungoverned one — so the principle is a security and governance stance, not just a packaging convenience.

## 2. What a Package may contain

A Package is a bundle that may carry any combination of:

| Content | Governed by | Notes |
|---------|-------------|-------|
| **Capabilities** | Book 03 §Ch06 | reusable typed behaviors + implementations |
| **Compilers / passes / backends** | Book 05 §Ch02, §Ch05 | optimization passes, lowering backends |
| **Runtimes** | Book 06 (AEP-0002) | execution engines |
| **Planners** | Book 08 (AEP-0001) | Goals→High IR adapters |
| **Policies** | Book 11 §06 | machine-checkable rules |
| **Knowledge packs** | Book 09 §Ch07 | curated Knowledge |
| **Templates / Workflows** | Book 02 §Ch07, Book 04 | reusable High-IR templates |
| **Providers** | Book 03 §Ch04 | graph/vault/secret/observability backends |
| **UI** | Book 13 | Web Runtime extensions |
| **Documentation** | — | rendered via Web Runtime (Book 13) |
| **Tests / conformance** | Book 06 §Ch07, Book 08 §Ch07 | proof of conformance |
| **Prompt libraries** | Book 08 | planner resources (non-secret) |
| **Controllers** | Book 07 §Ch03 | third-party reconcilers for their kinds |
| **Resource kinds (schemas)** | Book 02 §Ch05 | new ARM kinds |

A Package **declares** what it provides (its manifest, §Ch02); on install, those contributions are registered with the relevant Managers (Book 03 §Ch09 §3). One Package may bundle several related contributions (a runtime plus the capabilities and knowledge to use it).

## 3. A Package is itself a Resource (P2, P6)

A Package is modeled as a **`Package` Resource** (Book 02 §Ch07), so installation is not an imperative script but a **reconciled** operation (§Ch05, Book 03 §Ch09 §3): the `Package` Resource's desired state is "installed at version V," and a controller (Book 07) drives the actual state there. Consequences:
- Install/upgrade/rollback/uninstall are lifecycle transitions (Book 02 §Ch04), observable and auditable (P5, P10).
- Packages are versioned (P6) and their state is inspectable like any Resource.
- The ecosystem is governed by the same substrate as everything else — no special-case machinery.

## 4. Installation grants no authority by itself (P8)

A foundational security rule, stated up front because it shapes the whole Book: **installing a Package does not, by itself, grant its extensions any authority.** A Package *declares* the capabilities its extensions require (§Ch02); installation *surfaces* those for explicit authorization (Book 03 §Ch09 §5, Book 11 §03). Until granted, an installed extension can do nothing (P8, no ambient authority). This forecloses the classic supply-chain failure where "install = privilege" — a malicious Package cannot act merely by being installed; it can act only within capabilities a human/policy explicitly grants (Book 11 §10 §6).

## 5. Untrusted by default

Every Package is **untrusted** (Book 11 §01 §3, §Ch04): its author is assumed potentially hostile. The discipline that makes untrusted Packages safe is the subject of the rest of this Book — signing and provenance verified before install (§Ch04), deterministic dependency resolution (§Ch03), least-privilege capability grants (§4), reconciled install with rollback (§Ch05), and — for its executable contents — the isolation and output-verification of Book 11 §10. A Package is admitted not because it is trusted but because it is *contained*.

## 6. The Package format is AEP-0003

Because the Package format is a contract third parties build against, it is an AEP (**AEP-0003**, the AEP process): the manifest schema, layout, signing, and dependency semantics are a stability-graded public interface. The rest of Book 12 is the specification body of AEP-0003. Evolving the format follows the AEP stability discipline (Experimental → Beta → Stable) so the ecosystem is not broken by format changes (§Ch07).

## 7. Invariants (normative summary)

1. Every extension to Sankalpa ships as a Package; there is no ungoverned back door for adding extensions.
2. A Package may bundle capabilities, compilers/passes/backends, runtimes, planners, policies, knowledge packs, templates/workflows, providers, UI, docs, tests, prompt libraries, controllers, and Resource kinds, and declares what it provides.
3. A Package is a `Package` Resource; install/upgrade/rollback/uninstall are reconciled, observable, auditable, versioned lifecycle operations (P2, P6).
4. Installation grants no authority by itself; it surfaces the Package's required capabilities for explicit authorization; an installed extension can do nothing until granted (P8).
5. Every Package is untrusted by default and is admitted because it is contained (signing, resolution, least-privilege, isolation), not because it is trusted.
6. The Package format is AEP-0003, a stability-graded public interface specified by this Book.
