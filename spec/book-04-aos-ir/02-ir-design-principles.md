# Book 04 · Chapter 02 — IR Design Principles

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P7, P9, P13.*

> These are the invariants every AOS IR construct — in either level — MUST satisfy. They are to Book 04 what the Foundational Principles (Book 01 §04) are to the whole system. A proposed IR construct that violates one is rejected on that basis.

## IR-P1 — Deterministic evaluation
Given identical Low IR and identical resolved inputs, execution **MUST** be reproducible: the same observable effects and outputs, every time.
- Non-determinism is permitted **only** inside explicitly-typed **reasoning steps** (Ch 03 §4), which are the compiled record of a planner/model decision. A reasoning step's *result* is captured as a typed value; its *production* is the one place a model may be consulted.
- Every other construct (capability invocation, control flow, data operation) **MUST** be a pure function of its inputs and declared effects. No hidden clocks, randomness, or ambient state; time and randomness, where needed, are explicit typed inputs or capabilities.
- Rationale: determinism is the mission (Book 01 §05) and the precondition for caching, replay, and determinization (P13).

## IR-P2 — Fully typed
Every value and every construct **MUST** carry a static type (Ch 05). Ill-typed IR **MUST** be rejected before execution (Ch 08).
- Capabilities declare input and output types; invocations are type-checked against them.
- There is no untyped "any that bypasses checking"; a genuinely dynamic value uses an explicit `Dynamic`/`Union` type that is itself checked (Ch 05).
- Rationale: types catch whole classes of error before anything runs and make verification (Ch 08) sound.

## IR-P3 — Effects are explicit and declared
Every construct **MUST** declare its externally-visible effects (Ch 06): network, filesystem, secret use, cost, and other side effects.
- **Deny by default:** a construct with an *un-annotated* effect **MUST** be rejected. Silence is not permission.
- Effects compose upward: a step's effects include the union of its children's effects.
- Rationale: Policy validates on declared effects *before* compilation and *before* execution (P9). An effect that can escape annotation is a governance hole.

## IR-P4 — Secrets by reference only
Secret **values** **MUST NOT** appear anywhere in IR — not in any node, literal, annotation, type, or serialized form. A secret is present only as an opaque `SecretRef` (Book 02 §Ch06 §4).
- The value is materialized solely at execution, solely by the Secret Broker, solely to the runtime (Book 06 §06, Book 11 §04).
- Planners and compilers see references, never values (P7, P8).
- Rationale: the IR is stored, cached, diffed, logged, and sometimes shown to humans; a secret in IR would leak through all of those.

## IR-P5 — Versioned and stable in meaning
The IR has a versioned schema (Ch 09). The *meaning* of a construct **MUST NOT** silently change across versions.
- Evolution is additive by default; breaking changes advance the major IR version and require an RFC (P12) and a lowering-compatibility path (Ch 09).
- An `IRModule` records the exact IR version its body conforms to.
- Rationale: reproducibility must survive time; a RuntimeGraph produced last year must remain interpretable (P6).

## IR-P6 — Canonical and content-addressed
Every IR module **MUST** have a single **canonical serialization** and be identified by the content hash of that canonical form (Ch 07).
- Two modules are identical iff their hashes are equal; semantically identical work has one identity.
- Rationale: content addressing is the mechanism for caching, deduplication, replay, and determinization (P13). It only works if serialization is canonical (no incidental differences).

## IR-P7 — Runtime-agnostic
No IR construct — in either level — may name or assume a specific runtime, planner, or vendor.
- High IR is runtime-agnostic in intent; Low IR is runtime-agnostic in *form* (it fixes mechanics like ordering and retries, but not which runtime provides them). Runtime specificity appears only in the RuntimeGraph after backend lowering (Books 05–06).
- Anything a runtime uniquely offers is surfaced as a typed Capability, not as an IR special case.
- Rationale: portability and replaceability (P11); planners must stay ignorant of runtimes (P1, Book 08).

## IR-P8 — Verifiable in isolation
IR **MUST** be checkable — structure, types, effects, and policy well-formedness — by a verifier that consults no runtime (Ch 08).
- Verification is a total function: it terminates with accept or a diagnostic for any input.
- Rationale: execution admits only verified IR; verification is the gate that makes P1/P2/P3 mechanically true.

## IR-P9 — Explicit dataflow
Data dependencies between steps **MUST** be explicit edges (values produced by one step, consumed by another), not implicit shared mutable state.
- High IR is a **typed dataflow graph** with explicit control-flow constructs; there is no ambient mutable store that steps read/write behind the graph.
- Rationale: explicit dataflow is what makes ordering, parallelism, optimization, and deterministic replay analyzable.

## IR-P10 — Semantics-preserving transformation
Every compiler pass and every lowering (Book 05) **MUST** preserve observable semantics: optimization does not change results, and lowering High → Low → RuntimeGraph does not change observable behavior.
- Where a transformation could change behavior (e.g. adding a retry), it does so only in ways declared to be observationally equivalent under the effect/idempotency contract (Ch 04, Ch 06).
- Rationale: users reason about High IR; if lowering could silently change meaning, that reasoning — and determinism — would be void.

## Precedence

When principles tension, IR precedence follows the system precedence (Book 01 §04 §3): **security (IR-P4) and determinism/verification (IR-P1, IR-P2, IR-P3, IR-P8) outrank convenience and optimization (IR-P6, IR-P10 as applied).** A transformation that would improve caching but weaken secret-safety or determinism is rejected.
