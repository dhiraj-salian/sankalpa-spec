# Book 03 · Chapter 09 — Plugin and Package Managers

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P8, P11. Companion to Book 12 (Packages), Book 11 §10 (isolation).*

> Extensibility is a foundational principle (P11), and these two Managers are its engine. The **Package Manager** installs the versioned bundles third parties ship; the **Plugin Manager** loads, isolates, and runs the executable extensions inside them. Together they let Sankalpa grow without the core growing (§Ch01 §4) — safely, because every extension is untrusted by default.

## 1. Two Managers, two concerns

- **Package Manager** — *distribution and installation*: resolve dependencies, verify signatures/provenance, install/upgrade/rollback/uninstall, and register what a Package provides (kinds, capabilities, policies, passes, planners, runtimes, channels, knowledge packs, …). Specified in depth in Book 12.
- **Plugin Manager** — *runtime lifecycle and isolation*: load an extension into an isolated host, health-check it, route AEP-interface calls to it, and contain its failures. Specified in coordination with each extension class's Book (planners Book 08, runtimes Book 06, backends Book 05, providers Book 09/11).

A Package is the *artifact*; a Plugin is the *running instance* of an executable inside it. The Package Manager decides *what is installed*; the Plugin Manager decides *what is running and how it is contained*.

## 2. Untrusted by default

Every plugin — regardless of author — is **untrusted**. This is the security posture that makes radical extensibility compatible with the invariants (§Ch01 §1):

- A plugin receives **only** an explicitly granted, attenuated set of capabilities (§Ch06 §5, P8) — never ambient access to storage, secrets, other plugins, or other tenants.
- A plugin runs in an **isolation boundary** (Book 11 §10): resource-limited, sandboxed, and unable to reach anything except through the Kernel API/Event Bus (P4).
- The Kernel **re-verifies** anything security- or determinism-relevant a plugin produces (e.g. IR from a planner, Book 04 §Ch08; passes' output, §Ch08 §5) — trust is never extended to a plugin's output.

## 3. Package Manager responsibilities

1. **Resolution** — resolve a Package's dependency graph against installed Packages and spec/AEP version constraints (Book 12 §03), rejecting unsatisfiable or conflicting graphs deterministically.
2. **Verification** — check signature and supply-chain provenance *before* install (Book 12 §04, Book 11); an unsigned or untrusted Package is not installed unless policy explicitly permits.
3. **Install lifecycle** — install/upgrade/rollback/uninstall as **reconciled** Resource operations (Book 12 §05): a `Package` is an ARM Resource (Book 02 §Ch07) with a lifecycle, not an imperative script.
4. **Capability grants at install** — a Package declares the capabilities its plugins require; installation surfaces these for authorization (least-privilege, P8). Grants are explicit and auditable, never implied by mere installation.
5. **Registration** — register provided kinds/interfaces/capabilities with the relevant Managers (Resource, Capability, Compiler, Runtime, …) so they become usable.

## 4. Plugin Manager responsibilities

1. **Load** — instantiate a plugin into an isolated host appropriate to its class, negotiating the AEP interface version (Book 12 §02, each extension AEP).
2. **Version negotiation** — confirm the host and plugin agree on a compatible interface version (e.g. planner AEP-0001, runtime AEP-0002); refuse incompatible plugins with a typed error rather than degrade.
3. **Health & readiness** — probe liveness/readiness; mark unhealthy plugins so dependent Managers (Runtime Manager §Ch07) route around them.
4. **Isolation enforcement** — apply resource limits and sandboxing (Book 11 §10); a plugin cannot exceed its granted capabilities or its resource budget.
5. **Replacement** — support swapping a plugin implementation for another satisfying the same interface **without core changes** (P11) — the concrete expression of "everything is replaceable."
6. **Fault containment** — a crashing/hanging plugin is contained: its in-flight work fails or is rescheduled per safety (§Ch07 §6, §Ch13); the Kernel and other plugins are unaffected.

## 5. The install-to-run flow

```
Package (signed) ──► Package Manager
   ├─ verify signature/provenance (Book 12 §04)        ── fail ─► reject
   ├─ resolve dependencies (Book 12 §03)               ── conflict ─► reject
   ├─ install as reconciled Package Resource (Book 12 §05)
   ├─ surface required capabilities for authorization (P8)
   └─ register provided kinds/interfaces/capabilities
                     │
When an extension is needed ──► Plugin Manager
   ├─ load into isolated host (Book 11 §10)
   ├─ negotiate AEP interface version
   ├─ grant least-privilege attenuated capabilities (§Ch06)
   ├─ health/readiness probe
   └─ route interface calls; contain faults
```

Installation never auto-escalates authority: a freshly installed Package's plugins can do nothing until explicitly granted the capabilities they declared (P8). This prevents "install = privilege" — a common supply-chain failure the design forecloses structurally.

## 6. Versioning and compatibility

- Packages declare compatibility with spec/AEP versions (Book 12 §07); the Package Manager enforces **joint consistency** across version spaces (ARM kinds Book 02 §Ch05, IR Book 04 §Ch09, AEP stability). A Package requiring `Foo/v1` or `irVersion ≥ X` is not installed where those are unavailable.
- Upgrades/rollbacks preserve the reconciled-Resource discipline and never break a running dependent without a migration path (P6, review gates).

## 7. Invariants (normative summary)

1. Package Manager owns distribution/installation; Plugin Manager owns runtime lifecycle/isolation; a Package is the artifact, a Plugin the running instance.
2. Every plugin is untrusted: least-privilege attenuated capabilities only, isolated and resource-limited, reachable only via Kernel API/Event Bus (P4, P8).
3. Packages are signature/provenance-verified before install and installed as reconciled `Package` Resources; unsatisfiable/conflicting dependency graphs are rejected deterministically.
4. Installation surfaces required capabilities for explicit authorization; installation alone grants no authority.
5. Plugins negotiate AEP interface versions; incompatible plugins are refused, not degraded; any implementation is replaceable by another satisfying the interface (P11).
6. Faults are contained to the plugin; the Kernel re-verifies security/determinism-relevant plugin output; version-space consistency is enforced at install.
