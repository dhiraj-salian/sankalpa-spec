# Book 06 · Chapter 02 — The Runtime Interface

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0001; realizes principles P4, P7, P8, P11. This chapter is the specification body of **AEP-0002 (Runtime interface)**.*

> The runtime interface is the stable contract every runtime plugin implements. It is an AEP (AEP-0002) because it is a promise to third-party authors: implement this faithfully and your engine can execute any Sankalpa plan. This chapter defines the interface methods, the lifecycle, version negotiation, and the capability/isolation model.

## 1. Scope and stance

The interface is defined **semantically**, independent of wire protocol (like the Kernel API, Book 03 §Ch02 §5): a conforming runtime MAY be hosted in-process, as a subprocess, or as a remote service, as long as it provides identical semantics. The interface is deliberately small — a runtime is asked to *lower and execute*, *report*, and *be manageable* — because the intelligence lives in the IR and the compiler, not in the runtime.

## 2. Interface methods

A runtime plugin implements:

```
Runtime (AEP-0002):
  # Identity & capability
  describe() -> RuntimeDescriptor
        # declared capabilities, supported effects, ExecPolicy features
        # (retry/timeout/compensation), cost profile, supported Low irVersions

  # Lowering (the backend role, Book 05 §Ch05 §3)
  supports(lowModule) -> SupportReport
        # can this runtime honor every instruction/effect/ExecPolicy in this module?
  lower(lowModule) -> RuntimeGraph | Diagnostic
        # produce the runtime-specific execution artifact (or explain why not)

  # Execution
  execute(runtimeGraph, executionCtx) -> ExecutionHandle
        # begin executing; returns a handle for tracking/cancellation
  cancel(handle, reason) -> Ack
        # cooperative cancellation; triggers onError/compensation per ExecPolicy

  # Reporting
  events(handle) -> stream<RuntimeEvent>
        # typed, secret-free progress/effect/terminal events (§5)

  # Health & lifecycle (§3)
  health() -> HealthStatus
  init(grantSet, limits) / shutdown()
```

- `describe`/`supports` feed **runtime selection** (Ch 04): the Runtime Manager never selects a runtime that cannot honor a module.
- `lower` is the backend role of Book 05 §Ch05 §3; a runtime is both the lowering backend for its target *and* the executor — bundled behind one interface so a runtime owns its own translation.
- `execute` receives an **`executionCtx`** that carries *references* (secret binding tokens, resource handles) — never secret values (§6, P7).

## 3. Lifecycle and version negotiation

- **Load & init.** The Plugin Manager (Book 03 §Ch09) loads the runtime into an isolated host, then calls `init(grantSet, limits)`: the runtime receives its **least-privilege capability grants** (Book 03 §Ch06 §5) and **resource limits** (Book 11 §10). It has no authority beyond `grantSet` (P8).
- **Version negotiation.** `describe` declares supported Low `irVersion`s and the AEP-0002 interface version. The host refuses a runtime whose interface/IR versions are incompatible (Book 03 §Ch09 §4, Book 04 §Ch09 §3) — mismatch fails loudly, never degrades silently.
- **Health & readiness.** `health` is probed; an unhealthy runtime is marked and routed around by selection (Ch 04) and the Scheduler (Book 03 §Ch07).
- **Shutdown.** `shutdown` allows graceful drain; in-flight executions are cancelled cooperatively (§4) or allowed to reach a terminal state per policy.

## 4. Execution and cancellation contract

- `execute` MUST realize the RuntimeGraph faithfully (Ch 01 §3, Ch 03): honor each instruction's `ExecPolicy`, respect the schedule's partial order, produce only declared effects.
- `cancel` MUST be **cooperative and safe**: it stops further instructions and runs the specified `onError`/compensation (Book 04 §Ch04 §4) so external state is left consistent. A runtime MUST NOT leave a partially-applied non-idempotent write without either completing or compensating it.
- If the runtime fails mid-execution, it MUST emit a terminal failure Event with enough detail (which instructions completed/compensated) for the Execution to reach an *explained* terminal state (Book 03 §Ch13 §3), never a silent partial success.

## 5. Reporting: Events (P5, P7)

- `events` streams typed `RuntimeEvent`s that the Runtime Manager translates into domain Events on the Event Bus (Book 03 §Ch03) — preserving P4 (the runtime does not publish to the bus directly; the Manager reflects its reports).
- Every RuntimeEvent MUST be **secret-free** (P7): it reports effects, progress, and outcomes by reference and by non-secret fact, never a materialized secret value.
- Events MUST carry correlation/trace context (Book 14 §03) so the execution is traceable end-to-end (Intent → … → Execution).

## 6. Capability and secret model (P7, P8)

- A runtime acts **only** through its `grantSet` (P8). To materialize a secret an execution needs, the runtime presents the binding token from `executionCtx` to the **Secret Broker**, which resolves the value into the runtime's execution context **only if** the runtime holds the authorizing capability and **only at execution** (§Ch06, Book 11 §04).
- The interface gives a runtime **no** method to read secrets ahead of time, in bulk, or outside an execution. There is no "fetch all credentials" call — by construction, the interface cannot express it.
- A runtime MUST treat materialized secrets as ephemeral: used for the effect, never logged, never emitted, never persisted beyond the execution's need (Book 11 §04).

## 7. Isolation and trust (P8, P11)

- The runtime runs inside an isolation boundary (Book 11 §10) with the `limits` from `init`; it cannot exceed them. A crashing/hanging runtime is contained (Book 03 §Ch09 §4, §Ch13) — its executions fail or reschedule (where safe), the Kernel and other runtimes are unaffected.
- Anything the runtime returns that is security- or determinism-relevant — chiefly the `RuntimeGraph` from `lower` — is **conformance-checked** (Ch 07, Book 05 §Ch05 §5) before execution. Trust is never extended to the runtime's own claim of fidelity.

## 8. Stability (AEP-0002)

- AEP-0002 carries an explicit stability level (Experimental → Beta → Stable, per the AEP process). Runtimes declare the interface version they implement; promotion requires the conformance suite (Ch 07) and real adopters.
- The interface evolves additively (P11, P6); a breaking change is a new major AEP version with a negotiation/compatibility path so existing runtimes keep working.

## 9. Invariants (normative summary)

1. Every runtime implements the semantic AEP-0002 interface: describe/supports/lower/execute/cancel/events/health/init/shutdown; transport is an implementation choice.
2. `init` grants least-privilege capabilities and resource limits; the runtime has no authority beyond its grant set and cannot exceed its limits (P8).
3. Interface and Low IR versions are negotiated; incompatible runtimes are refused, not degraded.
4. `execute` realizes the RuntimeGraph faithfully; `cancel`/failure run compensation and reach explained terminal states — never silent partial success.
5. RuntimeEvents are reported through the Runtime Manager (not directly to the bus) and are always secret-free with trace context (P4, P5, P7).
6. The interface offers no way to read secrets except by-reference materialization at execution under capability; materialized secrets are ephemeral.
7. The runtime is isolated and untrusted; its `lower` output is conformance-checked before execution; the interface is a versioned, stability-graded AEP.
