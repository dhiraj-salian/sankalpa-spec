# Book 12 — Packages & Marketplace

*Status: Draft skeleton · Nature: Normative. · Reflects: AEP-0003 (Package format); realizes P6, P11.*

## Scope
Distribution and ecosystem: the Package format, dependency and version resolution, signing and provenance, installation lifecycle, and the Marketplace. Everything extensible ships as a Package.

## Chapters
1. **`01-everything-is-installable.md`** *(Normative)* — The Package as the unit of distribution; what a Package may contain (capabilities, compilers, policies, knowledge packs, templates, workflows, providers, UI, docs, tests, prompt libraries, controllers).
2. **`02-package-manifest.md`** *(Normative)* — Manifest schema: identity, version, dependencies, provided interfaces (AEP conformance claims), capabilities required.
3. **`03-dependency-resolution.md`** *(Normative)* — Version constraints, resolution semantics, and conflict handling (lessons from Cargo/npm/Go modules).
4. **`04-signing-and-provenance.md`** *(Normative)* — Signature, supply-chain provenance, and verification before install (ties to Book 11).
5. **`05-install-lifecycle.md`** *(Normative)* — Install/upgrade/rollback/uninstall as reconciled Resource operations; capability grants at install time.
6. **`06-marketplace.md`** *(Normative)* — Discovery, publishing, conformance badges, trust tiers, and moderation.
7. **`07-compatibility-and-stability.md`** *(Normative)* — How Packages declare compatibility with spec/AEP versions (ties to versioning-and-stability).
