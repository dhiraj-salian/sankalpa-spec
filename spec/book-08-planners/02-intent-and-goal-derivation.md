# Book 08 · Chapter 02 — Intent and Goal Derivation

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P2, P3, P8, P9. Companion to Book 02 §Ch07 (Intent/Goal), Book 09 (Knowledge), Book 11 §07 (Approval).*

> Before a planner can emit High IR, it must turn a vague human request into structured, checkable objectives. This chapter specifies **Intent → Goals** derivation: how an `Intent` becomes one or more `Goal` Resources, how underspecification is resolved through clarification (human-in-the-loop), and the boundaries that keep this stage safe.

## 1. Intent and Goal as Resources

Both ends of this stage are ARM Resources (Book 02 §Ch07), so derivation is observable, versioned, and reconciled like everything else (P2, P3):

- **`Intent`** — captures a human request: the natural-language `utterance`, the channel it arrived on, and the submitter (Book 02 §Ch02 example). It is non-executable (P1).
- **`Goal`** — a structured objective refined from Intent: what outcome is wanted, with **explicit success criteria**, constraints, and references to relevant Knowledge (Book 09). Still declarative — not yet a plan.

The Intent controller (Book 07) drives this derivation: an `Intent` in `Pending` is reconciled by invoking a planner (§Ch03) to produce `Goal`s, which the Intent **owns** (`ownerRef`, Book 02 §Ch06). Deleting the Intent cascades to its Goals.

## 2. Why an explicit Goal stage

Jumping straight from utterance to IR would bury two distinct concerns — *what is wanted* and *how to achieve it* — in one opaque leap. Separating them:

- Makes the **objective inspectable and confirmable** before any plan is built (a human can review Goals; §4).
- Lets **success criteria** be stated declaratively, so an execution's success is later *checkable* against them (Book 10 §07), not merely "it ran."
- Gives policy and Knowledge a clean place to attach (§3, §5) before planning commits to steps.

A Goal is the contract the plan must satisfy; the plan (High IR) is one way to satisfy it.

## 3. Deriving Goals

The planner derives Goals by reasoning over:
- the Intent's `utterance` and context (channel, submitter role, Conversation history — Book 13 §05);
- relevant **Knowledge** (Book 09) retrieved as **non-secret, provenance-tagged** context (§Ch04, Book 09 §06) — prior projects, runbooks, org facts that inform what "done" means;
- the **capabilities available** to this workspace (Book 03 §Ch06), so Goals are grounded in what can actually be done.

Each derived Goal MUST carry **explicit success criteria** — the checkable definition of done. A Goal without success criteria is incomplete and MUST NOT proceed to planning: you cannot later judge success, nor can Experience learn from the outcome (Book 10 §07).

## 4. Clarification and human-in-the-loop

Intent is frequently underspecified ("summarize the sales and send it round" — to whom? which period? which channel?). The planner MUST handle underspecification explicitly rather than guessing silently on consequential ambiguities:

- **Clarification loop.** When a Goal cannot be responsibly derived without more information, the planner emits a **clarification request** surfaced to the human through a channel/web experience (Book 13). The Intent remains `Pending`/`Progressing` with a condition explaining what is needed (Book 02 §Ch03 §3.5) — never silently stalled.
- **Bounded assumptions.** For low-consequence ambiguity, the planner MAY proceed on a **stated, recorded assumption** (captured on the Goal), so the human can see and correct it. High-consequence ambiguity (irreversible or costly effects) MUST NOT be resolved by assumption; it requires clarification or, at execution, an **Approval** (Book 11 §07).
- **No secret elicitation here.** Clarification MUST NOT ask the human to type a secret into planner context (P7). Credentials are acquired only through the Secret Broker's secure channels (Book 11 §05) and referenced, never entered into planning.

## 5. Policy and safety at the Goal stage

- Derivation is **capability-scoped** (P8): the planner may only ground Goals in capabilities the submitter/workspace is entitled to. A Goal implying an unentitled action is surfaced as needing authorization, not silently planned.
- Goals feed the downstream **policy-validation pass** (Book 05 §Ch04): because success criteria and intended effects are explicit by the Goal stage, governance later has a meaningful artifact to check. Derivation should therefore make intended effects *legible*, not hide them.
- The whole stage is **secret-free** (§Ch04): no Goal, clarification, or Knowledge context carries secret values (P7).

## 6. From Goals to planning

Once Goals with success criteria exist (and any blocking clarification is resolved), the planner proceeds to build the plan and emit High IR (§Ch03, Book 04 §Ch03). The Goals remain the reference against which:
- the **plan** is constructed (the High IR must plausibly achieve the Goals' criteria), and
- the **outcome** is later judged (Experience compares actual results to the Goals' success criteria, Book 10 §07).

This traceability — Intent → Goals(with criteria) → High IR → Execution → Experience-vs-criteria — is what makes the loop *evaluable*, not just runnable (Book 15 §01).

## 7. Invariants (normative summary)

1. Intent and Goal are ARM Resources; the Intent owns its derived Goals; derivation is reconciled, observable, and versioned (P2, P3).
2. Every Goal carries explicit, checkable success criteria; a Goal without them does not proceed to planning.
3. Consequential underspecification is resolved by clarification (human-in-the-loop) or a recorded, correctable assumption; high-consequence ambiguity is never resolved by silent assumption.
4. Derivation uses only non-secret, provenance-tagged Knowledge and never elicits secrets into planner context (P7); credentials flow only through the Secret Broker.
5. Derivation is capability-scoped; Goals implying unentitled actions are surfaced for authorization, not silently planned (P8, P9).
6. Goals are the contract the plan must satisfy and the yardstick the outcome is judged against, making the intent→experience loop evaluable.
