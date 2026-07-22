# RFC-0008: Capability-grant re-authorization on package upgrade — binding grants to verified version identity

| Field | Value |
|-------|-------|
| **Status** | Final |
| **Authors** | Dhiraj Salian (Phase 2 hardening review) |
| **Domain / Book** | Packages & Security / Books 12, 11 |
| **Shepherd (Domain Lead)** | Security Domain Lead |
| **Created** | 2026-07-16 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | — · re-review tracked in the [interim-acceptance ledger](../process/interim-acceptance-ledger.md) |

> Raised by the Phase 2 hardening pass (adversarial review toward v1.0). Number 0008 reserved; open for review by the Security Domain Lead and Reviewers. Second hardening batch (0005–0011); first supply-chain (Book 12) finding, independent of the others.

> **Accepted 2026-07-16.** FCP (accept disposition) was called and concluded the same day by the Security Domain Lead. For the solo-maintainer repo the 10-working-day window was shortened and the ≥2-Reviewer gate ([process §7](../process/rfc-process.md)) waived — both recorded here for auditability, not pretended. No blocking objections; all design questions resolved (see *Resolved questions*), and the review pass fixed the defects it found. Per [process §8](../process/rfc-process.md) this RFC becomes **normative only on reflection into `spec/`**; status advances to **Final** once the Documentation Changes (§12) land in Books 11 §03/§06, 12 §04/§05 and the Glossary.

> **Final 2026-07-16.** §12 reflected into `spec/` (Book 12 §04 §3 / §05 §2–§3.1/§4/§7, Book 11 §03 §2/§8, Book 11 §06 §3) and the Glossary in this change; the RFC is now normative. See CHANGELOG.

## 1. Executive Summary
Capability grants are established at **install**: installation "surfaces the Package's declared required capabilities for explicit authorization" ([Book 12 §Ch05 §3](../spec/book-12-packages/05-install-lifecycle.md), [Book 12 §Ch02 §4](../spec/book-12-packages/02-package-manifest.md), [Book 03 §Ch09 §5](../spec/book-03-kernel/09-plugin-and-package-managers.md)). But the **upgrade** flow has no such step — it is *"re-resolve jointly-consistent → verify → apply delta reconciled → Active"* ([Book 12 §Ch05 §2](../spec/book-12-packages/05-install-lifecycle.md)), with **no capability re-authorization**. Grants are attached to the `Package`, not to the specific verified code that was authorized, and they persist unchanged across a version change. Two holes follow:

1. **Same-capabilities, different-code laundering.** A publisher (compromised, or a new owner after a package transfer — the classic supply-chain takeover) ships v2 declaring the *same* capabilities as v1 but with malicious code. Signature verification ([Book 12 §Ch04](../spec/book-12-packages/04-signing-and-provenance.md)) confirms v2 is intact and identifies its signer — then v2 **silently inherits v1's grants** (`payments`, secret-materialization, egress) and exercises them with new code the human never consented to. The workspace approved v1's *behavior under authority*, not v2's.
2. **Silent broadening on upgrade.** If v2's manifest requires *new or broader* capabilities than were granted at install, the upgrade flow specifies no surfacing/authorization gate for the delta — the "no hidden privilege escalation" promise ([Book 12 §Ch02 §4](../spec/book-12-packages/02-package-manifest.md)) has no mechanism at upgrade time.

The spec's own posture — *"authority only by explicit grant," "install ≠ privilege," "a compromised trusted publisher cannot escalate beyond the capabilities a workspace explicitly grants"* ([Book 12 §Ch04 §… ](../spec/book-12-packages/04-signing-and-provenance.md), §Ch01 §4) — is true only for *net-new* capabilities and only if the upgrade delta is gated (which it is not). It does not cover the **re-use of already-granted authority by newly-substituted code**. This RFC binds each grant to the **verified version/signing identity** it was authorized against and makes **upgrade re-evaluate grants**: broadened authority and signing-identity changes force re-authorization (fail-safe for sensitive classes), while same-identity non-broadening upgrades may carry grants forward by policy — closing the laundering path without adding friction to routine patches.

## 2. Problem Statement
The install discipline is the platform's supply-chain backbone ([Book 12 §Ch01 §4](../spec/book-12-packages/01-everything-is-installable.md), [Book 12 §Ch04](../spec/book-12-packages/04-signing-and-provenance.md)): untrusted-by-default, authority only by explicit least-privilege grant, verified signature and provenance. **Upgrade silently breaks the "explicit" in "explicit grant":**

- **Grants are not bound to what was verified.** A grant is "specific," "attenuated," and "revocable" ([Book 11 §Ch03 §3–5](../spec/book-11-security/03-capability-based-security.md)), but nothing binds it to the **version or artifact identity** of the grantee. The `Package` desired state — "installed at version V, with these grants" ([Book 12 §Ch05 §1](../spec/book-12-packages/05-install-lifecycle.md)) — pairs V and grants without any rule that changing V re-evaluates the grants.
- **The platform verifies signer identity per artifact but does not act on a change of it.** Signature verification checks "the signature against the publisher's identity … before doing anything else" ([Book 12 §Ch04 §… signature verification](../spec/book-12-packages/04-signing-and-provenance.md)); provenance attests build origin. So a publisher change, a signing-key change, or an ownership transfer across an upgrade is **detectable** — but no rule requires re-consent when it happens.
- **The upgrade flow omits the authorization step by construction.** [Book 12 §Ch05 §2](../spec/book-12-packages/05-install-lifecycle.md) lists "surface & grant capabilities (§3)" under **Install** but not under **Upgrade**. Install-time authorization is thus a one-time event, not a per-verified-version property.
- **The threat is concrete and common.** Maintainer-account compromise and package-ownership transfer to a bad actor are among the most exploited supply-chain vectors. Sankalpa's isolation and least-privilege bound the *set* of authority, but a takeover upgrade that stays within the *already-granted* sensitive set (e.g. an extension legitimately granted `payments` for v1) can do real harm with no fresh human decision.

Cost of doing nothing: "explicit authorization" is defeated by the ordinary upgrade path; a workspace that vetted v1 is silently running v2's code under v1's authority; the strong install-time posture is undermined at exactly the moment code changes.

## 3. Alternatives Considered
- **Do nothing.** Rejected: leaves grant continuity across code changes implicit, defeating explicit authorization and enabling upgrade-laundering of sensitive grants.
- **Re-consent every capability on every upgrade.** Rejected: upgrade fatigue drives users to blanket-approve, degrading the security value of consent; punishes routine same-publisher patches that broaden nothing.
- **Uninstall + fresh install for every upgrade.** Rejected: forfeits the reconciled, reversible, dependency-aware upgrade the spec deliberately provides ([Book 12 §Ch05 §2/§4](../spec/book-12-packages/05-install-lifecycle.md)); disruptive and still needs the identity-binding rule to know when re-consent is due.
- **Bind grants to verified version identity + policy-graded re-authorization on upgrade (chosen).** Grants record the version/signing identity they were authorized against; upgrade re-evaluates against the new artifact and re-authorizes only when authority broadens or signing identity changes, carrying forward frictionlessly otherwise. Mirrors the capability model's existing "authority is specific and revocable" stance, extended along the version axis.

## 4. Proposed Design
**4.1 Grants bind to verified version identity (normative).** A capability grant to a Package ([Book 12 §Ch05 §3](../spec/book-12-packages/05-install-lifecycle.md), [Book 11 §Ch03](../spec/book-11-security/03-capability-based-security.md)) **MUST** record the **authorized-against identity**: the Package version, the publisher identity, and the signing-key/artifact identity ([Book 12 §Ch04](../spec/book-12-packages/04-signing-and-provenance.md)) in effect when the grant was made. A grant authorizes an extension's authority **for that verified identity**, not for the Package name in perpetuity.

**4.2 Upgrade re-evaluates grants (normative).** The Upgrade flow ([Book 12 §Ch05 §2](../spec/book-12-packages/05-install-lifecycle.md)) gains a **grant re-evaluation** step, after verify and before the upgraded contributions become `Active`. The controller compares the new artifact's declared `requires.capabilities` ([Book 12 §Ch02 §4](../spec/book-12-packages/02-package-manifest.md)) and signing identity against the recorded authorized-against identity (§4.1):

- **Broadened authority** — the new version requires a capability, effect, or scope not previously granted: the delta **MUST** be surfaced for explicit authorization exactly as at install ([Book 12 §Ch05 §3](../spec/book-12-packages/05-install-lifecycle.md), [Book 03 §Ch09 §5](../spec/book-03-kernel/09-plugin-and-package-managers.md)). Until authorized, contributions needing the new authority are **fail-closed** (P8); the upgrade either holds or proceeds for the unchanged-authority parts, per policy.
- **Signing-identity change** — different publisher, different signing key, or a provenance-attested ownership transfer ([Book 12 §Ch04](../spec/book-12-packages/04-signing-and-provenance.md)), *even with an unchanged capability set*: re-authorization **MUST** be required for **sensitive-class** capabilities (secret-materialization, `payments` and equivalent high-consequence classes, external egress / `Network(write)`, external `StateWrite`; [Book 04 §Ch06](../spec/book-04-aos-ir/06-effect-system.md)). This is the ownership-transfer/maintainer-compromise defense: a new signer does not silently inherit sensitive authority.
- **Same identity, non-broadening** — same publisher and signing key, same-or-narrowed capabilities, within a policy-permitted version step: grants **MAY** carry forward automatically. This keeps routine patches frictionless while preserving the invariant that authority is never *broadened* and *identity* never *changes* without explicit consent.

**4.3 Policy-governed thresholds, fail-safe default (normative).** Workspace policy ([Book 11 §Ch06](../spec/book-11-security/06-policy-engine.md)) sets re-authorization thresholds (e.g. "always re-consent `payments`", "re-consent on MAJOR", "auto-carry same-key patch"). Where policy is absent or does not decide, the default is **fail-safe**: re-authorize on any signing-identity change and for any sensitive-class capability; carry forward only same-identity, non-sensitive, non-broadening deltas. An unverifiable comparison (missing recorded identity from a pre-adoption install) is treated as an identity change (§4.2 middle case).

**4.4 Rollback symmetry (normative).** Rolling back ([Book 12 §Ch05 §4](../spec/book-12-packages/05-install-lifecycle.md)) to a **previously-authorized** version restores that version's already-consented grants without new consent — it was explicitly authorized before and is known-good. Two exceptions, both treated as an upgrade under §4.2:
- A rollback target **never authorized on this install** (unusual).
- A target whose capability set is **wider than what is currently granted** — because "broadening" is measured against the *currently granted* set, not against version ordering. Absent this rule, a publisher could ship a narrow v2 and then "roll back" to a wide v1 to reclaim authority without consent; version direction is not a safety property.

**4.5 Audit (normative).** Every carry-forward and every re-authorization is auditable ([Book 11 §Ch09](../spec/book-11-security/09-audit-and-attribution.md), P10): "grant G carried forward v1→v2 (same key K)" or "grant G re-authorized by subject U for v2 (key changed K→K′)". The chain "who authorized what authority for which verified code" is always answerable — the version-axis complement of the existing per-grant audit.

## 5. Tradeoffs
**Gain:** explicit authorization becomes a property of *verified code*, not a one-time Package event; upgrade-laundering of sensitive grants is closed; ownership-transfer/key-compromise forces re-consent; broadened authority is gated at upgrade as it is at install — while same-identity patches stay frictionless.
**Give up:** grants carry an authorized-against identity (small record growth); the upgrade controller performs a capability/identity diff and may pause for consent; workspace policy must express thresholds (with a safe default if it does not).

## 6. API Changes
Additive; no method signatures change:
- The grant record ([Book 11 §Ch03](../spec/book-11-security/03-capability-based-security.md), recorded on install per [Book 12 §Ch05 §3](../spec/book-12-packages/05-install-lifecycle.md)) gains `authorizedAgainst` (version + publisher + signing-key/artifact identity).
- The package install/upgrade controller ([Book 03 §Ch09](../spec/book-03-kernel/09-plugin-and-package-managers.md), [Book 12 §Ch05](../spec/book-12-packages/05-install-lifecycle.md)) gains the §4.2 re-evaluation step, reusing the existing install-time authorization surface ([Book 03 §Ch09 §5](../spec/book-03-kernel/09-plugin-and-package-managers.md)).
- No new Kernel API method; re-authorization uses the same explicit-grant path as install.

## 7. Resource Changes
- Capability grant records gain `authorizedAgainst` (secret-free identity metadata). The `Package` Resource ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md), [Book 12 §Ch05 §1](../spec/book-12-packages/05-install-lifecycle.md)) `status` records the last re-authorization outcome per grant. No lifecycle change: upgrade still `Progressing → Active`, but may dwell in `Progressing` with a surfaced `AwaitingReauthorization` condition ([Book 02 §Ch03 §3.5](../spec/book-02-resource-model/03-desired-vs-actual-state.md)) until consent — never silently `Active` on unconsented authority.

## 8. Event Changes
Additive, secret-free (P5): `package.grantCarriedForward` (records same-identity carry-forward), `package.reauthorizationRequired` and `package.reauthorized` (records the delta and the deciding subject). These extend the existing lifecycle Events ([Book 12 §Ch05 §6](../spec/book-12-packages/05-install-lifecycle.md), [Book 14 §Ch01](../spec/book-14-observability-governance/01-event-model.md)); per-subject ordering on the `Package` subject suffices ([Book 03 §Ch03 §5](../spec/book-03-kernel/03-event-bus.md)).

## 9. Security Impact
This RFC *strengthens* P8 along the version axis and introduces no new authority path. `authorizedAgainst` is identity metadata, secret-free, safe wherever grant records already live. It closes a concrete escalation-in-practice — a takeover or compromised-key upgrade exercising sensitive grants under new code — that the current "verified before install, granted once" model leaves open, and it does so without weakening the existing containment levers (grants remain revocable, [Book 11 §Ch03 §5](../spec/book-11-security/03-capability-based-security.md); malicious packages remain uninstallable, [Book 12 §Ch06](../spec/book-12-packages/06-marketplace.md)). It composes with, and does not replace, marketplace moderation (which handles *known* bad packages; this handles *silent* code substitution before anyone flags it). Add to the SI checklist ([Book 11 §Ch11](../spec/book-11-security/11-security-invariants.md)): *a capability grant authorizes a verified version identity; a signing-identity change or authority broadening on upgrade requires re-authorization for sensitive classes.* Security review required.

## 10. Performance Impact
The re-evaluation is a capability/identity diff per upgrade — negligible against the resolve/verify/lower cost already on the upgrade path. Same-identity patches add only the diff (no consent round-trip). No hot-path effect; this is control-plane-only.

## 11. Testing Strategy
- Laundering: v1 granted `payments`; upgrade to v2 with same declared capabilities but a **changed signing key** ⇒ upgrade requires re-authorization; unconsented ⇒ `payments`-using contribution fail-closed, Package dwells `AwaitingReauthorization`.
- Broadening: v2 requires a new egress capability ⇒ delta surfaced for authorization; unconsented ⇒ that contribution fail-closed.
- Frictionless path: same publisher/key, same-or-narrower capabilities, policy-permitted step ⇒ grants carry forward, `package.grantCarriedForward` emitted, no consent prompt.
- Rollback: to a previously-authorized version ⇒ grants restored without new consent.
- Fail-safe: missing `authorizedAgainst` (pre-adoption install) ⇒ treated as identity change ⇒ re-authorize sensitive classes.

## 12. Documentation Changes
Add the grant re-evaluation step to the Upgrade flow in [Book 12 §Ch05 §2/§3](../spec/book-12-packages/05-install-lifecycle.md) and the §4 rules; add `authorizedAgainst` to the grant model in [Book 11 §Ch03](../spec/book-11-security/03-capability-based-security.md); tighten the [Book 12 §Ch04](../spec/book-12-packages/04-signing-and-provenance.md) statement so "cannot escalate beyond explicitly-granted capabilities" explicitly covers the upgrade/code-substitution case; note re-authorization thresholds in [Book 11 §Ch06](../spec/book-11-security/06-policy-engine.md). New Glossary entries: *Authorized-against identity*, *Grant carry-forward*, *Upgrade re-authorization*.

## 13. Migration Strategy
Additive and pre-1.0. Existing grants lack `authorizedAgainst`; on first upgrade after adoption the fail-safe (§4.3) treats the missing baseline as an identity change and re-authorizes sensitive classes once, stamping `authorizedAgainst` thereafter. Non-sensitive same-name upgrades still carry forward. No stored artifact changes meaning; the change is to the upgrade *procedure* and the grant *record*.

## 14. Risks
- **Consent fatigue** if thresholds are too aggressive — mitigated by the frictionless same-identity carry-forward (§4.2) so only broadening/identity-change events prompt.
- **Sensitive-class list drift** — the set of sensitive capability classes must be maintained ([Book 04 §Ch06](../spec/book-04-aos-ir/06-effect-system.md), [Book 11 §Ch07](../spec/book-11-security/07-approval-engine.md)); flagged for the reflecting PR to define authoritatively.
- **Automated/unattended upgrades** (a policy that auto-applies patches) must still honor §4.2 for identity changes — an auto-upgrade policy MUST NOT auto-consent a signing-identity change for sensitive classes; verify in review.
- **Interaction with dependencies** — a transitive dependency's upgrade ([Book 12 §Ch03](../spec/book-12-packages/03-dependency-resolution.md)) that changes *its* signing identity must re-evaluate *its* grants too; the re-evaluation is per-Package across the resolved set, not only the top-level target.

## 15. Future Improvements
Transitive-capability transparency (surfacing dependencies' authority requests, not only the top-level Package's) as a companion RFC; reproducible-build / transparency-log attestation ([Book 12 §Ch04](../spec/book-12-packages/04-signing-and-provenance.md)) feeding an automatic trust signal that can *lower* re-consent friction for verified-reproducible upgrades; time-boxed grants that expire and force periodic re-authorization independent of upgrades.

---
### Resolved questions
- **Where does the "sensitive capability class" set live?** In [Book 11 §Ch07](../spec/book-11-security/07-approval-engine.md) (approval triggers), workspace-extensible by policy ([Book 11 §Ch06](../spec/book-11-security/06-policy-engine.md)); this RFC **references** it and does not define a rival. The approval engine already maintains the consequence classes that gate high-consequence actions, and a second taxonomy would inevitably drift out of sync with the first — leaving two answers to "is this sensitive?". One authoritative set, two consumers (approval gating, upgrade re-authorization).
- **Same-key upgrade that *narrows* capabilities: auto-revoke the now-unused grants?** **Yes, auto-revoke** (audited, [Book 11 §Ch09](../spec/book-11-security/09-audit-and-attribution.md)). Least-privilege (P8) says authority without a declared need should not persist, and such a grant is already inert: *"A Package that performs an effect requiring a capability it did not declare is denied at runtime"* ([Book 12 §Ch02 §4](../spec/book-12-packages/02-package-manifest.md)). So retaining it confers no capability the Package can lawfully use while leaving standing authority for a *later* version to silently reclaim under §4.2's carry-forward — pure risk, no benefit. Revocation here needs no consent: it only ever removes authority.
- **Does a downgrade that re-introduces a wider set trigger §4.2 broadening?** **Yes** — broadening is measured against the **currently granted** set, not against version ordering, so §4.2 applies regardless of direction. This also closes an obvious evasion: ship a narrow v2, then "roll back" to a wide v1 to reclaim authority without consent. §4.4's rollback exemption is therefore scoped precisely to a target **previously authorized on this install** and *not wider than what is currently granted*; anything else is an upgrade under §4.2.

### Unresolved questions
*(none — all resolved ahead of review)*
