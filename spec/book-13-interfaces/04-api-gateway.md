# Book 13 · Chapter 04 — The API Gateway

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P4, P7, P8, P10. Companion to Book 03 §Ch02 (Kernel API), Book 11 §08 (identity).*

> The API gateway is the **external face of the Kernel API** — how systems and programmatic clients reach Sankalpa over the network. This chapter specifies it, and its defining property: the gateway is a channel *in front of* the Kernel API, not a different API. It adds network concerns (TLS, rate limiting, external auth) but never a second path around the invariants.

## 1. The gateway is the Kernel API, externalized

The Kernel API is specified semantically and transport-independently (Book 03 §Ch02 §5); the API gateway is its **external, network-facing exposure** (Book 03 §Ch02 §5, Book 13 §Ch01 §2). Critically:
- The gateway exposes the *same* verbs, semantics, error model, and — above all — the *same* cross-cutting checks (auth → authorize → admit → policy → execute → audit, Book 03 §Ch02 §3) as the internal Kernel API.
- It is **not** a separate API with its own logic; it is a channel (§Ch02) that carries external requests to the Kernel API and responses back. There is **no** external operation that bypasses the Kernel's pipeline (P4).

This is why the gateway cannot become a security hole: it has no authority of its own and no path around the invariants — it forwards to the one front door (Book 03 §Ch02 §1).

## 2. What the gateway adds

The gateway handles the concerns of *being on the network* that the internal API need not:
- **Transport security** — TLS termination; the external boundary is encrypted.
- **External authentication** — mapping external credentials/tokens (API keys, OAuth, mTLS) to a Session/User (Book 11 §08, §Ch05), which the Kernel API pipeline then authorizes. External auth is establishing *identity*; authority is still capability-based inside (P8, Book 11 §03).
- **Rate limiting & quotas** — protecting the platform from external overload, complementing the Scheduler's backpressure (Book 03 §Ch07, §Ch13). The gateway sheds excess with typed errors rather than passing unbounded load inward.
- **Request shaping** — translating external protocol (HTTP/JSON, gRPC) to Kernel API calls, exactly as a channel adapter translates (§Ch02 §2).

These are *transport concerns*; none of them is *authority* or *governance* — those remain inside the Kernel.

## 3. The gateway enforces nothing it is trusted to enforce alone

A subtle but important discipline: the gateway MAY perform *early* checks (reject an unauthenticated request at the edge, rate-limit) but MUST NOT be the *sole* enforcer of any invariant. The authoritative checks happen at the Kernel API pipeline (Book 03 §Ch02 §3):
- Even if the gateway is bypassed or compromised, the Kernel API still applies full auth/capability/policy/audit (defense in depth, Book 11 §01 §6). The gateway is an *optimization and hardening* layer at the edge, not the security boundary itself.
- This means a gateway misconfiguration cannot grant unauthorized access: the request still faces the Kernel's checks. The gateway makes attacks *harder and cheaper to reject*; the Kernel makes them *impossible to succeed*.

## 4. Privacy and secrets at the network edge (P7)

- **No secrets in URLs/params.** Consistent with the platform-wide rule, the gateway MUST NOT place personal or sensitive data — and never a secret — in URL parameters or query strings (they are logged, cached, and referable). Secrets never transit the gateway as request/response payloads either (P7); credentials are acquired only via the Web Runtime → Broker path (Book 11 §05).
- **Secret-free audit.** Gateway request logging is secret-free (Book 14 §Ch04) and references by id/class, like all logging.

## 5. Attribution and audit (P10)

Every request through the gateway is **authenticated to an identity** (§2) and **audited** (Book 11 §09) — it is a Kernel API caller, so its actions are attributed and recorded exactly as any internal caller's. An external system acting through the gateway is a non-human identity (Book 11 §08 §6) with least-privilege capabilities; its every action is accountable. There is no anonymous external access.

## 6. Invariants (normative summary)

1. The API gateway is the external, network-facing exposure of the Kernel API — the same verbs, semantics, error model, and cross-cutting checks — not a separate API; no external operation bypasses the Kernel pipeline (P4).
2. The gateway adds transport concerns (TLS, external auth mapping, rate limiting/quotas, protocol shaping); it holds no authority of its own.
3. Early edge checks never substitute for the Kernel API's authoritative checks; a bypassed/compromised gateway cannot grant unauthorized access (defense in depth).
4. No secret or sensitive data appears in URLs/params or as gateway payloads; credentials flow only via the Web Runtime → Secret Broker path (P7); gateway logs are secret-free.
5. Every gateway request is authenticated to a least-privilege identity and audited; there is no anonymous external access (P8, P10).
