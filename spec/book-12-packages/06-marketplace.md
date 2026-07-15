# Book 12 · Chapter 06 — The Marketplace

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P8, P10, P11. Part of AEP-0003.*

> The Marketplace is where Packages are discovered, published, and assessed for trust. This chapter specifies its responsibilities: discovery, publishing, conformance and trust signals, and moderation. A healthy marketplace is what turns "everything is installable" (§Ch01) into a thriving ecosystem — while the discipline of the preceding chapters is what keeps that ecosystem safe.

## 1. Purpose

The Marketplace serves the ecosystem's supply and demand: publishers **offer** Packages (runtimes, planners, capabilities, knowledge packs, policies, …) and workspaces **discover and install** them. It is the coordination point that lets Sankalpa grow by community contribution (P11) rather than only by core development. Crucially, the Marketplace is a *discovery and trust* layer — it does not relax any install-time guarantee: everything discovered here is still verified, resolved, least-privilege-granted, and isolated per §Ch02–§Ch05 and Book 11.

## 2. Discovery

- Packages are **discoverable** by what they provide (§Ch02 §2 — "find a runtime that supports X", "find a knowledge pack for domain Y"), by publisher, by conformance level (§Ch02 §6), and by compatibility with the workspace's version spaces (§Ch07).
- Discovery surfaces the manifest's declarations (provides, requires, compatibility, conformance) so a workspace can assess a Package **before** installing — including exactly what authority it will request (§Ch02 §4). Transparency-before-install is a Marketplace requirement, not a courtesy: an operator must be able to see what a Package does and needs before committing.

## 3. Publishing

- Publishing a Package requires a **verified publisher identity** (§Ch04 §1) and produces a **signed, provenance-attested** artifact (§Ch04). Publishing validation **rejects secret-bearing Packages** (§Ch04 §4) and MAY require passing declared conformance suites (§Ch02 §6) before listing.
- Published Packages are **versioned and immutable per version** (§Ch07, P6): a given version's contents never change after publication (a change is a new version), so an installer's resolved version is reproducible and its provenance stable. This is the supply-chain integrity guarantee at the distribution layer.

## 4. Trust signals and tiers

The Marketplace makes trust **legible** (§Ch04 §5) so workspaces can make informed, policy-governed decisions (Book 11 §06):
- **Verified publisher** status (identity attestation).
- **Conformance badges** — which suites a Package passed, at what stability level (§Ch02 §6, Book 06 §Ch07) — *verifiable* claims, not marketing.
- **Provenance** — build origin attestation (§Ch04 §2).
- **Reputation** — adoption and community signals.

These are **signals for policy**, not automatic authorizations: a workspace's policy (Book 11 §06) decides what trust level it requires. The Marketplace informs; policy decides; installation enforces least-privilege regardless (§Ch05 §3). High trust never bypasses the capability model (§Ch04 §3).

## 5. Moderation and safety

- The Marketplace **moderates** listings: removing or flagging Packages found malicious, vulnerable, or non-conforming, and propagating that signal to installers (so a workspace can learn a previously-installed Package is now flagged).
- Because installed Packages are `Package` Resources with revocable grants (§Ch05 §3) and containment (Book 11 §10), a Package later found malicious can be **contained** (grants revoked) and **uninstalled** cleanly (§Ch05 §5) — moderation has teeth because the install discipline makes Packages reversible and their authority revocable.
- Moderation actions are **auditable** (P10): the provenance of a flag/removal is recorded, so trust decisions are themselves accountable.

## 6. The Marketplace does not weaken any guarantee

Stated plainly because it is the Marketplace's most important property: **discovery convenience never relaxes safety.** A Package installed from the Marketplace is subject to *exactly* the same verification (§Ch04), resolution (§Ch03), least-privilege authorization (§Ch05 §3), isolation and output-verification (Book 11 §10), and policy governance (Book 11 §06) as any other. The Marketplace makes Packages *findable and assessable*; the preceding chapters make them *safe*. Conflating the two — treating "it's in the marketplace" as "it's safe to grant broad authority" — is exactly the supply-chain error the design forecloses (§Ch01 §4).

## 7. Invariants (normative summary)

1. The Marketplace is a discovery and trust layer for publishing and finding Packages; it never relaxes any install-time guarantee.
2. Packages are discoverable by provides/publisher/conformance/compatibility, with pre-install transparency into what a Package does and what authority it requests.
3. Publishing requires a verified identity and produces a signed, provenance-attested, secret-free, version-immutable artifact; conformance may be required to list.
4. Trust signals (verified publisher, verifiable conformance badges, provenance, reputation) are legible inputs to policy — not automatic authorizations; least-privilege is enforced regardless.
5. Moderation flags/removes unsafe Packages with propagated signals; revocable grants and clean uninstall give moderation teeth; moderation actions are auditable.
6. Marketplace discovery convenience never weakens verification, resolution, least-privilege, isolation, or policy governance.
