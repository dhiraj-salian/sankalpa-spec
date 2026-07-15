# RFC-0009: Channel identity binding and assurance — authenticating messaging-channel senders and gating cross-channel session continuity

| Field | Value |
|-------|-------|
| **Status** | Draft |
| **Authors** | Dhiraj Salian (Phase 2 hardening review) |
| **Domain / Book** | Interfaces & Security / Books 13, 11 |
| **Shepherd (Domain Lead)** | Security Domain Lead |
| **Created** | 2026-07-16 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | TBD |

> Draft raised by the Phase 2 hardening pass (adversarial review toward v1.0). Numbering provisional until a maintainer reserves it at PR time. First Interfaces (Book 13) hardening finding; independent of RFC-0002–0008.

## 1. Executive Summary
A `Session` is "an authenticated interaction context [that] establishes *who* is interacting" ([Book 13 §Ch05 §2](../spec/book-13-interfaces/05-conversation-and-session.md), [Book 11 §Ch08 §3](../spec/book-11-security/08-identity-users-sessions.md)), and every Kernel API request "is authenticated before authorization" via "tokens, mTLS, OIDC" ([Book 11 §Ch08 §4](../spec/book-11-security/08-identity-users-sessions.md)). But a message from a messaging channel (Telegram, WhatsApp, Slack, Email, Voice — [Book 13 §Ch03](../spec/book-13-interfaces/03-reference-channels.md)) carries no such token — only a **channel-native, often-spoofable identifier**: an email `From`, a phone number, a Telegram user id. The spec says only that "the channel's transport identity is **mapped** to a Session/User at the Kernel API" ([Book 13 §Ch02 §… ingress](../spec/book-13-interfaces/02-channel-model.md), [Book 13 §Ch03 §… inbound](../spec/book-13-interfaces/03-reference-channels.md)) — a **mapping**, never an **authentication** — while simultaneously declaring the channel **untrusted** ([Book 13 §Ch02 §… untrusted](../spec/book-13-interfaces/02-channel-model.md)). Two holes result:

1. **The untrusted channel's identity claim is trusted with no verification.** There is no specified enrollment or proof binding a channel-native identifier to a `User`, so anyone who can spoof or compromise a channel handle (trivial for email `From`; SIM-swap for SMS/WhatsApp; account takeover for Slack/Telegram) can act **as** that user — issue `Intent`s, continue Conversations — because the Kernel accepts the untrusted channel's assertion of *who sent this*.
2. **Cross-channel continuity extends authority to unauthenticated legs.** "A user starts in Slack, continues by email … correlated by Session" ([Book 13 §Ch02 §… spans](../spec/book-13-interfaces/02-channel-model.md), [Book 13 §Ch05 §… spans](../spec/book-13-interfaces/05-conversation-and-session.md)). If the Session was authenticated on one leg, a message arriving on a *weaker* channel with a matching identifier silently joins it — a spoofed `From` continues an authenticated user's Session.

The strong front-door authentication ([Book 11 §Ch08](../spec/book-11-security/08-identity-users-sessions.md)) and the "authenticated interaction context" guarantee are therefore **unbacked for exactly the channels most humans use**. This RFC specifies (a) a verified **channel-identity binding** (a channel-native id becomes an identity only via enrollment from an already-authenticated context), (b) a per-channel **authentication assurance level** that caps a Session's effective authority, and (c) **cross-channel re-binding** so continuity never silently extends authority to an unauthenticated leg.

## 2. Problem Statement
Ingress is a declared trust boundary: at B1, everything a channel carries is "untrusted input … checked by the Kernel" ([Book 11 §Ch02 §B1](../spec/book-11-security/02-trust-boundaries.md), [Book 13 §Ch02 §… ingress](../spec/book-13-interfaces/02-channel-model.md)). Yet the one thing the Kernel does **not** independently check is *the identity the channel asserts*:

- **"Mapped," not "authenticated."** [Book 13 §Ch02](../spec/book-13-interfaces/02-channel-model.md) and [§Ch03](../spec/book-13-interfaces/03-reference-channels.md) both say the channel's user identity "maps to a Session/User at the Kernel API." No mechanism verifies that the Telegram id / email address genuinely belongs to that `User`. [Book 11 §Ch08 §4](../spec/book-11-security/08-identity-users-sessions.md) authenticates *API requests* (tokens/mTLS/OIDC); a chat message is not such a request, and the adapter that asserts "this is from user X" is itself **untrusted** ([Book 13 §Ch02 §… untrusted](../spec/book-13-interfaces/02-channel-model.md)). The trust model says *don't trust the channel*; the identity mapping *does*.
- **No enrollment/binding concept exists.** The core catalog has `User`, `Session`, `Conversation` ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)) but nothing binding *(channel, channel-native-id) → User*, and no verified enrollment step. So the mapping is, by omission, "trust the sender field."
- **The stolen-token defense doesn't cover this.** [Book 11 §Ch08 §3](../spec/book-11-security/08-identity-users-sessions.md) bounds "the blast radius of a stolen session token" via attenuation/expiry — but a spoofed channel identity needs no token at all; it manufactures a session from a forged sender.
- **Cross-channel is the sharp edge.** The very feature that a Conversation spans channels means a Session bootstrapped on a strong leg (Web Runtime OIDC) can be *continued* on a weak one (email). Whether the new leg is re-authenticated or trusted-by-matching-identifier is unspecified — and the natural reading (correlate by Session, map channel identity) is the insecure one.
- **What is already well-defended, by contrast.** The *decision surfaces* are safe: approvals and credential entry happen only on the trusted Web Runtime, which authenticates the actor, and the channel "only carries the link" ([Book 13 §Ch02 §… web-runtime](../spec/book-13-interfaces/02-channel-model.md), [Book 11 §Ch07 §… non-repudiable](../spec/book-11-security/07-approval-engine.md)). The hole is the **inbound** path — acting *as* the user — not the outbound decision surface.

Cost of doing nothing: a spoofable identifier is sufficient to impersonate a user to the platform over the most common channels; "authenticated interaction context" and "attributed to a real identity" (P10) become unfalsifiable; and cross-channel continuity is an authority-laundering path.

## 3. Alternatives Considered
- **Do nothing.** Rejected: leaves impersonation-by-spoofed-sender open and the authentication guarantee unbacked for messaging channels.
- **Require Web Runtime OIDC for every interaction (no messaging-channel origination).** Rejected: defeats the entire channel model ([Book 13 §Ch02](../spec/book-13-interfaces/02-channel-model.md), "reach the platform however you like") and the reference channels ([Book 13 §Ch03](../spec/book-13-interfaces/03-reference-channels.md)); users legitimately start in Slack/Telegram.
- **Trust the channel adapter to authenticate (push auth into the adapter).** Rejected: contradicts "channels carry, they do not decide" and "the channel is untrusted" ([Book 13 §Ch02](../spec/book-13-interfaces/02-channel-model.md)); authentication is a Kernel responsibility ([Book 11 §Ch08 §4](../spec/book-11-security/08-identity-users-sessions.md)).
- **Bind on first-seen identifier (trust-on-first-use, no verification).** Rejected: TOFU on a spoofable identifier still admits the spoofer as the first user; no verification means no binding integrity.
- **Verified enrollment + per-channel assurance + cross-channel re-binding (chosen).** A channel-native id is meaningless until *enrolled* from an already-authenticated context; each channel carries an assurance level that caps authority; consequential actions step up to a high-assurance surface (the Web Runtime, which already hosts approvals). Keeps origination on any channel while making authority proportional to proven identity — "channels carry, they do not decide," extended to identity.

## 4. Proposed Design
**4.1 Channel-identity binding (normative).** A channel-native identifier is not an identity until bound. A **`ChannelBinding`** Resource ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)) ties *(channelKind, channel-native-id) → User*, established by a **verified enrollment**:
- Enrollment originates from an **already-authenticated context** — a Web Runtime OIDC session ([Book 13 §Ch01](../spec/book-13-interfaces/01-web-runtime.md), [Book 11 §Ch08 §4](../spec/book-11-security/08-identity-users-sessions.md)) — that issues a one-time enrollment challenge the user completes *from the channel* (e.g. the user, logged in on the Web Runtime, is shown a code to send from their Telegram, or confirms a link delivered to the address), proving control of the channel identifier. The proof-of-control mechanism is a provider concern; the *requirement* — a verified binding before a channel identifier carries authority — is normative.
- Until a `ChannelBinding` exists, an inbound message from an unbound identifier maps to **no Session** and can do nothing beyond, at most, *initiating enrollment* (fail-closed, P8, [Book 11 §Ch08 §1](../spec/book-11-security/08-identity-users-sessions.md): identity with no grants can do nothing — here, *no verified identity at all*).
- Bindings are revocable and auditable ([Book 11 §Ch09](../spec/book-11-security/09-audit-and-attribution.md), P10): a user (or admin) can revoke a compromised handle's binding, promptly cutting off impersonation.

**4.2 Per-channel authentication assurance (normative).** Each channel/binding carries an **assurance level** reflecting how strongly the sender identity is proven per message (e.g. `low` — bare email `From`, no per-message proof; `medium` — a channel with authenticated accounts and workspace SSO, e.g. Slack; `high` — Web Runtime OIDC session, per-request token). A `Session` established or continued over a channel leg **MUST** carry that leg's assurance, and its **effective authority is capped by assurance**: a Session may hold no more than the intersection of the user's grants and what the assurance level permits for that consequence class.

**4.3 Consequential actions step up (normative).** Policy ([Book 11 §Ch06](../spec/book-11-security/06-policy-engine.md)) and approval ([Book 11 §Ch07](../spec/book-11-security/07-approval-engine.md)) declare a **minimum assurance** per consequence class. A high-consequence Intent arriving on a low-assurance channel is **not refused outright** — the channel *carries* it — but the consequential action is gated on a **step-up** to a high-assurance surface (the Web Runtime, which already hosts approvals and credential entry, [Book 13 §Ch02 §… web-runtime](../spec/book-13-interfaces/02-channel-model.md)). This composes with the existing "approvals decided on the Web Runtime, channel carries the link" design ([Book 13 §Ch06](../spec/book-13-interfaces/06-approval-and-human-in-the-loop.md)): the approval surface *is* the assurance step-up.

**4.4 Cross-channel continuity re-binds, never trusts (normative).** A Conversation may span channels ([Book 13 §Ch05](../spec/book-13-interfaces/05-conversation-and-session.md)), but each channel leg's inbound identity **MUST** be independently bound (§4.1) to the same `User` before it contributes turns/Intents under the Session's authority, and the Session's effective authority is re-capped to the **new leg's** assurance (§4.2) while that leg is active. A message on an unbound or lower-assurance channel does **not** silently inherit a Session bootstrapped on a stronger leg; at most it requests to join, gated by binding/step-up. This closes the "continues by email" spoof.

**4.5 Attribution records assurance (normative).** Every turn/Intent records the `ChannelBinding` and assurance level it was authenticated under ([Book 11 §Ch09](../spec/book-11-security/09-audit-and-attribution.md), P10). Audit answers not just *who* but *how strongly authenticated* — so a later review can distinguish a high-assurance Web Runtime action from a low-assurance email one.

## 5. Tradeoffs
**Gain:** "authenticated interaction context" becomes true for messaging channels; impersonation-by-spoofed-sender is closed by verified binding; authority is proportional to proven identity; cross-channel continuity cannot launder authority; audit gains assurance granularity — all while preserving origination on any channel.
**Give up:** a one-time enrollment per (user, channel handle); an assurance model channels must declare and policy must map to consequence classes; some low-assurance interactions require a Web Runtime step-up for consequential actions (already the norm for approvals).

## 6. API Changes
Additive; no method signatures change:
- New `ChannelBinding` kind ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)); enrollment is a governed flow through the Web Runtime ([Book 13 §Ch01](../spec/book-13-interfaces/01-web-runtime.md)) and the User/Session Managers ([Book 11 §Ch08](../spec/book-11-security/08-identity-users-sessions.md)).
- The channel adapter interface ([Book 13 §Ch02](../spec/book-13-interfaces/02-channel-model.md)) is unchanged — it still faithfully carries the transport identifier; the Kernel now *verifies* it against a binding rather than trusting it.
- `Session` gains `assurance`; the ingress auth step ([Book 03 §Ch02 §… auth](../spec/book-03-kernel/02-kernel-api.md), [Book 11 §Ch08 §4](../spec/book-11-security/08-identity-users-sessions.md)) resolves *(channel, id)*→binding→User and stamps assurance.

## 7. Resource Changes
- New `ChannelBinding` (secret-free: it holds identifiers and a `User` reference, never credentials — a channel handle is not a secret; proof-of-control artifacts are transient and not stored as values, P7). `Session` ([Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md)) gains `spec/status.assurance`. `Conversation` records per-leg binding/assurance. No lifecycle-model change; `ChannelBinding` is a long-lived, revocable Resource; `Session` remains authenticated/expiring ([Book 11 §Ch08 §3](../spec/book-11-security/08-identity-users-sessions.md)).

## 8. Event Changes
Additive, secret-free (P5): `channel.enrolled`, `channel.bindingRevoked`, `session.assuranceCapped`, `session.stepUpRequired`. These feed audit ([Book 11 §Ch09](../spec/book-11-security/09-audit-and-attribution.md)) and observability ([Book 14 §Ch01](../spec/book-14-observability-governance/01-event-model.md)); per-subject ordering on the `ChannelBinding`/`Session` subjects suffices ([Book 03 §Ch03 §5](../spec/book-03-kernel/03-event-bus.md)).

## 9. Security Impact
This is a **B1 ingress-boundary hardening** ([Book 11 §Ch02 §B1](../spec/book-11-security/02-trust-boundaries.md)) that closes an impersonation path the current "map the channel's identity" text leaves fully open. It strengthens P8 (no authority without *verified* identity) and P10 (attribution now carries assurance). It preserves the untrusted-channel posture: the adapter is still never trusted — the Kernel verifies the identifier against a binding it controls. It composes with, and does not weaken, the Web-Runtime decision-surface defense (approvals/credentials remain there; §4.3 makes that surface the assurance step-up). Threat-model update ([Book 11 §Ch01](../spec/book-11-security/01-threat-model.md)): add channel-identity spoofing/takeover as an enumerated ingress threat. SI checklist ([Book 11 §Ch11](../spec/book-11-security/11-security-invariants.md)): *a channel-native identifier carries authority only via a verified `ChannelBinding`; a Session's authority is capped by its channel leg's assurance; cross-channel continuity re-binds, never inherits.* Security review required.

## 10. Performance Impact
Enrollment is a one-time per-handle flow. Per-message overhead is a binding lookup (indexed by *(channelKind, id)*) plus an assurance cap check — O(1), control-plane. No hot-path (execution) effect.

## 11. Testing Strategy
- Impersonation: an inbound message from an **unbound** identifier ⇒ no Session, cannot issue an Intent (may only start enrollment).
- Spoof: a message with a forged `From` matching a bound address but from an unverified path ⇒ still requires the binding's per-message assurance; low assurance ⇒ consequential action blocked pending step-up.
- Cross-channel: a Session bootstrapped on Web Runtime (high) continued by an email leg (low) ⇒ effective authority re-capped to low; consequential action requires Web Runtime step-up; unbound email identity ⇒ does not join the Session.
- Revocation: revoking a `ChannelBinding` ⇒ subsequent messages from that handle map to no Session (fail-closed).
- Attribution: each turn records binding + assurance; audit distinguishes high- vs low-assurance actions.

## 12. Documentation Changes
Rewrite the identity-mapping sentences in [Book 13 §Ch02](../spec/book-13-interfaces/02-channel-model.md) and [§Ch03](../spec/book-13-interfaces/03-reference-channels.md) from "maps to a Session/User" to "verified against a `ChannelBinding`, with assurance"; extend [Book 11 §Ch08](../spec/book-11-security/08-identity-users-sessions.md) with §4.1–4.2 (binding, enrollment, assurance) and [§Ch02 §B1](../spec/book-11-security/02-trust-boundaries.md) with the identity-verification check; connect the step-up to [Book 13 §Ch06](../spec/book-13-interfaces/06-approval-and-human-in-the-loop.md) and [Book 11 §Ch07](../spec/book-11-security/07-approval-engine.md); add `ChannelBinding` to [Book 02 §Ch07](../spec/book-02-resource-model/07-core-resource-catalog.md). New Glossary entries: *Channel binding*, *Enrollment*, *Assurance level*, *Step-up*.

## 13. Migration Strategy
Additive and pre-1.0. Absent bindings, a workspace policy sets the transition posture: a permissive default MAY treat existing channel identities as `low` assurance (usable for non-consequential interaction, step-up required for anything consequential) with a deprecation window to enroll; a strict policy requires enrollment before any authority. New `ChannelBinding` and `Session.assurance` are additive; existing Sessions default to the assurance of their originating channel. No stored artifact changes meaning.

## 14. Risks
- **Enrollment friction** deterring adoption — mitigated by one-time-per-handle enrollment and the permissive migration posture (low-assurance usable for non-consequential flows).
- **Assurance-level taxonomy drift** across channels — the level definitions must be authoritative and per-channel-declared; flagged for the reflecting PR (a channel's AEP conformance could declare its max assurance).
- **Provider-specific proof-of-control** varies (email link vs. Slack OIDC vs. Telegram code) — the RFC fixes the *requirement*, not the mechanism, consistent with pluggable auth ([Book 11 §Ch08 §4](../spec/book-11-security/08-identity-users-sessions.md)).
- **Interaction with the credential/approval delivery** ([Book 11 §Ch05](../spec/book-11-security/05-credential-acquisition.md)): a one-time page link delivered to a low-assurance channel is already defended by Web Runtime auth at the page, but delivery to a bound, revocable handle further reduces misdelivery — verify the composition in review.

## 15. Future Improvements
Continuous/step-up assurance (re-prompt for high-assurance when risk rises mid-Conversation); channel-reputation signals feeding assurance dynamically; passkey/WebAuthn enrollment for the highest assurance tier; binding a Session to device posture for sensitive workspaces.

---
### Resolved questions
*(none yet)*

### Unresolved questions
- Should the assurance taxonomy be a fixed core ordinal (`low`/`medium`/`high`) or a policy-extensible lattice mapped from per-channel declarations?
- Should a `ChannelBinding` be per-workspace or per-`User`-across-workspaces (a user's Telegram handle is the same globally, but authority is tenant-scoped, [Book 11 §Ch08 §5](../spec/book-11-security/08-identity-users-sessions.md))?
- For voice channels ([Book 13 §Ch03](../spec/book-13-interfaces/03-reference-channels.md)), is caller-ID ever above `low` assurance, or must voice always step up for consequential actions?
