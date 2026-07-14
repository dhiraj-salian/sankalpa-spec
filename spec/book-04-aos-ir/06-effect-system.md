# Book 04 · Chapter 06 — Effect System

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P7, P9, P13 and IR-P1, IR-P3.*

> Effects are how AOS IR makes side effects *visible to analysis*. Every construct declares what it does to the outside world, so that Policy can validate before anything runs (P9) and the compiler can reason about determinism, ordering, and caching. Deny-by-default: an undeclared effect is a verification error (IR-P3).

## 1. Why an effect system

Determinism (IR-P1) and governance (P9) both require knowing, statically, *what a piece of IR can do*. A pure computation can be reordered, cached, and replayed freely; a network write cannot. Policy must forbid or require certain effects (e.g. "no external email without approval"). The effect system makes these facts first-class and checkable, rather than discovering them at runtime.

## 2. The effect lattice

An **Effect** is a typed, structured tag. The core effect kinds:

```
Effect :=
    Pure                                   # no external effect (the identity)
  | Reason                                 # non-deterministic model reasoning (Ch 03 §3.2)
  | Network(mode, target)                  # mode: read|write; target: a policy-visible label, e.g. "crm", "email"
  | Filesystem(mode, scope)                # mode: read|write; scope: a path/label
  | SecretUse(ref)                         # materialization of a SecretRef at execution (P7)
  | Time                                   # observes the clock (a determinism-relevant effect)
  | Random                                 # consumes randomness (a determinism-relevant effect)
  | Cost(model)                            # consumes a metered/paid resource (tokens, money, quota)
  | StateWrite(scope)                      # mutates durable external state (drives idempotency, Ch 04 §4)
  | Custom(namespace, name, payload)       # extension-declared effect (Book 12), still policy-visible
```

An **EffectSet** is a set of Effects. Effect kinds and their `target`/`scope` labels form a partial order (a lattice): a more specific effect (`Network(write, "email")`) refines a more general one (`Network(write, *)`). Policy (Book 11 §06) is written against this lattice.

- `Reason`, `Time`, and `Random` are **determinism-relevant**: their presence means the construct is not a pure function of its inputs, and they constrain reordering/caching (§4).
- `SecretUse` is **security-relevant**: it names a `SecretRef` whose value is materialized at execution only (P7); it never carries the value.
- `Cost` and `StateWrite` are **safety/economics-relevant**: they drive approval gates, budgets, and idempotency requirements.

## 3. Declaration, composition, and deny-by-default

- **Declaration.** Every node/instruction declares its `EffectSet` (Ch 03, Ch 04). A Capability declares its effects in its `Signature` (Ch 05 §2.3); an invocation's declared effects **MUST** equal the Capability's (Ch 03 §3.1).
- **Composition.** A composite construct's effect set is the **union** of its children's effects. A module's `effects` is the union over its nodes and **MUST** be exactly the checked union — not less (unsound) and not an unexplained superset.
- **Deny-by-default (IR-P3).** If a construct performs an effect not in its declared set, that is a **verification error** (Ch 08). There is no "unknown effect" that slips through; extensions must declare `Custom` effects rather than performing undeclared ones. Silence is never permission.

## 4. Effects and determinism

The effect set determines what transformations are legal (IR-P10) and whether replay is sound (IR-P1):

- `Pure` constructs MAY be freely reordered, deduplicated, cached, and eliminated if unused.
- `Network(read)`/`Filesystem(read)` MAY be cached only if policy marks the target as stable; otherwise they are re-evaluated.
- `Time`/`Random` constructs are **not** pure: to keep IR-P1, they are modeled as reading an explicit typed input (a supplied clock/seed value) so the *reasoning about* them stays deterministic, and their `Effect` records the dependency for audit.
- `StateWrite`/`Network(write)` are the effects that force **idempotency** discipline in Low IR: a retry (Ch 04 §4) on such an effect is legal only when idempotency is established, because re-executing a visible write could double it.

## 5. Effects and policy (P9)

The **policy-validation pass** (Book 05 §04) runs on High IR *before* lowering and consumes the declared effects:

- Policy MAY **forbid** effects (`no Network(write, "email") from workspace X`), **require gates** (`SecretUse(payments.*)` requires an Approval, Book 11 §07), or **bound** them (`Cost` under a budget).
- Because effects are declared on the human-meaningful High IR, a rejected plan produces an explainable diagnostic ("this plan would send external email, which policy P-42 forbids") rather than an opaque runtime failure.
- A second, runtime policy checkpoint (Book 14 §06) re-checks effects against live conditions before execution — defense in depth, since some facts (current budget, approval state) are only known at execution.

## 6. Effects and secrets (P7)

`SecretUse(ref)` is the only effect that touches secrets, and it is carefully bounded:
- It names a `SecretRef` (Book 02 §Ch06 §4), never a value.
- Its presence tells Policy and the Secret Broker which secrets a plan will materialize, enabling pre-authorization and audit *without* exposing the secret.
- Materialization happens exclusively at the execution-time binding site (Ch 04 §5); no pass, no cache, and no log ever holds the value (Book 14 §04).

## 7. Effects and determinization (P13)

`Cost` and `Reason` effects are the signals the determinization engine (Book 05 §06, Book 10 §06) watches: expensive, repeated `Reason` steps with equivalent typed inputs are the prime candidates to fold into cached deterministic Capabilities. After determinization, the folded Capability's declared effects **MUST** be a subset of the original `Reason` node's effects (it removed `Reason`, possibly kept `Network(read)`), and semantics are preserved (IR-P10). The effect system is thus not just a guardrail but the *instrumentation* that drives the platform toward determinism.

## 8. Invariants (normative summary)

1. Every construct declares an EffectSet; a Capability invocation's effects equal the Capability's declared effects.
2. Composite effects are the exact checked union; performing an undeclared effect is a verification error (deny-by-default).
3. Determinism-relevant effects (`Reason`, `Time`, `Random`) mark non-pure constructs and constrain reordering/caching; `Time`/`Random` are modeled as explicit typed inputs to preserve IR-P1.
4. Write effects (`Network(write)`, `StateWrite`) require established idempotency before any retry.
5. Policy validates on declared effects before lowering and again at runtime; rejections are explainable.
6. `SecretUse` names references only; values are materialized solely at execution-time binding sites (P7).
7. Determinization folds repeated `Reason` steps into Capabilities whose effects are a subset and whose semantics are preserved.
