# Book 09 · Chapter 05 — Vault ⇄ Graph Synchronization

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0006 (base-version three-way reconcile); realizes principles P3, P5, P6, P10. Companion to §Ch03 (vault), §Ch04 (graph), Book 07 (controllers).*

> The vault and the graph are two authoritative representations of one Knowledge (§Ch03 §3, §Ch04 §2). This chapter specifies how they are kept **synchronized**: the reconciliation model, conflict resolution, and consistency guarantees. Synchronization is not a batch export — it is a reconciled, bidirectional, controller-driven process, so the human face and the machine face never drift into disagreement.

## 1. The problem: two faces, one truth

A human edits a runbook in the vault; the system promotes a lesson into the graph (Book 10 §Ch04). Both are legitimate changes to the *same* Knowledge, arriving through *different* representations. Synchronization's job is to make each change appear, consistently, in both faces — so that a planner querying the graph (§Ch06) sees what a human wrote in the vault, and a human reading the vault sees what the loop learned. Without it, the two would diverge and Knowledge would become untrustworthy.

## 2. Synchronization is reconciliation (P3)

Because the vault and graph are representations of `Knowledge` Resources (Book 02 §Ch07), synchronization is **reconciliation** in the ARM sense (Book 02 §Ch03, Book 07), not a bespoke sync script:

- A **Knowledge controller** (Book 07) observes changes to the Knowledge Resource / its representations and drives both faces toward consistency with the Resource's desired state — the standard observe → diff → act → emit loop (Book 02 §Ch03 §3).
- It is **level-triggered and idempotent** (Book 02 §Ch03 §3.1–2): re-running sync on already-consistent Knowledge is a no-op; a missed or duplicated change event cannot corrupt consistency because the controller reconciles from observed current state, not from event contents.
- Every sync action **emits Events** (P5) and is **auditable** (P10, Book 11 §09).

Using reconciliation (rather than a fragile one-shot exporter) is what makes sync robust to partial failure, concurrent edits, and restarts — the same reasons ARM chose reconciliation everywhere (Book 02 §Ch01).

## 3. Bidirectional, with one identity

Sync is **bidirectional** because both faces are authoritative (§Ch03 §3):
- **Vault → graph.** A human's Markdown edit (front-matter + `[[links]]`) is parsed into node properties and typed edges (§Ch04 §2) and reconciled into the graph.
- **Graph → vault.** A machine-promoted or graph-side change is rendered into the corresponding Markdown note and front-matter.

Because **identity is shared** (§Ch04 §2 — the note `id`, the node `id`, and the `Knowledge` Resource id are one), sync never has to *match* two independently-named objects; it keeps two *representations of one identity* consistent. This is the design choice that makes bidirectional sync tractable rather than a perpetual matching problem.

## 4. Conflict resolution

Concurrent changes to both faces (a human edits the vault while the loop updates the graph) can conflict. Resolution rules:

- **Base-version three-way reconcile.** In-system changes carry the Knowledge Resource's `resourceVersion` and commit under optimistic concurrency (Book 02 §Ch08 §2). A **direct vault edit does not**: the vault is deliberately editable in Obsidian, offline, outside the Resource Manager (§Ch03 §1, §4), so such an edit observes no `resourceVersion` and compare-and-swap has nothing to check. Sync therefore reconciles against a recorded **merge base** — the note's controller-managed `syncedFrom` stamp (§Ch03 §2). For a changed note with `id` I, base `B = syncedFrom.generation`, against the Resource's current `generation` C:
  - **B == C (fast-forward).** The graph has not meaningfully changed since the note was synced; apply the note's fields/body as a store write with CAS at the recorded `resourceVersion`. A concurrent store write loses the CAS and re-reconciles — no lost update.
  - **B < C (concurrent change).** Both faces changed. The controller performs a **three-way merge** over base, current, and the edited note: fields/relations changed on only one side merge; fields changed on **both** sides to **different** values are a genuine conflict, resolved by the trust/provenance rule below or else surfaced. The controller **MUST NOT** blind-overwrite the side it did not originate.
  - **Base unknown** (`syncedFrom` absent, malformed, or naming a version the store does not know — e.g. a stripped stamp or a restored backup). Fail-safe: the edit is **conflict-surfaced**, never merged. An unverifiable base never authorizes an overwrite.

  `generation` decides fast-forward vs. merge because it advances only on `spec` change (Book 02 §Ch02 §4), so a status-only write cannot manufacture a phantom conflict; `resourceVersion` remains the CAS token. The base *content* is held in a controller-side last-synced cache; it is **not** authoritative — a miss is reconstructed from the versioned store/Event history, and failing that the base is unknown and the fail-safe applies. This is what makes "no lost update" true for the vault's primary edit path, rather than assuming an observed version that a direct edit never had.
- **Provenance- and trust-aware merge.** Where changes touch different fields/relations, they merge. Where they genuinely conflict on the same assertion, resolution weighs **trust and provenance** (§Ch02 §3): an `Authoritative` human assertion is not silently overwritten by an `Inferred` machine update; a contradicting higher-trust change supersedes a lower-trust one, recording the supersession (Book 10 §Ch04 §5). Trust adjudicates *only* where the sides differ in trust; two same-trust changes are decided by the base (above), not by trust.
- **Human-visible when consequential.** An unresolvable or consequential conflict — including any unknown-base edit and any same-trust both-sides conflict — surfaces to a human (via the Web Runtime, Book 13) rather than being auto-resolved silently, mirroring the clarification discipline (Book 08 §02 §4). The Knowledge is marked conflicted (a condition, Book 02 §Ch03 §3.5) until resolved, never left quietly inconsistent. A conflicted unit is **withheld from planning retrieval** (§Ch06) until resolved: it is not low-trust but *indeterminate*, and an indeterminate assertion must not shape a plan even at reduced weight.

## 5. Consistency guarantees

Synchronization provides **eventual consistency** between the two faces, with bounded, observable convergence:
- After a change to either face, the controller converges both to consistency (P3); the gap is observable via the Knowledge Resource's conditions/`observedGeneration` (Book 02 §Ch02 §4) — you can always see whether sync is pending.
- Sync does **not** promise instantaneous cross-face atomicity (there are no cross-Resource transactions, Book 02 §Ch08 §4); it promises convergence and honest visibility of the transient gap. A planner querying mid-sync gets a consistent (if momentarily slightly stale) graph, never a corrupt one — and the trust/provenance on results lets it weigh freshness (`lastReviewed`, §Ch02 §1).
- Because both faces derive from the one Knowledge Resource and its Event history (P5/P6), they cannot permanently disagree: reconciliation always drives them back together, and the audit trail records how (Book 11 §09).

## 6. Tenancy and secret-freedom

- Sync is **workspace-scoped** (Book 11 §08 §5): it reconciles only within a tenancy boundary; cross-tenant Knowledge is not synchronized across workspaces.
- Sync preserves **secret-freedom** (P7): neither face ever holds a secret (§Ch03 §5, §Ch04 §5), so sync never transports one. There is nothing secret to synchronize — by construction.

## 7. Invariants (normative summary)

1. The vault and graph are kept consistent by a level-triggered, idempotent, Event-emitting reconciliation controller — not a batch export (P3, P5).
2. Sync is bidirectional; shared identity (one id across Resource, note, and node) makes it a representation-consistency problem, not an identity-matching one.
3. Conflicts are resolved against a recorded merge base (`syncedFrom`, §Ch03 §2) by three-way reconcile, then by trust/provenance-aware merge; a direct/offline vault edit carries no `resourceVersion`, so the base — not optimistic concurrency alone — is what prevents lost updates. Higher-trust assertions are not silently overwritten; an unknown base is never merged; consequential and same-trust conflicts surface to a human, and the Knowledge is marked conflicted — and withheld from planning retrieval — until resolved.
4. Sync provides eventual consistency with observable, bounded convergence; it offers no cross-face atomic transaction but guarantees the two faces cannot permanently disagree.
5. Sync is workspace-scoped and preserves secret-freedom — there is nothing secret in either face to synchronize (P7).
