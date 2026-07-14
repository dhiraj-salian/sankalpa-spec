# Book 10 · Chapter 04 — Feedback into Knowledge

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P2, P6, P7, P10. Companion to Book 09 (Knowledge), Book 10 §02 (lessons).*

> The first feedback path: Experience promotes durable **lessons** into **Knowledge** (Book 09), so that what one execution learned improves how future Goals and plans are formed. This chapter specifies how lessons become Knowledge, with provenance, without exposing secrets, and without letting unvalidated learning silently corrupt the knowledge base.

## 1. Why Experience feeds Knowledge

Experience is per-execution and eventually reclaimed (Book 10 §02 §7); Knowledge is durable, curated understanding that informs planning (Book 09 §01, §06). The feedback path exists to **distill the transient into the durable**: a single run's outcome is a fact about that run; a *pattern* across runs — "this integration is flaky at month-end," "this Goal class is best achieved this way," "this org uses that convention" — is Knowledge worth keeping. Without this path, the system would relearn the same patterns forever (Book 10 §01 §1).

## 2. What gets promoted

The Experience Manager (Book 03 §Ch11 §3) promotes **lessons** (Book 10 §02 §5) — not raw Experiences — to Knowledge. Candidates for promotion:
- **Outcome patterns** — recurring success/failure of a Goal class, a Capability, or a Runtime.
- **Relationships & facts** — discovered facts about people, organizations, systems, and their conventions (grounded, non-secret).
- **Runbook/playbook material** — an effective sequence for a recurring objective (which may also become a reusable Workflow, Book 02 §Ch07).
- **Anti-patterns** — plans/approaches that reliably fail policy or execution, so planners avoid them (Book 05 §Ch07 §6).

Raw per-run facts are *not* promoted wholesale; promotion is **distillation** — a pattern supported by multiple Experiences, or a human-confirmed insight (§4).

## 3. Provenance is mandatory (P10, P6)

Every piece of Knowledge promoted from Experience MUST carry **provenance**: which Experiences (by reference) support it, when, and with what confidence (Book 09 §06 §provenance). Rationale:
- Planning must be able to weigh Knowledge by trust (Book 09 §06); a lesson from one execution is weaker than one from a hundred.
- Promoted Knowledge must be **revisable**: if later Experience contradicts it (the integration stopped being flaky), the Knowledge can be updated or retired *because its supporting evidence is tracked* (§5). Unprovenanced learning would be unrevisable and thus dangerous.
- Provenance keeps the learning trail accountable (P10), consistent with audit (Book 11 §09).

Knowledge without provenance MUST NOT be created by this path.

## 4. Human-in-the-loop for consequential Knowledge

Not all learning should be automatic. The path distinguishes:
- **Auto-promotable** — low-risk, well-supported patterns (e.g. cost/latency observations, clear anti-patterns) MAY be promoted automatically with provenance and confidence.
- **Human-confirmed** — consequential or ambiguous insights (a claimed fact about a person/org, a policy-relevant pattern) SHOULD be surfaced for human confirmation (Book 13) before becoming trusted Knowledge, or be recorded at low confidence until corroborated.

This mirrors the planning clarification discipline (Book 08 §02 §4): the system does not silently assert consequential things it merely inferred. Governance of what enters the knowledge base is itself policy-governed (P9, Book 11 §06).

## 5. Revisability and contradiction

Knowledge promoted from Experience is **not permanent truth** — it is the best current understanding, and the loop can correct it:
- When new Experience **contradicts** existing promoted Knowledge, the Experience Manager flags the conflict; the Knowledge is updated, down-weighted, or retired, with the change provenanced (Book 09 §06).
- This is the same reversibility principle that governs determinization (Book 05 §Ch06 §4): learning that proves wrong is *undone*, not left to mislead. The loop that adds Knowledge also maintains it.

## 6. Secret-freedom (P7)

Because lessons are secret-free (Book 10 §02 §6) and Knowledge never stores secrets (Book 09 §06, Book 11 §04 §5), the promotion path carries no secret. A lesson may note that "an execution used a `payments`-class credential" (by class/reference) but never the credential. The Knowledge base — which *is* surfaced into planning context (Book 09 §06, Book 08 §04) — thus remains a safe, non-secret surface, preserving the guarantee that planners never see secrets.

## 7. Closing one arc of the loop

This path closes the Experience → Knowledge arc of the core loop (Book 10 §01 §1). Its payoff is realized in Book 09 §06, where promoted Knowledge conditions future Goal derivation and planning (Book 08 §02–§03): the platform forms better objectives and plans because prior Experience taught it what "good" looks like for this workspace — durably, provenanced, revisably, and without a secret ever entering the loop.

## 8. Invariants (normative summary)

1. Experience feeds Knowledge by distilling recurring, supported *lessons* into durable Knowledge — not by promoting raw per-run facts wholesale.
2. Every promoted piece of Knowledge carries mandatory provenance (supporting Experiences, time, confidence); unprovenanced promotion is prohibited (P10, P6).
3. Low-risk well-supported patterns may auto-promote; consequential/ambiguous insights require human confirmation or low-confidence recording (governed by policy).
4. Promoted Knowledge is revisable: contradicting Experience updates, down-weights, or retires it, with the change provenanced (reversibility).
5. The path is secret-free: lessons and Knowledge reference secrets only by class/reference, so the planning-facing knowledge base never holds a secret (P7).
6. This path closes the Experience→Knowledge arc, improving future Goal derivation and planning durably and accountably.
