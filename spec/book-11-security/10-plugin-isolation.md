# Book 11 · Chapter 10 — Plugin Isolation

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0010 (observability output verified); realizes principles P8, P11 (and P1, P7). Companion to Book 03 §Ch09 (Plugin/Package Managers), Book 11 §02 (trust boundaries).*

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
- Compiler-pass / backend output → re-verified; RuntimeGraph conformance-checked (Book 05 §Ch02 §4, §Ch05 §5); and a pass's **effect graph checked against its input** (Book 05 §Ch02 §4.1). The comparative check is what bounds a *malicious pass*, which needs no undeclared effect, no non-determinism, and no secret to do harm: it retargets a declared, already-approved effect — rewriting an approved transfer's payee — and every single-artifact check passes, because the output is valid. Only its meaning changed. Conservation catches the retarget at the effect graph; an intra-value rewrite is caught only where the pass carries a translation-validation certificate (Book 05 §Ch02 §4.3), which is **required** for any pass running after an `Approval` is lowered in.
- **Runtime observability output** (`RuntimeEvent`s, logs, traces, metrics) → checked against the execution's materialized-value digest set before any sink (Book 14 §04 §2.1). This is the runtime-specific case, and it exists because the runtime is the one plugin whose *inputs* include a plaintext secret (Ch 02 §B2, §B3): the checks above would not catch it, because a leaked credential in a typed event field is perfectly well-formed output.
- A plugin's claims about itself (`supports`, `describe`) are treated as claims, cross-checked by conformance suites (Book 06 §Ch07, Book 08 §Ch07).

Thus a plugin cannot smuggle non-determinism or an undeclared effect into the system by *producing bad output* — the output is checked at the boundary before use. For **secrets** the claim must be scoped precisely rather than stated flatly: a plugin holding no secret has none to smuggle (planners, backends — Book 08 §04), and a **runtime**, which does hold one, cannot emit it *verbatim or under the declared common encodings* into any platform sink, because §2.1's boundary check drops it. A runtime that *transforms* the value beyond that closed set, or exfiltrates it across **B4** to its own endpoint, is **not** bounded by output-verification — it is bounded by effect declaration, egress policy, and isolation (Ch 02 §B4, Book 04 §Ch06, Book 14 §06). Conformance testing catches an honest runtime's bug; it is a *pre-admission* test and never an enforcement boundary against a malicious one.

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

It **cannot**: escalate authority, **obtain** a secret it was not authorized to materialize, read another tenant's data, corrupt the core, or cause an unverified/unauthorized effect.

**One plugin class needs its bound stated separately: the runtime.** "Cannot reach the Secret Broker's values" is true of every other plugin, and false of a runtime by design — B3 materializes values into it (Ch 02 §B3, Book 06 §Ch06). A malicious *runtime*, holding a credential it was legitimately granted, is bounded to: exercising that credential within its declared, policy-permitted effects (Book 14 §06); failing to place it in any platform sink verbatim or under the declared encodings (§4, Book 14 §04 §2.1); and being detected, attributed, and revocable when it tries. It is **not** bounded from transforming the value and exfiltrating it across **B4** — no boundary check can decide that in general — which is why B4 egress is itself a granted, declared, policy-governed capability, and why a runtime is granted only the secrets of the execution it is running (Ch 03 §4). The honest bound is *"a malicious runtime cannot quietly poison the platform's durable stores, and cannot use authority it was not granted"* — not *"cannot reach a secret."*

This bounded worst case — stated exactly, including where it stops — is the whole justification for admitting untrusted extensions (P11) into a security-critical platform. Overstating it would be worse than useless: it is precisely the "by construction, therefore unchecked" reasoning that left the observability path unguarded.

## 7. Invariants (normative summary)

1. Every plugin runs isolated; there is no trusted-plugin tier; isolation is a precondition of loading.
2. Isolation provides confinement (no ambient access), resource limits, controlled Kernel-mediated communication, and governed egress.
3. Isolation (the box) and least-privilege capabilities (what may be done outside it) together bound a plugin to the minimum its function needs (P8).
4. Security/determinism-relevant plugin output is re-verified and conformance-checked — High IR, pass/backend output, and a **runtime's observability output** against the execution's materialized-value digest set (§4, Book 14 §04 §2.1); plugin self-claims are cross-checked, never trusted (P1). Conformance testing is pre-admission and is not an enforcement boundary.
5. Faults are contained to the plugin's own work, bounded by limits, rescheduled only where safe, and audited.
6. A fully malicious plugin is bounded to harmless outcomes; it cannot escalate authority, obtain a secret it was not authorized to materialize, cross tenants, corrupt the core, or cause unverified/unauthorized effects. The **runtime** is bounded separately and more narrowly (§6): it legitimately holds a materialized secret, so its bound is that it cannot place one in a platform sink verbatim or under the declared encodings, and cannot use authority it was not granted — not that it "cannot reach a secret". Transformation and B4 exfiltration are bounded by effect declaration, egress policy, and isolation, not by output-verification.
