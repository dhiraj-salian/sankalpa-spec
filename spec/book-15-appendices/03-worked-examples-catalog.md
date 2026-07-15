# Book 15 · Chapter 03 — Worked Examples Catalog

*Nature: **Informative** (a library of scenarios; the owning Books are authoritative).*

> This chapter catalogs representative scenarios that exercise the platform across planners, runtimes, and features. Each is a pointer to where it is worked in detail plus a note on what it demonstrates. It is a reading aid — a way to find "an example of X" — not new normative material. The scenarios grow as the specification and ecosystem mature.

## 1. The canonical example (worked in full)

- **"Monday sales email"** — Book 15 §Ch01 (full end-to-end) and Book 04 §Ch10 A (IR detail). Demonstrates the *entire* loop: channel → intent → goals → High IR → policy (allow-with-guard/approval) → Low IR (retry/compensation/binding sites) → runtime selection (fidelity gate) → execution (secret materialization, approval) → Experience → Knowledge. **Start here.**

## 2. Feature-focused examples

| Scenario | Demonstrates | Worked in |
|----------|--------------|-----------|
| **Ticket classification** | Determinization: a stable `Reasoning` node folded into a deterministic Capability (P13). | Book 04 §Ch10 B, Book 10 §Ch06, Book 05 §Ch06 |
| **Verification rejections** | The four ways bad IR is stopped before execution (secret-into-reasoning, undeclared effect, retry-on-non-idempotent, type mismatch). | Book 04 §Ch10 C, Book 04 §Ch08 |
| **Policy denial** | A plan forbidden at compile-time with an explainable diagnostic. | Book 05 §Ch04 §4 |
| **Runtime portability** | The same Low IR run on two runtimes with equivalent observable results. | Book 06 §Ch04 §6, Book 05 §Ch05 §4 |
| **Secret materialization** | A secret as reference throughout, value only transiently in the runtime. | Book 11 §Ch04, Book 06 §Ch06 |
| **Credential acquisition** | Out-of-band one-time page / OAuth into the Broker. | Book 11 §Ch05 |
| **Approval flow** | High-consequence effect gated on non-repudiable human authorization at the runtime checkpoint. | Book 11 §Ch07, Book 14 §Ch06, Book 13 §Ch06 |
| **Writing a controller** | The reconcile/finalize loop and the common-mistakes table. | Book 07 §Ch06 |
| **Knowledge sync** | Bidirectional vault ⇄ graph reconciliation with conflict resolution. | Book 09 §Ch05 |
| **Package install** | Reconciled install with least-privilege capability grants and rollback. | Book 12 §Ch05 |
| **MCP interop** | External tool wrapped as a typed governed Capability. | Book 13 §Ch07 |

## 3. Cross-cutting reference runs

Illustrative, open-ended mappings (each names *no* runtime/planner in core, P11):
- **Planner variety** (Book 08 §Ch06): the same Goals planned by a graph framework, an agent SDK, a multi-agent crew, and a rule engine — all emitting verified High IR; the rule engine yielding a fully-deterministic pipeline.
- **Runtime variety** (Book 06 §Ch05): the same Low IR targeting a durable-workflow engine, a container orchestrator, an edge/function platform, an n8n-class engine, and Bash — each honoring exactly what it `supports`.

## 4. How to use this catalog

- Looking for *how a feature works end-to-end*? Find it here, follow the pointer to its worked location.
- Building a *test corpus* (Phase 2+)? These scenarios are the seed set for conformance and integration tests (the conformance suites, Book 04 §Ch08, Book 06 §Ch07, Book 08 §Ch07).
- Onboarding? Read §Ch01 (the capstone), then the feature examples relevant to your area.

## 5. Summary (informative)

The catalog is a map from "an example of X" to where X is worked in the specification. The canonical Monday-sales-email run (§Ch01) is the spine; the feature examples exercise individual mechanisms; the reference runs demonstrate planner/runtime variety. All point back to their owning Books for the authoritative detail.
