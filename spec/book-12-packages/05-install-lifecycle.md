# Book 12 · Chapter 05 — Install Lifecycle

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P5, P8. Part of AEP-0003. Companion to Book 07 (Controllers), Book 02 §Ch04 (Lifecycle).*

> Installing a Package is not running a script — it is **reconciling a `Package` Resource** toward a desired state (Book 03 §Ch09 §3). This chapter specifies the install/upgrade/rollback/uninstall lifecycle as governed, reconciled, reversible operations. Treating installation as reconciliation is what gives the ecosystem the same reliability, observability, and rollback guarantees as everything else in Sankalpa.

## 1. Installation is reconciliation

A `Package` Resource (Book 02 §Ch07) has a desired state ("installed at version V, with these grants") that a **package install controller** (Book 07) drives the actual state toward. Consequences (inheriting the whole controller model, Book 07):
- Install is **idempotent** and **level-triggered**: re-reconciling an installed Package is a no-op; a partial install is completed by the next reconcile (Book 07 §Ch05 §4).
- Every transition **emits Events** (P5) and is **observable/auditable** (P10, Book 11 §09).
- Failure **rolls back** to a consistent state (§4) via the same recovery discipline as any controller (Book 07 §Ch05).

There is no bespoke installer with its own failure modes; installation is the reconciliation model applied to Packages.

## 2. The lifecycle operations

```
Install:   resolve (§Ch03) → verify (§Ch04) → surface & grant capabilities (§3) →
           register contributions (Book 03 §Ch09 §3) → Active
Upgrade:   re-resolve jointly-consistent (§Ch03 §6) → verify → apply delta reconciled → Active
Rollback:  revert to the prior known-good version (§4)
Uninstall: deregister contributions → run finalizers (release grants, GC provided kinds) → removed
```

Each maps onto the ARM lifecycle (Book 02 §Ch04): a `Package` moves `Pending → Progressing → Active`, and through `Terminating` on uninstall, with finalizers ensuring clean teardown.

## 3. Capability grants at install (P8)

Installation **surfaces the Package's declared required capabilities** (§Ch02 §4) for **explicit authorization** (Book 03 §Ch09 §5, Book 11 §03):
- A human/policy grants the least-privilege capabilities the Package needs, or refuses. The grants are **attenuated** (Book 11 §03 §4) to exactly what is needed and are recorded on the install.
- Until granted, the installed contributions **can do nothing** (install ≠ privilege, §Ch01 §4). A runtime installed but not granted secret-materialization capability simply cannot materialize secrets.
- Grants are **revocable** (Book 11 §03 §5): revoking a Package's grants disables its authority without uninstalling it — a fast containment lever if a Package misbehaves.

This is the point at which the "untrusted by default, authority only by explicit grant" posture (§Ch01 §4–5) becomes concrete.

## 4. Rollback and reversibility

Every install/upgrade is **reversible**:
- A failed install/upgrade **rolls back** to the prior consistent state (Book 07 §Ch05 §4) — the operation is all-or-nothing at the plan level (§Ch03 §5), so a failure never leaves a half-installed, inconsistent Package set.
- An upgrade that proves faulty can be **rolled back** to the prior known-good version, re-resolving joint consistency (§Ch03 §6). Because Packages are versioned (§Ch07) and installs are reconciled, rollback is a normal lifecycle operation, not an emergency hack.
- Reversibility is what makes it safe to adopt ecosystem Packages: a bad install/upgrade is *undoable*, cleanly.

## 5. Uninstall and clean teardown

Uninstall follows the finalizer-gated deletion path (Book 02 §Ch04 §4, Book 07 §Ch03 §4):
- The install controller **deregisters** the Package's contributions from their Managers (Book 03 §Ch09), **revokes** its capability grants (Book 11 §03 §5), and **garbage-collects** provided kinds/resources per ownership (Book 02 §Ch04 §5) — all through finalizers, idempotently.
- A finalizer that cannot complete (an external teardown fails) keeps the Package `Terminating` with a surfaced condition (Book 02 §Ch04 §4.3) rather than silently leaking — the same discipline as any Resource deletion.
- Uninstalling a Package that others depend on is rejected or requires resolving the dependents first (§Ch03) — the dependency graph is respected on teardown as on install.

## 6. Everything is observable and governed (P5, P9, P10)

- Every lifecycle operation emits Events (`package.installed`, etc., Book 14 §Ch01) and is auditable (Book 11 §09): "who installed/upgraded/removed what, when, with which grants" is always answerable.
- What may be installed/upgraded, by whom, is **policy-governed** (§Ch04 §5, Book 11 §06). The lifecycle is not an ops side channel; it is governed like every consequential action.

## 7. Invariants (normative summary)

1. Installation is reconciliation of a `Package` Resource — idempotent, level-triggered, Evented, auditable, and rollback-capable — not a bespoke script.
2. Install/upgrade/rollback/uninstall map onto the ARM lifecycle with finalizer-gated teardown.
3. Installation surfaces declared required capabilities for explicit least-privilege authorization; contributions can do nothing until granted; grants are attenuated and revocable (P8).
4. Every install/upgrade is reversible: failures roll back all-or-nothing; faulty upgrades roll back to prior known-good, re-resolving joint consistency.
5. Uninstall deregisters contributions, revokes grants, and GCs provided kinds via idempotent finalizers; incomplete teardown surfaces a condition rather than leaking; dependents are respected.
6. Every lifecycle operation is Evented, auditable, and policy-governed (P5, P9, P10).
