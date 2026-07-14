# Book 04 · Chapter 05 — Type System

*Nature: **Normative**. · Reflects: RFC-0001; realizes IR-P2, IR-P8.*

> Every value and construct in AOS IR is typed (IR-P2). This chapter defines the type language, assignability, and checking. The type system is deliberately small and decidable: its job is to reject ill-formed IR before execution and to make verification (Ch 08) sound — not to be a general-purpose programming type system.

## 1. Design stance

- **Structural, not nominal.** Two types are the same if their structure is the same. Rationale: IR is produced by many independent planners and Packages that never coordinate on names; structural typing lets them interoperate by shape. A Capability declaring `{ id: String, total: Money }` accepts any value of that shape.
- **Decidable and total.** Type checking **MUST** terminate with accept or a diagnostic for any module (supports IR-P8). No feature may make checking undecidable.
- **Explicit over inferred.** Ports declare their types; the checker verifies, it does not perform global inference. Local convenience inference (e.g. a literal's type) is permitted where it stays decidable.

## 2. The type language

```
Type :=
    Primitive
  | Record { field: Type, ... }
  | List<Type>
  | Map<Primitive, Type>            # keys are primitive (comparable)
  | Optional<Type>                  # value or absent
  | Union<Type, Type, ...>          # tagged; checked at use (Ch 06 has no bearing)
  | Enum { variant, ... }
  | Ref<Kind>                       # a typed reference to an ARM Resource (Book 02 §Ch06)
  | SecretRef                       # opaque handle to a Secret; carries NO value type (P7)
  | Capability<Signature>           # a first-class capability value
  | Dynamic                         # explicitly dynamic; see §5
  | Named(hash)                     # a named type defined in the module/type-pack, by content hash
```

### 2.1 Primitives
`Bool, Int, Float, String, Bytes, Timestamp, Duration, DateRange, Money, Json`. Primitives have fixed semantics defined in the [notation reference](../book-15-appendices/02-notation-reference.md). `Bytes`/`String` literals **MUST NOT** carry secret material (IR-P4).

### 2.2 `SecretRef` is not a value type
`SecretRef` is a first-class type so the graph can *carry* a secret reference, but it exposes **no** operations to read a value in IR. There is no `deref` node in any level of IR; the only place a `SecretRef` is resolved is an execution-time binding site (Ch 04 §5). Any construct that attempts to convert `SecretRef` to `String`/`Bytes` **MUST** fail verification. This makes P7 a *type-level* guarantee, not merely a convention.

### 2.3 `Signature`
```
Signature: { inputs: Record, outputs: Record, effects: EffectSet }
```
Capabilities, modules, and `Reasoning` outputs are typed by signatures. A `CapabilityInvocation` (Ch 03 §3.1) type-checks its `inputs`/`outputs` and effects against the referenced Capability's `Signature`.

## 3. Assignability

A value of type `S` may feed an input of type `T` iff `S` is **assignable to** `T` (`S <: T`). Assignability is structural and covariant where sound:

- Primitive: `S <: T` iff identical, plus the explicit widenings in §4.
- `Record`: width + depth subtyping — `S` assignable to `T` if `S` has (at least) every field of `T` with an assignable type. Extra fields are allowed (structural width subtyping).
- `List<S> <: List<T>` iff `S <: T`; likewise `Optional`, `Map` value.
- `Union`: `S <: Union<…,T,…>` if `S <: T` for some member; assigning *from* a union requires a checked `Branch`/`Convert` that narrows it.
- `Optional<T>`: `T <: Optional<T>`; the reverse requires an explicit presence check.
- `SecretRef <: SecretRef` only; it is assignable to nothing else and nothing else is assignable to it.
- `Dynamic`: see §5.

Assignability **MUST** be reflexive, transitive, and decidable; the checker computes it structurally.

## 4. Conversions

Implicit conversions are limited to sound, lossless widenings (`Int <: Float` is **not** implicit — it can lose precision; it requires an explicit `Convert`). All other conversions are explicit `Convert` data nodes (Ch 03 §3.4) that are **checked** and **total or failable**:

- A total conversion (e.g. `Int → Json`) always succeeds.
- A failable conversion (e.g. `String → Timestamp`, `Json → Record{…}`) **MUST** produce `Optional<T>` or declare a failure effect (Ch 06) and an `onError` in Low IR (Ch 04 §4). Unchecked conversions are prohibited — they would reintroduce runtime type errors that IR-P2 exists to prevent.

## 5. `Dynamic` — the escape hatch, fenced

Some data genuinely has an unknown shape until runtime (e.g. an arbitrary API response). `Dynamic` models this, but it is *fenced*, not an untyped hole:

- A `Dynamic` value **MUST** be introduced explicitly (a Capability output typed `Dynamic`, or a `Convert` into `Dynamic`).
- A `Dynamic` value **MUST** be *narrowed* by a checked `Convert`/`Branch` before feeding any non-`Dynamic` input; feeding `Dynamic` directly into a typed input **MUST** fail verification.
- Narrowing is failable (§4) and thus forces explicit error handling in Low IR.
- Rationale: this preserves IR-P2/IR-P8 (checking stays total and sound) while admitting real-world dynamism. There is deliberately **no** `Any`-that-bypasses-checking.

## 6. Type identity and content addressing

Because typing is structural, type identity is by **canonical structure hash** (Ch 07): a `Named(hash)` refers to a type by the hash of its canonical form. Two modules that independently define the same-shaped record share a type identity. This is what lets independently-produced IR interoperate and lets the checker deduplicate types.

## 7. Checking obligations

The type checker, as part of verification (Ch 08), **MUST**:
1. Assign a type to every value/port; reject any untyped element.
2. Verify every data edge: producer type assignable to consumer type (§3).
3. Verify every `CapabilityInvocation`/`Reasoning` against its `Signature` (inputs, outputs, effects).
4. Verify all conversions are explicit and checked (§4); reject implicit lossy conversions.
5. Enforce `SecretRef` opacity (§2.2) and `Dynamic` narrowing (§5).
6. Terminate with accept or a precise diagnostic (Book 05 §07) — always (IR-P8).

## 8. Invariants (normative summary)

1. The type system is structural, decidable, and total; checking always terminates with accept or a diagnostic.
2. Every value/port is typed; no untyped elements; the only dynamism is fenced `Dynamic`, which must be checked-narrowed before typed use.
3. `SecretRef` is opaque: no IR construct can derive a value from it; attempts fail verification (P7 as a type-level guarantee).
4. Implicit conversions are limited to sound lossless widenings; all others are explicit, checked, and failable-with-handling.
5. Type identity is by canonical structure hash, enabling interoperation across independently produced IR.
