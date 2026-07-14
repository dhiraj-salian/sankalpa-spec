# Book 08 · Chapter 03 — The Planner Interface

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P8, P11. This chapter is the specification body of **AEP-0001 (Planner interface)**.*

> The planner interface is the stable contract every planner plugin implements. It is an AEP (AEP-0001) — a promise to third-party authors: implement this and any planning technology can drive Sankalpa. This chapter defines the interface, the input/output contract, uncertainty reporting, version negotiation, and the isolation model. The overriding rule from §Ch01 governs it: **input is Goals + non-secret context; output is verified High IR.**

## 1. Scope and stance

Defined **semantically**, independent of wire protocol and independent of *how* the planner reasons (single model call, multi-agent graph, search, or a deterministic rule engine). Sankalpa does not prescribe the planner's internals — only its contract. The interface is small because the platform's leverage is the IR and the deterministic machine, not a particular planning paradigm.

## 2. Interface methods

```
Planner (AEP-0001):
  describe() -> PlannerDescriptor
        # supported Goal classes, required Knowledge/capability context,
        # supported High-IR irVersions, determinism/uncertainty characteristics

  deriveGoals(intent, context) -> GoalSet | ClarificationRequest | Diagnostic
        # Intent → structured Goals (Ch 02); may request clarification

  plan(goals, context) -> HighIRModule | ClarificationRequest | Diagnostic
        # Goals → High IR (Book 04 §Ch03); the core transformation

  # Lifecycle
  init(grantSet, limits) / health() / shutdown()
```

- `deriveGoals` and `plan` are separated to match the two-stage model (Ch 02 §2): objective first, plan second. A planner MAY implement them jointly internally but MUST expose both so the Goal stage is inspectable and clarifiable.
- **`context`** carries only **non-secret** material: retrieved Knowledge (Book 09), available capability *descriptors* (signatures/effects, not grants of authority), workspace/role, and Conversation history. It **never** contains secret values (P7, §Ch04).
- Every output path is typed; a planner that cannot proceed returns a `ClarificationRequest` (Ch 02 §4) or a `Diagnostic`, never a malformed result.

## 3. The output contract (the crux)

`plan` MUST return a **High IR module that the compiler will verify** (Book 04 §Ch08). Specifically, the returned IR MUST be:

- **Well-formed and fully typed** (Book 04 §Ch05) — every value/port typed, every edge assignable.
- **Fully effect-annotated** (Book 04 §Ch06) — every node declares its effects; deny-by-default means an under-annotated node fails verification.
- **Secret-safe** (Book 04 §Ch05 §2.2) — secrets appear only as `SecretRef` values inside capability invocations; no secret value, and no `SecretRef` fed into a `Reasoning` node (§Ch04).
- **Runtime-agnostic** (IR-P7) — it names no runtime, no retry policy, no lowering detail (those are downstream).

The planner does **not** self-certify this; the compiler **verifies** it (Book 04 §Ch08, Book 05 §Ch08). If verification fails, the compiler returns a structured, stage-`Verify` diagnostic (Book 05 §Ch07 §2) identifying the planner as the source, which the planner MAY use to correct and re-emit. This feedback loop is how planners are held to the contract mechanically rather than by trust.

## 4. Uncertainty and non-determinism reporting

Planning is non-deterministic (§Ch01 §4); the interface makes this **explicit and honest** rather than hidden:

- **Residual reasoning → `Reasoning` nodes.** Decisions the planner cannot resolve deterministically at plan time are emitted as typed `Reasoning` nodes (Book 04 §Ch03 §3.2), marked `determinize` per the planner's judgment of stability. The planner does *not* bake in a guessed answer; it defers the decision into the plan where it is governable and later determinizable (P13, Book 05 §Ch06).
- **Confidence/uncertainty.** A planner SHOULD surface its confidence and any material assumptions (Ch 02 §4) on the Goals/IR so humans and policy can react. Low-confidence, high-consequence plans SHOULD route through Approval (Book 11 §07).
- **Determinism characteristics.** `describe` declares whether the planner is itself deterministic (a rule engine) or model-driven; this informs caching and expectations but does not change the output contract — *all* planners emit verifiable High IR.

## 5. Lifecycle, version negotiation, isolation

- **Init & grants.** The Plugin Manager loads the planner into an isolated host and calls `init(grantSet, limits)`: the planner receives **least-privilege planning grants** (capability *descriptors* to reason with, not authority to execute — P8) and resource limits (Book 11 §10). It cannot cause effects; it can only produce IR.
- **Version negotiation.** `describe` declares supported High `irVersion`s and the AEP-0001 interface version; incompatible planners are refused, not degraded (Book 03 §Ch09 §4, Book 04 §Ch09 §3).
- **Isolation & containment.** The planner is untrusted (§Ch01 §6, §Ch04): sandboxed, resource-limited, reachable only via the Kernel-mediated interface. A crashing/hanging planner is contained (Book 03 §Ch13); its Intent surfaces a failure condition rather than stalling silently.

## 6. Secrets and the interface (P7)

The interface gives a planner **no** way to obtain a secret value — by construction, as with the runtime interface (Book 06 §Ch02 §6). `context` is secret-free; `plan` outputs `SecretRef`s only. This is not a guideline a planner must remember; it is a property of the contract: there is no method that returns a secret, so a planner cannot request or leak one. Secret handling is entirely downstream, at execution (Book 06 §Ch06). See §Ch04 for the full planner-isolation model.

## 7. Stability (AEP-0001)

AEP-0001 carries an explicit stability level (Experimental → Beta → Stable, per the AEP process); planners declare the interface version they implement, and promotion requires the conformance suite (§Ch07) and real adopters. The interface evolves additively (P6, P11); breaking changes are a new major AEP version with negotiation/compatibility.

## 8. Invariants (normative summary)

1. Planners implement the semantic AEP-0001 interface (describe/deriveGoals/plan/init/health/shutdown); internals and paradigm are unconstrained.
2. Input is Goals + **non-secret** context (Knowledge, capability descriptors, role, conversation); output is **verified** High IR, or a clarification/diagnostic — never malformed output.
3. Emitted High IR is well-formed, fully typed, fully effect-annotated, secret-safe, and runtime-agnostic; the compiler verifies it and returns `Verify`-stage diagnostics on failure.
4. Residual non-determinism is emitted as typed `Reasoning` nodes with confidence/assumptions surfaced; planners never bake in guessed answers.
5. Planners are isolated, resource-limited, and granted only least-privilege planning capabilities (descriptors to reason with, no execution authority — P8).
6. The interface offers no method to obtain a secret value; context is secret-free and output carries only `SecretRef`s (P7); the interface is a versioned, stability-graded AEP.
