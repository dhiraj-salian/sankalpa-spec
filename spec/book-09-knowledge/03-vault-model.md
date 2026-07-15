# Book 09 · Chapter 03 — The Vault Model

*Nature: **Normative**. · Reflects: ADR-0001; realizes principles P2, P6, P7, P11. Companion to §Ch04 (graph), §Ch05 (sync).*

> The vault is Knowledge's **human face**: an Obsidian-compatible Markdown vault humans can read, author, and curate directly. This chapter specifies the vault's structure, conventions, and the rule that it is an *authoritative representation* of Knowledge, not a disposable export. Choosing an open, human-native format is a deliberate bet on human curation as a first-class source of Knowledge (§Ch01 §4).

## 1. Why an Obsidian-compatible Markdown vault

Knowledge is only as good as humans' willingness to maintain it. Sankalpa therefore stores the human-readable representation in a format humans already use to think: **Markdown files with wiki-style links**, compatible with **Obsidian** (and the broader Markdown-vault ecosystem). Rationale:

- **Human-native.** People read, write, and link Markdown without training; curation is not gated behind a bespoke UI.
- **Open and portable (P11, ADR-0001).** The vault is plain files, not a proprietary store. It can be edited in Obsidian, an editor, or the Web Runtime (Book 13 §01), and it survives independently of any one tool.
- **Link-structured.** Wiki-links (`[[...]]`) map naturally to the relationships (§Ch04) that make Knowledge a graph, so the human structure and the machine structure align (§Ch05).

The vault is a *representation* of `Knowledge` Resources (Book 02 §Ch07), kept synchronized with the graph (§Ch05) — not a separate, ungoverned pile of files.

## 2. Vault structure

Each `Knowledge` unit corresponds to a Markdown **note** with front-matter and body:

```markdown
---
id: <Knowledge Resource id>          # binds the note to its ARM Resource (immutable)
kindOfKnowledge: runbook             # the taxonomy category (§Ch02)
title: Deploy the reporting service
scope: ws-acme                       # workspace (tenancy, Book 11 §08)
trust: Corroborated                  # trust level (§Ch02 §3)
provenance:                          # source(s) + supporting Experiences (by ref)
  author: user/01J9...
  experiences: [exp/01K2..., exp/01K5...]
tags: [deploy, reporting]
lastReviewed: 2026-07-10
---

# Deploy the reporting service

Steps and context in Markdown, linking related knowledge:
see [[reporting-service-architecture]] and [[acme-deploy-policy]].
```

Rules:
- **Front-matter is the Resource binding.** `id` binds the note to its `Knowledge` Resource; front-matter carries the structured fields (kind, scope, trust, provenance) that the graph and governance rely on. A note without valid front-matter is not a governed Knowledge unit.
- **Body is the human content.** Free Markdown, for people.
- **Links are relationships.** `[[wiki-links]]` express relations (§Ch04); they are extracted into typed graph edges on sync (§Ch05).
- **Organization** (folders, naming) SHOULD follow the workspace's conventions; the taxonomy `kindOfKnowledge`, not folder location, is authoritative for category.

## 3. The vault is authoritative, not an export

A central rule: **the vault is an authoritative representation of Knowledge, co-equal with the graph** (§Ch05) — not a read-only export of the graph, and not a scratch space the system ignores. Consequences:
- A human editing the vault **is editing Knowledge**; the change is synchronized into the graph (§Ch05) and governed like any Knowledge change (§5).
- The vault therefore participates fully in provenance (a human edit is an authored change, Book 11 §08) and versioning (P6).
- Neither representation is "the real one" — both are authoritative views of the same Knowledge, and §Ch05 keeps them consistent.

This co-equality is what makes human curation first-class: a person's edits are not second-class annotations on a machine-owned graph; they are Knowledge.

## 4. Human authoring and the Web Runtime

- Humans MAY author/edit the vault through Obsidian directly, or through the **Web Runtime** (Book 13 §01), which renders and edits the vault as part of the integrated web layer. Either path produces a governed Knowledge change (§5).
- Rich Knowledge experiences (browsing, search, graph views) are surfaced through the Web Runtime as URLs (Book 13 §01), consistent with the platform's "return URLs for rich experiences" pattern.

## 5. Governance, tenancy, and secret-freedom

- **Tenancy.** Each note's `scope` places it in a workspace (Book 11 §08 §5); the vault is partitioned by tenancy, and cross-tenant linking requires an explicit grant (Book 02 §Ch06 §5).
- **Governance.** Vault edits are Knowledge changes subject to policy (P9, Book 11 §06) — e.g. who may assert `Authoritative`-trust facts. Machine-promoted notes (Book 10 §Ch04) carry their Experience provenance in front-matter.
- **Secret-freedom (P7).** The vault MUST NOT contain secret values — it is a human-editable surface that is also fed to planning (§Ch06), doubly requiring the guarantee. Credential-shaped content is never stored here; secrets live only in the Broker (Book 11 §04). A note may reference a secret by class ("uses the `payments` credential"), never by value.

## 6. Invariants (normative summary)

1. The human-readable representation of Knowledge is an Obsidian-compatible Markdown vault: open, portable, human-native, link-structured (P11).
2. Each Knowledge unit is a note whose front-matter binds it to its `Knowledge` Resource `id` and carries structured fields (kind, scope, trust, provenance); the body is human Markdown; `[[links]]` are relationships.
3. The vault is an authoritative representation co-equal with the graph — not an export; human edits are governed Knowledge changes, synchronized into the graph (§Ch05).
4. Humans author via Obsidian or the Web Runtime; rich Knowledge experiences are surfaced as Web Runtime URLs.
5. The vault is workspace-scoped, policy-governed, and secret-free; notes reference secrets only by class, never by value (P7).
