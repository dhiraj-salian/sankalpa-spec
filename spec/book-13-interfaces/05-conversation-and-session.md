# Book 13 · Chapter 05 — Conversation and Session

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P2, P8, P10. Companion to Book 11 §08 (identity/sessions), Book 02 §Ch07, Book 09 §01 (knowledge vs. memory).*

> `Conversation` and `Session` are the Resources that give interaction continuity and identity. This chapter specifies them: how a Conversation spans channels, how a Session scopes authority, and how they correlate a user's activity into a coherent, attributable whole — without becoming the "memory" that Sankalpa deliberately rejects (Book 09 §01).

## 1. Two Resources, two roles

- A **`Session`** (Book 02 §Ch07, Book 11 §08 §3) is an authenticated interaction context: it establishes *who* is interacting and *with what bounded authority*, and it expires. It is the security/identity envelope.
- A **`Conversation`** (Book 02 §Ch07) is a dialogue: the sequence of turns (intents, replies, clarifications, approvals) exchanged with a user, potentially across channels. It is the continuity envelope.

Both are ARM Resources (P2), so they are observable, versioned, governed, and attributable like everything else — interaction state is first-class, not ad-hoc.

## 2. Conversations span channels

Because channels are pure transport and behavior lives in the Kernel (§Ch02 §6), a **Conversation may span multiple channels**: a user starts in Slack, continues by email, approves on the Web Runtime — one Conversation, correlated by Session (§3), not fragmented per transport. This is only possible because the conversation lives in the Kernel, not in any channel adapter (§Ch02 §1). The user experiences continuity; the platform experiences one coherent, attributable dialogue.

Conversations correlate their turns to the `Intent`s, `Goal`s, and `Execution`s they produce (by reference, Book 02 §Ch06), so the whole thread — request → plan → execution → result — is traceable (P10, Book 14 §Ch03).

## 3. Sessions scope authority (P8)

A Session carries a **bounded, least-privilege capability scope** that is an **attenuation** of the user's authority (Book 11 §08 §3): a session never holds more than the user, and often less (scoped to a channel, a task, a time window). Consequences:
- Every request in a Conversation is authorized against the **Session's** scope (Book 03 §Ch02 §3) — so interaction authority is bounded and expiring, limiting the blast radius of a stolen session token.
- Session **expiry is fail-closed** (Book 11 §08 §3): an expired session's requests are denied, not treated as anonymous-with-defaults.
- A Session ties a Conversation's activity to an **authenticated identity** (Book 11 §08), so every turn is attributed (P10).

## 4. Continuity is not memory (Book 09 §01)

A crucial boundary: **Conversation continuity is not the "memory" anti-pattern.** A Conversation records its turns for *continuity, correlation, and audit* — it is a first-class, structured, attributed Resource. It is **not** a rolling buffer of recent turns silently re-injected into planner context (Book 09 §01 §2). Specifically:
- What a planner receives is **non-secret Knowledge context** (Book 09 §06) plus the *current* intent — not the raw Conversation history stuffed into a prompt.
- Short-term conversational state that *is* needed for continuity lives in the Conversation Resource with its own lifecycle and tenancy — governed, not smuggled into reasoning context.
- This keeps the P7 discipline intact: a Conversation may contain what a user typed, but it is never a bypass that pours conversation (possibly including something sensitive a user pasted) into a model prompt or a durable Knowledge base without governance.

The distinction matters because conflating conversation-continuity with planner-memory is exactly how sensitive data leaks into reasoning context in naive systems (Book 09 §01 §2, Book 11 §01).

## 5. Secrets and Conversations (P7)

- A Conversation MUST NOT be used to collect or store secret values (§Ch02 §5): a user is never asked to type a credential into the dialogue; the platform sends a Web Runtime credential-page URL (Book 11 §05). A Conversation that captured a secret would breach P7 at the interaction layer.
- If a user nonetheless includes sensitive content in a message, the platform treats Conversation content as tenant-sensitive data (classified, governed, tenant-scoped) — and, per §4, it is not silently promoted into planner context or durable Knowledge without governance.

## 6. Tenancy and lifecycle

- Both Resources are **workspace-scoped** (Book 11 §08 §5) and **attributed** (P10); a Conversation/Session belongs to a tenant and its identity.
- They follow the ARM lifecycle (Book 02 §Ch04): a Session expires (terminal), a Conversation is retained per policy (for continuity/audit) and reclaimed per retention (Book 02 §Ch04 §5). Retention is tenant-scoped and governed.

## 7. Invariants (normative summary)

1. `Session` (authenticated, bounded-authority, expiring identity envelope) and `Conversation` (cross-channel dialogue continuity) are first-class ARM Resources — observable, versioned, attributable (P2, P10).
2. A Conversation may span channels because dialogue lives in the Kernel, correlated by Session; its turns link to the Intents/Goals/Executions they produce.
3. A Session's authority is a least-privilege, expiring attenuation of the user's; every conversational request is authorized against the Session scope, fail-closed on expiry (P8).
4. Conversation continuity is not the memory anti-pattern: turns are recorded as a governed Resource, never silently re-injected into planner context; planners receive non-secret Knowledge context plus the current intent (Book 09 §01).
5. Conversations never collect or store secret values; credentials route to the Web Runtime → Broker path; user-provided sensitive content is classified, governed, and not silently promoted (P7).
6. Both Resources are workspace-scoped, attributed, and lifecycle-governed with tenant-scoped retention.
