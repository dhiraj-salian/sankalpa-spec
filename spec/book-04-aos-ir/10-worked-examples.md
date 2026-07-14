# Book 04 · Chapter 10 — Worked Examples

*Nature: **Informative**. · Illustrates Chapters 02–09 with complete, traceable examples. Notation is illustrative pseudo-IR; the canonical form (Ch 07) is normative.*

> These examples show real IR at both levels and how it upholds the principles. They are teaching artifacts, not normative schemas; where an example and a normative chapter disagree, the chapter wins.

## Example A — "Every Monday, summarize last week's sales and email the team"

This is the running example from Book 02 §Ch02 (the `Intent`), followed to executable IR.

### A.1 Goals (Book 08 output, ARM Resources)
The planner derives two `Goal`s: *(G1)* obtain last week's sales data; *(G2)* produce and deliver a summary. The recurring schedule ("every Monday") is a property of the long-lived `Intent`, which spawns a fresh set of Goals per run (Book 02 §Ch07 note).

### A.2 High IR (planner emits; verifier checks — Ch 03, Ch 08)
```
module (High) irVersion=0.1
  inputs:  { period: DateRange }              # last week, supplied per run
  outputs: { receipt: SendReceipt }

  n1 = CapabilityInvocation cap=crm.sales.query      # a typed, reusable Capability
         inputs  = { range: period }
         outputs = { rows: List<SalesRow> }
         effects = { Network(read, "crm"), SecretUse(crm-token), Cost(api.crm) }

  n2 = Reasoning intent="Write a concise executive summary of these sales rows"
         inputs  = { data: n1.rows }           # typed, NON-secret context only (Ch 03 §3.2)
         output  = Markdown
         effects = { Reason, Cost(model.tokens) }
         determinize = false                    # summaries vary; not a determinization target

  n3 = CapabilityInvocation cap=email.send
         inputs  = { to: literal("sales-team@acme"), subject: literal("Weekly Sales"), body: n2.output }
         outputs = { receipt: SendReceipt }
         effects = { Network(write, "email"), SecretUse(smtp-cred), StateWrite("email:sent") }

  edges: period→n1.range, n1.rows→n2.data, n2.output→n3.body
  effects: { Network(read,"crm"), SecretUse(crm-token), Cost(api.crm),
             Reason, Cost(model.tokens),
             Network(write,"email"), SecretUse(smtp-cred), StateWrite("email:sent") }
```
Observe the principles at work:
- **No runtime named** (IR-P7); no retry/order (that is Low IR).
- **Secrets are references** inside the Capabilities (`crm-token`, `smtp-cred` are `SecretRef`s); the summary reasoning `n2` receives **no** secret-derived value (P7, Ch 03 §3.2).
- **Effects are fully declared** and the module set is their union (Ch 06); Policy can now see "this plan sends external email and spends CRM API + model budget."

### A.3 Policy validation (Book 05 §04, on High IR — P9)
Policy `P-email-approval` says: any `Network(write,"email")` to an external domain requires an `Approval`. The pass does not reject; it **annotates** that lowering must insert an approval gate before `n3`, and confirms `Cost` is within the workspace budget. A different policy forbidding external email entirely would instead **reject** here with an explainable diagnostic (Ch 06 §5).

### A.4 Low IR (compiler lowers; verifier re-checks — Ch 04, Ch 08)
```
module (Low) irVersion=0.1  fromModule=H(<high-hash>)
  schedule: i1 → i_approval → i2 ∥ (none) → i3        # partial order; i2 has no concurrency here

  i1 = LoweredCapability crm.sales.query
        outputs = { rows: List<SalesRow> }
        effects = { Network(read,"crm"), SecretUse(crm-token), Cost(api.crm) }
        bindings = [ { name: crm-token, ref: SecretRef(crm-token), resolveAt: Execution,
                       grantedBy: cap.crm.sales.query } ]
        policy   = { idempotency: Idempotent, retry: {maxAttempts:3, backoff:exp}, timeout: 30s, onError: Fail }

  i_approval = LoweredCapability approval.request        # inserted by policy annotation (A.3)
        inputs  = { summaryOf: i1.rows }
        outputs = { decision: ApprovalDecision }
        effects = { StateWrite("approval") }
        policy  = { idempotency: IdempotentWithKey(run-id), onError: Fail }

  i2 = CapturedReasoning intent="executive summary…"
        inputs  = { data: i1.rows }
        output  = Markdown
        effects = { Reason, Cost(model.tokens) }
        policy  = { idempotency: NonIdempotent, retry: none, timeout: 60s, onError: Fail }   # no retry: non-idempotent reasoning (Ch 08 §2.5)

  i3 = LoweredCapability email.send
        inputs  = { to: "sales-team@acme", subject: "Weekly Sales", body: i2.output }
        outputs = { receipt: SendReceipt }
        effects = { Network(write,"email"), SecretUse(smtp-cred), StateWrite("email:sent") }
        bindings = [ { name: smtp-cred, ref: SecretRef(smtp-cred), resolveAt: Execution, grantedBy: cap.email.send } ]
        policy   = { idempotency: IdempotentWithKey(run-id), retry: {maxAttempts:2}, timeout: 20s, onError: Compensate(email.retract) }
```
Now everything execution needs is explicit and **still no runtime is named**:
- **Ordering** is a partial order; **retries** attach only where idempotency permits (`i2` reasoning gets no retry — Ch 08 §2.5); the **email** write is keyed-idempotent with a **compensation** for rollback (Ch 04 §4).
- **Secrets** are execution-time **binding sites**, capability-gated (`grantedBy`), never values (P7).
- The **approval gate** materialized from the policy annotation (A.3), realizing P9 as an actual instruction.

### A.5 Runtime selection + RuntimeGraph (Books 05–06)
Only now is a runtime chosen (Book 06 §04) — say **Temporal** for its durable retries/compensation. The backend lowers Low IR to a Temporal workflow graph. The **same** Low IR lowered to a different runtime (e.g. a Bash+cron target) MUST produce an observably equivalent execution (IR-P10). Runtime specifics live only in the RuntimeGraph, never upstream.

### A.6 Execution → Events → Experience (Books 06, 10, 14)
Execution emits Events (`i1.started`, `approval.granted`, `email.sent`, …), all secret-free (P7). The `Experience` (Book 02 §Ch07, Book 10) records the Intent, the High/Low hashes, the runtime, metrics, and outcome — feeding Knowledge and future planning.

## Example B — Determinization in action (P13)

Suppose a *different* recurring intent runs `n2' = Reasoning intent="classify this support ticket into {billing, bug, other}"` with `determinize = true`. Over many runs, the Experience/determinization engine (Book 05 §06, Book 10 §06) observes that for equivalent typed inputs the reasoning yields stable outputs.

- It synthesizes a deterministic `Capability ticket.classify` (typed `SupportTicket → TicketClass`) whose declared effects are a **subset** of the original (`Reason` removed, perhaps `Network(read)` kept) — Ch 06 §7.
- The next lowering replaces the `CapturedReasoning` with a `LoweredCapability ticket.classify` and records the substitution (Ch 04 §3).
- Result: the same intent now executes **deterministically and cheaper** — repeated reasoning became a reusable deterministic capability. This is the platform's compounding advantage (Book 01 §05) made mechanical by content addressing (Ch 07) and the effect system (Ch 06).

## Example C — What verification rejects (Ch 08)

Each of these fails verification with a precise, secret-free diagnostic — *before* anything runs:

1. **Secret into reasoning.** Feeding a `SecretRef` (or a value derived from one) into a `Reasoning` node → fails IR-P4/type opacity (Ch 05 §2.2). *"E-SEC-001: SecretRef flows into Reasoning input `data`; secrets must not enter reasoning context."*
2. **Undeclared effect.** A Capability that performs `Network(write)` but declares only `Network(read)` → fails deny-by-default (Ch 06 §3). *"E-EFF-004: node n3 performs Network(write,'email') not in its declared effect set."*
3. **Retry on non-idempotent write.** A `Network(write)` instruction with `idempotency: NonIdempotent` and a `retry` policy → fails determinism constraints (Ch 08 §2.5). *"E-DET-002: retry attached to non-idempotent effect; supply an idempotency key or remove retry."*
4. **Type mismatch.** `n1.rows : List<SalesRow>` fed into an input typed `Table` with no `Convert` → fails assignability (Ch 05 §3). *"E-TYP-003: List<SalesRow> not assignable to Table at n2.data."*

Each rejection is the system stopping non-determinism, ungoverned effects, or secret leakage at the IR boundary — exactly what Book 04 exists to guarantee.
