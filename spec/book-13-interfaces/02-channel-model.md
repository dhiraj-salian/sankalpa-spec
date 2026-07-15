# Book 13 · Chapter 02 — The Channel Model

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P4, P7, P8, P11. Companion to Book 11 §02 (trust boundaries), Book 01 (channels).*

> A channel is a **transport adapter** — Telegram, Slack, Email, CLI, Voice, REST, MCP — that carries intent in and results out. This chapter specifies the channel model and its defining constraint: **channels carry, they do not decide.** A channel moves messages across the ingress boundary; all interpretation, authority, and governance happen inside the Kernel. This keeps the platform's behavior independent of how a user happens to reach it.

## 1. Channels are transport, not intelligence

The defining rule: **a channel is a transport adapter that carries messages; it does not interpret intent, hold authority, or make decisions.** A channel:
- receives a user's message and delivers it to the Kernel as an `Intent` (or a reply, approval, etc.) through the Kernel API (P4);
- receives results/requests from the Kernel and renders them to the user in the channel's medium.

Everything *between* — deriving Goals, planning, governing, executing — happens inside the Kernel, identically regardless of channel (Book 08, Book 05, Book 06). A Slack message and an email produce the *same* platform behavior because the channel only transported them differently. This is why "channels are transport adapters" (Book 01) is a hard architectural rule, not a description: putting decision logic in a channel would fork the platform's behavior per transport and bypass the Kernel's governance.

## 2. The channel adapter interface (P11)

Channels are **plugins** behind an AEP-governed adapter interface (Book 12), like planners and runtimes:

```
Channel (adapter interface):
  describe() -> ChannelDescriptor          # medium, capabilities (text/rich/voice), auth model
  receive(inbound) -> KernelRequest         # transport message → Intent/reply/approval (via Kernel API)
  deliver(outbound)                          # Kernel result/request → rendered channel message
  init(grantSet, limits) / health() / shutdown()
```

- The adapter translates between the channel's native format and the Kernel's vocabulary (Intents, replies, approval decisions). It does **not** add behavior beyond faithful translation and rendering.
- A channel is **untrusted** (Book 11 §10): isolated, least-privilege, granted only what transport requires (P8). It cannot reach Resources, secrets, or other tenants except through the Kernel-mediated requests it faithfully carries.
- New channels are added by implementing the interface and packaging them (Book 12) — the core names no specific channel (Book 03 §Ch04 §3, P11).

## 3. The ingress boundary and its checks (Book 11 §02 §B1)

A channel sits at the **ingress boundary** (Book 11 §02 §B1). Everything it carries inbound is **untrusted input** and is checked inside the Kernel, never by the channel:
- **Authentication** — the channel's transport identity is mapped to a Session/User (Book 11 §08, §Ch05) at the Kernel API; the channel does not self-authorize.
- **Authorization/capability, admission, policy** — applied by the Kernel API pipeline (Book 03 §Ch02 §3) to whatever the channel delivered.
- **Content is data, not command** — a message arriving via a channel is *input to be governed*, never an instruction that bypasses governance. (This mirrors the platform's own instruction-source discipline: transported content is data.)

The channel's job ends at faithful delivery; the Kernel's governance begins there.

## 4. Rich experiences return URLs (into the Web Runtime)

Channels are often narrow (a chat line, an SMS). When an interaction warrants richness — an approval with full context, a dashboard, a file, a generated site — the Kernel's response is a **URL into the Web Runtime** (§Ch01 §2), which the channel delivers. Rationale:
- The rich, security-sensitive experience (approval, credential entry) is rendered on the **trusted Web Runtime surface** the platform controls (§Ch01 §4), not reconstructed insecurely inside a third-party channel.
- **Secrets and approvals never happen inside a channel** (P7): a credential is entered on a Web Runtime page (Book 11 §05), an approval is decided on a Web Runtime page (Book 11 §07) — the channel only carries the *link*, never the secret or the raw decision surface.

This "return a URL for rich experiences" pattern is how channels stay simple transports while users still get rich, secure interactions.

## 5. Secrets never flow through channels (P7)

An absolute rule at this boundary: **a secret value never transits a channel.** Credentials are acquired out-of-band via the Web Runtime → Secret Broker path (Book 11 §05), never pasted into a chat/email/CLI. A channel that solicited or carried a secret would breach P7 at the ingress boundary (Book 11 §02 §B1). The channel adapter MUST NOT log or retain message content in a way that could capture a secret, and the platform never asks a user to enter a secret *into* a channel — it sends a Web Runtime URL instead (§4).

## 6. Communication is uniform across channels

Because channels are pure transport and all behavior is in the Kernel:
- A **Conversation** (Book 02 §Ch07, §Ch05) may **span channels** — a user starts in Slack and continues by email — because the conversation lives in the Kernel, correlated by Session, not in any one channel (§Ch05).
- Adding, removing, or swapping a channel changes *how users reach* the platform, never *what the platform does* (P11). The same intent, the same governance, the same execution — different doors.

## 7. Invariants (normative summary)

1. A channel is a transport adapter that carries messages; it does not interpret intent, hold authority, or make decisions — all behavior is in the Kernel, identical across channels.
2. Channels are untrusted, isolated, least-privilege plugins behind an AEP-governed adapter interface; the core names no specific channel (P11).
3. At the ingress boundary, everything a channel carries is untrusted input checked by the Kernel API pipeline (auth, capability, admission, policy); transported content is data, not command.
4. Rich or security-sensitive interactions return URLs into the trusted Web Runtime; approvals and credential entry never happen inside a channel.
5. A secret value never transits a channel; credentials flow only via the Web Runtime → Secret Broker out-of-band path (P7).
6. Conversations may span channels because behavior lives in the Kernel; adding/swapping a channel changes how users reach the platform, never what it does.
