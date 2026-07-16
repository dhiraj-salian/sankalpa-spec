# Prior-Art Study: Temporal

*Status: Accepted · Informs: Books 06 (execution semantics), 07 (controllers), 10 (replay/Experience); RFC-0002.*

## 1. System in one paragraph
Temporal is a durable-execution platform: you write ordinary code, and the platform makes it survive process crashes, machine loss, and multi-day waits. It does this by **event-sourcing the execution itself** — every step's result is appended to a history, and on recovery the workflow function is *re-executed from the start* against that history, replaying past decisions from the log instead of redoing them. The price of this trick is a hard requirement: workflow code must be **deterministic**. That bargain — determinism buys durability — is the same bargain Sankalpa makes, which makes Temporal our closest ancestor for Book 06 §Ch03.

## 2. Core ideas
1. **Durable execution via replay.** State is not checkpointed; it is *reconstructed* by re-running deterministic code over an append-only event history.
2. **The determinism requirement.** Workflow code may not read the clock, generate randomness, or perform I/O directly — all non-determinism is delegated to *activities*, whose results are recorded.
3. **Activities as the effect boundary.** Anything that touches the world is an activity: retried independently, with declared timeouts and retry policy, and idempotent by the author's obligation.
4. **Saga compensation.** Long-running work rolls back by running compensating actions in reverse, because distributed transactions do not exist.
5. **Timers and waits are free.** A workflow can sleep for a month; durability makes long waits an ordinary control-flow construct rather than an infrastructure project.
6. **Versioning as a first-class hazard.** Changing workflow code breaks replay of in-flight executions; the platform exposes explicit versioning gates because there is no way to hide this.

## 3. Design decisions & trade-offs
- **Replay-based recovery** buys full durability with no explicit state management, at the cost of the determinism constraint leaking into the programmer's daily experience — the constraint is invisible until it is violated, and then it is a production incident.
- **Code-as-workflow** buys expressiveness and familiar tooling, at the cost of making determinism *unenforceable statically*: nothing stops you writing `time.Now()`, and the failure surfaces only on replay.
- **Activities as the sole effect escape hatch** buys a clean boundary, at the cost of an idempotency obligation the platform states but cannot check.
- **History as state of record** buys auditability and replayability, at the cost of history size limits and continue-as-new mechanics.

## 4. Relevance to Sankalpa
Book 06 §Ch03 is Temporal's semantics, restated against IR. Our determinism obligation (§Ch03 §1), idempotency and retries (§2), timeouts and cancellation (§3), and saga compensation (§4) all have Temporal ancestry. RFC-0002 (replay semantics and recorded reasoning) is our version of Temporal's central move — record the non-deterministic result, replay the deterministic remainder — applied to a source of non-determinism Temporal never had: model reasoning. RFC-0004 (compensation failure, terminal, escalation) confronts a question Temporal's saga support leaves to the author.

## 5. What we adopt
- **Durable execution with replay as the recovery model**, driven by IR `ExecPolicy` and runtime-agnostic (Book 15 §Ch04); Execution is a Resource with observable status (Book 06 §Ch03 §5).
- **The determinism/effect split**, which is exactly our Pure-vs-effectful IR distinction: Temporal's "workflow code" is our deterministic IR, and Temporal's "activities" are our effectful Capability invocations (Book 04 §Ch06).
- **Record the non-deterministic result and replay against it.** `Reason`, `Time`, and `Random` are our determinism-relevant effects (Book 04 §Ch06 §4); recorded reasoning (RFC-0002) is Temporal's recorded activity result, applied to the model.
- **Idempotency as an explicit obligation tied to a declared effect** — `StateWrite(scope)` (Book 04 §Ch06 §2) marks precisely where the obligation bites (Book 06 §Ch03 §2), rather than leaving it to author discipline.
- **Saga compensation** as the answer to partial failure (Book 06 §Ch03 §4) — no distributed transactions, as in Book 02 §Ch08 §4.
- **Versioning as a stated hazard, not a footnote.** Temporal's replay-versioning pain is why IR is content-addressed and immutable (Book 04 §Ch07) and why P6 forbids silent history rewriting: an Execution replays against *the IR it started with*.
- **Long waits are ordinary** — human approval (Book 11 §Ch07, Book 13 §Ch06) is a wait that can last days, and durable execution is what makes that unremarkable.

## 6. What we reject / change
- **Determinism by author discipline.** This is the divergence that matters. Temporal *asks* you to be deterministic and finds out at replay; we make determinism a **property of the representation** — IR-P1 requires deterministic evaluation, and non-determinism cannot be written without declaring an effect the verifier checks (Book 04 §Ch06 §3, deny-by-default). We refuse a constraint that is only discoverable in production.
- **Code as the executable artifact.** Temporal executes a general-purpose program, which is why it cannot check anything statically. Our executable artifact is verified IR (P1, RFC-0001), which is why we can validate policy (Book 05 §Ch04) *before* execution — Temporal has no analog and structurally cannot.
- **Temporal as the platform.** It is a **lowering target** (Book 01 §Ch06 §2, Book 06): a runtime a plan may be lowered to after selection, not the architecture. The same IR must run on a different runtime with the same meaning (IR-P7).
- **Its own history as state of record.** Our state of record is ARM plus the Event Bus (Book 02 §Ch08, Book 14 §Ch01); a runtime's internal history is an implementation detail of that runtime, and we specify observable semantics instead (Book 06 §Ch07 conformance).
- **Compensation left to the author.** RFC-0004 specifies what happens when compensation *itself* fails — terminal states and escalation — rather than treating it as the author's problem.
- **No policy, no capability model, no secret discipline.** Activities run with ambient credentials; P7/P8 require the Secret Broker (Book 11 §Ch04) and capability grants instead.

## 7. Open questions
- Recorded reasoning (RFC-0002) makes a replay faithful to a past model output. But is a replay that *skips the model* still a meaningful re-execution, or a recording playback? Book 06 §Ch03 §7 ("determinism vs. the real world") is where this must be honest.
- Temporal's history-size limits forced continue-as-new. What is our bound on Execution history, and does a long-lived Execution need an analogous escape?
- If a runtime backend *is* Temporal, how much of Book 06 §Ch03 is enforced by us versus delegated to it — and what does the conformance suite (§Ch07) actually test to tell the difference?

## 8. References
- Temporal documentation on workflow determinism, activities, retries, and versioning; the Cadence/Temporal design lineage (Uber); Garcia-Molina & Salem, "Sagas" (1987); Helland, "Life beyond Distributed Transactions" (2007).
