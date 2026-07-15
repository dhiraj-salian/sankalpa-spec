# Book 09 · Chapter 04 — The Graph Model

*Nature: **Normative**. · Reflects: ADR-0001; realizes principles P2, P8, P10, P11. Companion to §Ch03 (vault), §Ch05 (sync), §Ch06 (planning).*

> The graph is Knowledge's **machine face**: a graph database over which the system retrieves relevant, related Knowledge efficiently. This chapter specifies the graph's node/edge model, its identity relationship to the vault, its query surface for planning, and the rule that its backend is a replaceable provider — the *contract* is the model, not the database.

## 1. Why a graph

Knowledge is fundamentally **relational**: a project relates to an architecture, which relates to capabilities, which relate to organizations and people; a lesson relates to the Experiences that support it. Retrieval for planning is a *traversal* problem — "given this Goal, what related, trusted Knowledge is relevant?" — which a graph answers naturally and a flat store does not. The graph is what makes Knowledge *usable by machines* the way the vault (§Ch03) makes it usable by humans; §Ch05 keeps the two consistent.

## 2. The node/edge model

```
Node:
  id            = the Knowledge Resource id (§Ch03 §2; same identity as the vault note)
  kindOfKnowledge   (§Ch02)
  properties    = structured fields (title, tags, scope, trust, provenance refs)
  # NOTE: the human body lives in the vault; the graph holds structure + a reference,
  #       not a duplicate of the prose.

Edge (Relation):
  type          = a typed relation (e.g. relates-to, part-of, supersedes,
                  supported-by, uses-capability, authored-by, derived-from)
  from, to      = node ids (or refs to other Resources, Book 02 §Ch06)
  properties    = provenance, trust, time
```

- **Identity is shared with the vault.** A graph node's `id` **is** the `Knowledge` Resource id and the vault note's front-matter `id` (§Ch03 §2). There is one Knowledge identity with two representations — not two objects to reconcile by name. This is what makes synchronization (§Ch05) a matter of keeping *representations* consistent, not *identities* mapped.
- **Edges are typed and provenanced.** Relations carry a type and their own provenance/trust, so a traversal can weigh not just nodes but the *reliability of the connections* between them.
- **Edges may reference non-Knowledge Resources** (by id, Book 02 §Ch06): a Knowledge node may relate to a `Capability`, `Workflow`, `Policy`, or `Experience` Resource, tying understanding to mechanism (§Ch02 §4).

## 3. The query surface for planning

The graph exposes a **retrieval interface** (through the Knowledge Manager, Book 03 §Ch11 §2) that planners use during Goal derivation and planning (§Ch06, Book 08 §02–§03):

- **Relevance retrieval** — given a Goal/context, return related Knowledge, traversing typed edges and ranked by relevance **and trust** (§Ch02 §3). Retrieval MUST weigh trust: an `Authoritative` fact outranks an `Inferred` one.
- **Relationship traversal** — follow typed edges (part-of, uses-capability, supersedes) to assemble a coherent context.
- **Provenance retrieval** — return the provenance of any Knowledge (supporting Experiences, author) so the planner can weigh and cite it (P10).

The retrieval interface returns **structured, non-secret, provenance-tagged** Knowledge (§Ch06, Book 08 §04) — never raw prompt-dumps and never secrets (§5).

## 4. The backend is a replaceable provider (P11)

The Knowledge Manager is **core** (Book 03 §Ch11 §2, §4) — it owns the model and the sync — but the **graph database backend is a provider plugin** (Book 03 §Ch04 §1). The specification mandates the *graph model and query contract* (§2–§3), not a specific database (ADR-0001). A deployment may back the graph with any store that satisfies the contract; swapping it is a provider change, not a core change (P11). This is the same properties-not-products discipline applied to ARM storage (Book 02 §Ch08 §1).

## 5. Security: tenancy, capability, secret-freedom

- **Tenancy (Book 11 §08 §5).** The graph is workspace-partitioned; a traversal is scoped to the querying subject's authorized workspaces and MUST NOT cross tenants without an explicit grant (Book 02 §Ch06 §5). A planner for workspace A cannot retrieve workspace B's Knowledge.
- **Capability-gated (P8).** Querying the graph is a capability-gated operation (Book 03 §Ch06); a subject retrieves only what it is authorized to.
- **Secret-free (P7).** The graph, like the vault, holds no secret values (§Ch01 §6). Because its whole purpose is to feed planning context (§Ch06), a secret in the graph would breach P7 directly. Nodes/edges reference secrets only by class/reference (Book 11 §04 §5).
- **Attributable (P10).** Knowledge changes and their provenance are recorded; graph mutations emit Events (P5) and are auditable (Book 11 §09).

## 6. Invariants (normative summary)

1. The machine-readable representation of Knowledge is a graph of typed, provenanced nodes and edges; retrieval for planning is a trust-weighted traversal.
2. A graph node's identity **is** the Knowledge Resource id and the vault note's id — one identity, two representations — so sync reconciles representations, not identities.
3. Edges are typed and provenanced and may reference non-Knowledge Resources, tying understanding to mechanism.
4. The retrieval interface returns structured, non-secret, provenance-tagged, trust-weighted Knowledge — never prompt-dumps or secrets.
5. The graph model and query contract are normative; the graph database backend is a replaceable provider plugin (P11).
6. The graph is workspace-partitioned, capability-gated, secret-free, and attributable (P7, P8, P10).
