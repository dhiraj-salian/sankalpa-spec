# RFC-0006: Vault base-version stamping for lost-update-safe bidirectional Knowledge synchronization

| Field | Value |
|-------|-------|
| **Status** | Proposed |
| **Authors** | Dhiraj Salian (Phase 2 hardening review) |
| **Domain / Book** | Knowledge / Book 09 (and Book 02) |
| **Shepherd (Domain Lead)** | Knowledge Domain Lead |
| **Created** | 2026-07-16 |
| **Supersedes / Superseded by** | — |
| **Tracking issue** | TBD |

> Raised by the Phase 2 hardening pass (adversarial review toward v1.0). Number 0006 reserved; open for review by the Knowledge Domain Lead and Reviewers. Second hardening batch (0005–0011); first Knowledge-domain finding, independent of the others.

> **Final Comment Period — disposition: accept.** Called 2026-07-16 by the Knowledge Domain Lead; concludes **2026-07-30** (10 working days). Solo-maintainer repo — author, Domain Lead, and Reviewer roles are currently held by one maintainer, so the FCP is recorded here for auditability rather than run on a thread; the ≥2-Reviewer gate ([process §7](../process/rfc-process.md)) is waived and noted until a second maintainer joins. Blocking objections must cite concrete technical harm. **No FCP-blocking items**: all 21 open design questions across the batch were resolved before proposing, and the review pass fixed the two defects it found (0005's value/authority split, 0007's unbounded hold).

## 1. Executive Summary
Vault ⇄ graph synchronization ([Book 09 §Ch05](../spec/book-09-knowledge/05-vault-graph-synchronization.md)) claims to prevent lost updates via optimistic concurrency: *"Each representation change carries the Knowledge Resource's `resourceVersion`; a stale change is rejected and re-reconciled against current state — no lost update"* (§Ch05 §4). Optimistic concurrency is defined as a compare-and-swap: *"Every write carries the `resourceVersion` the client observed; the store commits only if it still matches"* ([Book 02 §Ch08 §2](../spec/book-02-resource-model/08-storage-and-consistency.md)). **But the vault is Obsidian-compatible Markdown, edited by humans directly on the filesystem, offline, outside the Resource Manager** — the deliberate design of [Book 09 §Ch03](../spec/book-09-knowledge/03-vault-model.md) ("humans MAY author/edit the vault through Obsidian directly", §Ch03 §4). A direct file edit reads and carries **no** `resourceVersion`; the note front-matter ([Book 09 §Ch03 §2](../spec/book-09-knowledge/03-vault-model.md)) has no field for one. So for exactly the edit path the vault exists to support, the optimistic-concurrency premise does not hold: when the Knowledge controller later observes a directly-edited note, it has **no baseline** to reject a stale change against — and the "no lost update" guarantee is unbacked.

This RFC introduces a **synced-base version stamp** in vault front-matter — the `resourceVersion`/`generation` the note was last reconciled from, written by the controller on every graph→vault render — turning vault→graph reconciliation into a **three-way merge** (base, current Resource, edited note) with a defined common ancestor. This makes optimistic concurrency real for offline direct edits, resolves same-trust concurrent edits that §Ch05 §4's trust/provenance rule cannot adjudicate, and makes a missing/tampered base stamp **fail safe** (surface a conflict, never blind-overwrite).

## 2. Problem Statement
Both faces of Knowledge are authoritative and independently editable ([Book 09 §Ch03 §3](../spec/book-09-knowledge/03-vault-model.md), §Ch05 §3). Sync's stated safety rests on three pillars ([Book 09 §Ch05 §4](../spec/book-09-knowledge/05-vault-graph-synchronization.md)): optimistic concurrency, trust/provenance-aware merge, and human-surfacing of consequential conflicts. Pillars two and three are sound. **Pillar one is unfounded for the vault's primary edit path**, and the other two do not cover the gap it leaves:

- **The CAS premise is violated by design.** [Book 02 §Ch08 §2](../spec/book-02-resource-model/08-storage-and-consistency.md) optimistic concurrency requires the writer to have *observed* a `resourceVersion` and carry it back. An Obsidian/filesystem edit ([Book 09 §Ch03 §4](../spec/book-09-knowledge/03-vault-model.md)) is not a store write and observes no version. The front-matter schema ([Book 09 §Ch03 §2](../spec/book-09-knowledge/03-vault-model.md)) carries `id`, `kindOfKnowledge`, `trust`, `provenance`, `lastReviewed` — but **no version/base stamp**. The controller, on observing the edited file, cannot distinguish "edited from current state" from "edited from a week-old copy."
- **Offline widens the window arbitrarily.** The vault is "plain files … survives independently of any one tool" ([Book 09 §Ch03 §1](../spec/book-09-knowledge/03-vault-model.md)); a human can edit a note offline (a laptop clone) while the Experience loop promotes an updated fact into the graph for the same `id` ([Book 10 §Ch04](../spec/book-10-experience/04-feedback-into-knowledge.md)). On reconnect, both faces changed with no shared reference version.
- **Trust/provenance cannot adjudicate the common case.** Pillar two only helps when the conflicting changes differ in trust (`Authoritative` beats `Inferred`). Two same-trust changes — a human vault edit and a `Corroborated` machine update, or edits from two of a user's devices — have no tiebreak. Without a base version there is no way to compute *which* change is the stale one, so the controller either silently clobbers one (lost update — the very thing §Ch05 §4 pillar one promises against) or cannot even detect that a conflict occurred.
- **The guarantee is stated absolutely.** §Ch05 §7 invariant 3 asserts higher-trust assertions "are not silently overwritten" and Knowledge is "never left quietly inconsistent." Both can fail for a same-trust offline edit under the current mechanism.

Cost of doing nothing: the flagship Knowledge-consistency guarantee is false for the vault's headline feature (direct human curation, offline, in the tool people already use), producing silent lost updates precisely where human-authored `Authoritative` Knowledge is most valuable and least recoverable.

## 3. Alternatives Considered
- **Do nothing.** Rejected: leaves "no lost update" unbacked for direct/offline vault edits and same-trust conflicts silently resolved or lost.
- **Make the vault read-only / graph-authoritative.** Rejected: contradicts the co-equality rule ([Book 09 §Ch03 §3](../spec/book-09-knowledge/03-vault-model.md), §Ch05 §3) and the "human curation is first-class" bet ([Book 09 §Ch01 §4](../spec/book-09-knowledge/01-knowledge-vs-memory.md)) — the entire reason the vault exists.
- **Route all vault edits exclusively through the Web Runtime** (so every edit is an in-system, versioned write. Rejected: forbids direct Obsidian/offline editing, which [Book 09 §Ch03 §1/§4](../spec/book-09-knowledge/03-vault-model.md) makes normative; portability ("survives independently of any one tool") is a stated P11 goal.
- **Last-writer-wins by `lastReviewed`/mtime.** Rejected: filesystem mtimes and human-set `lastReviewed` are unreliable and unsynchronized across offline clones; LWW silently destroys the loser and cannot detect a genuine conflict — the anti-pattern §Ch05 §4 pillar three exists to prevent.
- **Base-version stamp in front-matter + three-way reconcile (chosen).** The controller writes the reconciled `resourceVersion`/`generation` into each note on graph→vault render; vault→graph reconcile reads it as the common ancestor. This is the git-merge insight (a three-way merge needs a merge base) applied to two authoritative representations, and it makes [Book 02 §Ch08 §2](../spec/book-02-resource-model/08-storage-and-consistency.md) optimistic concurrency actually applicable to offline edits.

## 4. Proposed Design
**4.1 Synced-base stamp in front-matter (normative).** The vault note front-matter ([Book 09 §Ch03 §2](../spec/book-09-knowledge/03-vault-model.md)) gains a controller-managed field:

```yaml
syncedFrom:
  resourceVersion: <the Knowledge Resource resourceVersion this note was last reconciled from>
  generation: <metadata.generation at that reconcile>   # spec-version the body/fields reflect
```

- The **Knowledge controller** ([Book 09 §Ch05 §2](../spec/book-09-knowledge/05-vault-graph-synchronization.md)) **MUST** write `syncedFrom` on every graph→vault render, recording the exact `resourceVersion`/`generation` the rendered note reflects.
- The field is **controller-owned**: humans are not expected to set it, and a human-authored *new* note (no `id`) has none — that is a create, not a conflict (§4.3).
- It is an **opaque monotonic token**, carrying no value material, so it is safe in a front-matter surface that is fed to planning ([Book 09 §Ch06](../spec/book-09-knowledge/06-knowledge-in-planning.md)) and is secret-free by the same rule as the rest of the note ([Book 09 §Ch03 §5](../spec/book-09-knowledge/03-vault-model.md), P7) — a version counter is not a secret.

**4.2 Three-way reconcile on vault→graph (normative).** When the controller observes a changed note carrying `id` I and base B = `syncedFrom.resourceVersion`, against the current Knowledge Resource at `resourceVersion` C:
- **B == C (fast-forward).** The graph has not changed since the note was synced; apply the note's fields/body as a store write with CAS at `resourceVersion == C` ([Book 02 §Ch08 §2](../spec/book-02-resource-model/08-storage-and-consistency.md)). A concurrent store write races the CAS and re-reconciles — the existing guarantee, now with a real observed version.
- **B < C (concurrent change).** The Resource advanced since the note's base. The controller computes a **three-way merge** over base B, current C, and the edited note: fields/relations changed on only one side merge; fields changed on **both** sides to **different** values are a genuine conflict → resolve by trust/provenance where the pillar-two rule decides ([Book 09 §Ch05 §4](../spec/book-09-knowledge/05-vault-graph-synchronization.md)), else **mark the Knowledge conflicted** (a condition, [Book 02 §Ch03 §3.5](../spec/book-02-resource-model/03-desired-vs-actual-state.md)) and surface to a human (§Ch05 §4 pillar three). The controller **MUST NOT** blind-overwrite the side it did not originate.
- **B > C or B unknown to the store.** Treated as §4.3.

**4.3 Missing / invalid / new-note handling (fail-safe, normative).**
- **No `id`** (human authored a brand-new note): the controller mints a `Knowledge` Resource, treats the whole note as a create, and stamps `syncedFrom` — never a conflict.
- **`id` present but `syncedFrom` absent, malformed, or referencing an unknown `resourceVersion`** (a human hand-edited or stripped it, or restored an ancient backup): the base is **unknown**, so the controller **MUST** treat the edit as conflicting and surface it for human resolution (§Ch05 §4 pillar three) rather than assume it is current. Fail-safe over fail-silent: an unverifiable base never authorizes an overwrite.

**4.4 Relationship to existing pillars.** This RFC does not replace trust/provenance merge or human-surfacing; it supplies the **missing common ancestor** those pillars assumed. With a base version, "which change is stale" becomes decidable, so pillar two adjudicates real conflicts (not phantom ones from re-observing unchanged fields) and pillar three fires only when a true same-trust conflict remains.

## 5. Tradeoffs
**Gain:** optimistic concurrency becomes real for the vault's primary (direct, offline) edit path; same-trust concurrent edits are detected instead of silently lost; the "no lost update" / "never quietly inconsistent" invariants become true as stated; conflict surfacing fires precisely, not on every re-observation.
**Give up:** one controller-managed field appears in front-matter (minor human-visible noise in the vault); the sync controller must retain or recompute a base representation to diff against (a three-way merge needs base content, not just the base version — see §14); a human who deletes the stamp forces a conflict-surface (intentional fail-safe, mild friction).

## 6. API Changes
Additive; no method signatures change:
- Knowledge controller ([Book 09 §Ch05](../spec/book-09-knowledge/05-vault-graph-synchronization.md)) gains the graph→vault stamp-write and the vault→graph three-way reconcile as internal obligations; both ride the existing reconciliation loop ([Book 07](../spec/book-07-controllers/README.md)).
- No Kernel API change: `resourceVersion`/`generation` already exist ([Book 02 §Ch02 §4](../spec/book-02-resource-model/02-resource-anatomy.md), §Ch08 §2); this RFC only *records* them in the vault representation.

## 7. Resource Changes
None to the `Knowledge` Resource schema. The change is to the **vault representation** ([Book 09 §Ch03 §2](../spec/book-09-knowledge/03-vault-model.md)): front-matter gains `syncedFrom` (controller-owned, secret-free). The graph representation is unchanged (it already lives at a `resourceVersion`).

## 8. Event Changes
Additive: the existing sync Events ([Book 09 §Ch05 §2](../spec/book-09-knowledge/05-vault-graph-synchronization.md), P5) gain a `knowledge.conflictDetected` domain Event (references the Knowledge `id` and the base/current versions, secret-free) when §4.2's both-sides case surfaces a conflict, so operators and the Web Runtime see unresolved conflicts. Per-subject ordering on the `Knowledge` subject suffices ([Book 03 §Ch03 §5](../spec/book-03-kernel/03-event-bus.md)).

## 9. Security Impact
No new secret flow and no relaxation. `syncedFrom` is an opaque version token with no value material — safe wherever `id`/`resourceVersion` already are; the vault's secret-freedom rule ([Book 09 §Ch03 §5](../spec/book-09-knowledge/03-vault-model.md), P7) is unchanged. The fail-safe of §4.3 *strengthens* integrity: an unverifiable base can never silently overwrite an `Authoritative` human assertion, closing a path by which a restored-backup or tampered note could clobber governed Knowledge. Governance of who may assert trust levels ([Book 09 §Ch03 §5](../spec/book-09-knowledge/03-vault-model.md), P9) is unaffected; conflict resolution remains policy-governed. Security review advisable (touches Knowledge integrity, which feeds planning).

## 10. Performance Impact
The stamp write is one small field per graph→vault render (already a write). Three-way reconcile adds a base-vs-current diff on vault→graph sync — O(size of the note), only when a note actually changed, and it *reduces* spurious churn by not re-applying unchanged fields. Conflict surfacing is rare by construction.

## 11. Testing Strategy
- Conformance: offline-edit lost-update test — sync note; advance the graph for the same `id`; edit the note's *stamped-base* copy offline; reconcile ⇒ conflict detected/merged, **no** graph change silently lost.
- Same-trust concurrent edit ⇒ conflict surfaced (marked conflicted), not silently resolved.
- Disjoint-field concurrent edit ⇒ clean three-way merge, no conflict.
- Fail-safe: `id` present, `syncedFrom` stripped/mangled ⇒ conflict-surface, never overwrite; new note without `id` ⇒ create, never conflict.
- Idempotence: re-reconciling a consistent, stamped note is a no-op ([Book 09 §Ch05 §2](../spec/book-09-knowledge/05-vault-graph-synchronization.md)).

## 12. Documentation Changes
Amend [Book 09 §Ch03 §2](../spec/book-09-knowledge/03-vault-model.md) (add `syncedFrom` to the front-matter schema and rules; note it is controller-owned); rewrite [Book 09 §Ch05 §4](../spec/book-09-knowledge/05-vault-graph-synchronization.md) pillar one to the base-version three-way model and state the offline/direct-edit case explicitly; add the fail-safe of §4.3 to §Ch05 §4 pillar three; update the §Ch05 §7 invariants. New Glossary entries: *Synced-base stamp*, *Three-way Knowledge reconcile*.

## 13. Migration Strategy
Additive and pre-1.0. Existing notes lack `syncedFrom`; on first reconcile after adoption the controller stamps them (treating the current graph state as the base, §4.3's unknown-base rule applies only to *edited* unstamped notes, so an unedited legacy note is simply stamped, not conflicted). Notes authored by tools unaware of the field keep working — the field is controller-managed and its absence triggers the fail-safe path, never data loss.

## 14. Risks
- **Base content availability.** A three-way merge needs the base *content*, not only the base version; the controller must retain the last-synced representation (or reconstruct it from the Knowledge Resource's Event history / versioned store, [Book 02 §Ch08 §5](../spec/book-02-resource-model/08-storage-and-consistency.md)) — flagged for design in the reflecting PR; if only version (not content) is retained, the mechanism degrades to conflict-on-any-divergence (still safe, less ergonomic).
- **Human edits the stamp.** Mitigated by §4.3 fail-safe (unknown base ⇒ conflict-surface, never overwrite).
- **Stamp noise in the vault.** A controller-owned front-matter field is mildly unergonomic in Obsidian; mitigated by rendering it unobtrusively and documenting it as machine-managed.
- **Interaction with Knowledge Packs.** Pack-imported Knowledge ([Book 09 §Ch07](../spec/book-09-knowledge/07-knowledge-packs.md)) already merges by trust/provenance; `syncedFrom` composes with, and does not replace, that path — verify in review.

## 15. Future Improvements
Field-level conflict markers rendered inline in the note (git-style) for human resolution in Obsidian; a vault-side lightweight client that observes `resourceVersion` to pre-empt conflicts before they occur; content-hash-based base identity to make the merge base tool-independent.

---
### Resolved questions
- **Record `generation` as well as `resourceVersion`?** Yes — **both**, per §4.1. They answer different questions: `resourceVersion` is the CAS token for the write ([Book 02 §Ch08 §2](../spec/book-02-resource-model/08-storage-and-consistency.md)) and advances on *any* write including status-only; `generation` advances **only on `spec` change** ([Book 02 §Ch02 §4](../spec/book-02-resource-model/02-resource-anatomy.md)). The three-way merge cares about *content* divergence, so `generation` is what distinguishes "the graph meaningfully changed" from "a status-only write happened" — without it, routine status writes would manufacture spurious conflicts. Use `generation` to decide fast-forward vs. merge; use `resourceVersion` for the CAS.
- **Where does the base *content* live?** A controller-side **last-synced cache**, with reconstruction from the versioned store / Event history as fallback, and the §4.3 fail-safe as the floor. The cache is fast but explicitly **not authoritative**: a cache miss attempts reconstruction, and if the base content cannot be established the base is **unknown** and §4.3 applies (conflict-surface, never blind-overwrite). This reuses the existing fail-safe rather than inventing a new failure mode, so cache eviction degrades to safe-but-noisier, never to data loss.
- **Conflicted Knowledge: excluded from planning, or lowered trust?** **Excluded** from planning retrieval ([Book 09 §Ch06](../spec/book-09-knowledge/06-knowledge-in-planning.md)) until resolved. Trust weighting answers *"how much should this count?"*; a conflicted unit is not low-trust but **indeterminate** — the system does not know which assertion is true. Feeding an indeterminate fact in at reduced weight still lets a possibly-wrong assertion shape a plan. Fail-closed and surface it, consistent with §Ch05 §4's "never left quietly inconsistent."

### Unresolved questions
*(none — all resolved ahead of review)*
