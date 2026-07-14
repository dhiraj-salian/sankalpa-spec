# Book 01 · Chapter 04 — Foundational Principles

*Nature: **Normative**. · Reflects: ADR-0001, ADR-0002, RFC-0001, SECURITY.md.*

> This chapter states the invariant principles of Sankalpa. Every other Book, RFC, ADR, and AEP is subordinate to these. A design that violates a principle here is rejected on that basis alone; changing a principle requires an RFC, a governance FCP, and a Steering Council supermajority.

## 1. Purpose

Long-lived systems keep their shape because a small set of principles is treated as non-negotiable. These principles are the load-bearing walls of Sankalpa. They are deliberately few, so they can be remembered; deliberately strict, so they can be enforced; and deliberately justified, so they can be defended.

Each principle below is stated as a **MUST**, followed by its **rationale** and its **enforcement point** (where in the architecture the principle is made true, not merely hoped for).

## 2. The principles

### P1 — Natural language is never executable
Natural language, prompts, and model outputs **MUST NOT** be executed directly. The **only** executable representation is AOS IR.
*Rationale:* determinism, static policy validation, portability, and auditability are impossible over free-form text. *Enforcement:* the compiler pipeline (Book 05) is the sole path to execution; runtimes accept only lowered Low IR (RFC-0001).

### P2 — Everything is a Resource
Every entity in the system — Mission, Intent, Goal, Capability, Workflow, Runtime, Compiler, Planner, Package, Policy, Execution, Conversation, Secret reference, and the rest — **MUST** be modeled as an ARM Resource with Metadata, Spec, Status, Desired State, Actual State, Version, Lifecycle, Controller, and Events.
*Rationale:* one uniform model yields uniform tooling, observability, versioning, and governance. *Enforcement:* the Resource Manager (Book 03) admits only well-formed Resources (Book 02).

### P3 — Everything has a lifecycle and a controller
Every Resource **MUST** have an explicit lifecycle and a Controller that reconciles Actual State toward Desired State.
*Rationale:* declarative desired-state + reconciliation is how the system stays correct under partial failure. *Enforcement:* the Controller Runtime (Book 07).

### P4 — The Kernel mediates all communication
Components **MUST NOT** communicate directly. All interaction flows through the **Kernel API** (synchronous) or the **Event Bus** (asynchronous).
*Rationale:* a single mediation point is the only place global invariants (policy, secrets, audit) can be enforced and the only way plugins stay decoupled and replaceable. *Enforcement:* the microkernel boundary (ADR-0002, Book 03).

### P5 — Everything emits events
Every state change **MUST** emit an Event on the Event Bus.
*Rationale:* observability, auditability, event-sourced reconstruction, and Experience capture all depend on a complete event stream. *Enforcement:* Resource Manager and Controller Runtime emit on every transition (Books 03, 14).

### P6 — Everything is versioned
Resources, IR, APIs, and Packages **MUST** be versioned; history **MUST NOT** be silently rewritten.
*Rationale:* reproducibility, migration, and honest evolution. *Enforcement:* versioning rules in Book 02 and [`../../process/versioning-and-stability.md`](../../process/versioning-and-stability.md).

### P7 — Secrets flow only by reference
Secret **values** **MUST NOT** enter planner context, prompts, logs, memory, vector stores, or IR. Secrets **MUST** be delivered to Execution by reference via the Secret Broker.
*Rationale:* the single most consequential leak surface in an AI system is reasoning context; closing it is non-negotiable. *Enforcement:* the Secret Broker and the IR effect/reference model (Book 11, RFC-0001).

### P8 — No ambient authority
A component **MUST** be able to perform only what its granted Capabilities permit. There is no implicit, ambient authority.
*Rationale:* capability-based security contains the blast radius of any bug or malicious plugin. *Enforcement:* the Capability Manager and Security Manager (Book 11).

### P9 — Everything is governed by policy
Plans **MUST** be validated by the Policy Engine before compilation, and actions **MUST** be validated before execution.
*Rationale:* governance must be mechanical and pre-emptive, not advisory and after-the-fact. *Enforcement:* policy-validation pass in the compiler (Book 05) and the runtime policy checkpoint (Book 14).

### P10 — Everything is observable
Every Resource and Execution **MUST** be inspectable through Events, metrics, and traces.
*Rationale:* you cannot operate, debug, or improve what you cannot see. *Enforcement:* the Observability Manager (Book 14).

### P11 — Everything is extensible and replaceable
Planners, runtimes, compilers, channels, and providers **MUST** be plugins behind AEP-defined interfaces; no such component may be irreplaceable.
*Rationale:* a decade-scale platform cannot bet on today's tools; it must outlive them. *Enforcement:* the Plugin/Package Managers and the AEP process (Books 03, 12).

### P12 — Everything is documented, testable, and reasoned
No normative behavior exists without specification, a testing strategy, and recorded rationale (RFC/ADR).
*Rationale:* specification-first (ADR-0001) is only real if it is enforced per change. *Enforcement:* the review gates ([`../../process/review-gates.md`](../../process/review-gates.md)).

### P13 — The system tends toward determinism
When repeated non-deterministic reasoning is observed, the system **SHOULD** convert it into a reusable deterministic Capability (Determinization).
*Rationale:* this is the platform's compounding advantage — every solved problem becomes cheaper and more reliable next time. *Enforcement:* the Experience Engine and determinization passes (Books 10, 05).

## 3. Precedence and conflict

When principles appear to conflict, precedence is: **security invariants (P1, P7, P8, P9) > correctness (P2–P6, P10) > extensibility (P11) > optimization (P13).** A design **MUST NOT** trade a higher-precedence principle for a lower one. Any apparent need to do so is a signal to redesign, and MUST be raised as an RFC.

## 4. Relationship to the rest of the spec

Every subsequent Book cites the principles it enforces. A reviewer's first question for any change is: *which principles does this touch, and does it uphold every one?* The [security gate](../../process/review-gates.md) checks P1, P7, P8, P9 explicitly; the architecture gate checks the rest.

## 5. Non-goals of this chapter

This chapter does not specify *mechanisms* — how the Secret Broker works, what the IR looks like, how reconciliation runs. Those are the subjects of later Books. Here we fix only the invariants those mechanisms must satisfy.
