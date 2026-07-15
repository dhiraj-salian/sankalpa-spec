# Book 09 · Chapter 07 — Knowledge Packs

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P6, P7, P11. Companion to Book 12 (Packages & Marketplace), §Ch02 (taxonomy).*

> Knowledge is distributable. A **Knowledge Pack** is a Package (Book 12) whose payload is Knowledge — curated facts, documentation, runbooks, playbooks, and relationships that a workspace can install to bootstrap or extend what the system knows. This chapter specifies how Knowledge is packaged, installed, provenanced, and kept safe, so that shared Knowledge is as governed as authored Knowledge.

## 1. Why package Knowledge

Much valuable Knowledge is not workspace-specific: how a common SaaS API works, best-practice runbooks for a technology, a domain's conventions. Rather than every workspace re-authoring it, such Knowledge can be **curated once and distributed** as a Knowledge Pack (Book 12 §01 lists knowledge packs among Package contents). This is the Knowledge expression of the platform's "everything is installable" ecosystem (P11, Book 12) — the same way capabilities, compilers, and policies are distributed.

## 2. A Knowledge Pack is a Package

A Knowledge Pack is not a special mechanism — it is a **Package** (Book 12) whose contents are Knowledge units (vault notes + their graph structure, §Ch03–§Ch04). It therefore inherits the entire Package discipline (Book 12):
- **Manifest** (Book 12 §02) — identity, version, dependencies, and the Knowledge it provides.
- **Signing & provenance** (Book 12 §04) — verified before install; an unsigned/untrusted pack is not installed unless policy permits.
- **Dependency resolution & versioning** (Book 12 §03, §07) — a pack declares compatibility with spec versions and may depend on other packs.
- **Reconciled install lifecycle** (Book 12 §05) — install/upgrade/rollback/uninstall as reconciled `Package` Resource operations.

Nothing about Knowledge Packs bypasses Package governance; a Knowledge Pack is governed exactly as a code-bearing Package, because imported Knowledge shapes planning (§Ch06) and is therefore security-relevant.

## 3. Provenance survives installation (P10, P6)

When a Knowledge Pack is installed, the imported Knowledge's **provenance records its pack origin** (§Ch02 §3): each unit is marked as sourced from Pack P, version V, publisher X — not silently merged as if locally authored. Consequences:
- Planning weighs installed Knowledge by its **pack provenance and trust** (§Ch06 §3): a unit from a trusted publisher's pack is weighed accordingly; an unvetted pack's Knowledge is low-trust until corroborated.
- Installed Knowledge is **revisable and removable**: because its provenance ties it to the pack, uninstalling the pack (or upgrading it) cleanly reconciles the affected Knowledge (Book 12 §05), and local edits/overrides are tracked distinctly from pack-provided content.

Imported Knowledge that erased its origin would be unweighable and unremovable — so origin-preservation is mandatory.

## 4. Installed Knowledge is tenant-scoped and merges by trust

- A Knowledge Pack installs into a **workspace** (Book 11 §08 §5); its Knowledge is tenant-scoped like all Knowledge and does not leak across workspaces.
- Where installed Knowledge overlaps or conflicts with existing local Knowledge, it **merges by the trust/provenance rules** (§Ch05 §4): a workspace's `Authoritative` local assertion is not overwritten by a pack's `Inferred` unit; conflicts surface as they would for any Knowledge change. Installing a pack is a Knowledge change, subject to the same reconciliation and conflict handling (§Ch05).

## 5. Secret-freedom is a publishing invariant (P7)

Knowledge Packs are **secret-free** — and this is enforced at both publish and install:
- A pack MUST NOT contain secret values (§Ch01 §6); Knowledge is secret-free everywhere, and a distributed artifact makes this doubly critical (a leaked secret in a published pack would spread to every installer).
- Packing/publishing validation (Book 12 §04) MUST reject a pack containing secret-shaped material; installation MUST NOT introduce secrets into the vault/graph (§Ch03 §5, §Ch04 §5).
- A pack may reference a secret *class* ("this runbook uses a `payments`-class credential") so that an installing workspace knows to acquire the appropriate credential through the Broker (Book 11 §05) — but the pack never carries the credential itself.

## 6. Marketplace and governance

- Knowledge Packs are discoverable and distributable through the **Marketplace** (Book 12 §06), with the same trust tiers, conformance/quality signals, and moderation as other Packages.
- What packs a workspace may install, and at what trust, is **policy-governed** (P9, Book 11 §06): a workspace may restrict Knowledge imports to vetted publishers, mirroring how it governs any capability or policy import.

## 7. Invariants (normative summary)

1. A Knowledge Pack is a Package whose payload is Knowledge; it inherits the full Package discipline (manifest, signing/provenance, dependency resolution, versioning, reconciled install lifecycle).
2. Imported Knowledge preserves its pack origin in provenance; planning weighs it by that provenance/trust, and it is cleanly revisable/removable via the pack lifecycle.
3. Installed Knowledge is tenant-scoped and merges with local Knowledge by the trust/provenance conflict rules; installing a pack is a governed Knowledge change.
4. Knowledge Packs are secret-free, enforced at publish and install; they reference secret *classes* only, never values (P7).
5. Knowledge Packs are distributed via the Marketplace under the same trust/moderation as other Packages, and their installation is policy-governed.
