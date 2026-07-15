# Book 13 · Chapter 07 — MCP Integration

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P1, P7, P8, P11. Companion to §Ch02 (channel model), Book 03 §Ch06 (Capabilities), Book 11 (Security).*

> The Model Context Protocol (MCP) is both a **channel** into Sankalpa and a surface through which Sankalpa **exposes** capabilities to other systems. This chapter specifies both directions and the governance that keeps them safe. MCP is significant because it is how Sankalpa interoperates with the broader agent/tool ecosystem — as a client and as a server — without weakening any invariant.

## 1. Two directions

MCP integration is bidirectional:
- **Sankalpa as MCP client** — the platform consumes external MCP servers' tools/resources, mapping them into governed Capabilities (§3).
- **Sankalpa as MCP server** — the platform exposes selected Capabilities to external MCP clients, as a governed channel (§4).

Both directions pass through the Kernel's governance; neither is a back door (P4). MCP is powerful *because* it interoperates widely — so its governance discipline is correspondingly strict.

## 2. MCP as a channel (§Ch02)

As a transport, MCP is a **channel adapter** (§Ch02 §2): an MCP client's requests become Kernel requests, and results render back over MCP. It obeys the channel model (§Ch02 §1): it carries and renders, it does not decide; it is untrusted, isolated, least-privilege (Book 11 §10); and everything it carries is untrusted input checked by the Kernel API pipeline (§Ch02 §3). A secret never transits MCP (§Ch02 §5, P7).

## 3. Sankalpa as MCP client — external tools become governed Capabilities

When Sankalpa consumes an external MCP server, its tools are **not** invoked directly by planners or executed as opaque calls. Instead they are **mapped into typed, governed Capabilities** (Book 03 §Ch06, Book 04 §Ch06):
- An external MCP tool is wrapped as a `Capability` with a **typed signature** and **declared effects** (Book 04 §Ch06) — so a plan invoking it is type-checked, effect-annotated, policy-validated (Book 05 §Ch04), and executed under the runtime discipline (Book 06) like any other Capability.
- Invocation is **capability-gated** (P8) and its effects (network, cost, secret use) are **declared and governed** — an external tool cannot cause an ungoverned or undeclared effect just because it came via MCP.
- Any credential the external tool needs is a **`SecretRef`** materialized only at execution by the Broker (Book 11 §04, Book 06 §Ch06) — never handed to the tool ahead of time or exposed to planning (P7).

This is the crucial discipline: **MCP does not bypass the IR/capability/effect model.** External tools enter as governed Capabilities, so consuming the MCP ecosystem does not create an ungoverned execution path. Natural-language tool descriptions from an MCP server are *data* for wrapping into a typed Capability, never executable instructions (P1, and consistent with the platform's instruction-source discipline).

## 4. Sankalpa as MCP server — exposing Capabilities safely

When Sankalpa exposes Capabilities to external MCP clients:
- Only **explicitly published** Capabilities are exposed, **capability-gated** per client identity (Book 11 §08 §6): an external client is a non-human identity with least-privilege grants (P8), and sees only what it is authorized for.
- Exposed invocations pass the **full Kernel pipeline** (Book 03 §Ch02 §3) — auth, capability, admission, policy, audit — exactly as internal ones. An external MCP client cannot invoke anything ungoverned.
- **No secret is exposed** (P7): an exposed Capability may *use* secrets internally (materialized at execution, Book 06 §Ch06), but never returns or reveals a secret value to the MCP client.
- Exposure is **policy-governed and audited** (P9, P10): a workspace controls what it exposes, to whom, and every external invocation is attributed and recorded (Book 11 §09).

## 5. MCP interoperation never weakens an invariant

The governing principle, stated plainly: **MCP interoperability is achieved without relaxing any invariant.** Whether consuming or exposing:
- Effects are declared and governed (P9); nothing MCP does escapes the effect/policy model.
- Authority is capability-based (P8); MCP grants no ambient authority in either direction.
- Secrets stay references, materialized only at execution (P7); MCP never carries a value.
- Only AOS IR executes (P1); an MCP tool description is data wrapped into a typed Capability, never executed as text.

MCP is thus a *governed interoperability surface*, not an exception to the architecture. This is what lets Sankalpa participate in the wider ecosystem — as client and server — while remaining the deterministic, secure platform the rest of the spec defines.

## 6. Extensibility (P11)

MCP support ships and evolves as governed Package contributions (Book 12): MCP client/server adapters and Capability-wrapping providers are Packages, isolated and least-privilege like any extension. The core defines the *governance* (Capabilities, effects, policy, secrets); the MCP specifics are an extension surface — so Sankalpa tracks the evolving MCP ecosystem without core changes.

## 7. Invariants (normative summary)

1. MCP integration is bidirectional (Sankalpa as client and as server); both directions pass through the Kernel's governance and are not back doors (P4).
2. As a channel, MCP obeys the channel model: transport-only, untrusted, isolated, least-privilege, carrying no secret (§Ch02).
3. As a client, Sankalpa wraps external MCP tools into typed, effect-declared, capability-gated Capabilities; MCP never bypasses the IR/capability/effect/policy model, and tool descriptions are data, not executable instructions (P1, P8, P9).
4. As a server, Sankalpa exposes only explicitly published Capabilities, capability-gated per external identity, through the full Kernel pipeline, exposing no secret, policy-governed and audited (P7, P8, P9, P10).
5. MCP interoperability never relaxes an invariant: effects governed, authority capability-based, secrets by reference at execution, only IR executes.
6. MCP support ships as governed, isolated Package contributions, so Sankalpa tracks the MCP ecosystem without core changes (P11).
