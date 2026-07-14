# Book 05 · Chapter 04 — Policy-Validation Pass

*Nature: **Normative**. · Reflects: RFC-0001; realizes principle P9 (and P1, P7, P8). Companion to Book 11 §06 (Policy Engine), Book 14 §06 (runtime checkpoint).*

> This pass is where principle **P9 — everything is governed by policy** is enforced *before execution*. It runs on the optimized High IR, consumes the declared effects (Book 04 §Ch06), and either annotates the plan with required guards or rejects it with an explainable diagnostic. A plan that fails this pass is **never lowered or executed**.

## 1. Why policy validates here

Two facts make High IR the right place for pre-execution governance:
- **It is the human-meaningful artifact.** High IR expresses intent and *declared effects* in terms a person and a policy can reason about ("this plan sends external email, spends CRM budget, reads customer PII") — before it is buried under execution mechanics. A rejection here is explainable (§4); a rejection at raw execution would be opaque.
- **It is post-optimization, pre-lowering.** Optimization has already removed dead effects (Ch 03), so policy validates the plan's *true* effect set; and because lowering has not run, rejecting costs nothing downstream (Ch 01 §2).

This is the **pre-execution** P9 checkpoint. It is complemented by the Kernel API control-plane policy (Book 03 §Ch02 §3) upstream and the **runtime** policy checkpoint (Book 14 §06) downstream — defense in depth, because some facts (live budget, current approval state) are only known at execution.

## 2. Inputs and the policy model

The pass evaluates the module against the applicable **Policy** Resources (Book 02 §Ch07, Book 11 §06) for the workspace/context. It consumes:
- the module's **effect set** and per-node effects (Book 04 §Ch06) — the primary subject of policy;
- **types** and capability/secret references (Book 04 §Ch05, §Ch06) — e.g. "no Capability tagged `pii-export`", "no `SecretRef` in class `payments` without approval";
- **cost** estimates (Ch 03 §3) — e.g. budget ceilings;
- non-secret **context** (workspace, submitter role) — never secret material (P7).

Policies are machine-checkable rules over these inputs. They are Resources (versioned, P6) and MAY be shipped as Packages (Book 12); the policy *version* participates in the compilation cache key (Ch 08) so a policy change re-validates affected plans.

## 3. Three outcomes

For each applicable policy, evaluation yields one of:

1. **Allow** — the plan satisfies the policy; nothing changes.
2. **Allow-with-guard** — the plan is permitted *iff* a guard is inserted. The pass **annotates** the module so that lowering (Ch 05) materializes the guard — e.g. an **Approval** instruction before an external `Network(write)` (Book 04 §Ch10 A.4), a budget check, or a redaction step. The pass does not itself rewrite execution mechanics; it records the obligation for lowering.
3. **Deny** — the plan violates the policy. Compilation **fails** with a policy diagnostic (§4). The module is not lowered.

Conflicting policies are resolved by the Policy Engine's precedence rules (Book 11 §06); an unresolved conflict is a **deny** (fail-closed, Book 03 §Ch13 §1) — governance never defaults to permissive.

## 4. Explainable diagnostics

A deny (or a required guard) **MUST** produce a structured, secret-free diagnostic (Ch 07): the policy id and version, the offending node/effect (by canonical id), a human explanation, and — where possible — what would make the plan compliant. Example: *"Denied by policy `P-no-external-email/v3`: node n3 declares Network(write,'email') to an external domain; this workspace forbids external email. Route through an approved internal relay Capability to comply."*

Rationale: governance that says only "no" teaches nothing; an explainable diagnostic lets a planner (or its author) adjust and lets a human understand the guardrail. Diagnostics are deterministic (same plan+policy ⇒ same diagnostic) and secret-free even when reporting on secret-referencing nodes (P7).

## 5. The pass is not verification, and not the only checkpoint

- **Not verification.** Verification (Book 04 §Ch08) checks *well-formedness* (is this valid IR?); this pass checks *permission* (is this allowed here?). They are separate and run in order: policy validates already-verified IR. A plan can be perfectly valid yet forbidden.
- **Not the only checkpoint.** Because policy can only see what is statically known, the **runtime checkpoint** (Book 14 §06) re-evaluates policy against live conditions immediately before execution (current budget, live approval decision, runtime health). A plan that passed here may still be stopped there if conditions changed — and, being fail-closed, is stopped rather than run on stale permission.

## 6. Security coupling (P7, P8)

- The pass makes secret **use** visible to policy without exposing secret **values**: it reasons over `SecretUse(ref)` effects (Book 04 §Ch06 §6), enabling rules like "materializing a `payments`-class secret requires an Approval" — all by reference (P7).
- It enforces capability-scoped rules (P8): a plan invoking a Capability the submitter is not entitled to is denied, coordinating with the Capability Manager (Book 03 §Ch06).

## 7. Invariants (normative summary)

1. Policy validation runs on optimized High IR, before lowering; a denied plan is never lowered or executed (P9).
2. It evaluates versioned Policy Resources over the plan's declared effects, types, references, cost, and non-secret context — never over secret values (P7).
3. Outcomes are allow / allow-with-guard (annotate for lowering) / deny; unresolved policy conflicts deny (fail-closed).
4. Denials and required guards produce deterministic, secret-free, explainable diagnostics.
5. Policy validation (permission) is distinct from verification (well-formedness) and is complemented by control-plane and runtime checkpoints; the runtime checkpoint re-checks live conditions.
6. Secret use is governed by reference; capability entitlements are enforced in coordination with the Capability Manager.
