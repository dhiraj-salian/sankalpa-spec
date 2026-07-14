# Book 11 · Chapter 02 — Trust Boundaries

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P4, P7, P8. Companion to Book 03 (Kernel), Book 01 §04 (principles).*

> A trust boundary is a line across which data or control passes between components of different trust levels; at every such line, something must be checked. This chapter enumerates Sankalpa's trust boundaries, states what crosses each and what is checked, and identifies the one boundary that dominates all others: the untrusted-plugin boundary.

## 1. The trust topology

Sankalpa has one **trusted core** (the Kernel's Managers, Book 03 §Ch04) surrounded by concentric rings of decreasing trust:

```
         ┌──────────────────────────────────────────────┐
         │  UNTRUSTED: users, channels, model providers  │
         │   ┌────────────────────────────────────────┐  │
         │   │ UNTRUSTED PLUGINS: planners, runtimes,  │  │
         │   │  compiler backends, providers, pkgs     │  │
         │   │   ┌────────────────────────────────┐    │  │
         │   │   │  TRUSTED CORE (Kernel Managers) │    │  │
         │   │   │  + Secret Broker protected store│    │  │
         │   │   └────────────────────────────────┘    │  │
         │   └────────────────────────────────────────┘  │
         └──────────────────────────────────────────────┘
```

The core is the only thing that must be correct (Ch 01 §4). Everything in the outer rings is assumed potentially hostile. Communication crosses rings **only** through the Kernel API and Event Bus (P4) — there is no side channel between an outer ring and the core, or between two plugins.

## 2. The boundaries, enumerated

For each boundary: what crosses, and what is checked at the crossing.

### B1 — User/channel ↔ Kernel (the ingress boundary)
- **Crosses:** Intent (natural language), commands, approvals, credential submissions.
- **Checked:** authentication and session (Ch 08); capability authorization of the requested verb (Ch 03); admission/validation (Book 03 §Ch05); control-plane policy (Ch 06). Credential submissions are diverted to the Secret Broker's secure channel (Ch 05), never handled inline.

### B2 — Plugin ↔ Kernel (the dominant boundary, §3)
- **Crosses:** a planner receives Goals + non-secret context and returns High IR; a runtime receives a RuntimeGraph + reference-bearing `executionCtx` and returns Events; a backend returns a RuntimeGraph; providers exchange domain data.
- **Checked:** the plugin holds only least-privilege grants (P8, Ch 03); its inputs are secret-free (P7); its security/determinism-relevant output is **re-verified** (Book 04 §Ch08, Book 05 §Ch05 §5); it runs isolated and resource-limited (Ch 10).

### B3 — Kernel ↔ Secret Broker protected store (the secret boundary)
- **Crosses:** secret *references* in; materialized *values* out — but only into a runtime's execution context, only at execution (Ch 04, Book 06 §Ch06).
- **Checked:** capability to resolve *this* secret (P8); policy/approval conditions (Ch 06, Ch 07); audit of the materialization by reference (Ch 09). The store is **physically separate** from the ARM store (Book 02 §Ch08 §6) so an ARM-store compromise cannot cross this boundary.

### B4 — Runtime ↔ external systems (the effect boundary)
- **Crosses:** the actual external effects (API calls, writes) and their results.
- **Checked:** effects were declared (Book 04 §Ch06) and policy-permitted (Book 05 §Ch04, Book 14 §06); the runtime acts only within its grants (P8); materialized secrets are used here and never surfaced back (Book 06 §Ch06).

### B5 — Planner ↔ model provider (the reasoning-egress boundary)
- **Crosses:** non-secret reasoning context out; model output back into the planner (which becomes IR, never executable text — Book 08 §01).
- **Checked:** egress is a *granted, policy-governed* capability (Book 08 §04 §4) — a workspace may require on-prem or forbid external egress; **no secret ever crosses** this boundary (P7), because planner context is secret-free by construction.

### B6 — Tenant ↔ tenant (the isolation boundary)
- **Crosses:** nothing, by default.
- **Checked:** all reads/writes/watches are workspace-scoped (Book 02 §Ch08 §7); cross-workspace references require an explicit, audited, capability-gated grant (Book 02 §Ch06 §5). Silent cross-tenant flow is prohibited.

## 3. The untrusted-plugin boundary dominates

B2 is the boundary the architecture is most shaped around, because plugins run arbitrary code and are the richest attack surface (Ch 01 §3). Three rules govern every crossing of B2:

1. **Least privilege in.** A plugin receives only the attenuated capabilities it needs (Ch 03 §attenuation) and secret-free inputs (P7). It cannot reach what it was not granted.
2. **Verify out.** Anything security/determinism-relevant a plugin returns is re-verified by the core before use (Book 04 §Ch08, Book 05 §Ch08). The core never trusts a plugin's claim about its own output.
3. **Contain always.** The plugin is isolated and resource-limited (Ch 10); its compromise or crash is contained to its own work (Book 03 §Ch13).

Together these mean a fully malicious plugin is bounded to: producing un-verifiable/forbidden output (caught), consuming its resource budget (bounded), and leaking its own secret-free inputs (harmless). It cannot escalate, leak secrets, or corrupt the core.

## 4. Data classification across boundaries

To reason about crossings uniformly, data is classified:
- **Secret** — never crosses B1(inline)/B2/B5; crosses B3 only as a reference, and B4 only transiently inside the runtime (P7).
- **Sensitive (tenant) data** — crosses B2 to plugins only as needed and within tenancy (B6); never crosses B6 without a grant.
- **Non-secret context / references / IR / Events** — may cross B2/B5, and are the *only* things that do at those boundaries. IR and Events are content-addressed and secret-free (Book 04 §Ch07 §5, Book 03 §Ch03 §8), so they are safe to cross widely.

The design goal, restated: **the more a datum can leak (secrets), the fewer boundaries it is ever allowed to cross** — ideally none but B3(by reference)→B4(transient).

## 5. Invariants (normative summary)

1. One trusted core surrounded by untrusted rings; all cross-ring communication is Kernel-mediated (P4); there are no side channels.
2. Every enumerated boundary (ingress, plugin, secret, effect, reasoning-egress, tenant) has a defined crossing set and defined checks.
3. The untrusted-plugin boundary is governed by least-privilege-in, verify-out, contain-always; a malicious plugin is bounded to harmless outcomes.
4. The Secret Broker's store is physically separate from ARM; secrets cross the secret boundary only as references, materialized only into a runtime at execution.
5. No secret ever crosses the reasoning-egress boundary; model egress is a granted, policy-governed capability.
6. Tenant isolation is default-deny; cross-tenant flow requires an explicit, audited, capability-gated grant; the most leakable data crosses the fewest boundaries.
