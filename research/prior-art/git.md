# Prior-Art Study: Git

*Status: Accepted · Informs: Book 04 §Ch07 (serialization and content addressing), Book 03 §Ch03 (Event Bus), Book 02 (versioning, history).*

## 1. System in one paragraph
Git is a content-addressed object store with a version-control system layered on top — and the object store is the part worth studying. Every object (blob, tree, commit) is named by the hash of its content, which makes identity a *function of the data* rather than an assignment by an authority. From that one decision follow deduplication, cheap comparison, integrity verification, distribution without coordination, and an append-only history that cannot be silently altered. Sankalpa applies the same trick to IR (Book 04 §Ch07).

## 2. Core ideas
1. **Content addressing.** An object's name is the hash of its bytes. Identical content is the same object, everywhere, with no coordination.
2. **Canonical serialization.** Content addressing only works if identical *meaning* produces identical *bytes* — the object format is exact and normalized.
3. **A Merkle DAG.** Objects reference objects by hash, so a commit's hash transitively fixes the entire tree beneath it. Verifying the root verifies everything.
4. **Immutable objects, mutable refs.** The object store is append-only; branches are mutable *pointers* into it. All the change lives in the pointers, none in the history.
5. **Distributed by construction.** Because names are global and content-derived, syncing is "do you have this hash?" — no central allocator, no merge of identity.
6. **Plumbing vs. porcelain.** A small, stable set of primitive commands with a user-facing layer built on top; the primitives are the contract.

## 3. Design decisions & trade-offs
- **Hash-as-identity** buys integrity, dedup, and distribution for free, at the cost of pinning you to a hash function — Git's SHA-1 migration is a decade-long lesson in what "for free" excludes.
- **Storing snapshots, not diffs** buys simple, fast reads, at the cost of storage, reclaimed by packfiles later. The lesson: get the model right, optimize the encoding after.
- **Append-only history** buys auditability and trust, at the cost of making deletion genuinely hard — a real problem when secrets or personal data get committed.
- **Mutable refs over immutable objects** buys a clean separation of "what happened" from "what we currently think," at the cost of rewritable branch pointers (force-push) that erase context the objects still hold.
- **Porcelain complexity** is the standing evidence that a beautiful data model does not confer a usable interface.

## 4. Relevance to Sankalpa
Book 04 §Ch07 is Git's object model applied to IR: canonicalization requirements (§2), content addressing (§3), and what it enables (§4) — caching, replay, and determinization recognizing "we have done this before" (Book 05 §Ch01 §4). Book 03 §Ch03's Event Bus and Book 02 §Ch08 §5's history-and-reconstruction inherit Git's immutable, append-only stance, which P6 states as principle.

## 5. What we adopt
- **Content addressing for IR** (IR-P6, Book 04 §Ch07 §3): an IRModule is named by the hash of its canonical form, which is what makes compilation cacheable by input hash (Book 05 §Ch01 §4), replay meaningful (RFC-0002), and "identical plan" a decidable question rather than a heuristic.
- **Canonicalization as a hard prerequisite** (Book 04 §Ch07 §2). Git's discipline: identical meaning MUST produce identical bytes, or content addressing silently degrades into a cache-miss generator.
- **Immutable objects, mutable pointers.** IR artifacts and Events are immutable; Resource state is the moving pointer (Book 02 §Ch03). This is Git's split, and it is why "what did this Execution run?" always has an exact answer.
- **Append-only history that is never silently rewritten** (P6) — the Event Bus (Book 03 §Ch03) and audit (Book 11 §Ch09) depend on it.
- **Merkle integrity.** A module's hash transitively fixes its referenced content, which is what lets provenance and signing (Book 12 §Ch04) attest to a whole artifact by attesting to one hash.
- **Small stable primitives underneath a friendlier layer** — the plumbing/porcelain split is the Kernel API (Book 03 §Ch02) vs. the interfaces of Book 13.

## 6. What we reject / change
- **Secrets in the history.** Git's append-only history plus content addressing means a committed secret is effectively permanent — the canonical "rewrite history and rotate everything" incident. We refuse this at the model level: secret *values* never enter IR (P7, IR-P4), Book 04 §Ch07 §5 governs hashing and secrets, and Events are secret-free by invariant (Book 14 §Ch01 §6). Only `SecretRef`s are ever hashed.
- **Human-authored, hand-merged content.** Git's whole ergonomic surface — merges, conflicts, rebases — exists because humans edit the same text. IR is machine-produced from Intent; there is no IR merge, and there must not be one. (Where humans *do* edit — the knowledge vault, Book 09 §Ch05 — we adopt Git-like reconciliation and RFC-0006's base-version stamping, which is optimistic concurrency, not three-way merge.)
- **Branching and rewritable refs.** No force-push analog exists for Resources; lifecycle transitions (Book 02 §Ch04) are governed and evented, not repointed.
- **A single hash function baked into identity.** Git's SHA-1 migration is the warning. Book 04 §Ch07 should treat the digest algorithm as versioned and agile from day one — the cost of getting this wrong is paid once, forever.
- **Unbounded retention.** Git keeps everything by default; we have retention and tenancy requirements (Book 02 §Ch08 §7) and legal deletion obligations that an append-only store must be designed to honor, not discover later.
- **Filesystem-shaped storage.** We specify storage *properties* (Book 02 §Ch08 §1), not a `.git` directory.

## 7. Open questions
- Append-only history vs. the right to erasure: how do audit (Book 11 §Ch09) and retention (Book 02 §Ch08 §7) reconcile when a deletion obligation lands on content-addressed, hash-referenced data? Git has no answer; we need one.
- Is IR canonicalization (Book 04 §Ch07 §2) specified tightly enough that two independent implementations produce byte-identical output? If not, content addressing is a per-implementation illusion and the conformance suite must test it.
- Does the digest algorithm carry a version tag in the address itself (multihash-style), and what is the migration path when it must change?

## 8. References
- *Pro Git*, ch. 10 ("Git Internals"); the Git object-format documentation; the SHA-1 transition plan (`hash-function-transition`); the SHAttered collision (2017); Merkle, "A Digital Signature Based on a Conventional Encryption Function" (1987).
