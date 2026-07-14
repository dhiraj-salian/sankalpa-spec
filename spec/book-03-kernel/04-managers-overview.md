# Book 03 · Chapter 04 — Managers Overview

*Nature: **Informative** (map); each Manager's normative contract is its own chapter or Book. · Reflects: ADR-0002; realizes P4, P8, P9, P11.*

> The Kernel's trusted core is a set of **Managers**, each owning one domain and exposing its function only through the Kernel API (§Ch02) and Event Bus (§Ch03). This chapter is the map: what each Manager owns, its boundary, and where it is specified. It is the antidote to a "god core" — the core is decomposed into cohesive, individually-reasoned services.

## 1. What a Manager is

A Manager is a core component that:
- Owns a **single domain** (resources, capabilities, scheduling, …) — high cohesion.
- Exposes its function **only** through Kernel API verbs and Events (P4); Managers do not reach into each other's storage.
- Enforces the invariants relevant to its domain (a Manager is *trusted core*, unlike a plugin).
- Is itself **not replaceable by third parties** (that is the plugin/core litmus test, §Ch01 §4) — though its *backing providers* often are (e.g. the Knowledge Manager is core, but its graph backend is a provider plugin).

Managers collaborate by the same rules everyone else follows: Kernel API calls and Events. This keeps the core internally decoupled and independently testable, and means the core is not a monolith but a small microservice-like federation under one trust boundary.

## 2. The Managers

Grouped by role. "Spec" points to the authoritative chapter/Book.

### 2.1 Resource & lifecycle plane
| Manager | Owns | Spec |
|---------|------|------|
| **Resource Manager** | Admission, validation, storage, versioning, and Event emission for all Resources. The heart of ARM. | §Ch05, Book 02 |
| **Lifecycle Manager** | Phase transitions, finalizers, deletion, retention/GC across kinds. | §Ch10, Book 02 §Ch04 |
| **Controller Runtime** | Hosting, scheduling, and rate-limiting of Controllers (the reconcilers). | §Ch10, Book 07 |

### 2.2 Execution plane
| Manager | Owns | Spec |
|---------|------|------|
| **Capability Manager** | Registry, granting, attenuation, revocation, and invocation of Capabilities (P8). | §Ch06, Book 11 §03 |
| **Scheduler** | Placement and admission of work (compilations, executions) under capacity/policy. | §Ch07 |
| **Runtime Manager** | Lifecycle and selection of Runtime plugins; runtime chosen *after* planning. | §Ch07, Book 06 |
| **Compiler Manager** | Orchestrates the compile pipeline (verify → optimize → policy → lower). | §Ch08, Book 05 |

### 2.3 Extension plane
| Manager | Owns | Spec |
|---------|------|------|
| **Plugin Manager** | Loading, isolating, versioning, health, and replacement of plugins (all classes). | §Ch09, Book 06/08 |
| **Package Manager** | Install/upgrade/rollback/uninstall of Packages; dependency resolution; signing. | §Ch09, Book 12 |

### 2.4 Security plane
| Manager | Owns | Spec |
|---------|------|------|
| **Security Manager** | Authn/authz orchestration; the enforcement point for capability checks and policy at the API. | §Ch11, Book 11 |
| **Policy Engine** | Machine-checkable policy: control-plane policy (at the API) and the compiler's policy-validation pass hook. | §Ch11, Book 11 §06, Book 05 §04 |
| **Secret Broker** | Sole custodian of secrets; issues/resolves references; materializes values only at execution (P7). | §Ch11, Book 11 §04 |
| **Approval Engine** | Human-authorization gates; approval requests/decisions. | §Ch11, Book 11 §07 |
| **User Manager** | Identities, roles, membership. | §Ch11, Book 11 §08 |
| **Session Manager** | Authenticated interaction contexts; correlation; expiry. | §Ch11, Book 11 §08, Book 13 §05 |

### 2.5 Knowledge & experience plane
| Manager | Owns | Spec |
|---------|------|------|
| **Knowledge Manager** | The vault ⇄ graph knowledge system and its sync. | §Ch11, Book 09 |
| **Experience Manager** | Capture, storage, and feedback of Experience; determinization signals. | §Ch11, Book 10 |

### 2.6 Observability plane
| Manager | Owns | Spec |
|---------|------|------|
| **Observability Manager** | Metrics, tracing, logging (secret-free), and the audit trail derived from Events. | §Ch12, Book 14 |

## 3. Boundaries and anti-patterns

- **No Manager reaches into another's storage.** The Resource Manager owns storage; other Managers persist their domain state as Resources through it, or own a domain store fronted only by their own Kernel API verbs.
- **No Manager embeds plugin-specific knowledge.** The Runtime Manager knows the *Runtime interface* (AEP-0002), not "n8n"; the Compiler Manager knows the *pass/backend interface*, not a specific backend.
- **One writer per concern.** Each piece of state has exactly one owning Manager/Controller (Book 02 §Ch03 §4); overlapping ownership is a design defect to be resolved before shipping.
- **Failure isolation.** A Manager's failure degrades its domain, not the whole Kernel, wherever the invariants allow (§Ch13).

## 4. Reading order

The execution-plane and lifecycle-plane Managers get full chapters next (§Ch05–§Ch10) because they are the Kernel's spine. The security, knowledge, experience, and observability Managers are specified in depth in their own Books (11, 09, 10, 14) and summarized here in §Ch11–§Ch12 to keep Book 03 the authoritative *map* without duplicating those Books.
