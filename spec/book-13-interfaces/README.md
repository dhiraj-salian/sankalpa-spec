# Book 13 — Interfaces & Channels

*Status: **Draft-complete** (all chapters authored) · Nature: Normative. · Reflects: realizes P4, P10, P11.*

## Scope
How humans and systems interact with Sankalpa: the integrated **Web Runtime**, the **channel** adapters (transport only), and the API gateway. Channels carry intent and results; they never make decisions.

## Chapters
1. **`01-web-runtime.md`** *(Normative)* — The integrated web layer: dashboards, markdown/documentation rendering, generated websites, project hosting, reverse proxy, streaming, approval pages, downloads, file browser, API gateway.
2. **`02-channel-model.md`** *(Normative)* — Channels as transport adapters; the adapter interface (AEP-governed); the rule that rich experiences return URLs into the Web Runtime.
3. **`03-reference-channels.md`** *(Informative)* — Telegram, WhatsApp, Slack, Discord, Teams, Email, REST, CLI, Web UI, Voice, MCP, and future channels — each mapped to the adapter interface.
4. **`04-api-gateway.md`** *(Normative)* — External API surface over the Kernel API; authn/authz (Book 11); rate limiting and quotas.
5. **`05-conversation-and-session.md`** *(Normative)* — Conversation and Session as Resources; correlation across channels; continuity.
6. **`06-approval-and-human-in-the-loop.md`** *(Normative)* — Rendering Approval Engine requests (Book 11) as web experiences; secure one-time pages.
7. **`07-mcp-integration.md`** *(Normative)* — Sankalpa as an MCP client and server; mapping MCP to channels and capabilities.
