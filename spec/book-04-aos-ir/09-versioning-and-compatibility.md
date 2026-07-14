# Book 04 · Chapter 09 — Versioning and Compatibility

*Nature: **Normative**. · Reflects: RFC-0001, ADR-0001; realizes principle P6 and IR-P5. Companion to [`../../process/versioning-and-stability.md`](../../process/versioning-and-stability.md) and Book 02 §Ch05.*

> AOS IR must remain interpretable for as long as executions are reproducible — years. This chapter defines how the IR schema evolves without breaking that promise. It governs the IR *body* version, which is distinct from the `IRModule` *kind*'s ARM `apiVersion` (Book 02 §Ch05 §5).

## 1. Three distinct version spaces (do not conflate)

1. **IR schema version** — the version of the IR language itself (node kinds, type language, effect lattice, canonical form, hash algorithm). Recorded in every module's `irVersion` (Ch 03, Ch 04) as `MAJOR.MINOR.PATCH`.
2. **`IRModule` kind version** — the ARM `apiVersion` of the `IRModule` Resource envelope (Book 02 §Ch05). Independent of the body's `irVersion`.
3. **Module content hash** — the identity of a *specific* module (Ch 07). Not a version; an identity.

This chapter is about (1).

## 2. SemVer for the IR schema

- **MAJOR** — a change that would make a previously valid module invalid, change the meaning of an existing construct, or change the canonical form/hash such that identity shifts. Requires an RFC (P12) and a compatibility path (§4).
- **MINOR** — backward-compatible additions: a new optional node attribute, a new Capability-expressible pattern, a new `Custom` effect kind, a new primitive type. Old modules remain valid and unchanged in meaning.
- **PATCH** — clarifications and verifier/serializer fixes that do not change which modules are valid or their canonical form.

IR-P5 is the backbone: **the meaning of a construct MUST NOT silently change.** A change to meaning is always MAJOR and always explicit.

## 3. Compatibility guarantees

- **Additive-by-default (P11).** New capability is added via new optional constructs, new Capabilities, or new `Custom` effects — not by mutating existing constructs. A producer targeting `irVersion = X.Y` MUST keep validating and executing under `X.(Y+n)`.
- **Forward tolerance is bounded.** A consumer (verifier/compiler/runtime backend) that encounters a *minor* it does not fully know MUST reject constructs it does not understand rather than guess — deny-by-default (IR-P3) extends to unknown constructs. It MUST NOT silently ignore an unknown node or effect, because that could drop meaning or an effect Policy needed to see.
- **No silent hash drift.** Because identity is the content hash (Ch 07), a serialization change that altered hashes of unchanged modules would break replay and caching. Such a change is MAJOR and handled by §4/§5.

## 4. Breaking changes and lowering-compatibility

When a MAJOR IR change is unavoidable, reproducibility of *existing* work must still be preserved (P6). The mechanism:

1. **Version-tagged modules.** Every stored module records the exact `irVersion` (and hash algorithm, §5) its body conforms to. Nothing is reinterpreted under a different version.
2. **Multi-version verifier/compiler.** The toolchain MUST be able to verify and lower modules of any still-supported `irVersion`, not only the latest. A `v2` compiler still understands `v1` modules for as long as `v1` is supported.
3. **Lowering-compatibility shims (RFC-0001 §13).** A previously produced RuntimeGraph (or Low IR) MUST remain re-executable. Where a MAJOR change alters lowering, a shim preserves the observable behavior of already-lowered artifacts so historical executions replay identically.
4. **Migration, not mutation.** Old modules are not rewritten in place; if a new storage version is desired, migration follows the reconciled, gated process of Book 02 §Ch05 §4, and the original hashes/executions remain reproducible.

## 5. Hash-algorithm agility

The content-address hash (Ch 07 §3) is itself versioned in IR metadata so it can be migrated if a hash function weakens:

- A module records which hash algorithm produced its identity.
- Introducing a new algorithm is MINOR (additive) for new modules; existing modules keep their existing-algorithm identity.
- The store MAY carry dual identities during a transition; references resolve under the algorithm recorded with the reference. Retiring an old algorithm is MAJOR and follows §4.

## 6. Deprecation

- A construct is deprecated by marking it in the schema and the [Glossary](../../GLOSSARY.md), emitting verifier warnings, and documenting the replacement. It remains valid for at least one MINOR before any MAJOR removal (mirrors versioning-and-stability §Stability guarantees).
- Producers SHOULD stop emitting deprecated constructs; consumers MUST keep accepting them until removal.

## 7. Stability trajectory

Per the roadmap, IR is `Experimental` through Roadmap Phase 5 and hardens toward the spec's v1.0 (Phase 2 gate). Pre-1.0, MINOR MAY break (Book 02 §Ch05 §2.2 rationale applies to IR too); post-1.0, the guarantees above are firm. `IRModule` schemas and AEPs that reference IR (planner AEP-0001, runtime AEP-0002) declare which `irVersion` range they support, and the Package Manager (Book 12 §07) enforces joint consistency across version spaces.

## 8. Invariants (normative summary)

1. IR schema version (`irVersion`), `IRModule` kind version, and module hash are distinct and never conflated.
2. Meaning never changes silently: any change to an existing construct's meaning or to canonical form/hash is MAJOR and RFC-gated.
3. Evolution is additive-by-default; unknown constructs/effects are rejected, never silently ignored.
4. Existing modules and lowered artifacts remain reproducible across MAJOR changes via version-tagging, multi-version tooling, and lowering-compatibility shims.
5. The hash algorithm is versioned and migratable without breaking existing identities.
6. Deprecation precedes removal by at least one MINOR; consumers accept deprecated constructs until removal.
