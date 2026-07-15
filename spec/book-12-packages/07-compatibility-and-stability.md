# Book 12 · Chapter 07 — Compatibility and Stability

*Nature: **Normative**. · Reflects: ADR-0001; realizes principles P6, P11. Part of AEP-0003. Companion to `process/versioning-and-stability.md`, Book 02 §Ch05, Book 04 §Ch09.*

> Packages live for years across an evolving platform. This chapter specifies how a Package declares compatibility with Sankalpa's multiple version spaces, how the Package Manager enforces joint consistency, and how the Package format itself (AEP-0003) evolves without breaking the ecosystem. It ties the Package layer into the project-wide versioning discipline.

## 1. Packages version by SemVer (P6)

A Package versions its own contents by **SemVer** (§Ch02 §1, `process/versioning-and-stability.md`): MAJOR for a breaking change to what it provides, MINOR for backward-compatible additions, PATCH for fixes. A published version is **immutable** (§Ch06 §3); change means a new version. This gives dependents (§Ch03) stable, reproducible targets and lets resolution reason about compatibility (§Ch03 §3).

## 2. The multiple version spaces a Package touches

A Package sits at the intersection of Sankalpa's **independent version spaces** (Book 04 §Ch09 §1), and its manifest declares compatibility with each (§Ch02 §5):
- **Spec version** — which `sankalpa-spec` version(s) it targets (`process/versioning-and-stability.md`).
- **ARM kind versions** — the `apiVersion`s of kinds it provides or consumes (Book 02 §Ch05).
- **AOS IR version(s)** — for anything touching IR (a runtime, a compiler pass, a planner) (Book 04 §Ch09).
- **AEP version(s)** — the extension interfaces it implements (planner AEP-0001, runtime AEP-0002, package AEP-0003).

These spaces evolve **independently but must be jointly consistent** (Book 04 §Ch09 §1, Book 03 §Ch09 §6): a Package is installable only where *all* its declared compatibility constraints hold.

## 3. Joint consistency enforcement

The Package Manager enforces **joint consistency** at install/upgrade (Book 03 §Ch09 §6, §Ch03 §3):
- A Package requiring `irVersion ≥ X`, or `AEP-0002 v2`, or a kind at `v1` is installable only where those are available — otherwise resolution rejects it (§Ch03 §4) with an explainable diagnostic.
- Joint consistency spans *both* the platform's version spaces *and* the other selected Packages: a version set is valid only if every member is mutually compatible and platform-compatible. This is what prevents an incompatible Package from installing and failing mysteriously — incompatibility is a resolution-time rejection, not a runtime surprise.

## 4. Additive-by-default evolution (P11)

All the version spaces evolve **additively by default** (P6/P11, mirrored from ARM Book 02 §Ch05 and IR Book 04 §Ch09):
- New capability is added via new optional manifest fields, new provided-interface versions, or new Package versions — not by mutating existing contracts.
- A Package targeting a given spec/IR/AEP version keeps working as those advance by MINOR (backward-compatible). A breaking (MAJOR) change in a version space requires the affected Packages to declare the new range explicitly — an opt-in, not a silent break.

Additive evolution is what lets the ecosystem persist across years of platform change: most upgrades don't break most Packages, and the ones that would are caught at resolution (§3).

## 5. The Package format itself is stability-graded (AEP-0003)

AEP-0003 — the manifest schema, layout, signing, and dependency semantics (§Ch01 §6) — is itself an AEP with a stability level (the AEP process):
- The format evolves additively (new optional manifest fields are MINOR); a breaking format change is a MAJOR AEP-0003 bump with a negotiation/migration path so existing Packages remain installable (Book 04 §Ch09 §4 analog).
- The Package Manager understands a **range** of AEP-0003 versions, not only the latest, so Packages published against an older format version keep installing until that version is retired (with deprecation, `process/versioning-and-stability.md`).

The format that carries the whole ecosystem is thus held to the strictest stability discipline — because breaking *it* would break *every* Package.

## 6. Deprecation and retirement

- A Package version, an interface version, or a format version is **deprecated** before removal (with warnings and a documented successor), for at least one MINOR (`process/versioning-and-stability.md`), so dependents have a migration window.
- Retirement follows the same reconciled, auditable discipline as any lifecycle change; nothing the ecosystem depends on is removed without a deprecation period and a migration path (P6).

## 7. Invariants (normative summary)

1. Packages version by SemVer with immutable published versions, giving dependents stable, reproducible targets.
2. A Package declares compatibility with the spec, ARM-kind, IR, and AEP version spaces it touches; these evolve independently but must be jointly consistent.
3. The Package Manager enforces joint consistency at install/upgrade across platform version spaces and co-selected Packages; incompatibility is a resolution-time rejection, not a runtime failure.
4. All version spaces evolve additively by default; a Package keeps working across MINOR advances, and MAJOR breaks require explicit opt-in range declarations.
5. The Package format (AEP-0003) is itself stability-graded, evolves additively, and the Package Manager supports a range of format versions so older Packages keep installing until retirement.
6. Package/interface/format versions are deprecated with a migration window before removal; nothing depended-upon is removed without deprecation and a migration path (P6).
