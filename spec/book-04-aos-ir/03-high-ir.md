# Book 04 · Chapter 03 — High IR

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P1, P8, P9, P11 and IR-P1…P10.*

> High IR is the planner-facing, runtime-agnostic form: the artifact a Planner (Book 08) emits and the compiler's optimization/policy passes (Book 05) consume. It expresses *what should happen and with what declared effects*, as a typed dataflow graph — and nothing about scheduling mechanics, retries, or runtimes.

## 1. Structure: a typed dataflow graph

A High IR **Module** is a directed graph:

```
Module (High):
  irVersion   string           # IR schema version (Ch 09)
  signature   Signature        # typed inputs and outputs of the module (Ch 05)
  nodes       list<Node>       # the steps
  edges       list<DataEdge>   # explicit dataflow: producer.output → consumer.input
  effects     EffectSet        # union of node effects (Ch 06), computed & checked
```

- **Nodes** are steps; **edges** are explicit dataflow (IR-P9). A consumer node's input port is fed by exactly one producer output (or a module input). There is no ambient mutable store.
- The graph over data edges **MUST** be acyclic; iteration is expressed by explicit control-flow nodes (§3), not by data cycles.
- Every node and every value is typed (IR-P2); every node declares effects (IR-P3).

## 2. Values and ports

- A **value** is an immutable, typed datum flowing along an edge. Types are defined in Ch 05.
- A node has typed **input ports** and **output ports**. The edge feeding an input **MUST** have a type assignable to that input's type (Ch 05 §assignability).
- **Literals** are constant values embedded in a node. A literal **MUST NOT** be, or contain, secret material (IR-P4); secrets appear only as `SecretRef` values (Book 02 §Ch06 §4).

## 3. Node kinds

High IR has a small, fixed set of node kinds. New behavior is added via Capabilities, not new node kinds (IR-P7).

### 3.1 `CapabilityInvocation`
Invokes a typed, reusable **Capability** (Book 02 §Ch07 §4).
```
CapabilityInvocation:
  capability   Ref<Capability>     # by id (Book 02 §Ch06)
  inputs       map<port, ValueRef>
  outputs      map<port, Type>
  effects      EffectSet           # MUST equal the capability's declared effects
```
- Inputs/outputs **MUST** type-check against the Capability's signature.
- The node's declared effects **MUST** match the Capability's declared effects; a mismatch is a verification error (Ch 08).

### 3.2 `Reasoning`  — *the sole locus of non-determinism*
Represents a decision delegated to a planner/model. This is the one node kind whose *production* may be non-deterministic; its *result* is a captured typed value (IR-P1).
```
Reasoning:
  intent       string             # what the model was asked to decide (NOT a secret; NO secret interpolation)
  inputs       map<port, ValueRef> # typed context provided to the reasoning
  output       Type               # the typed result the reasoning must produce
  effects      EffectSet          # MUST include effect `reason`; MAY include others only if declared
  determinize  bool               # eligible for determinization into a Capability (P13)?
```
- A `Reasoning` node **MUST NOT** receive any `SecretRef`-derived value as context (P7, Book 08 §04); its `inputs` are non-secret typed values only.
- Downstream nodes consume only the *typed output*, never the reasoning process. This confines non-determinism to the node boundary (IR-P1).
- `determinize: true` marks the node as a candidate for the determinization passes (Book 05 §06): if the same reasoning recurs with equivalent inputs, it may be folded into a cached Capability.

### 3.3 Control-flow nodes
Iteration and branching are explicit nodes, keeping data edges acyclic (§1):
- `Branch` — selects one of N typed sub-regions based on a typed predicate value.
- `Parallel` — declares independent sub-regions that MAY execute concurrently; results join into typed outputs. Concurrency is a *permission*, not a mechanic — Low IR (Ch 04) decides realization.
- `Loop` — bounded iteration over a typed collection or until a typed condition; the loop body is a sub-region with typed carry values. Unbounded loops **MUST** declare a termination bound or a cost effect (Ch 06) so verification can reason about termination/cost.
- `Map` — a common special case of `Parallel` over a collection; sugar that lowers to `Parallel`.

### 3.4 Data nodes
Pure, effect-free value manipulation: `Construct`/`Destructure` (records), `Select` (fields), `Convert` (checked type conversions per Ch 05). Data nodes **MUST** be pure (effect set empty) — they exist so dataflow can be shaped without smuggling effects.

## 4. What High IR deliberately omits

High IR does **not** express (these belong to Low IR, Ch 04):
- Evaluation **order** beyond what data/control edges imply (Low IR fixes a total/partial order).
- **Retry, timeout, idempotency,** and **error-handling** policy.
- **Secret binding** to execution (High IR carries only `SecretRef` values as opaque data; it does not resolve them).
- Any **runtime** selection or runtime-specific detail (IR-P7).

Keeping these out is what lets a Planner emit High IR without knowing anything about runtimes or execution mechanics (Book 08).

## 5. Effects in High IR

Every node declares an `EffectSet` (Ch 06); the module's `effects` is their union and **MUST** be consistent with the nodes (IR-P3, deny-by-default). The **policy-validation pass** (Book 05 §04) consumes this: a plan whose declared effects violate policy is rejected *before* lowering (P9). Because effects are declared at the High level, governance happens on the clean, human-meaningful artifact rather than on execution minutiae.

## 6. Relationship to Goals and Planners

A Planner (Book 08) transforms `Goal` Resources (Book 02) into a High IR Module and wraps it in an `IRModule` Resource (`level: High`). The Planner **MUST** emit only well-typed, fully effect-annotated High IR (its conformance obligation, Book 08 §07); the compiler then verifies (Ch 08) before doing anything else. A Planner that emits ill-typed or under-annotated High IR fails verification and the plan does not proceed — non-determinism is thereby stopped at the planner boundary.

## 7. Example (illustrative; full examples in Ch 10)

A High IR fragment for "summarize last week's sales, then email it":
```
module (High) irVersion=0.1:
  inputs:  { period: DateRange }
  n1 = CapabilityInvocation cap=sales.query
          inputs={ range: period }
          outputs={ rows: Table }
          effects={ network(read: crm) }
  n2 = Reasoning intent="Write an executive summary of these sales rows"
          inputs={ data: n1.rows }
          output=Markdown
          effects={ reason }
          determinize=false
  n3 = CapabilityInvocation cap=email.send
          inputs={ to: literal("sales-team"), body: n2.output }
          outputs={ receipt: SendReceipt }
          effects={ network(write: email) }
  outputs: { receipt: n3.receipt }
  effects: { network(read: crm), reason, network(write: email) }
```
Note: no runtime, no retry policy, no secret values — only the `sales.query`/`email.send` Capabilities carry `SecretRef`s internally, resolved at execution (Book 06 §06).

## 8. Invariants (normative summary)

1. High IR is a typed dataflow graph; data edges are acyclic; iteration/branching are explicit control-flow nodes.
2. Every value and node is typed; every node declares effects; the module effect set is the checked union (deny-by-default).
3. `Reasoning` is the only non-deterministic node; it receives no secret-derived context and exposes only a typed output.
4. New behavior is added via Capabilities, not new node kinds; High IR names no runtime.
5. High IR omits ordering/retry/error/secret-binding mechanics (those are Low IR).
6. Planners emit only well-typed, fully effect-annotated High IR; the compiler verifies before proceeding.
