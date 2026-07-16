# Book 11 · Chapter 07 — The Approval Engine

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0009 (assurance step-up); realizes principles P9, P10 (and P7, P8). Companion to Book 05 §Ch04 (guards), Book 13 (Web Runtime), Book 14 §06 (runtime checkpoint).*

> Some actions are too consequential — irreversible, costly, or sensitive — to run on machine judgment alone. The Approval Engine gates such actions on **explicit human authorization**, captured non-repudiably. It is where "human-in-the-loop" becomes a first-class, governed mechanism rather than an ad-hoc pause.

## 1. When approval is required

Approval is required when **policy demands it** (Book 05 §Ch04 §3, Ch 06) or when **planning flags high-consequence ambiguity** (Book 08 §02 §4). Typical triggers:
- An **irreversible or high-cost effect** (`Network(write)` to an external party, a payment, a deletion) — especially where compensation (Book 04 §Ch04 §4) cannot fully undo it.
- **Materializing a sensitive-class secret** (e.g. `payments`) — policy requires an approval before the Broker resolves it (Ch 04 §4, Ch 06 §3).
- A **low-confidence plan** with consequential effects (Book 08 §03 §4).
- Any action a workspace's policy designates as approval-gated.

Approval requirements are declared upstream (by policy) and **materialized as an `Approval` instruction** in Low IR by lowering (Book 05 §Ch04 §3, Book 04 §Ch10 A.4). Approval is thus part of the governed plan, not a side pause.

## 2. The approval as a first-class Resource

An **`Approval`** is an ARM-modeled request/decision (Book 02): it records *what* is being authorized (the specific effect/instruction, by reference), *why* (the policy or planning trigger), *who* may decide (the authorized approver(s)), and the *decision* with its rationale and timestamp. Because it is a Resource, an approval is observable, versioned, and auditable like everything else (P10).

## 3. The approval flow

```
Policy/planning requires approval
   → lowering inserts an Approval instruction (Book 05 §Ch04)
   → at execution, the runtime checkpoint (Book 14 §06) reaches it
   → Approval Engine renders a request to the authorized human via
        a secure Web Runtime page / channel (Book 13 §06)
   → human reviews WHAT will happen (effects, targets, cost) and decides
   → decision (approve/deny, with rationale) recorded non-repudiably (§4)
   → approve → execution proceeds;  deny → execution stops (fail-closed), compensation runs
```

- The request presents the human with the **concrete consequence** — the declared effects, targets, and cost of the gated action (Book 04 §Ch06) — not an opaque "approve?". Informed consent requires showing *what* is being authorized.
- The request is rendered through the Web Runtime / channels (Book 13); like credential acquisition (Ch 05), **no secret value is shown** — the human authorizes the *use* of a `payments` secret by reference, never sees the secret (P7).
- Approval is **time-bounded**: an un-actioned request expires, and the execution fails closed rather than waiting indefinitely (Ch 01 §6).

## 4. Non-repudiation and attribution (P10)

An approval decision MUST be **non-repudiable**: bound to the authenticated identity of the approver (Ch 08), timestamped, and recorded in the tamper-evident audit trail (Ch 09). After the fact, it is always provable *who* authorized *what*, *when*, and on *what basis*. This is essential both for accountability and for the platform's own integrity — an unaccountable approval would be an unaccountable action.

## 5. Fail-closed and least-authority

- **Fail-closed.** Absence of an approval, a denial, or an expiry all **stop** the gated action (Book 03 §Ch13 §1). The safe default is *do not act*. An approval-gated effect never proceeds on a missing or ambiguous decision.
- **Least-authority approvers.** *Who* may approve is itself capability-gated (P8): only designated approvers for the workspace/effect-class may decide. A subject cannot approve an action it is not authorized to approve; separation-of-duty policies (the requester ≠ the approver) are expressible.
- **Minimum assurance per consequence class.** An approval requirement declares the **minimum assurance** (Ch 08 §4.2) at which its decision may be taken. Because the decision is rendered on the Web Runtime (Book 13 §Ch06 §2), which authenticates the approver, the approval surface *is* the step-up (Ch 08 §4.3): a request may arrive on a `low`-assurance channel, but the decision is taken at `high`. A decision offered below the declared minimum is refused, fail-closed.
- **Scoped to the specific action.** An approval authorizes the *specific* gated instruction, not a category. It is not a reusable blanket grant (that would be ambient authority by another name, Ch 03 §1).

## 6. Interaction with the runtime checkpoint (P9)

Approval is enforced at the **runtime checkpoint** (Book 14 §06) precisely because approval state is *live* — it is known only at execution whether a decision is present, fresh, and affirmative. The compile-time policy pass (Book 05 §Ch04) can only *require* the approval and insert the guard; the runtime checkpoint *checks the decision*. This is why the three-checkpoint model (Ch 06 §2) matters: approval is the archetypal condition that must be re-verified against live state before the irreversible effect.

## 7. Invariants (normative summary)

1. Approval gates high-consequence actions when policy requires or planning flags them; the requirement is materialized as an `Approval` instruction in the governed plan.
2. An `Approval` is a first-class, observable, auditable Resource recording what/why/who/decision.
3. The human is shown the concrete consequence (effects, targets, cost) and never a secret value; approvals authorize secret *use* by reference (P7).
4. Decisions are non-repudiable — bound to an authenticated identity, timestamped, and in the tamper-evident audit trail (P10).
5. Missing, denied, or expired approvals fail closed; approval authority is capability-gated and scoped to the specific action (P8), with separation-of-duty expressible.
6. Approval is checked at the runtime checkpoint against live decision state, complementing the compile-time requirement (P9).
