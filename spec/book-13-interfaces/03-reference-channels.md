# Book 13 · Chapter 03 — Reference Channels

*Nature: **Informative**. · Reflects: illustrates P4, P7, P11. These are transport *adapters*, not dependencies of the core.*

> This chapter sketches how representative channels map to the adapter interface (§Ch02). It is illustrative: each is a *plugin adapter*, and the core names none of them (Book 03 §Ch04 §3, P11). The purpose is to show that the "channels carry, they do not decide" model (§Ch02 §1) holds uniformly across media as different as a chat app, an email inbox, a CLI, and voice — the evidence that the platform is genuinely transport-agnostic.

## 1. How to read these mappings

For each channel the questions are: how does it translate an inbound message into a Kernel request (an `Intent`/reply/approval), and how does it render Kernel outputs — including the "return a URL into the Web Runtime for rich/secure experiences" pattern (§Ch02 §4)? Whatever the medium, the adapter only *transports and renders*; it never decides (§Ch02 §1), and no secret ever flows through it (§Ch02 §5).

## 2. Messaging channels (Telegram, WhatsApp, Slack, Discord, Teams)

- **Inbound:** a user message becomes an `Intent` (or a reply within a Conversation, §Ch05); the channel's user identity maps to a Session (Book 11 §08) at the Kernel API.
- **Outbound:** text results render inline; rich results (approvals, dashboards, files) render as **Web Runtime URLs** (§Ch02 §4) the user clicks — so an approval is decided on the trusted web page, not via a chat button that reconstructs the decision surface insecurely.
- **Multi-channel:** a Conversation may start in one messaging app and continue in another or by email, correlated by Session in the Kernel (§Ch05).

## 3. Email

- **Inbound:** an email becomes an `Intent`/reply; threading maps to a Conversation (§Ch05).
- **Outbound:** replies render results; rich experiences link to the Web Runtime. Email's asynchrony suits long-running work — the platform can email a URL to an approval or a finished artifact.
- **Secrets:** a credential is **never** solicited by email; the platform emails a one-time Web Runtime credential-page URL (Book 11 §05), never a request to reply with a secret (§Ch02 §5, P7).

## 4. REST and CLI

- **REST:** a programmatic channel — requests map to Kernel API operations through the **API gateway** (§Ch04), which is itself the external face of the Kernel API. REST is how systems (not humans) submit intents and read results, under the same auth/capability/policy (§Ch04).
- **CLI:** a text channel for operators/developers; commands map to Kernel requests. The CLI renders results and opens Web Runtime URLs for rich views. Like all channels, it carries and renders; the Kernel decides.

## 5. Voice

- **Inbound:** speech is transcribed by the adapter into an `Intent`; the adapter handles the medium (audio ↔ text) but adds no decision logic (§Ch02 §1).
- **Outbound:** spoken responses for simple results; for anything rich or security-sensitive (an approval, a credential), the adapter directs the user to a Web Runtime URL (e.g. sent via a companion channel) — a voice channel MUST NOT collect a secret by voice (§Ch02 §5).

## 6. MCP

- **MCP** (Model Context Protocol) is both a channel and a deeper integration surface; its mapping is specified normatively in §Ch07. As a channel, an MCP client's requests become Kernel requests, and Sankalpa may also *expose* capabilities via MCP — under the same governance as any channel (§Ch07).

## 7. Web UI

- The **Web UI** is the human-facing front of the Web Runtime (§Ch01) — itself an interface surface rather than a separate transport adapter, but listed here for completeness. It is where dashboards, approvals, credential pages, and file browsing live, all acting through the Kernel API (§Ch01 §3).

## 8. What the spread demonstrates

A chat app, an inbox, a CLI, a phone call, and a protocol integration are wildly different media — yet each, as an adapter, does the *same* thing: transport an intent in, render results out, return Web Runtime URLs for richness, and never carry a secret or make a decision. That is the payoff of the channel model (§Ch02, Book 01):

- **Transport-agnostic** (P11): the platform behaves identically regardless of channel; adding one is writing an adapter, never a core change.
- **Secure by uniformity** (P7): because secrets and approvals always route to the trusted Web Runtime (§Ch02 §4–5), no channel — however untrusted — is ever a secret-leak or unauthorized-decision surface.
- **Coherent across media** (§Ch05): one Conversation can span channels because the conversation lives in the Kernel, not the transport.

## 9. Note on adding a channel

A new channel is added by implementing the adapter interface (§Ch02 §2), packaging it (Book 12), and registering it — never by editing this specification. This chapter's list is illustrative and open-ended, exactly as P11 intends.
