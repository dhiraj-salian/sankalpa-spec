# Book 12 · Chapter 03 — Dependency Resolution

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P6, P11. Part of the specification body of **AEP-0003**.*

> Packages depend on Packages, and on specific spec/IR/AEP versions. This chapter specifies how the Package Manager resolves a dependency graph deterministically, how it handles conflicts, and the rule that an unsatisfiable graph is rejected cleanly rather than installed partially. Predictable resolution is what keeps the ecosystem composable without descending into dependency chaos.

## 1. What is resolved

At install (or upgrade), the Package Manager resolves the transitive dependency graph of the target Package against:
- **Installed Packages** and their versions (§Ch02 §3),
- the declared **version constraints** on each dependency (§Ch02 §3), and
- the **compatibility constraints** — spec/IR/AEP version ranges (§Ch02 §5, Book 04 §Ch09) — which are themselves resolution constraints.

Resolution produces either a **consistent install plan** (a set of Packages at specific versions satisfying all constraints) or a **rejection** with an explanation of the conflict (§4). It never produces a partial or best-effort result.

## 2. Deterministic resolution

Resolution MUST be **deterministic**: the same target, the same installed set, and the same available versions always yield the same result. Rationale (mirroring the determinism discipline everywhere in Sankalpa): reproducible installs are debuggable, auditable, and portable — a resolution that varied run-to-run would make deployments unreproducible. Determinism is achieved by a defined resolution algorithm with defined tie-breaking (e.g. prefer the highest version satisfying all constraints, with a stable order), learning from the package-manager lineage (Cargo/npm/Go modules — Book 12 references prior art).

## 3. Version constraints and SemVer

Dependencies are constrained by SemVer ranges (§Ch02, P6):
- Constraints express compatibility intent ("compatible with 2.x", "at least 1.4"), and resolution finds versions satisfying *all* constraints simultaneously.
- Because Packages are SemVer-versioned (§Ch07) and interfaces evolve additively (P11), a compatible range is usually satisfiable; a breaking major version is an explicit, opt-in constraint change.
- **Compatibility constraints** (spec/IR/AEP, §Ch02 §5) are resolved *jointly* with package dependencies (Book 03 §Ch09 §6): a Package version is a candidate only if it is compatible with the installed platform's version spaces *and* the other selected Packages.

## 4. Conflicts are rejected, not papered over

When constraints cannot be simultaneously satisfied — two Packages require incompatible versions of a shared dependency, or a required IR version is unavailable — resolution **fails cleanly**:
- The Package Manager **rejects** the install with a **structured, explainable** diagnostic (which constraints conflict, which Packages/versions are involved) — mirroring the explainability discipline of policy and compiler diagnostics (Book 11 §06 §5, Book 05 §Ch07).
- It MUST NOT install a partial or inconsistent set, and MUST NOT silently downgrade/override a constraint to force a fit. A conflicting graph is a *rejection*, not a *best guess* — because a silently-forced fit would break an extension in ways discovered only at runtime.
- The rejection is actionable: it tells the operator what would need to change (a different version, an upgrade) to resolve the conflict.

## 5. Reconciled, transactional install of the plan

Once a consistent plan is resolved, its application is **reconciled** (§Ch05, Book 03 §Ch09 §3): the Package Manager drives the install plan to its desired state as `Package` Resource operations. If any step fails, the operation reaches an explained terminal state and rolls back (§Ch05 §4) — the install is all-or-nothing at the plan level, so a failed install does not leave a half-resolved, inconsistent Package set. This uses the same reconciliation/rollback discipline as everything else (Book 07), not a bespoke installer.

## 6. Upgrades and the graph over time

- An **upgrade** re-resolves the graph with the new target version and applies the delta reconciled (§Ch05), preserving joint consistency (§3) — an upgrade that would break a dependent's constraints is rejected, not forced.
- Because resolution is deterministic (§2) and Packages are versioned (§Ch07), the dependency graph's evolution is reproducible and auditable: you can always determine why a given version set is installed.

## 7. Invariants (normative summary)

1. The Package Manager resolves the transitive dependency graph against installed Packages, declared version constraints, and spec/IR/AEP compatibility — jointly.
2. Resolution is deterministic: identical inputs yield identical results, for reproducible, auditable, portable installs.
3. Version constraints are SemVer ranges resolved to satisfy all constraints simultaneously; compatibility constraints are resolved jointly with package dependencies.
4. An unsatisfiable or conflicting graph is rejected with a structured, actionable diagnostic; resolution never installs a partial set or silently overrides a constraint.
5. A resolved plan is applied reconciled and all-or-nothing; a failed install rolls back rather than leaving an inconsistent set.
6. Upgrades re-resolve jointly-consistently and are rejected rather than forced when they would break a dependent; graph evolution is reproducible and auditable.
