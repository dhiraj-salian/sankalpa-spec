# Book 11 · Chapter 08 — Identity, Users, and Sessions

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P8, P10 (and tenancy). Companion to Book 03 §Ch02 (Kernel API pipeline), Book 13 §05 (Conversation/Session).*

> Authorization is capability-based (Ch 03), but capabilities are granted *to someone* and actions are attributed *to someone*. This chapter specifies identity: the User and Session Managers, authentication, how identity feeds the capability model **without** becoming ambient authority, and multi-tenant isolation.

## 1. Identity feeds capabilities but is not authority

A subtlety that must be stated plainly: **identity is not authority** (P8, Ch 03 §1). Knowing *who* a subject is does not, by itself, permit anything. Identity's role is narrower and precise:
- It is the basis on which **capabilities are granted** to a subject (a user, a session, a plugin instance).
- It is the basis on which **actions are attributed** (P10, Ch 09).

But an authenticated identity with no grants can do nothing (Ch 03 §3). This is the deliberate inversion of ambient-authority systems where identity implies broad power. Here, identity is a *name to grant capabilities to and attribute actions to* — never a source of implicit permission.

## 2. Users

A **`User`** (Book 02 §Ch07) represents an identity — a human, a service account, or an organizational principal — with roles and workspace membership. The **User Manager** (Book 03 §Ch11 §1.4) owns:
- **Registration & identity federation** — integrating external identity providers (SSO/OIDC) as provider plugins (P11); the *contract* is authentication, the *provider* is pluggable (ADR-0001).
- **Roles & membership** — a user's roles within workspaces, which *inform which capabilities are granted* (§1) but are not themselves ambient permissions.
- **Lifecycle** — deactivation/removal, which revokes the user's grants (Ch 03 §5) promptly.

Roles are a *convenience for granting* (a role bundles a least-privilege capability set), not a bypass of the capability model: exercising a role's authority still means holding and invoking specific capabilities.

## 3. Sessions

A **`Session`** (Book 02 §Ch07, Book 13 §05) is an authenticated interaction context — the scope within which a user (via a channel) interacts with the platform. The **Session Manager** owns:
- **Authentication** — establishing, at the Kernel API front door (Book 03 §Ch02 §3 step 1), that a request belongs to a valid session/identity.
- **Scope & expiry** — a session carries a bounded, least-privilege capability scope and a lifetime; it expires (fail-closed on expiry, Ch 01 §6).
- **Correlation** — linking a session's requests, Conversations, Intents, and resulting Executions so activity is coherent and attributable across channels (Book 13 §05).

A session's authority is itself an **attenuation** (Ch 03 §4) of the user's: a session never holds more than the user, and often less (scoped to a channel, a task, a time window). This limits the blast radius of a stolen session token.

## 4. Authentication at the front door

- **Every** Kernel API request is authenticated before authorization (Book 03 §Ch02 §3). There is no unauthenticated side door (P4).
- Authentication failure is **fail-closed**: an unauthenticated or expired request is denied, never treated as anonymous-with-defaults (Ch 01 §6).
- The mechanism (tokens, mTLS, OIDC) is a provider concern; the *requirement* — authenticate before authorize before act — is normative and uniform.

## 5. Multi-tenant isolation

Tenancy is enforced through **Workspaces** (Book 02 §Ch02, §Ch08 §7) as the isolation boundary (Ch 02 §B6):
- Every Resource read/write/watch and every Event subscription is **workspace-scoped**; a subject can touch or observe only workspaces it is authorized for (P8). A buggy or hostile subject cannot reach another tenant's data.
- **Cross-workspace access requires an explicit, audited, capability-gated grant** (Book 02 §Ch06 §5); silent cross-tenant flow is prohibited (Ch 02 §B6).
- Isolation extends through the whole stack: plugins are granted capabilities scoped to a workspace; the Secret Broker resolves only secrets the workspace owns; Experience and Knowledge are tenant-scoped (Books 09–10). Tenancy is not a single check but a property threaded through every layer.

## 6. Non-human identities

Plugins, controllers, and service accounts are identities too (§1): each is granted a least-privilege capability set at load/registration (Book 03 §Ch06 §5, §Ch09), and each action is attributed to it (Ch 09). A runtime materializing a secret, a controller writing status, a planner calling a model — all act *as* an attributed, capability-scoped identity, never anonymously. This is what makes the audit trail (Ch 09) complete: there is no unattributed actor in the system.

## 7. Invariants (normative summary)

1. Identity is the basis for granting capabilities and attributing actions — not a source of permission; an authenticated identity with no grants can do nothing (P8).
2. Users carry roles/membership that inform (bundle) least-privilege grants; roles are a granting convenience, not an authority bypass; deactivation revokes grants.
3. Sessions are authenticated, bounded-scope, expiring attenuations of a user's authority; expiry and auth failure are fail-closed.
4. Every Kernel API request is authenticated before authorization; there is no unauthenticated side door.
5. Workspaces are the tenancy boundary; all access is workspace-scoped and cross-tenant flow requires an explicit, audited, capability-gated grant, threaded through every layer.
6. Non-human actors (plugins, controllers, service accounts) are capability-scoped, attributed identities; no actor is anonymous.
