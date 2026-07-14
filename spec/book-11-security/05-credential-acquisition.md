# Book 11 · Chapter 05 — Credential Acquisition

*Nature: **Normative**. · Reflects: SECURITY.md; realizes principle P7 (and P8, P10). Companion to Book 11 §04 (Secret Broker), Book 13 (Web Runtime/channels).*

> Chapter 04 specified how a stored secret is *used* (materialized at execution by reference). This chapter specifies how a secret first *enters* the Broker — **acquisition** — without ever passing through a planner, a chat channel, a log, or any reasoning context (P7). Acquisition is where the reference model is seeded; if a secret leaked *here*, the whole downstream discipline would be moot.

## 1. The problem: getting a secret in without leaking it

An agent platform routinely needs credentials it does not yet hold ("connect my CRM," "post to my Slack"). The naive path — ask the user to paste a token into the chat — is catastrophic: the secret enters the channel, the Conversation Resource, the planner context, the model provider, and the logs. Acquisition must therefore be **out-of-band** from the conversational/planning flow: the secret goes *directly* into the Secret Broker and nowhere else.

## 2. Secure one-time acquisition pages

The primary mechanism is a **secure, one-time HTTPS acquisition page** served by the Web Runtime (Book 13 §01):

- When a plan or setup needs a credential, the system issues a **one-time, short-lived, capability-scoped URL** to the human through a channel (Book 13). The channel message carries the *URL*, never a prompt for the secret itself.
- The human enters the credential **directly into the HTTPS page**, whose backend delivers it **straight to the Secret Broker** (Ch 04). The value never transits the chat channel, the Conversation Resource, the planner, or any log.
- The page is **one-time and expiring**: the URL is single-use and time-bounded; after submission (or expiry) it is dead. This bounds the window and prevents reuse.
- The page and its transport are TLS-protected; the backend treats the value as write-only into the Broker (it does not echo, log, or store it elsewhere).

The result: the secret's *only* journey is human → HTTPS page → Secret Broker. It never touches any surface P7 protects.

## 3. OAuth preferred over passwords

Where a target system supports it, **OAuth (or equivalent delegated authorization) is preferred over raw passwords/keys** (SECURITY.md):

- The acquisition flow initiates an OAuth authorization-code exchange; the human authorizes at the provider; the system receives **tokens** (access/refresh), which the Broker custodies as secrets.
- Advantages: no long-lived password is ever held; scopes are explicit and least-privilege (aligning with P8); tokens are revocable at the provider; refresh is automatable.
- Passwords/API keys are accepted only when OAuth is unavailable, and are treated as higher-risk secrets (stricter policy, rotation — §5).

Preferring OAuth is a security *default*, not merely a convenience: it minimizes the value of what the Broker must hold and shrinks the blast radius of a compromise.

## 4. Acquisition is capability-gated and audited (P8, P10)

- Initiating an acquisition (issuing a one-time page or an OAuth flow) is itself a **capability-gated** action (P8): only an authorized subject, for an authorized target and scope, may trigger it. A plugin cannot silently provoke credential collection it was not entitled to request.
- Every acquisition is **audited** (Ch 09): that an acquisition for target T, scope S, was initiated for workspace W and completed — **by reference**, never the value (P10). The audit proves a credential was obtained appropriately without recording it.
- The human is shown *what* is being connected and *with what scope* before authorizing — informed consent, and a defense against a malicious plugin trying to over-scope a grant.

## 5. Storage, scope, and rotation

- On receipt, the Broker stores the credential in its separate protected store (Ch 04 §2) with its **class** (e.g. `oauth-token`, `api-key`, `payments`), **scope**, and **rotation policy**.
- **Least-privilege scope.** Acquired credentials SHOULD carry the narrowest scope that satisfies the need (OAuth scopes; restricted API keys). Over-broad credentials are a policy concern (Ch 06).
- **Rotation.** Rotatable credentials (OAuth refresh, rotatable keys) are rotated per policy; the reference stays stable while the value changes (Ch 04 §7), so no consumer is disrupted and exposure windows shrink.

## 6. What acquisition must never do

- MUST NOT accept a secret through a conversational channel, a planner interaction, or any path that would place it in a Conversation Resource, planner context, log, or the model provider (P7). Any UI that could collect a secret routes exclusively through the Broker-backed acquisition page.
- MUST NOT return an acquired value to the initiating plan/planner; the plan receives only a `SecretRef` (Ch 04 §3).
- MUST NOT persist the value outside the Broker's protected store.

A violation here is the most serious kind of security defect, because it would undermine the entire reference model that everything downstream relies on.

## 7. Invariants (normative summary)

1. Secrets enter the system only out-of-band: human → secure one-time HTTPS page (or OAuth flow) → Secret Broker; never through a channel, planner, Conversation, or log (P7).
2. Acquisition pages/URLs are one-time, short-lived, capability-scoped, and TLS-protected; the backend delivers the value write-only into the Broker.
3. OAuth (delegated, scoped, revocable) is preferred over passwords/keys; raw credentials are accepted only when necessary and treated as higher-risk.
4. Initiating acquisition is capability-gated with informed human consent on target and scope; every acquisition is audited by reference (P8, P10).
5. Acquired credentials are stored only in the Broker's separate protected store with class, least-privilege scope, and rotation policy; references stay stable across rotation.
6. Acquisition never routes a secret through a conversational/planning path, never returns a value to the plan (only a `SecretRef`), and never persists it outside the Broker.
