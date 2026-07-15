# Book 12 · Chapter 04 — Signing and Provenance

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P7, P8, P10. Part of AEP-0003. Companion to Book 11 (Security).*

> A Package is untrusted code from a potentially hostile author (§Ch01 §5). Before the platform admits it, it must verify *who* published it and *that it has not been tampered with* — signing and provenance. This chapter specifies verification before install, supply-chain provenance, and the secret-freedom of published Packages. This is the ecosystem's first line of defense against supply-chain attack.

## 1. Verify before install

A Package MUST be **verified before it is installed** (Book 03 §Ch09 §3):
- **Signature verification** — the manifest and contents are signed by the publisher; the Package Manager verifies the signature against the publisher's identity before doing anything else. A Package whose signature is missing or invalid is **not installed** (fail-closed, Book 11 §01 §6) unless workspace policy explicitly permits unsigned Packages (a deliberate, audited relaxation, not a default).
- **Integrity** — the signature covers the whole artifact (manifest + contents, §Ch02 §1), so tampering with either invalidates it. What is installed is exactly what the publisher signed — nothing added or altered in transit.

Verification precedes resolution's *application* and registration (§Ch03 §5, §Ch05): the platform commits to nothing from an unverified Package.

## 2. Supply-chain provenance (P10)

Beyond "who signed it," provenance attests **how the Package was built and where it came from** (§Ch02 `integrity.provenance`):
- Provenance records the build origin, source, and attestations, so an installer can assess the *supply chain*, not just the final signature — defending against a compromised build pipeline that signs malicious output.
- Provenance is **retained and auditable** (Book 11 §09): the installed `Package` Resource records its verified provenance, so "where did this extension come from?" is always answerable after the fact.
- This makes the ecosystem accountable: every installed extension traces to a verified publisher and build origin.

## 3. Signing/provenance and the capability model (P8)

Verification establishes *identity and integrity*; it does **not** grant *authority*. Even a perfectly-signed Package from a trusted publisher gets **no** authority on install (§Ch01 §4): its declared capabilities are surfaced for explicit least-privilege authorization (Book 03 §Ch09 §5, Book 11 §03). Signing answers "is this really from X, untampered?"; capabilities answer "what may it do?" — two independent questions. A trusted signature does not imply broad authority; a Package from a trusted publisher still runs least-privilege and isolated (Book 11 §10). This separation is deliberate: it means a compromised trusted publisher cannot escalate beyond the capabilities a workspace explicitly grants.

## 4. Published Packages are secret-free (P7)

A Package is a **distributed artifact** — installed by many workspaces — so a secret embedded in one would leak to *every* installer. Therefore:
- A Package MUST NOT contain secret values (extending the secret-free guarantee, Book 11 §04). **Publishing validation MUST reject** a Package containing secret-shaped material (mirroring knowledge-pack publishing, Book 09 §Ch07 §5).
- A Package may reference a secret **class** ("this runtime needs a `payments`-class credential") so an installing workspace knows to acquire the credential through the Broker (Book 11 §05) — but never carries the credential itself.
- This is checked at publish and preserved at install: installation MUST NOT introduce a secret into the platform from a Package.

## 5. Trust tiers and policy (P9)

- Publishers and Packages carry **trust signals** (verified publisher, conformance claims §Ch02 §6, marketplace reputation §Ch06). Trust is *earned and legible*, not binary.
- What a workspace will install, from whom, and at what trust is **policy-governed** (Book 11 §06): a workspace may require verified publishers, forbid unsigned Packages, or restrict to vetted sources. Installation is thus governed by the same policy engine as every other consequential action — supply-chain risk is managed as policy, not left to chance.

## 6. Invariants (normative summary)

1. A Package's signature and integrity are verified before install; a missing/invalid signature blocks install (fail-closed) unless policy explicitly permits unsigned Packages as an audited relaxation.
2. The signature covers manifest + contents, so what is installed is exactly what was signed; supply-chain provenance attests build origin and is retained/auditable (P10).
3. Verification establishes identity and integrity but grants no authority; even a trusted-publisher Package runs least-privilege, isolated, and authorized only by explicit capability grants (P8).
4. Published Packages are secret-free: publishing validation rejects secret-shaped material; Packages reference secret classes only, never values, and install never introduces a secret (P7).
5. Trust is legible (verified publisher, conformance, reputation) and installation is policy-governed, so supply-chain risk is managed as policy (P9).
