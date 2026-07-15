# Book 13 · Chapter 06 — Approval and Human-in-the-Loop

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P7, P8, P9, P10. Companion to Book 11 §07 (Approval Engine), Book 08 §02 (clarification), §Ch01 (Web Runtime).*

> Human-in-the-loop interactions — approvals and clarifications — are where a person is brought into the platform's flow to authorize or disambiguate. This chapter specifies how these are *rendered and captured* at the interface layer. The governance mechanisms are normative in their home Books (Approval Engine Book 11 §07, clarification Book 08 §02); this chapter specifies the human-facing surface that makes them real, and its security discipline.

## 1. Two human-in-the-loop interactions

- **Approval** (Book 11 §07) — gating a high-consequence action on explicit human authorization.
- **Clarification** (Book 08 §02 §4) — resolving consequential ambiguity in intent before planning proceeds.

Both bring a human into the loop; both are surfaced through channels (§Ch02) and rendered on the Web Runtime (§Ch01) when richness or security demands. This chapter specifies the *rendering and capture* discipline common to both.

## 2. Approvals are rendered with full consequence, on a trusted surface

When the Approval Engine (Book 11 §07) requires a decision, the interface layer renders the request as a **Web Runtime approval page** (§Ch01 §4), reached via a URL delivered through the user's channel (§Ch02 §4). The rendering MUST (Book 11 §07 §3):
- Show the **concrete consequence** — the declared effects, targets, and cost of the gated action (Book 04 §Ch06) — so the human gives *informed* consent, not a blind "approve?".
- **Never display a secret value** (P7): an approval authorizes secret *use* by reference/class (Book 11 §07 §3), so the page shows "will use the `payments` credential," never the credential.
- Capture a **non-repudiable decision** (Book 11 §07 §4) bound to the authenticated approver's identity (Book 11 §08), timestamped, and audited (P10).

Rendering on the **trusted Web Runtime** — not reconstructed inside a third-party channel (§Ch02 §4) — is what makes the decision surface trustworthy: the platform controls what the approver sees and how the decision is captured.

## 3. Clarifications route the same way

A clarification request (Book 08 §02 §4) — "to whom should this be sent? which period?" — is surfaced to the human through the channel, and for anything beyond a trivial text reply, via the Web Runtime. Like approvals:
- It presents enough context for an informed answer, and records the answer against the Intent (Book 08 §02 §4) so planning can proceed.
- It **never solicits a secret** into the dialogue (§Ch02 §5, Book 08 §04): a clarification that needs a credential directs the user to a Web Runtime credential page (Book 11 §05), never asks them to type it in chat.
- It is attributed and audited (P10).

## 4. Fail-closed and bounded (P9)

Human-in-the-loop interactions are **fail-closed and time-bounded** (Book 11 §07 §5, Book 03 §Ch13 §1):
- An **un-actioned** approval/clarification **expires**, and the gated action **stops** (fail-closed) rather than waiting indefinitely or proceeding on a guess. An expired approval fails the runtime checkpoint (Book 14 §06); an unresolved consequential clarification blocks planning with a surfaced condition (Book 08 §02 §4).
- A **denied** approval stops the action, with compensation for any completed effects (Book 06 §Ch03 §4).

The interface layer MUST surface the *pending* state visibly (a condition, Book 02 §Ch03 §3.5) so the interaction is never a silent stall — the human and operators can see what is waiting on whom.

## 5. Authority to approve is itself gated (P8)

Who may approve/answer is **capability-gated** (Book 11 §07 §5): the interface layer renders the request only to authorized approvers, and captures a decision only from one. Separation-of-duty (requester ≠ approver) is expressible and enforced (Book 11 §07 §5). The interface does not *decide* who may approve — it *enforces* what the Approval Engine and capability model determine (a channel/UI carries and renders; the Kernel governs, §Ch02 §1).

## 6. The interface renders; the Kernel governs

Restating the layering: this chapter's mechanisms (approval pages, clarification prompts) **render and capture** human decisions, but the *governance* — when approval is required (Book 11 §07 §1, policy Book 11 §06), what the decision authorizes, whether it is valid at execution (the runtime checkpoint, Book 14 §06) — lives in the Kernel. The Web Runtime shows the consequence and captures the decision; the Approval Engine and Policy Engine decide what that decision means and enforce it. This separation is what keeps human-in-the-loop trustworthy: the interface cannot fabricate or bypass an authorization, only faithfully render and capture one.

## 7. Invariants (normative summary)

1. Approvals and clarifications are rendered through channels and the trusted Web Runtime; their governance is normative in Book 11 §07 / Book 08 §02, this chapter specifies rendering and capture.
2. Approval pages show the concrete consequence (effects, targets, cost), never a secret value, and capture a non-repudiable, attributed, audited decision (P7, P10).
3. Clarifications present context, record the answer against the Intent, and never solicit a secret into the dialogue (routing to a Web Runtime credential page instead).
4. Human-in-the-loop interactions are fail-closed and time-bounded: expiry/denial stops the gated action (with compensation); the pending state is surfaced, never a silent stall (P9).
5. Authority to approve/answer is capability-gated with separation-of-duty enforceable; the interface enforces, it does not decide, who may approve (P8).
6. The interface renders and captures human decisions; the Kernel governs what they require, mean, and enforce — the interface cannot fabricate or bypass an authorization.
