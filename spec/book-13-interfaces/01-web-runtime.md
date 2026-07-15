# Book 13 · Chapter 01 — The Web Runtime

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P7, P8, P10. Companion to Book 11 §05 (credential pages), §07 (approval), Book 09 §03 (vault editing).*

> The Web Runtime is Sankalpa's integrated web layer — the surface through which humans see dashboards, review approvals, enter credentials securely, browse files, read documentation, and interact with rich outputs. This chapter specifies its responsibilities and the security discipline that governs it. It is not an optional add-on: several core mechanisms (secure credential acquisition, approval rendering) *require* it.

## 1. An integrated web layer, always provided

Sankalpa **always provides an integrated Web Runtime** — it is a first-class part of the platform, not a separate app a deployment must assemble. Rationale: many essential interactions are inherently richer than a chat message (a dashboard, an approval with full context, a secure credential form, a file browser), and the platform's own security mechanisms depend on a trusted web surface it controls (§4). The Web Runtime is that surface.

## 2. Responsibilities

The Web Runtime is responsible for:

| Responsibility | Serves | Notes |
|----------------|--------|-------|
| **Dashboards** | observability (Book 14) | metrics, traces, health, audit views |
| **Markdown & documentation rendering** | Knowledge (Book 09), docs | renders the vault and package docs |
| **Generated websites / project hosting** | Artifacts, Services (Book 02 §Ch07) | hosts produced outputs |
| **Reverse proxy** | hosted services | fronts project-hosted endpoints |
| **Streaming** | live output | streams execution progress |
| **Approval pages** | Approval Engine (Book 11 §07) | renders approval requests with full context |
| **Secure credential pages** | Secret Broker (Book 11 §05) | one-time HTTPS acquisition |
| **Downloads / file browser** | Artifacts | browse and retrieve produced files |
| **API gateway** | external API (§04) | the external face of the Kernel API |

The "return URLs for rich experiences" pattern (Book 01, channels §02) points here: when a channel interaction warrants richness (an approval, a dashboard, a file), the response is a **URL into the Web Runtime**, not a cramped inline rendering.

## 3. The Web Runtime fronts the Kernel; it does not bypass it (P4)

The Web Runtime is an **interface** to the platform, not a parallel authority. Every action it performs on the user's behalf goes through the **Kernel API** (Book 03 §Ch02) with the same authentication, capability checks, admission, policy, and audit as any other caller (P4, P8). The Web Runtime holds no special back door — a dashboard reads Resources via the Kernel API; an approval decision is recorded via the Kernel API; a file browse is a capability-gated read. This keeps the invariants intact regardless of interface (Book 03 §Ch02 §5): the API gateway (§04) is a channel *in front of* the Kernel API, not a different API.

## 4. Security-critical surfaces (P7, P8)

Two Web Runtime surfaces are **security-critical** and specified in Book 11; the Web Runtime is their rendering host:

- **Secure credential pages** (Book 11 §05). The Web Runtime serves the one-time, short-lived, TLS-protected HTTPS pages into which humans enter credentials — which flow **directly to the Secret Broker**, never through a channel, planner, or log (P7). The Web Runtime is trusted to render the page and transport the value write-only to the Broker; it MUST NOT log, echo, or retain the value.
- **Approval pages** (Book 11 §07). The Web Runtime renders approval requests showing the **concrete consequence** (effects, targets, cost) of the gated action, and captures the non-repudiable decision (Book 11 §07 §4) — never displaying a secret value (approvals authorize secret *use* by reference, P7).

Because these depend on a trusted web surface the platform controls, the Web Runtime being *integrated* (§1) is a security requirement, not a convenience.

## 5. Tenancy, capability, and audit (P8, P10)

- Every Web Runtime interaction is **authenticated** (a Session, Book 11 §08, §Ch05) and **capability-gated** (P8): a user sees and does only what their grants permit, scoped to their **workspace** (Book 11 §08 §5). A dashboard shows one tenant's data; another tenant's is unreachable.
- Every consequential action through the Web Runtime is **audited** (P10, Book 11 §09) — it is a Kernel API caller like any other, so its actions are attributed and recorded.
- Hosted/generated content (Artifacts, Services) is served within tenancy and policy: the reverse proxy and file browser respect workspace scoping and never expose one tenant's artifacts to another.

## 6. Extensible via Packages (P11)

The Web Runtime's UI is **extensible**: Packages may contribute UI (Book 12 §01) — custom dashboards, project views, rendered experiences — as governed contributions (registered, isolated, least-privilege like any Package content, Book 11 §10). The core Web Runtime provides the security-critical and standard surfaces (§2, §4); the ecosystem extends the rest. Contributed UI is subject to the same tenancy/capability/audit discipline (§5).

## 7. Invariants (normative summary)

1. Sankalpa always provides an integrated Web Runtime as a first-class surface; several core mechanisms (credential pages, approval rendering) require it.
2. It provides dashboards, doc/markdown rendering, project hosting/reverse proxy, streaming, approval and secure credential pages, downloads/file browser, and the API gateway; rich channel responses return URLs into it.
3. The Web Runtime acts through the Kernel API with full auth/capability/policy/audit; it holds no back door and never bypasses the invariants (P4).
4. Its security-critical surfaces — secure credential pages (value goes write-only to the Broker, P7) and approval pages (consequence shown, decision non-repudiable, no secret shown) — are specified in Book 11 and hosted here.
5. Every interaction is authenticated, capability-gated, workspace-scoped, and audited (P8, P10); hosted content respects tenancy and policy.
6. The UI is extensible via governed Package contributions under the same isolation/tenancy/audit discipline (P11).
