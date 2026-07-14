# Book 11 · Chapter 10 — Plugin Isolation

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P8, P11 (and P1, P7). Companion to Book 03 §Ch09 (Plugin/Package Managers), Book 11 §02 (trust boundaries).*

> Plugins run arbitrary third-party code and are the richest attack surface (Ch 01 §3, Ch 02 §B2). Isolation is the mechanism that lets Sankalpa run them anyway: sandbox them, bound their resources, and contain their failures so that a malicious or broken plugin harms only its own work. Isolation, least-privilege (Ch 03), and output-verification together are what make untrusted extensibility (P11) safe.

## 1. Isolation is not optional

Every plugin — planner, runtime, compiler backend, channel adapter, provider, third-party controller — runs inside an **isolation boundary** enforced by the Plugin Manager (Book 03 §Ch09). There is no "trusted plugin" tier; the core is the only trusted code (Ch 01 §4). A plugin that could run un-isolated would be a hole in the entire security model, so isolation is a precondition of loading, not a configuration.

## 2. What isolation provides

The isolation boundary MUST provide, at minimum:

1. **Confinement.** The plugin cannot access memory, storage, secrets, network, or filesystem except through explicitly granted capabilities (P8, Ch 03). It has **no ambient access** to the host, the ARM store, the Secret Broker, other plugins, or other tenants (Ch 02 §B2/§B6).
2. **Resource limits.** CPU, memory, execution time, and concurrency are bounded by the `limits` supplied at `init` (Book 03 §Ch09 §4, Book 06 §Ch02 §3). A plugin cannot exhaust host resources or starve others.
3. **Controlled communication.** The plugin interacts only through its AEP interface and the Kernel-mediated paths (P4). It cannot open side channels to other components.
4. **Egress control.** Network egress (e.g. a planner calling a model provider, Ch 02 §B5) is itself a granted, policy-governed capability — scoped and audited — not an open door.

The strength of the boundary (process, container, VM, WASM sandbox) is an implementation choice matched to the threat and the plugin class (ADR-0001); the *properties* above are normative.

## 3. Least privilege within isolation (P8)

Isolation and capabilities are complementary layers:
- Isolation says *"you are in a box."*
- Capabilities say *"here are the specific, attenuated things you may do outside the box"* (Ch 03 §4).

A plugin is granted, at load, the **minimum** capability set its function requires (Book 03 §Ch09 §2). A planner gets capability *descriptors* to reason with and model-egress — not execution authority (Book 08 §04). A runtime gets exactly the secrets-materialization and effect capabilities of the execution it runs — attenuated to that execution (Book 06 §Ch06). The box plus the least-privilege grants together bound what the plugin can *possibly* do to the minimum.

## 4. Output is verified, not trusted (P1 support)

Isolation contains *behavior*; it does not make a plugin's *output* trustworthy. So the core **re-verifies** anything security- or determinism-relevant a plugin returns (Ch 02 §B2 rule 2):
- Planner High IR → verified (Book 04 §Ch08).
- Compiler-pass / backend output → re-verified; RuntimeGraph conformance-checked (Book 05 §Ch02 §4, §Ch05 §5).
- A plugin's claims about itself (`supports`, `describe`) are treated as claims, cross-checked by conformance suites (Book 06 §Ch07, Book 08 §Ch07).

Thus a plugin cannot smuggle non-determinism, an undeclared effect, or a secret into the system by *producing bad output* — the output is checked at the boundary before use.

## 5. Fault containment

A plugin that crashes, hangs, or misbehaves is **contained** (Book 03 §Ch09 §4, §Ch13):
- Its failure degrades *its own work* (its executions/plans/compilations), not the Kernel or other plugins.
- Hangs are bounded by timeouts/limits (§2); a runaway plugin is terminated, not allowed to consume the host.
- Its in-flight work fails or reschedules **only where safe** (idempotency/compensation, Book 03 §Ch07 §6, Book 06 §Ch03 §4) — never re-run unsafely.
- The failure is attributed and audited (Ch 09) and feeds Experience (Book 10) so a chronically faulty plugin is visible.

## 6. The bounded worst case

Combining isolation (§2), least-privilege (§3), output-verification (§4), and containment (§5), a **fully malicious plugin** is bounded to:
- producing output that fails verification/policy (caught, discarded),
- consuming its own resource budget (bounded),
- exercising exactly its granted, attenuated capabilities and no more (which are the minimum for its function), and
- leaking its own secret-free inputs (harmless — planners get no secrets at all, Book 08 §04).

It **cannot**: escalate authority, reach the Secret Broker's values, read another tenant's data, corrupt the core, or cause an unverified/unauthorized effect. This bounded worst case is the whole justification for admitting untrusted extensions (P11) into a security-critical platform.

## 7. Invariants (normative summary)

1. Every plugin runs isolated; there is no trusted-plugin tier; isolation is a precondition of loading.
2. Isolation provides confinement (no ambient access), resource limits, controlled Kernel-mediated communication, and governed egress.
3. Isolation (the box) and least-privilege capabilities (what may be done outside it) together bound a plugin to the minimum its function needs (P8).
4. Security/determinism-relevant plugin output is re-verified and conformance-checked; plugin self-claims are cross-checked, never trusted (P1).
5. Faults are contained to the plugin's own work, bounded by limits, rescheduled only where safe, and audited.
6. A fully malicious plugin is bounded to harmless outcomes; it cannot escalate, reach secret values, cross tenants, corrupt the core, or cause unverified/unauthorized effects.
