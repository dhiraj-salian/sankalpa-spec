# Book 04 · Chapter 07 — Serialization and Content Addressing

*Nature: **Normative**. · Reflects: RFC-0001, RFC-0002 (reasoning ledger, replay variants); realizes principles P6, P13 and IR-P6.*

> AOS IR must have a single canonical serialization and be identified by the content hash of that form (IR-P6). Content addressing is the mechanism behind caching, deduplication, deterministic replay, and determinization (P13). It only works if serialization is *canonical* — free of incidental variation.

## 1. Two representations, one meaning

IR has:
- an **in-memory / textual** form for humans and tools (the notation used throughout this Book), and
- a **canonical binary/normalized** form used for hashing, storage, and transport.

The two **MUST** be interconvertible without semantic loss. Hashing and identity are defined over the *canonical* form only; the textual form is a projection for readability and is never hashed directly.

## 2. Canonicalization requirements

The canonical form **MUST** eliminate every degree of freedom that does not change meaning, so that semantically identical modules serialize identically (and thus hash identically). A conforming serializer **MUST**:

1. **Normalize node identity.** Node/instruction ids are assigned canonically (e.g. by a deterministic topological numbering), so two graphs differing only in id labels are identical.
2. **Canonical ordering.** Sets (effect sets, record fields, map entries, sibling nodes with no ordering constraint) are emitted in a defined canonical order (e.g. by structural hash), not authoring order.
3. **Normalize types.** Types are emitted in canonical structural form (Ch 05 §6); `Named(hash)` uses the canonical type hash.
4. **Fixed encodings.** Numbers, timestamps, and strings use single canonical encodings (no alternate representations, no insignificant whitespace, normalized Unicode).
5. **Exclude non-semantic metadata.** Human comments, source positions, and authoring annotations are **not** part of the canonical form and **MUST NOT** affect the hash. (They may travel alongside, keyed by node id, but outside the hashed body.)
6. **Preserve semantic structure exactly.** Everything that *does* affect meaning — dataflow edges, control structure, effects, policies (Low IR), bindings-by-reference — is included.

Canonicalization **MUST** be deterministic and total: the same module always yields the same bytes.

## 3. Content addressing

- The identity of an IR module is `Hash = H(canonical_bytes)` for a fixed, versioned cryptographic hash `H` (the hash algorithm is itself part of the IR version metadata so it can be migrated, Ch 09).
- Two modules are **the same module** iff their hashes are equal. This is the definition of IR identity used everywhere: the `IRModule` Resource (Book 02 §Ch07) stores this hash; Low IR references its source High IR by hash (`fromModule`, Ch 04 §2); the cache is keyed by it.
- Hashing is **structural and recursive**: sub-regions, types, and Capabilities referenced by content each have their own hash, so a change deep in the graph changes only the hashes on the path to the root (a Merkle-DAG). This makes fine-grained caching and diffing efficient.

## 4. What content addressing enables

- **Deduplication.** Identical IR — produced independently by different planners or runs — has one identity and is stored once.
- **Deterministic replay.** An Execution records the exact Low IR hash **and its reasoning ledger** — the content-addressed record of every `CapturedReasoning`/`Time`/`Random` output (RFC-0002, Book 06 §Ch03 §1). Replay reproduces behavior (IR-P1) by injecting those recorded outputs rather than re-invoking the model. Golden-file replay tests (Book 05, Book 10) compare hashes in the **reconstruction** variant, where recorded outputs are injected exactly and no external effect re-fires; the **re-execution** variant instead re-fires effects under live policy (Book 06 §Ch03 §1). Content addressing is what makes "same recorded output" a decidable check for both replay injection and determinization evidence.
- **Caching of compilation and results.** The compiler caches pass outputs keyed by input hash (Book 05 §03); a `Compilation` (Book 02 §Ch07) need not re-run if its input hash is unchanged.
- **Determinization (P13).** The determinization engine (Book 05 §06, Book 10 §06) recognizes a recurring reasoning/subgraph *by hash of its inputs and shape* and folds it into a cached Capability. Content addressing is precisely what makes "we have seen this exact work before" a decidable, cheap check.

## 5. Secrets and hashing (P7)

Because a `SecretRef` is opaque and carries no value (Ch 05 §2.2), the canonical form — and therefore the hash — contains **no secret material** (IR-P4). This is essential: hashes are stored, logged, transported, and compared widely; if a secret influenced a hash it could leak or enable oracle attacks. The hash depends only on the *reference*, never the value. Two modules that differ only in which secret they reference differ in hash (the references differ); two that would use the same secret share the reference and thus that part of the hash — with no exposure of the secret itself.

## 6. Transport and storage

- The canonical form is the wire and storage format across the Kernel API and Event Bus (Book 03); tools MAY exchange the textual form but MUST canonicalize before hashing or executing.
- Modules are stored in a content-addressed store (an implementation choice satisfying Book 02 §Ch08 properties); references between modules are by hash, forming the Merkle-DAG of §3.
- Backward/forward compatibility of the serialization across IR versions is governed by Ch 09.

## 7. Invariants (normative summary)

1. IR has one canonical form; identity is the content hash of that form; the textual form is never hashed directly.
2. Canonicalization is deterministic, total, and strips all non-semantic variation (ids, ordering, comments, source positions, encodings) while preserving all semantic structure.
3. Hashing is structural/recursive (a Merkle-DAG); localized changes cause localized hash changes.
4. The hash contains no secret material; identity depends on secret *references*, never values (P7).
5. Content addressing is the basis for deduplication, deterministic replay (via the content-addressed reasoning ledger, injected exactly in reconstruction), compilation/result caching, and determinization.
6. The canonical form is the wire/storage format; tools canonicalize before hashing or executing.
