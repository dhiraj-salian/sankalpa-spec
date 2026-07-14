# Book 06 · Chapter 05 — Reference Runtimes

*Nature: **Informative**. · Reflects: RFC-0001; illustrates P11, IR-P7, IR-P10. These are lowering/execution *targets*, not dependencies of the core.*

> This chapter sketches how representative runtimes map to the runtime interface (Ch 02) and execution semantics (Ch 03). It is illustrative: each is a *plugin target*, and the core names none of them (IR-P7, Book 03 §Ch04 §3). The purpose is to show that the runtime-agnostic Low IR contract is realizable across genuinely different engines — the evidence for the portability claim (Ch 04 §6).

## 1. How to read these mappings

For each runtime the key questions are the ones the **fidelity gate** (Ch 04 §3.1) asks:
- Which **effects** can it perform (Book 04 §Ch06)?
- Which **`ExecPolicy`** features can it honor natively — retry, timeout, idempotency keys, **compensation** (Book 04 §Ch04 §4)?
- How does it **materialize secrets** at execution only (Ch 06)?
- What is its **cost/latency** profile (informs selection among valid candidates, Ch 04 §3.4)?

A runtime that lacks a required feature simply reports `supports = false` for modules needing it (Ch 02 §2) and is not selected — it is never asked to fake fidelity.

## 2. Durable workflow engines (e.g. Temporal-class)

- **Strengths:** native durable retries, timeouts, and long-running orchestration; **first-class compensation** (saga) — an excellent match for Low IR's `ExecPolicy`.
- **Mapping:** Low IR instructions → workflow activities; schedule partial order → workflow control flow; `Compensate(ref)` → saga compensation handlers; idempotency keys → activity idempotency.
- **Selection niche:** the default choice for plans requiring compensation, long durations, or strong durability guarantees.

## 3. Container / job orchestrators (e.g. Kubernetes-class)

- **Strengths:** scalable execution of containerized capability implementations; resource isolation; scheduling.
- **Mapping:** `CapabilityInvocation`s whose implementations are container images → Jobs/Pods; effects → the container's declared network/filesystem access; schedule → job dependencies.
- **Considerations:** native compensation is weaker than a workflow engine's; a plan needing rich compensation may prefer a durable engine or require a compensation-capable wrapper. Secret materialization uses the orchestrator's secret-injection mechanism, driven *only* by Broker-resolved references at execution (Ch 06).

## 4. Function / edge platforms (e.g. Cloudflare-Workers-class, Trigger.dev/Windmill/Kestra-class)

- **Strengths:** low-latency, fine-grained execution; good for short, stateless capability invocations and event-driven steps.
- **Mapping:** instructions → function invocations; schedule → orchestrated function calls or the platform's own workflow primitives (for the ones that have them, e.g. Windmill/Kestra flows, Trigger.dev tasks).
- **Considerations:** durability/compensation vary widely by platform; the `supports` report must honestly reflect what each can guarantee. Edge platforms suit cost/latency-sensitive plans whose `ExecPolicy` they can honor.

## 5. Node-based automation (e.g. n8n-class)

- **Strengths:** large library of pre-built integrations (which map naturally to Capabilities); visual, operator-friendly execution graphs.
- **Mapping:** Low IR → an n8n-style node graph; `CapabilityInvocation`s → nodes; dataflow edges → node connections; effects → the nodes' external calls.
- **Role in the roadmap:** the **first** runtime compiler target (Roadmap Phase 7) precisely because a rich integration library lets the intent→execution arc be demonstrated end-to-end quickly. Its retry/compensation features are mapped where present and reported honestly where absent.

## 6. Scripting / shell (e.g. Bash-class)

- **Strengths:** universal, zero-infrastructure execution of simple, local capability implementations.
- **Mapping:** instructions → commands; schedule → sequencing; effects → filesystem/process/network the script touches.
- **Considerations:** minimal native durability/idempotency/compensation — so a Bash runtime reports `supports = false` for modules requiring durable retry or compensation. Suited to simple, idempotent, local plans; a valuable *lowest-common-denominator* target that proves the contract works even on a trivial engine.

## 7. What the spread demonstrates

These runtimes differ enormously — durable orchestrator vs. edge function vs. shell — yet each executes the **same** Low IR contract to the extent it honestly `supports` it, and each materializes secrets only at execution by reference. That is the whole point (Book 01 §05, Ch 01 §5):

- **Portability is real** (Ch 04 §6): a plan runs on whichever valid runtime is selected, with equivalent observable results.
- **The core owns none of them** (IR-P7): adding a new runtime is writing a plugin (Ch 02) and a `Runtime` Resource, never a core change (P11).
- **Fidelity is preserved by honesty** (Ch 02 §2, Ch 04 §3.1): a runtime advertises exactly what it can do; the fidelity gate does the rest.

## 8. Note on adding a runtime

A new runtime is added by: implementing AEP-0002 (Ch 02), passing the conformance suite (Ch 07) at a declared stability level, packaging it (Book 12), and registering a `Runtime` Resource. No Book in this specification is edited to add a runtime — this chapter's list is illustrative and open-ended, exactly as P11 intends.
