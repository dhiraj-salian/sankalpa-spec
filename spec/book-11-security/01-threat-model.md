# Book 11 · Chapter 01 — Threat Model

*Nature: **Normative**. · Reflects: SECURITY.md, ADR-0002; realizes principles P1, P7, P8, P9, P10. Foundational to the whole system.*

> Security in Sankalpa is not a feature bolted on; it is a set of *invariants* the whole architecture is shaped to preserve (Book 01 §04). This chapter states what we are defending, against whom, and why the platform's single most dangerous surface — reasoning context — drives the entire design. Every later chapter is a mechanism that closes a threat named here.

## 1. What we are protecting (assets)

In priority order (this order is the security precedence, Book 01 §04 §3):

1. **Secrets** — credentials, tokens, keys. Their exposure is catastrophic and often irreversible (a leaked key stays leaked). Protecting them is P7.
2. **Execution authority** — the ability to cause effects (send money, email, delete data). Unauthorized action is the second-worst outcome. Protecting it is P8.
3. **Governance integrity** — the guarantee that policy actually constrains what runs (P9). If policy can be bypassed, 1 and 2 follow.
4. **User data and tenancy** — one tenant's data, intents, and Experience must not leak to another.
5. **Attribution and audit integrity** — the guarantee that every action is truthfully attributable (P10). Without it, the other protections cannot be verified or enforced after the fact.

## 2. The defining threat: leakage through reasoning context

The founding observation (SECURITY.md) is that **an AI platform's most dangerous leak surface is the reasoning context of a language model.** Anything placed into a model's context can escape into:
- a prompt transmitted to a model provider (outside the trust boundary),
- a debug log or trace (durable, widely readable),
- a vector store / memory (durable, searchable, hard to scrub),
- the generated plan itself.

A single secret in reasoning context can therefore leak through four independent, high-fanout, hard-to-clean channels. This is why Sankalpa does not try to *carefully handle* secrets near the model — it ensures **secrets never enter reasoning context at all** (P7, Book 08 §04). The threat is so consequential that the architecture is bent around eliminating it, not mitigating it.

## 3. Adversaries

We model these adversaries explicitly:

| Adversary | Capability assumed | Primary defenses |
|-----------|-------------------|------------------|
| **Malicious/compromised plugin** (planner, runtime, backend, provider) | Runs arbitrary code inside its isolation boundary; sees its own inputs. | Untrusted-by-default; least-privilege capabilities (P8); no secrets in planner context (P7); output re-verified (Book 04 §Ch08); isolation (Ch 10). |
| **Malicious/naive Package author** | Ships a Package with hidden intent or vulnerable code. | Signing/provenance (Ch 04→Book 12); install ≠ privilege (Book 03 §Ch09); capability grants explicit and audited. |
| **Compromised model provider / prompt-injection via inputs** | Influences model output; may attempt to exfiltrate context or induce unsafe actions. | No secrets in context (P7); output-always-verified (Book 08 §01); policy on effects (P9); no ambient authority (P8). |
| **Malicious user / tenant** | Submits crafted Intent; tries to exceed entitlements or reach other tenants. | Capability-scoped derivation (Book 08 §02); tenancy isolation; policy; approval gates (Ch 07). |
| **Compromised storage / infrastructure** | Reads the ARM store or intercepts internal traffic. | Secrets never in ARM/IR/Events (P7); Secret Broker's separate protected store (Ch 04); audit integrity (Ch 09). |
| **Insider / operator error** | Has elevated access; may misconfigure. | Least-privilege even for operators; audited privileged operations (Ch 09); policy-as-code (Ch 06). |

We do **not** claim to defend against a total compromise of the Secret Broker's protected store *and* the runtime host simultaneously — but the design ensures no *other* component's compromise reaches secrets (Ch 04 §separation).

## 4. Trust posture: untrusted by default

The architecture's stance (ADR-0002, Book 03 §Ch09 §2): **everything outside the trusted Kernel core is untrusted.** Plugins, Packages, planners, runtimes, model providers, and user inputs are all assumed potentially hostile. Trust is *earned narrowly* through granted capabilities and *verified*, never *assumed*. The small trusted core (the Managers, Book 03 §Ch04) is the only thing that must be correct; everything else is contained.

This is why the microkernel shape is a *security* decision as much as an extensibility one: a single mediated chokepoint (P4) is the only place invariants can be enforced against untrusted surroundings.

## 5. The invariants that answer the threats

Each foundational security principle is a direct response to a threat above; the rest of Book 11 specifies the mechanism:

| Threat | Invariant | Mechanism (chapter) |
|--------|-----------|---------------------|
| Reasoning-context leakage | Secrets never in context/IR/logs/memory; by reference only (P7) | Secret Broker (Ch 04), planner isolation (Book 08 §04) |
| Unauthorized effect | No ambient authority (P8) | Capability model (Ch 03) |
| Policy bypass | Policy enforced before compile & execute (P9) | Policy Engine (Ch 06) |
| Untrusted extension | Isolation + least privilege | Plugin isolation (Ch 10), capabilities (Ch 03) |
| Undetectable action | Everything attributable (P10) | Audit & attribution (Ch 09) |
| Irreversible/high-stakes action | Human authorization | Approval Engine (Ch 07) |
| Cross-tenant leakage | Tenancy isolation | Workspaces + capability scoping (Ch 03, Ch 08) |

## 6. Defense in depth

No single mechanism is trusted alone. Secrets are protected by *reference-only IR* **and** *the Broker* **and** *isolation* **and** *audit*. Effects are governed by *capability checks* **and** *policy at compile time* **and** *policy at runtime* (Book 05 §Ch04, Book 14 §06) **and** *approval*. This layering means a single failure or bypass does not breach an asset — the next layer catches it (Book 03 §Ch13 §1, fail-closed). Depth is deliberate: adversaries are assumed capable, so no layer is load-bearing alone.

## 7. Invariants (normative summary)

1. Assets, in precedence order: secrets, execution authority, governance integrity, tenancy, audit integrity.
2. The defining threat is leakage through model reasoning context; the design eliminates secrets from context rather than mitigating their presence (P7).
3. Everything outside the trusted Kernel core is untrusted by default; trust is granted narrowly and verified, never assumed.
4. Each foundational security invariant (P1, P7, P8, P9, P10) answers a named threat and is realized by a specified mechanism in this Book.
5. Defenses are layered (defense in depth); no single mechanism is load-bearing alone, and every security failure mode is fail-closed.
