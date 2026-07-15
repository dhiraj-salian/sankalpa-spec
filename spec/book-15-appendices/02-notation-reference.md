# Book 15 · Chapter 02 — Notation Reference

*Nature: **Informative** (consolidated reference; the owning Books are authoritative). · Orientation is Book 00 §Ch03.*

> This chapter consolidates the notation used across the specification into one reference. It is informative: each notation's *normative* definition lives in its owning Book (types Book 04 §Ch05, canonical IR serialization Book 04 §Ch07, Resource storage Book 02 §Ch08, Events Book 14 §Ch01). Use this as a lookup; use the owning Book for the authoritative rules.

## 1. Requirement keywords (Book 00 §Ch02)

`MUST` / `MUST NOT` / `REQUIRED` / `SHALL` — absolute. `SHOULD` / `SHOULD NOT` / `RECOMMENDED` — documented-deviation recommendation. `MAY` / `OPTIONAL` — free choice. Normative statements only.

## 2. Identifiers

- **Principles**: `P1`–`P13` (Book 01 §Ch04).
- **IR principles**: `IR-P1`–`IR-P10` (Book 04 §Ch02).
- **Security invariants**: `SI-1`–`SI-12` (Book 11 §Ch11).
- **Artifacts**: `RFC-NNNN`, `ADR-NNNN`, `AEP-NNNN`.
- **Cross-references**: `Book NN §ChNN §S`; `§ChNN` within a Book is that Book's chapter.

## 3. Types (Book 04 §Ch05)

```
Primitive: Bool Int Float String Bytes Timestamp Duration DateRange Money Json
Composite: Record{field:Type,...}  List<Type>  Map<Prim,Type>  Optional<Type>
           Union<...>  Enum{...}  Ref<Kind>  SecretRef  Capability<Signature>
           Dynamic (fenced, must be narrowed)  Named(hash)
Signature: { inputs: Record, outputs: Record, effects: EffectSet }
```
Structural typing; `<:` is assignability; `SecretRef` is opaque (no value derivable). Full rules: Book 04 §Ch05.

## 4. Resource schema (Book 02 §Ch02)

```
Kind:
  apiVersion: <group>/<version>       # e.g. core.sankalpa.dev/v1alpha1
  kind: <PascalCase>
  metadata: { id, name, workspace, labels, annotations, generation,
              resourceVersion, createdAt, updatedAt, ownerRefs, finalizers, deletionTimestamp }
  spec:   <desired state>
  status: { observedGeneration, phase, conditions[], actualState }
Condition: { type, status(True|False|Unknown), reason, message, observedGeneration, lastTransitionTime }
Phase: Pending | Progressing | Active | Succeeded | Failed | Terminating
```
Full rules: Book 02 §Ch02–§Ch04.

## 5. AOS IR (Book 04 §Ch03–§Ch04)

```
module (High|Low) irVersion=X [fromModule=Hash]:
  inputs/outputs: { port: Type }
  # High node kinds: CapabilityInvocation, Reasoning, Branch, Parallel, Loop, Map, Construct/Select/Convert
  # Low instruction: op, inputs, outputs, effects, policy(ExecPolicy), bindings(BindingSite)
  nN = NodeKind ... effects={...}
  edges: producer.out → consumer.in
ExecPolicy: { idempotency(Idempotent|NonIdempotent|IdempotentWithKey), retry?, timeout?, onError(Fail|Compensate|Fallback|Ignore), compensation? }
```
Canonical serialization (hashed/executed) is normative: Book 04 §Ch07. Textual form here is illustrative.

## 6. Effects (Book 04 §Ch06)

```
Effect: Pure | Reason | Network(mode,target) | Filesystem(mode,scope) | SecretUse(ref)
      | Time | Random | Cost(model) | StateWrite(scope) | Custom(ns,name,payload)
```
Deny-by-default; module effect set = checked union. Full rules: Book 04 §Ch06.

## 7. Events (Book 14 §Ch01, Book 03 §Ch03)

```
type: <domain>.<subject>.<action>     # e.g. execution.succeeded, secret.materialized
Event: { id, type, source, subject, time, workspace, sequence, data, traceContext }
```
Immutable, attributed, per-subject-ordered, secret-free. Full taxonomy: Book 14 §Ch01.

## 8. Diagrams

Text sources in [`../../diagrams/src/`](../../diagrams/) (`.mmd`/`.puml`/`.d2`), linked not embedded. Flowchart (pipeline), sequence (protocol), component (structure), state (lifecycle). See `diagrams/README`.

## 9. Summary (informative)

This reference collects the notations; the owning Books (04 for types/IR/effects, 02 for Resources, 14 for Events) hold the authoritative rules and the canonical/normative forms.
