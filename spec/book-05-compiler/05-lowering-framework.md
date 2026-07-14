# Book 05 · Chapter 05 — Lowering Framework

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P7, P11 and IR-P7, IR-P10.*

> Lowering is where the compiler *decides execution mechanics*. It happens in two stages: **High → Low** (owned by the compiler, runtime-agnostic) and **Low → RuntimeGraph** (owned by a runtime **backend**, runtime-specific). This chapter specifies both, the backend interface, and the semantics-preservation obligation that ties a runtime-agnostic plan to a concrete execution.

## 1. Two lowerings, one guarantee

```
High IR ──[compiler lowering]──► Low IR ──[backend lowering]──► RuntimeGraph ──► execution
(runtime-agnostic)              (runtime-agnostic)             (runtime-specific)
```

Both lowerings are transform steps under the pass framework (Ch 02) and both **MUST** preserve observable semantics (IR-P10): the RuntimeGraph, executed with equal inputs and resolved bindings, produces the same observable effects and outputs as the High IR it derives from. This single guarantee — carried unbroken across two lowerings — is what makes a runtime-agnostic plan safe to run on *any* selected runtime (§4).

## 2. High → Low lowering (compiler-owned)

This lowering adds exactly the mechanics High IR omitted (Book 04 §Ch04 §1), making each decision explicit and verifiable:

- **Schedule.** Derive an explicit partial order from High IR's dataflow/control edges; realize or serialize `Parallel` regions per the optimization stage's tuning (Ch 03 §2.5). Any total order respecting the partial order must be observationally equivalent (Book 04 §Ch04 §2).
- **Execution policy.** Assign each instruction its `ExecPolicy` (idempotency, retry, timeout, `onError`, compensation — Book 04 §Ch04 §4), derived from Capability metadata, effect analysis, and policy annotations. **Retries only on established idempotency**; failable effects get explicit `onError` (deny-by-default).
- **Guards from policy.** Materialize the guards the policy-validation pass annotated (Ch 04 §3) — approval instructions, budget checks, redaction — as real Low IR instructions.
- **Binding sites.** Turn `SecretRef`/`ResourceRef` values into execution-time **binding sites** (Book 04 §Ch04 §5), each capability-gated; **no reference is resolved at compile time** (P7).

The compiler records `fromModule` (the High IR hash) on the Low module (Book 04 §Ch04 §2), keeping lowering auditable and reproducible.

## 3. The backend interface (Low → RuntimeGraph)

A **runtime backend** is a plugin (AEP-0002, Book 06 §02) that lowers Low IR into a specific runtime's execution graph. The framework calls backends through a stable interface:

```
Backend:
  runtimeClass   string                 # e.g. the runtime it targets (not named in core, P11)
  supports(lowModule) -> Capability report  # which instructions/effects/policies it can honor
  lower(lowModule) -> RuntimeGraph | Diagnostic
  irVersions     range                  # supported Low IR versions (Book 04 §Ch09)
```

Rules:
- A backend **MUST** faithfully realize every instruction's `ExecPolicy`. If it cannot honor a required policy (e.g. no durable retry/compensation), it **MUST** report itself unsupported for that module (`supports` fails) rather than lower it lossily — the Runtime Manager then does not select it (Book 03 §Ch07 §3.2). A backend that silently dropped a retry or a compensation would break IR-P10.
- A backend is **untrusted** (Book 03 §Ch09 §2): isolated, least-privilege, and its output subject to the checks in §5.
- A backend declares supported Low `irVersion`s; the framework refuses incompatible pairings (Book 04 §Ch09 §3).

## 4. Runtime-agnosticism and portability (IR-P7)

Low IR names no runtime (Book 04 §Ch04 §7); runtime **selection** happens after Low IR exists, in the Runtime Manager (Book 03 §Ch07), based on which backends `supports` the module plus cost/policy/health. Consequences fixed here:

- The **same Low IR MAY be lowered by different backends** and each RuntimeGraph **MUST** execute observably-equivalently (IR-P10). Portability is a *guarantee*, not a hope.
- Runtime-specific detail exists **only** in the RuntimeGraph, never upstream. Nothing a backend does may leak back into Low/High IR.

## 5. Semantics-preservation checks

Because lowering is where an abstract plan becomes a concrete execution, it is checked twice over:

1. **Low IR is re-verified** after compiler lowering (Book 04 §Ch08; Book 03 §Ch08 §3) — structure, types, effects, secret-safety, and the Low-IR-specific determinism constraints (retry↔idempotency, `onError` presence).
2. **RuntimeGraph conformance** — the backend's output is checked against a conformance contract (Book 06 §07): it must preserve the declared effects, honor the `ExecPolicy`, and resolve bindings only at execution. A backend whose RuntimeGraph fails conformance is rejected and not executed.

Trust is never extended to a backend's claim of fidelity; it is verified — the same discipline applied to every extension (Book 03 §Ch09 §2).

## 6. Secrets across lowering (P7)

Through both lowerings, secrets remain **references**:
- High → Low turns `SecretRef` values into binding sites (§2); Low → RuntimeGraph maps binding sites to the runtime's secret-injection mechanism **without resolving them**.
- The value is materialized **only** at execution, **only** by the Secret Broker, **only** into the runtime's execution context (Book 06 §06, Book 11 §04). No lowering stage, cache, log, or RuntimeGraph artifact ever contains a value. A RuntimeGraph is as safe to store and inspect as the IR it came from.

## 7. Invariants (normative summary)

1. Lowering is two stages — compiler (High→Low, runtime-agnostic) and backend (Low→RuntimeGraph, runtime-specific) — both semantics-preserving (IR-P10).
2. High→Low fixes schedule, execution policy, policy guards, and binding sites; retries require idempotency; failable effects get `onError`; no reference is resolved at compile time (P7).
3. A backend must faithfully realize every `ExecPolicy` or report itself unsupported; it never lowers lossily; it is untrusted, isolated, and version-checked.
4. The same Low IR lowered by different backends executes observably-equivalently; runtime specifics live only in the RuntimeGraph (IR-P7).
5. Low IR is re-verified and the RuntimeGraph is conformance-checked before execution; backends are not trusted on claim.
6. Secrets remain references through both lowerings and are materialized only at execution; no lowering artifact holds a value.
