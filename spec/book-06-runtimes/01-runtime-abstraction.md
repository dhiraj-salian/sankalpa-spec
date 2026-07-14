# Book 06 · Chapter 01 — Runtime Abstraction

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0001; realizes principles P1, P7, P11 and IR-P7, IR-P10. Companion to Book 05 §Ch05 (lowering), Book 03 §Ch07 (Scheduler & Runtime Manager).*

> A **Runtime** is where a plan finally *happens*. This chapter defines what a runtime is, the boundary between the compiler's RuntimeGraph and runtime execution, and the contract a runtime owes the rest of the system. Runtimes are plugins (P11); the runtime is selected *after* planning (RFC-0001); and no IR construct ever names one (IR-P7).

## 1. What a Runtime is

A **Runtime** is an execution engine that takes a **RuntimeGraph** — the runtime-specific artifact a backend lowered from Low IR (Book 05 §Ch05) — and *executes* it, producing observable effects, typed outputs, and a stream of Events. Examples of the *class* of thing a runtime is: a durable workflow engine, a container orchestrator, a function platform, a shell. Sankalpa names none of these in its core (IR-P7, Book 03 §Ch04 §3); each is a plugin behind the runtime interface (Ch 02).

A Runtime is registered as a `Runtime` Resource (Book 02 §Ch07) carrying its declared capabilities, cost profile, and health — the information the Runtime Manager uses to select it (Ch 04).

## 2. The boundary: where the compiler stops and the runtime begins

The handoff is precise and is the crux of the whole architecture:

```
Low IR (runtime-agnostic, verified)  ──[backend.lower]──►  RuntimeGraph (runtime-specific)  ──[runtime.execute]──►  effects + outputs + Events
        Book 04 §Ch04                    Book 05 §Ch05                Book 06 (this Book)
```

- **Everything semantic is decided before the runtime.** The *what* (High IR, planner), the *mechanics* (Low IR: order, retries, idempotency, error handling, binding sites — Book 04 §Ch04), and the *governance* (policy validation — Book 05 §Ch04) are all fixed upstream. The runtime does not re-plan, re-order for correctness, or reinterpret intent.
- **The runtime realizes, it does not decide.** Its job is to faithfully execute the RuntimeGraph: honor the `ExecPolicy` on each instruction (retries, timeouts, compensation), resolve binding sites at execution (secrets, §Ch06), run capability implementations, and report Events. A runtime that *added* behavior, *dropped* a retry, or *reordered* across an effect boundary would break IR-P10.

This boundary is why the same Low IR can run on radically different runtimes with equivalent observable results (§3): all the meaning lives above the line; only the *how-on-this-engine* lives below it.

## 3. The fidelity contract (IR-P10)

A Runtime **MUST** execute a RuntimeGraph **observably-equivalently** to the semantics of the Low IR (and thus the High IR) it derives from. Concretely, a Runtime:
- **MUST** produce the declared effects (Book 04 §Ch06) and no undeclared ones — a runtime cannot make a call the plan did not sanction.
- **MUST** honor every instruction's `ExecPolicy` (Book 04 §Ch04 §4): apply retries only where idempotency permits, enforce timeouts, run `onError`/compensation as specified.
- **MUST** respect the schedule's partial order; any total order it chooses must be within the legal set (Book 04 §Ch04 §2).
- **MUST NOT** introduce non-determinism beyond what the IR declared (`Reason`/`Time`/`Random` effects, Book 04 §Ch06 §4).

A runtime that *cannot* uphold this contract for a given RuntimeGraph MUST be excluded from selection for it (Ch 04, Book 05 §Ch05 §3) — it is never asked to run something it would run unfaithfully. Fidelity is enforced, not assumed: the RuntimeGraph is conformance-checked (Ch 07) and the runtime is conformance-tested.

## 4. Runtimes are untrusted plugins (P11, P8)

Every Runtime is an **untrusted plugin** (Book 03 §Ch09 §2), regardless of author:
- Loaded and isolated by the Plugin Manager; resource-limited and sandboxed (Book 11 §10).
- Granted **only** the attenuated capabilities its executions need (Book 03 §Ch06 §5) — never ambient access to storage, secrets, other tenants, or other plugins (P8).
- Able to obtain the secrets an execution requires **only** at execution, **only** by reference resolution through the Secret Broker (§Ch06, P7).
- Reachable and reporting only through the Kernel-mediated paths (P4): it receives a RuntimeGraph to run and emits Events; it does not call other components directly.

Because runtimes touch the outside world and (transiently) handle materialized secrets, they are the most security-sensitive plugin class — which is exactly why the isolation, least-privilege, and secret-by-reference disciplines are strongest here.

## 5. Replaceability and portability (P11)

- **Any runtime is replaceable** by another satisfying the interface (Ch 02), without core changes (Book 03 §Ch09 §4). A deployment may add, remove, or swap runtimes freely.
- **Portability is a guarantee, not a hope** (Book 05 §Ch05 §4): the same Low IR lowered for two runtimes MUST execute observably-equivalently. This is what lets Sankalpa outlive any particular execution technology — the platform's value is the deterministic machine above the runtime, not the runtime itself (Book 01 §05).

## 6. What this Book specifies next

The [runtime interface](02-runtime-interface.md) (the AEP-0002 contract every runtime implements); [execution semantics](03-execution-semantics.md) (determinism, idempotency, retries, compensation at runtime); [runtime selection](04-runtime-selection.md) (choosing a runtime after planning); the [reference runtimes](05-reference-runtimes.md) (informative mappings for n8n, Temporal, Kubernetes, Bash, …); [secret materialization](06-secret-materialization.md) (the execution-only secret path); and the [conformance suite](07-conformance-suite.md) (how a runtime proves fidelity).

## 7. Invariants (normative summary)

1. A Runtime executes a runtime-specific RuntimeGraph lowered from runtime-agnostic Low IR; all meaning, mechanics, and governance are fixed upstream — the runtime realizes, it does not decide.
2. A Runtime executes observably-equivalently to the source IR: producing only declared effects, honoring every `ExecPolicy`, respecting the schedule, introducing no undeclared non-determinism (IR-P10).
3. A runtime that cannot uphold the fidelity contract for a RuntimeGraph is excluded from selection for it; fidelity is conformance-checked, not assumed.
4. Runtimes are untrusted, isolated, least-privilege plugins; they obtain secrets only at execution, only by reference (P7, P8, P11).
5. Any runtime is replaceable by another satisfying the interface without core changes; the same Low IR is portable across runtimes with equivalent observable results.
