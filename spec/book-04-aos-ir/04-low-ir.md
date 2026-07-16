# Book 04 · Chapter 04 — Low IR

*Nature: **Normative**. · Reflects: RFC-0001, RFC-0002 (recorded-reasoning carve-out), RFC-0003 (`shadowSource`), RFC-0005 (secret carve-out in the bindings term); realizes principles P1, P7, P9 and IR-P1…P10.*

> Low IR is the compiler-produced form (Book 05): High IR after optimization and policy validation, lowered to fix everything needed for **deterministic execution** — evaluation order, error handling, retries, idempotency, and resource/secret **bindings by reference** — while still naming **no specific runtime** (IR-P7). A runtime backend lowers Low IR into a runtime-specific RuntimeGraph (Books 05–06).

## 1. What lowering adds

Lowering High → Low is where the compiler *decides mechanics* that High IR deliberately omitted (Ch 03 §4). For each, Low IR makes an explicit, verifiable choice:

| Concern | High IR | Low IR |
|---------|---------|--------|
| Evaluation order | implied by data/control edges | an explicit **schedule** (§2) |
| Concurrency | *permission* (`Parallel`) | a concrete **execution plan** honoring or serializing it (§2) |
| Errors | undefined | explicit **error handling** per instruction (§4) |
| Retries/timeouts | undefined | explicit **retry/timeout/idempotency** policy (§4) |
| Secret binding | opaque `SecretRef` values | **binding sites** resolved at execution by reference (§5) |
| Runtime | none | still none — form only (IR-P7) |

Lowering **MUST** be semantics-preserving (IR-P10): the observable outcome of Low IR **MUST** equal that of the High IR it came from, given equal inputs.

## 2. Structure: scheduled instruction graph

```
Module (Low):
  irVersion   string
  fromModule  Hash            # content hash of the High IR it lowered from (Ch 07)
  signature   Signature
  instructions list<Instruction>
  schedule    Schedule        # partial order (or total order) over instructions
  effects     EffectSet       # preserved & re-checked from High IR
```

- Each **Instruction** corresponds to a lowered High-IR node (a `CapabilityInvocation`, a captured `Reasoning`, a control construct, or a data op) plus its execution policy (§4).
- The **Schedule** is an explicit partial order consistent with High IR's dataflow/control edges. Where High IR granted `Parallel`, the schedule MAY keep instructions concurrent; where determinism requires it, the schedule serializes. Any total order the runtime picks that respects the partial order **MUST** yield identical observable results (IR-P1) — a property the effect/idempotency contract (§4, Ch 06) guarantees.
- `fromModule` links Low IR to its High IR by content hash, making lowering auditable and reproducible.

## 3. Instructions

```
Instruction:
  id           InstrId
  op           InstrOp         # LoweredCapability | CapturedReasoning | Branch | Fork/Join | LoopStep | DataOp
  inputs       map<port, ValueRef | BindingRef>
  outputs      map<port, Type>
  effects      EffectSet
  policy       ExecPolicy      # retry/timeout/idempotency/error (§4)
  bindings     list<BindingSite> # secret/resource references resolved at execution (§5)
```

- `LoweredCapability` — a concrete, ordered invocation of a Capability. When produced by determinization substitution (Book 05 §06 §2), it MAY carry an optional **`shadowSource`** — the identity of the `Reasoning` node it replaced — so the runtime can reconstruct and shadow-sample the original reasoning for drift detection (Book 10 §Ch06 §5). `shadowSource` is metadata for monitoring only; it never changes the instruction's typed behavior.
- `CapturedReasoning` — the lowered form of a High `Reasoning` node. Its non-determinism is confined exactly as at the High level (IR-P1): the instruction produces a typed value; if determinization (Book 05 §06) has folded it into a cached Capability, lowering replaces it with a `LoweredCapability` (carrying `shadowSource`) and records the substitution. Where it remains, its output is recorded to the execution's reasoning ledger so the run is replayable-with-record (Book 06 §Ch03 §1).
- Control ops (`Branch`, `Fork`/`Join`, `LoopStep`) realize High control-flow within the schedule.

## 4. Execution policy (the heart of Low IR)

Every instruction carries an `ExecPolicy` making failure behavior explicit and deterministic-by-construction:

```
ExecPolicy:
  idempotency  enum   # Idempotent | NonIdempotent | IdempotentWithKey(keyExpr)
  retry        RetryPolicy?   # maxAttempts, backoff, retryable-error classes
  timeout      Duration?
  onError      enum   # Fail | Compensate(ref) | Fallback(region) | Ignore(justified)
  compensation Ref?   # a compensating Capability for saga-style rollback
```

Normative rules:
- **Retries require idempotency.** A `retry` policy **MUST NOT** be attached to a `NonIdempotent` instruction unless an idempotency key (`IdempotentWithKey`) makes re-execution safe. Verification (Ch 08) enforces this — silently retrying a non-idempotent effect would break IR-P1 and could double external side effects.
- **Timeouts and error handling are explicit;** an instruction with a retryable/failable effect (Ch 06) **MUST** declare `onError`. Deny-by-default: an unhandled failable effect is a verification error.
- **Compensation** encodes saga-style rollback for effects that cannot be undone by retry; the compensating Capability is itself typed and effect-declared.
- These policies are the compiler's decisions, derived from High-level intent + Capability metadata + Policy (Book 05); they are not authored by planners (Book 08 never sees them).

## 5. Bindings — secrets and resources by reference (P7)

Low IR introduces **binding sites**: named points where an external reference (a secret, a connection, a resource handle) is resolved **at execution**.

```
BindingSite:
  name         string
  ref          SecretRef | ResourceRef    # opaque reference, never a value (IR-P4)
  resolveAt    Execution                    # ALWAYS execution-time, never compile-time
  grantedBy    CapabilityRef                # the capability authorizing resolution (P8)
```

- A `SecretRef` binding is resolved **only** at execution, **only** by the Secret Broker, **only** into the runtime's execution context, **never** back into IR (P7, Book 06 §06, Book 11 §04).
- Resolution is capability-gated (`grantedBy`, P8): possessing the reference is insufficient without the capability.
- No binding is ever resolved during compilation; the compiler handles references, never values. This is the mechanism that keeps IR safe to store, cache, diff, and display.

## 6. Determinism obligation restated

Low IR is the artifact executed (via its RuntimeGraph). Therefore IR-P1 lands here most concretely: **identical Low IR + identical resolved bindings + identical inputs + identical recorded reasoning/`Time`/`Random` outputs ⇒ identical observable behavior.** The recorded-reasoning term (RFC-0002) is required because a live `CapturedReasoning` node is non-deterministic by construction; for a plan with no such nodes it is empty and the guarantee reduces to the three-part form. The **bindings** term carries the secret carve-out (RFC-0005): a `SecretRef` binding site (§5) resolves at execution to a value P7 forbids recording, so "identical bindings" is guaranteed *within* an Execution (Book 06 §Ch06 §2.1) and, across runs, only where the reference's rotation generation is unchanged (Book 06 §Ch03 §1). The schedule's freedom (any order respecting the partial order) does not weaken this, because the effect/idempotency contract (§4, Ch 06) guarantees observational equivalence across legal orders. Replay tests (Book 05 testing, Book 10) assert exactly this property against golden Low IR, in the reconstruction variant (Book 06 §Ch03 §1).

## 7. Relationship to RuntimeGraph and runtime selection

Low IR is still runtime-agnostic (IR-P7). A **runtime backend** (Book 06, AEP-0002) lowers Low IR into a `RuntimeGraph` for a chosen runtime — and the runtime is chosen **after** planning and (usually) after Low IR exists, based on capabilities, policy, and cost (Book 06 §04). The same Low IR MAY be lowered to different runtimes and **MUST** produce observably equivalent executions on each (IR-P10) — the portability guarantee.

## 8. Invariants (normative summary)

1. Low IR fixes ordering, error handling, retries, idempotency, and bindings; it still names no runtime.
2. Lowering is semantics-preserving; any schedule order respecting the partial order yields identical observable results.
3. Retries attach only to idempotent (or keyed-idempotent) instructions; failable effects declare `onError` (deny-by-default).
4. Bindings (secrets/resources) are references resolved only at execution, capability-gated; no value ever enters IR.
5. `CapturedReasoning` confines non-determinism to its node; determinized reasoning is replaced by a cached Capability with the substitution recorded and an optional `shadowSource` for drift detection.
6. Identical Low IR + inputs + resolved bindings + recorded reasoning/`Time`/`Random` outputs ⇒ identical observable behavior, verifiable by reconstruction replay.
