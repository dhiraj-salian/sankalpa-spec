# Book 02 · Chapter 07 — Core Resource Catalog

*Nature: **Normative** (the catalog is authoritative for which kinds exist and who owns them; each kind's full schema is specified in its owning Book).*
*Reflects: all foundational principles; each row realizes P2/P3/P5.*

> This chapter enumerates the **core** Resource kinds — the ones the Kernel ships with. Each entry fixes the kind's identity, its owning Book/Controller, its lifecycle style, and the shape of its `spec`/`status`. Full schemas live in the owning Book; this catalog is the index of record so cross-references and the [Glossary](../../GLOSSARY.md) stay consistent. Third-party kinds are added as Packages (Book 12) and are **not** listed here.

## 1. Conventions

- **Group:** core kinds live in the `core.sankalpa.dev` group unless noted. Subsystems MAY use sub-groups (e.g. `ir.sankalpa.dev`, `pkg.sankalpa.dev`).
- **Lifecycle style:** *Long-lived* (steady state `Active`) or *One-shot* (terminal `Succeeded`/`Failed`), per Ch 04.
- **Owner:** the Book specifying the schema and the Controller reconciling the kind.

## 2. The intent chain

| Kind | Lifecycle | Owner | `spec` (essence) | `status.actualState` (essence) |
|------|-----------|-------|------------------|-------------------------------|
| `Mission` | Long-lived | Book 01/03 | The enduring purpose of a Workspace. | Derived strategies; health. |
| `Strategy` | Long-lived | Book 01/03 | How the Mission is pursued; constraints, priorities. | Active intents advancing it. |
| `Intent` | One-shot* | Book 08 | `utterance` (natural language), channel, submitter. | Derived Goals; clarification state. |
| `Goal` | One-shot* | Book 08 | Structured objective refined from Intent; success criteria. | Plan/IR produced; progress. |

\* *Intent/Goal are "one-shot" in that they converge to a plan; recurring intents (e.g. "every Monday") are modeled as a long-lived Intent that spawns per-run Goals.*

## 3. The compilation & execution chain

| Kind | Group | Lifecycle | Owner | `spec` | `status.actualState` |
|------|-------|-----------|-------|--------|----------------------|
| `IRModule` | `ir` | One-shot | Book 04/05 | The IR body + `level: High\|Low`, IR schema version, content hash. | Verification result; derived module. |
| `Compilation` | `ir` | One-shot | Book 05 | Input IRModule ref; requested passes; target constraints. | Passes run; diagnostics; output refs. |
| `RuntimeGraph` | `ir` | One-shot | Book 05/06 | Lowered graph; target runtime; secret refs (by reference). | Chosen runtime; validation result. |
| `Execution` | `core` | One-shot | Book 06 | RuntimeGraph ref; runtime ref; inputs; secret refs. | Progress, metrics, terminal outcome; **reasoning ledger** (recorded reasoning/`Time`/`Random` outputs, secret-free, RFC-0002). |
| `Experience` | `core` | Long-lived† | Book 10 | (system-produced) the full record of one Execution. | Lessons; knowledge updates; generated capabilities. |
| `RemediationTask` | `core` | One-shot | Book 06 | (system-produced on `CompensationFailed`, RFC-0004) refs to the `Failed` Execution, affected effects, and residual external state (by reference, secret-free). | Reconciliation outcomes; acknowledgment record (two-party for high-consequence residuals); `open → acknowledged → resolved`, non-droppable until audited resolution. |

† *Experience outlives the Execution that produced it (Ch 04 §5) and is retained for learning/audit.*

*`RemediationTask` is RFC-0004-gated (P12): it is the durable, non-droppable operator surface for a compensation failure's residual inconsistency. Its full schema is specified in Book 06; escalation and two-party resolution semantics are in Book 06 §Ch03 §4.*

## 4. Capabilities, workflows, and reuse

| Kind | Lifecycle | Owner | `spec` | `status.actualState` |
|------|-----------|-------|--------|----------------------|
| `Capability` | Long-lived | Book 03/05 | A reusable deterministic unit: typed inputs/outputs, declared effects, implementation ref. | Availability; version; provenance (hand-authored vs. determinized). |
| `Workflow` | Long-lived | Book 04 | A named, reusable High-IR template parameterized for reuse. | Instantiations; validity. |
| `Project` | Long-lived | Book 13 | A unit of work grouping intents, artifacts, hosting. | Contained resources; hosting state. |
| `Workspace` | Long-lived | Book 03/11 | A tenancy boundary; quotas; default policies. | Membership; resource counts. |

## 5. Extension & platform kinds

| Kind | Group | Lifecycle | Owner | `spec` | `status.actualState` |
|------|-------|-----------|-------|--------|----------------------|
| `Planner` | `core` | Long-lived | Book 08 | Registered planner plugin; AEP conformance; version. | Health; loaded state. |
| `Compiler` | `core` | Long-lived | Book 05 | Registered compiler/backend plugin. | Health; supported targets. |
| `Runtime` | `core` | Long-lived | Book 06 | Registered runtime plugin; capabilities; cost profile. | Health; capacity. |
| `Plugin` | `core` | Long-lived | Book 03 | A loaded extension of any class; isolation config. | Load/health state. |
| `Package` | `pkg` | Long-lived | Book 12 | Manifest, version, contents, signature. | Install state; provided kinds/interfaces. |
| `Policy` | `core` | Long-lived | Book 11 | Machine-checkable rules; scope. | Enforcement status; violations observed. |
| `Secret` | `core` | Long-lived | Book 11 | **Reference metadata only** (how the Broker resolves it); **no value** (P7, Ch 06 §4). | Availability; last rotation; never the value. |

## 6. Interaction & knowledge kinds

| Kind | Lifecycle | Owner | `spec` | `status.actualState` |
|------|-----------|-------|--------|----------------------|
| `Conversation` | Long-lived | Book 13 | A channel-spanning dialogue; participants; correlation. | Turns; linked intents. |
| `Session` | One-shot | Book 11/13 | An authenticated interaction context; identity; scope. | Activity; expiry. |
| `Artifact` | Long-lived | Book 13 | A produced output (doc, site, file); reference to content; content **hash**. | Hosting/availability; version. |
| `Service` | Long-lived | Book 13 | A long-running exposed capability (e.g. a hosted API/site). | Endpoints; health. |
| `Knowledge` | Long-lived | Book 09 | A unit of curated knowledge (fact/runbook/relationship…); vault + graph refs. | Sync state (vault ⇄ graph); provenance/trust. |

## 7. Rules for the catalog

- Every core kind above **MUST** obey the envelope (Ch 02), the desired/actual contract (Ch 03), the lifecycle model (Ch 04), and the versioning rules (Ch 05).
- Adding, renaming, or removing a **core** kind is a breaking, RFC-gated change (P12); the [Glossary](../../GLOSSARY.md) and this catalog are updated in the same change.
- A kind **MUST NOT** duplicate another's responsibility. If two kinds seem to overlap, the design is wrong and must be resolved before either ships (conceptual integrity, ADR-0001).
- `Secret` is the one kind whose `spec`/`status` are *constrained to hold no payload* (P7); this constraint is enforced by its schema and by admission (Book 11).

## 8. Invariants (normative summary)

1. The set of *core* kinds is exactly this catalog; changes are RFC-gated and reflected here and in the Glossary.
2. Every core kind conforms to Chapters 02–05.
3. Ownership/dependency edges among these kinds match Ch 06 §3 and are kept referentially consistent.
4. `Secret` carries reference metadata only; no core kind's `spec`/`status`/Events ever hold secret values (P7).
5. Third-party kinds arrive only via Packages and are never added to this core catalog.
