# Pattern Study: Event Sourcing

*Status: Accepted · Informs: Books 02 §Ch08 (history), 03 §Ch03 (Event Bus), 10 (Experience), 14 (event model, audit).*

## 1. Pattern in one paragraph
Event sourcing stores **what happened** rather than **what is**: the system's record is an append-only log of immutable domain events, and current state is a fold over that log. The consequences are large and mostly good — a complete audit trail by construction, time travel, the ability to answer questions nobody thought to ask when the data was written, and state that can be rebuilt from first principles. The consequences are also large and mostly underestimated: the log is forever, schema evolution has no `ALTER TABLE`, and "delete this person's data" collides with the pattern's central premise.

## 2. Core ideas
1. **Events are the record; state is derived.** The log is authoritative, and any state view is a cache of a fold over it.
2. **Events are immutable, append-only, and past-tense facts.** You do not edit history; you append a correcting event.
3. **State is reconstructible** by replaying from the beginning — the property that makes projections disposable and bugs recoverable.
4. **Snapshots** as a performance concession: fold once, checkpoint, replay from there.
5. **Temporal queries.** "What did we believe on Tuesday?" is answerable, which no state-only store can do.
6. **It pairs with CQRS** ([study](cqrs.md)) — projections are folds — but neither requires the other.
7. **Events derive from committed state.** The dual-write problem (write the state, then publish the event, and crash between) is the pattern's classic failure and demands an outbox or a log-as-source design.

## 3. Design decisions & trade-offs
- **The log as the record** buys perfect audit and reconstruction, at the cost of the log being *permanent* — every mistake, every leak, every regrettable payload is preserved by design.
- **No schema migration.** Old events were written by old code and must be readable forever, so evolution means upcasting, versioned events, and additive-only change. This is the cost teams discover in year two.
- **Rebuildable projections** buy operational forgiveness, at the cost of a replay path that must be maintained and, at volume, is not fast.
- **Immutability vs. erasure.** GDPR-style deletion is genuinely at odds with an append-only log; the escapes (crypto-shredding, tombstones, log rewriting) are all compromises of one property to save another.
- **Over-application** is the usual failure: event sourcing is right where history *is* the domain (ledgers, audit, workflow) and expensive ceremony where it is not.

## 4. Relevance to Sankalpa
The pattern is close to load-bearing here. P5 — "everything emits events" — requires a complete stream, and its rationale names event-sourced reconstruction explicitly. Book 02 §Ch08 §5 covers history and reconstruction; Book 03 §Ch03 specifies the Event Bus; Book 14 §Ch01 is the event model, with §5 addressing the dual-write problem directly ("events derive from committed state") and §4 the additive-evolution rule. Book 10's Experience capture is a fold over execution history, and determinization (Book 10 §Ch06) mines it for evidence — for us, history is not an audit nicety but the *input to the system's central learning loop*.

## 5. What we adopt
- **A complete event stream as an invariant** (P5) — every state change emits, so the stream has no holes to reason around.
- **Immutable, append-only history that is never silently rewritten** (P6), giving audit (Book 11 §Ch09) and attribution their foundation.
- **No dual-write.** Book 14 §Ch01 §5's rule — events derive from committed state — is the pattern's hardest-won lesson, adopted as a model-level invariant rather than an implementation note.
- **Additive-only event-schema evolution** (Book 14 §Ch01 §4), because old events are read by new code forever; a stable naming scheme (§Ch03) makes the stream navigable.
- **Reconstruction from history** (Book 02 §Ch08 §5) as a specified capability.
- **Events as first-class domain facts**, not log lines — typed, named, and secret-free (Book 14 §Ch01 §6).
- **History as the substrate for learning.** Experience (Book 10 §Ch02–03) is captured from the stream, and determinization's evidence thresholds (Book 10 §Ch06 §3) are queries over it. This is the pattern's "answer questions you did not know to ask" benefit, cashed in as the platform's central bet (P13).
- **Content-addressed, immutable artifacts alongside the log** ([Git study](../prior-art/git.md)), so an event can reference exactly the IR that ran.

## 6. How faithfully we apply it (and where we deviate)
- **We are event-*driven* with an authoritative state store, not purely event-sourced.** ARM is the state of record (Book 02 §Ch08); `Get` reads current state directly and is linearizable — it is not a fold over the log at read time. The log is authoritative for *history*; ARM is authoritative for *now*. A purist would call this a compromise, and it is a deliberate one: a control plane whose reads require a replay cannot reconcile at the rate P3 demands. Book 02 §Ch08 §5 should be read as "reconstruction is possible," not "reconstruction is the read path."
- **The dual-write problem is therefore ours to solve, and we solve it by rule.** Having both a state store and a stream is exactly the configuration that produces dual-write bugs; Book 14 §Ch01 §5 forbids the pattern rather than hoping implementations avoid it.
- **Secret-freedom is a model-level invariant, not a review practice.** This is the deviation with the most force. An append-only log plus a secret is a permanent breach — the [Git](../prior-art/git.md) and [Terraform](../prior-art/terraform.md) studies both show it landing in practice. P7 makes it structurally impossible: values never enter Events (Book 14 §Ch01 §6), IR (IR-P4), or storage (Book 02 §Ch08 §6). Only references. The pattern's permanence is safe only because there is nothing dangerous to preserve.
- **Retention is bounded, and this genuinely costs us.** Book 02 §Ch08 §7 has retention and tenancy rules; the pattern says keep everything forever. Where they conflict, the deletion obligation wins — which means reconstruction has a horizon, and any Experience or audit claim that depends on events older than that horizon is unsound. Naming this is better than discovering it: **what is retained bounds what can be reconstructed, replayed, or learned from**, and Books 10 and 11 §Ch09 both depend on that bound being stated.
- **No aggregate/command-handler ceremony.** We take the log discipline; we do not prescribe the tactical apparatus (Book 00 §Ch05).
- **Execution history is not our log.** A runtime may itself be event-sourced ([Temporal study](../prior-art/temporal.md)); that is its internal business. Our stream records observable, specified semantics (Book 06 §Ch03 §6).

## 7. Open questions
- What *is* the retention horizon, and what depends on it? Determinization mines history for evidence (Book 10 §Ch06 §3) — if events expire before a pattern accumulates, the learning loop has a silent floor.
- Erasure vs. immutability is unresolved in the general case. Does Book 02 §Ch08 §7 specify a mechanism (crypto-shredding? tombstones?), or leave an operator with an obligation the model cannot honor?
- If ARM is authoritative for current state and the log for history, can they disagree — and what detects it? A purely event-sourced system cannot have this bug; we can.
- Additive-only schema evolution is a promise about the *future* readability of old events. Is there a version-tolerant reader requirement in the conformance suites, or only in prose?

## 8. References
- Fowler, "Event Sourcing" (bliki); Young, "CQRS and Event Sourcing" talks and his later cautions; Vernon, *Implementing Domain-Driven Design* (events chapter); Kleppmann, *Designing Data-Intensive Applications* (ch. 11) and "Turning the Database Inside Out"; the transactional-outbox literature on dual writes; the [CQRS](cqrs.md) and [Git](../prior-art/git.md) studies.
