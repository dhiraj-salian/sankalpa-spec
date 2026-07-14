# Versioning & Stability

*Status: Draft*

Sankalpa versions the **specification** as a product. This document defines the status labels used on documents, how SemVer applies to normative meaning, and the stability guarantees the project makes.

## Document status labels

Every document carries one of:

| Label | Meaning |
|-------|---------|
| **Draft** | Being written; may change wholesale. |
| **Proposed** | Complete and under review. |
| **Accepted** | Consensus reached (RFC/ADR/AEP). |
| **Final** | Accepted *and* reflected across the spec. |
| **Deprecated** | Discouraged; scheduled for removal. |
| **Superseded** | Replaced; links to successor. |

## Specification SemVer

The `spec/` corpus is released under `MAJOR.MINOR.PATCH` where versions describe *meaning*, not word count:

- **MAJOR** — a change that would break a conforming implementation or extension (e.g., removing an IR op, changing a Resource's required fields, altering a Kernel API contract).
- **MINOR** — backward-compatible additions (a new optional field, a new Resource kind, a new AEP at Experimental).
- **PATCH** — clarifications and editorial fixes with no change in conforming behavior.

Pre-1.0 (`0.x`), MINOR may break: the architecture is still hardening (Roadmap Phase 1–2). **v1.0** is the stability watershed after which implementation (Phase 3+) may rely on the spec.

## Stability guarantees (post-1.0)

1. **Stable interfaces** (AEP `Stable`, core Kernel API, ARM core fields, AOS IR) do not break within a MAJOR version.
2. **Deprecation before removal.** Anything removed is first `Deprecated` for at least one MINOR version with a documented migration path.
3. **Additive by default.** New capability is added behind optional fields / new kinds, not by mutating existing contracts.
4. **Experimental means experimental.** Anything labeled Experimental (or AEP-Experimental) carries no stability guarantee and is opt-in.

## Conformance

Post-1.0, an implementation or extension may claim conformance to a specific spec version if it passes that version's conformance suite (defined per AEP and per subsystem). Conformance is always *to a version*, never to "latest."

## Change traceability

Every version bump references the RFCs/ADRs that justify it in [`../CHANGELOG.md`](../CHANGELOG.md). The chain **spec text → RFC/ADR → CHANGELOG entry → version** must be unbroken.
