# Book 09 · Chapter 02 — Knowledge Taxonomy

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P2, P6, P10. Companion to Book 02 §Ch07 (Knowledge Resource), Book 10 §Ch04 (promotion).*

> This chapter enumerates the kinds of Knowledge Sankalpa curates and the common structure every Knowledge unit shares. A defined taxonomy is what lets both humans (in the vault, §Ch03) and planners (over the graph, §Ch04) find and trust the right Knowledge — and lets promotion (Book 10 §Ch04) know what shape to produce.

## 1. Every Knowledge unit shares a common structure

Regardless of category, a `Knowledge` Resource (Book 02 §Ch07) carries a common envelope so that tooling, retrieval, provenance, and governance are uniform (P2):

```
Knowledge:
  spec:
    kindOfKnowledge  enum        # the category (§2)
    title            string
    body             Markdown     # human-readable content (vault form, §Ch03)
    relations        list<Relation>   # typed edges to other Knowledge/Resources (graph form, §Ch04)
    tags             list<string>
    scope            Ref<Workspace>   # tenancy (Book 11 §08)
  status:
    provenance       Provenance   # source(s), supporting Experiences, author (§3, Book 10 §Ch04)
    trust            TrustLevel   # how strongly to weigh this (§3)
    vaultRef / graphRef           # the two synchronized representations (§Ch05)
    lastReviewed     Timestamp
```

Two properties are mandatory on **every** unit and are the reason the taxonomy is trustworthy:
- **Provenance** (§3) — where this Knowledge came from and what supports it.
- **Trust level** (§3) — how strongly planning should weigh it.

Knowledge without provenance and a trust level MUST NOT be created (Book 10 §Ch04 §3); an unattributed, unweighted assertion is a liability, not an asset.

## 2. The categories

The taxonomy (from the mission's knowledge definition, Book 01) — each category is a `kindOfKnowledge`:

| Category | What it captures | Primary consumer |
|----------|------------------|------------------|
| **Facts** | Discrete asserted truths (a config value, a deadline, a preference). | Goal derivation, planning. |
| **Architecture** | How systems/projects are structured. | Planning technical work. |
| **Projects** | Ongoing work, its goals, state, and constraints. | Goal derivation, prioritization. |
| **Policies** | Human-readable statements of rules (the machine-checkable form is a `Policy` Resource, Book 11 §06). | Planning within constraints; authoring policies. |
| **Documentation** | Reference material for systems, APIs, domains. | Planning, capability grounding. |
| **Runbooks** | Step-by-step procedures for known operations. | Planning recurring operations; candidate Workflows (§4). |
| **Playbooks** | Higher-level strategies for classes of situations. | Approach selection. |
| **Meetings** | Records/decisions from human gatherings. | Context, provenance for decisions. |
| **People** | Individuals, roles, contact/authority (non-secret). | Attribution, approval routing (Book 11 §07). |
| **Organizations** | Orgs, teams, their conventions and relationships. | Planning within org norms. |
| **Capabilities** | What reusable Capabilities exist and mean (mirrors `Capability` Resources, Book 02 §Ch07). | Grounding plans in what can be done. |
| **Workflows** | Reusable plan templates (mirrors `Workflow` Resources). | Plan reuse. |
| **Experience** | Distilled lessons promoted from executions (Book 10 §Ch04). | The learning loop's durable output. |
| **Relationships** | Typed connections among any of the above. | Graph traversal, relevance (§Ch04). |

The taxonomy is **open**: Packages (§Ch07, Book 12) may introduce additional categories via knowledge packs, but core Knowledge uses these.

## 3. Provenance and trust are first-class

Because Knowledge is fed to planning (§Ch06) and accrues from both humans and machines (§Ch01 §4), **not all Knowledge is equally reliable**, and the system must represent that explicitly:

- **Provenance** records the source(s): a human author (by identity, Book 11 §08), the supporting Experiences (by reference, Book 10 §Ch04 §3), the time, and the derivation. Provenance makes Knowledge **auditable** (P10) and **revisable** (Book 10 §Ch04 §5) — you cannot correct what you cannot trace.
- **Trust level** is a first-class weighting (e.g. `Authoritative` human-asserted · `Corroborated` multi-source · `Inferred` single-Experience · `Unverified`). Planning weighs Knowledge by trust (§Ch06 §3); a low-trust inference informs but does not override an authoritative fact.

These are not metadata afterthoughts — they are what make a *pile of assertions* into a *usable knowledge base*. Retrieval (§Ch06), promotion (Book 10 §Ch04), and contradiction-handling (§Ch05) all key on provenance and trust.

## 4. Knowledge that mirrors executable Resources

Several categories (Capabilities, Workflows, Policies) have **executable counterparts** elsewhere in the spec:
- A **Capability** Knowledge unit *describes* what a `Capability` Resource (Book 02 §Ch07) is and when to use it; the executable Capability is grounded in the Capability Manager (Book 03 §Ch06).
- A **Workflow** Knowledge unit *documents* a reusable `Workflow` (High-IR template, Book 02 §Ch07).
- A **Policy** Knowledge unit is the *human-readable* rendering of a machine-checkable `Policy` (Book 11 §06).

The Knowledge form is the *understanding*; the Resource form is the *mechanism*. They reference each other (by id) and MUST stay consistent — a runbook promoted to a reusable Workflow (Book 10 §Ch04 §2) links its Knowledge and Workflow forms. This keeps "what the system knows about X" and "the executable X" from drifting apart.

## 5. Tenancy and governance

- Every Knowledge unit is **workspace-scoped** (`scope`, Book 11 §08 §5); Knowledge does not cross tenants without an explicit grant (Book 02 §Ch06 §5).
- What Knowledge may be created, by whom, and at what trust — especially machine-promoted Knowledge — is **policy-governed** (P9, Book 11 §06, Book 10 §Ch04 §4).
- All Knowledge is **secret-free** (§Ch01 §6): no category ever holds a secret value.

## 6. Invariants (normative summary)

1. Every Knowledge unit shares a common envelope and carries mandatory provenance and trust level; Knowledge without them is not created.
2. The core taxonomy spans facts, architecture, projects, policies, documentation, runbooks, playbooks, meetings, people, organizations, capabilities, workflows, experience, and relationships; the taxonomy is open via knowledge packs.
3. Provenance (source, supporting Experiences, author, time) makes Knowledge auditable and revisable; trust level weights it in planning.
4. Knowledge categories with executable counterparts (Capability/Workflow/Policy) reference and stay consistent with their Resource forms; Knowledge is the understanding, the Resource is the mechanism.
5. Knowledge is workspace-scoped, policy-governed as to creation/trust, and secret-free in every category.
