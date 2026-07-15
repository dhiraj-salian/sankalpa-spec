# Book 00 · Chapter 03 — Notation

*Nature: **Informative** (defines conventions; the full reference is Book 15 §Ch02).*

> This chapter defines the notation conventions used throughout the specification, so that schemas, IR, Events, and diagrams read consistently. The consolidated notation reference is Book 15 §Ch02; this chapter is the orientation.

## 1. Resource schema notation

ARM Resources (Book 02) are shown as field listings with types:

```
Kind:
  metadata: { id, name, workspace, generation, ... }   # (Book 02 §Ch02)
  spec:   { <desired-state fields> }
  status: { observedGeneration, phase, conditions, actualState }
```

Types use the type language of Book 04 §Ch05 (`String`, `Int`, `Record{...}`, `List<T>`, `Optional<T>`, `Ref<Kind>`, `SecretRef`, …). Examples use YAML for readability, but YAML is not privileged — the canonical serialization is defined per artifact (Book 04 §Ch07 for IR; Book 02 §Ch08 for Resources).

## 2. IR notation

AOS IR (Book 04) is shown in an illustrative textual pseudo-IR:

```
module (High|Low) irVersion=X:
  inputs/outputs: { port: Type }
  nN = NodeKind ... inputs={...} outputs={...} effects={...}
  edges: producer.out → consumer.in
```

This textual form is for reading; the **canonical serialization** (Book 04 §Ch07) is normative and is what is hashed/executed. Where an example and Book 04 differ, Book 04 wins (Book 04 §Ch10 preamble).

## 3. Event notation

Events (Book 03 §Ch03, Book 14 §Ch01) use the dotted `<domain>.<subject>.<action>` naming scheme (Book 14 §Ch01 §3), e.g. `execution.succeeded`, `secret.materialized`. Event payloads are shown as field listings; all are secret-free (P7).

## 4. Diagram conventions

Diagrams live as text sources in [`../../diagrams/src/`](../../diagrams/) (Mermaid/PlantUML/D2) and are referenced by relative link (Book 15 §Ch02, `diagrams/README`):
- **Flowcharts** for the transformation pipeline (e.g. [`layered-architecture.mmd`](../../diagrams/src/layered-architecture.mmd)).
- **Sequence diagrams** for protocols/flows.
- **Component diagrams** for structure.
- **State diagrams** for lifecycles.

Normative text does not embed rendered images; it links diagram sources (Book 00 §Ch01 §4 style).

## 5. Requirement and reference notation

- **Requirement keywords** (MUST/SHOULD/MAY) per Book 00 §Ch02.
- **Principle references**: `P1`–`P13` (Book 01 §Ch04); IR principles `IR-P1`–`IR-P10` (Book 04 §Ch02); security invariants `SI-1`–`SI-12` (Book 11 §Ch11).
- **Cross-references**: `Book NN §ChNN §S` (Book, Chapter, Section). `§Ch04` within a Book means that Book's Chapter 04.
- **Artifact references**: `RFC-NNNN`, `ADR-NNNN`, `AEP-NNNN`.

## 6. Summary (informative)

The specification uses consistent notation for Resource schemas (typed field listings), IR (textual pseudo-IR, canonical form normative), Events (dotted names, secret-free), diagrams (text sources, linked not embedded), and references (`P`/`IR-P`/`SI` identifiers and `Book §Ch §` cross-refs). The full reference is Book 15 §Ch02.
