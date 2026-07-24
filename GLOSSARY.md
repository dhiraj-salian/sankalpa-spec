# Glossary

*Status: Living document. Every term capitalized as a proper noun in the specification MUST be defined here. Add terms in the same PR that introduces them.*

Terms are grouped for reading but should be kept alphabetized within each group.

## Core loop

- **Intent** — A human-expressed desired outcome, in natural language. Non-executable. The top of the transformation chain.
- **Goal** — A structured, machine-derived objective refined from Intent. Still declarative, not yet a plan.
- **Plan** — An ordered/branching structure of steps that, if executed, achieves the Goals. Produced by a Planner.
- **AOS IR** — *Agent Operating System Intermediate Representation.* The **only** executable representation in Sankalpa. Has a High level (planner-facing, runtime-agnostic) and a Low level (compiler-produced, closer to execution).
- **Execution** — The runtime-specific realization of Low IR; produces Events.
- **Experience** — The first-class record of one Execution: its Intent, Goals, IR, runtime, metrics, outcome, feedback, and lessons. Feeds Knowledge.
- **Knowledge** — Curated, durable understanding (facts, architecture, policies, runbooks, relationships) that improves future Planning. *Not* short-term memory.
- **Determinization** — The process of converting repeated non-deterministic reasoning into a reusable, deterministic Capability.

## Compiler correctness

- **Dead-effect witness** — The dataflow fact a pass must emit to justify removing an effect: evidence that no observable output depends on it. No witness, no removal. Turns an undecidable "is this really dead?" into a checkable artifact. (RFC-0011)
- **Effect-graph conservation** — The mandatory comparative check of a transform's output against its input: the multiset of externally-visible effects, their observable order, and each effect's **target identity** are conserved, modulo the pass's declared effect-refinement relation. The first check in the pipeline that consults the original module. Necessary, not sufficient. (RFC-0011)
- **Equivalence certificate** — The per-compilation proof a `validated: true` pass emits for *this* input/output pair, checked by a small trusted validator in the core. (RFC-0011)
- **Translation validation** — Validating one concrete rewrite rather than a pass in general — decidable per instance, unlike full equivalence. Required for any pass running after an `Approval` is lowered into the plan. (RFC-0011)

## Observability egress

- **Materialized-value digest set** — The per-Execution set of keyed digests `HMAC(k_E, value)` of every secret materialized into it, held by the Broker/Runtime Manager under a per-execution key the runtime never sees. Keyed (not a bare hash) so it is not an offline-guessable oracle; the substrate the egress check compares against. (RFC-0010)
- **Observability egress verification** — The Runtime Manager's check of a runtime's `RuntimeEvent`s, logs, traces, and metrics against the materialized-value digest set before any sink. It exists because the runtime is simultaneously the sole holder of a plaintext secret and an untrusted plugin, so the Event stream is not "already secret-free" upstream of it. Catches verbatim and declared-encoding emission; not a bound on a transforming or B4-exfiltrating runtime. (RFC-0010)

## Scheduling

- **Aging** — Raising a `Pending` unit's effective priority monotonically with wait time so it eventually reaches the front of the queue. One of the two mechanisms (with reservation) that make starvation-freedom real rather than asserted. (RFC-0007)
- **DeadlineExceeded** — The terminal condition on `Failed` for a unit whose `deadline` passed before run-admission, or before completion for an execution-spanning deadline. Fail-closed and explained; compensation runs where effects were already produced. (RFC-0007)
- **Run-admission** — The Scheduler's decision to advance a unit from `Pending` to `Progressing`. Distinct from *store-admission* (the write succeeding, which is what `Pending` records); the gap between them is bounded and visible, never a silent stall. (RFC-0007)
- **Scheduling class** — A named priority class on schedulable work, mapped to a priority by workspace policy and capability-gated so priority is governed, not self-asserted. (RFC-0007)
- **Shed (work)** — The terminal condition on `Failed` for a unit that cannot be run-admitted within its deadline or backpressure budget — the async-work counterpart of the Kernel API's typed `Busy`/`Unavailable`. (RFC-0007)

## Secrets & determinism

- **Intra-execution secret stability** — The rule that a `SecretRef` resolves to a single value for an Execution's lifetime, keyed `(SecretRef, grantedBy)`, so "the same resolved bindings" holds for a plan that uses one credential twice. The memoization caches the *value*, never the *authority*: every `SecretUse` re-checks capability and policy, so revocation still bites "even past any caching". (RFC-0005)
- **Rotation generation** — An opaque, monotonic version counter on a `Secret`, incremented on rotation or revocation. Carries no value material (it is not a hash of the value), so it is safe in audit, Events, and `Execution` status. Lets re-execution replay detect that a secret changed since the recorded run and fail closed. (RFC-0005)
- **Secret carve-out (determinism)** — The acknowledgement that a materialized secret is a *resolved binding* the platform can never record (P7 forbids it), so the determinism guarantee holds *modulo* materialized secret values: pinned within an Execution, not across one. Rotation therefore takes effect at Execution boundaries; revocation takes effect immediately. (RFC-0005)

## Channel identity

- **Assurance level** — How strongly a channel proves the sender's identity *per message*: a fixed total order `low` < `medium` < `high` (bare identifier / authenticated account with SSO / Web Runtime OIDC or per-request token). A Session's effective authority is capped by its channel leg's assurance. Deliberately a total order, not a lattice, so "minimum assurance" is decidable. (RFC-0009)
- **Channel binding** — A verified, workspace-scoped, revocable `ChannelBinding` Resource tying a channel-native identifier (Telegram id, email address, phone number) to a `User`. A channel-native identifier carries authority only via such a binding; unbound, it resolves to no Session. (RFC-0009)
- **Enrollment** — Establishing a channel binding by proving control of the identifier from an already-authenticated context (a Web Runtime OIDC session issues a one-time challenge completed from the channel). The mechanism is a provider concern; the requirement is normative. (RFC-0009)
- **Step-up** — Requiring a consequential action to be decided at a higher assurance than the channel it arrived on, on the Web Runtime — which, because it authenticates the approver, already *is* the step-up surface. (RFC-0009)

## Supply chain

- **Authorized-against identity** — The verified identity a capability grant was made against: for a Package, its version, publisher identity, and signing-key/artifact identity at the moment of authorization. Authority is conveyed to the verified *code* that was authorized, not to a Package name in perpetuity. (RFC-0008)
- **Grant carry-forward** — Retaining a Package's existing grants across an upgrade without fresh consent, permitted only when the signing identity is unchanged and the declared capabilities are the same or narrower. Recorded and auditable. (RFC-0008)
- **Upgrade re-authorization** — Re-evaluating a Package's grants against the newly verified artifact before it becomes `Active`: broadened authority is surfaced for explicit authorization, and a signing-identity change re-authorizes sensitive-class capabilities even when the capability set is unchanged — the maintainer-compromise and package-takeover defense. (RFC-0008)

## Knowledge sync

- **Synced-base stamp** — The controller-managed `syncedFrom` front-matter field on a vault note, recording the `resourceVersion`/`generation` the note was last reconciled from. It is the merge base a three-way Knowledge reconcile needs, and exists because a direct/offline vault edit observes no `resourceVersion` and so cannot rely on optimistic concurrency alone. An absent, malformed, or unknown stamp means an *unknown* base, which is conflict-surfaced, never merged. (RFC-0006)
- **Three-way Knowledge reconcile** — Vault⇄graph synchronization of a changed note against three inputs — the synced base, the Resource's current state, and the edited note — merging one-sided changes and surfacing genuine both-sided conflicts, so a direct or offline vault edit cannot silently clobber a concurrent graph change. (RFC-0006)

## Execution & replay

- **Fresh execution** — An execution mode in which each `CapturedReasoning` node invokes the model and each `Time`/`Random` effect reads its live input; every such output is recorded to the reasoning ledger. Contrast **Replay-with-record**. (RFC-0002)
- **Reasoning ledger** — The ordered, append-only, content-addressed, secret-free record on an `Execution` of every `CapturedReasoning`/`Time`/`Random` output, keyed by `(instructionId, invocationIndex)`. The substrate replay is bound to and that determinization discovery and shadow drift detection are defined over. (RFC-0002)
- **Reconstruction replay** — The replay-with-record variant that performs **no** external effect: every effectful instruction's result is injected from the recorded outcome. Golden-file conformance and audit reconstruction are defined over it; it must be unable to cause any external effect. (RFC-0002)
- **Re-execution replay** — The replay-with-record variant that re-fires external effects live and therefore re-runs policy and approval checks against current state; never an approval-bypass path. (RFC-0002)
- **Replay-with-record** — Executing a recorded plan by injecting the reasoning ledger's recorded outputs instead of re-invoking the model (fail-closed on a ledger miss). Has two variants: **Reconstruction** and **Re-execution**. (RFC-0002)

## Determinization health

- **Drift** — Divergence of a determinized Capability's output from what current reasoning would produce (the world changed), measured by shadow sampling under the synthesis-time equivalence bound; sustained drift retires the Capability. (RFC-0003)
- **Re-validation window** — The period opened when a shadow source's version changes (e.g. a model upgrade), during which confidence re-accrues against the new version instead of the change being counted as drift. (RFC-0003)
- **Shadow sampling** — Running a determinized Capability's replaced reasoning on a policy-governed, non-blocking fraction of executions to measure drift, without affecting observable behavior. (RFC-0003)
- **Shadow source** — The identity (`shadowSource`) of the `Reasoning` node a determinized `CapabilityInvocation` replaced, recorded so the runtime can reconstruct and shadow-sample the original reasoning. (RFC-0003)
- **Version bucket** — A grouping of shadow observations by shadow-source version; retirement decisions aggregate only within a single bucket so a version change re-validates rather than retires. (RFC-0003)

## Failure & remediation

- **Compensation failure** — The condition where a mandated compensation itself fails after exhausting its bounded policy, leaving external state inconsistent; surfaces as a `Failed`/`CompensationFailed` Execution. (RFC-0004)
- **Remediation task** — The durable, non-droppable `RemediationTask` Resource raised on a compensation failure, carrying the residual inconsistency for operator reconciliation (two-party acknowledgment for high-consequence residuals). (RFC-0004)
- **Residual inconsistency** — The external state left inconsistent when compensation fails, described by reference (secret-free) in the `CompensationFailed` condition and its remediation task. (RFC-0004)

## Substrate

- **Kernel** — The microkernel that owns orchestration. Nothing bypasses it; components communicate only via the Kernel API or the Event Bus.
- **Kernel API** — The synchronous request/response contract for interacting with the Kernel and its managers.
- **Event Bus** — The asynchronous publish/subscribe backbone. Every state change emits an Event.
- **Controller** — A reconciliation loop that drives a Resource's Actual State toward its Desired State.
- **Controller Runtime** — The Kernel subsystem that hosts and schedules Controllers.
- **Manager** — A Kernel-owned service for a domain (Resource, Capability, Scheduler, Runtime, Compiler, Plugin, Package, Security, Knowledge, Experience, Policy, Lifecycle, User, Session, Approval, Secret Broker, Observability).

## Resource model

- **ARM** — *Agent Resource Model.* The uniform model in which *everything is a Resource*.
- **Resource** — A versioned, observable, controller-backed entity with Metadata, Spec, Status, Desired State, Actual State, Lifecycle, and Events.
- **Desired State / Actual State** — The declared target vs. the observed reality of a Resource; the gap the Controller closes.
- **Spec / Status / Metadata** — The declared intent, the observed condition, and the identifying/annotating data of a Resource (Kubernetes-derived convention).

## Compiler & extension

- **Planner** — A plugin that transforms Goals into High AOS IR. Never knows about specific runtimes. (e.g., LangGraph adapter.)
- **Compiler** — Optimizes High IR, validates against Policy, and lowers it to Low IR and then to runtime-specific graphs.
- **Runtime** — A plugin that executes runtime-specific graphs (e.g., n8n, Temporal, Kubernetes, Bash).
- **Capability** — A reusable, deterministic unit of behavior a Plan can invoke; the target of Determinization.
- **Policy** — A machine-checkable rule set enforced by the Policy Engine before compilation and before execution.
- **Package** — A distributable, versioned bundle (capabilities, compilers, policies, knowledge packs, templates, workflows, providers, UI, docs, tests, controllers).
- **Plugin** — A runtime-loaded extension conforming to an AEP-defined interface (planners, runtimes, compilers, channels are plugins).

## Security

- **Secret Broker** — The sole custodian of secrets; delivers them to Execution **by reference**, never by value into any reasoning context.
- **Capability-based security** — Authority granted as unforgeable references to specific capabilities; no ambient authority.
- **Approval Engine** — The subsystem that gates actions requiring human authorization.

## Interfaces

- **Channel** — A transport adapter (Telegram, Slack, Email, REST, CLI, Web UI, Voice, MCP, …). Channels carry, they do not decide.
- **Web Runtime** — The integrated web layer for dashboards, approvals, rendered artifacts, file browsing, reverse proxy, and the API gateway.

## Process

- **RFC / ADR / AEP** — Request for Comments / Architecture Decision Record / Architecture Extension Proposal. See [`process/`](process/README.md).
- **FCP** — Final Comment Period; the fixed window before a proposal is accepted or rejected.
- **Interim Review Process** — The bootstrap-period substitute for the full review quorum, used while a domain has no seated Domain Lead or Reviewers. Replaces the missing reviewers with a Compensating-Control package rather than waiving scrutiny. (RFC-0012)
- **Bootstrap Authority** — The temporary, self-limiting acceptance authority held by the founding maintainer for unstaffed domains under the Interim Review Process. Lapses automatically as domains are staffed; not a Domain Lead seat. (RFC-0012)
- **Compensating Control** — A required substitute for a waived reviewer under the Interim Review Process: a cooling-off period, an adversarial self-review, an independent adversarial pass, and provenance stamping. (RFC-0012)
- **Interim-Acceptance Ledger** — The standing queue of every artifact accepted under the Interim Review Process, each owing an independent re-review before v1.0. (RFC-0012)
- **Bootstrap-seated reviewer** — A Reviewer or Domain Lead seated solely by the founding maintainer's Bootstrap Authority, with no endorsement from an already-independent party. Such reviewers alone cannot clear a ledger entry or satisfy the v1.0 gate. (RFC-0012 §4.5)
- **Staffing-status note** — The dated note the founding maintainer MUST publish in the Interim-Acceptance Ledger at least every 90 days while Bootstrap Authority is in force, recording which domains remain unstaffed and what recruitment was attempted. (RFC-0012 §4.6)
- **Agent reviewer** — An AI system performing a cold, adversarial review of a finished artifact in a separate session, given only the artifact and its cited spec context and tasked to find concrete harm. Distinct from the author's own in-session AI pass; supplies review, never accountability, and never fills a governance seat. (RFC-0013 §4.1)
- **Agent review quorum** — ≥2 independent Agent reviewers (distinct sessions, distinct model families or a disclosed third, never the authoring session, full transcript recorded) with no unresolved blocking finding; equivalent to two independent reviewers for ledger clearance and the v1.0 gate, with disclosed provenance. High-stakes changes require ≥3 agents spanning ≥2 families. (RFC-0013 §4.4–4.5)
- **`cleared (agent)`** — The Interim-Acceptance Ledger status of an entry re-reviewed by an Agent review quorum: records the reviewers (model ids/versions), panel size, and transcript link. Upgradeable to `cleared (human)` by a later human review. (RFC-0013 §4.6)
