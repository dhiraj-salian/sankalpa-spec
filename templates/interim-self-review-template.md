# Interim Self-Review — <artifact id and title>

*Filed under [RFC-0012](../rfcs/0012-interim-review-process.md) §4.3. Attach to the PR that accepts the artifact. This is an **adversarial** self-review: your job here is to argue against your own change, not to defend it.*

| Field | Value |
|-------|-------|
| **Artifact** | <RFC/ADR/AEP/PR id> |
| **Author** | <name> |
| **Domain** | <Book/subsystem — confirm its Domain Lead seat is vacant> |
| **Reached `Proposed`** | YYYY-MM-DD |
| **Earliest acceptance (Proposed + 3 working days)** | YYYY-MM-DD |

## 1. The strongest case against this change

*Mandatory. State the best argument that this change is wrong, unnecessary, or worse than an alternative. What would a skeptical Domain Lead say? What would falsify the design?*

## 2. Review gates

*Answer each with a concrete finding or "N/A — because …". Do not write "pass".*

| Gate | Finding |
|------|---------|
| **Architecture** — conceptual integrity; layered flow; everything-is-a-Resource; Kernel-mediated communication | |
| **Security** — every SI-1…SI-13 invariant (Book 11 §Ch11) still holds; new trust boundaries / secret flows / capabilities named; security-review artifact attached for T3 | |
| **Performance** — latency/throughput/memory/scaling; cost model for T3 | |
| **Testability** — deterministically verifiable; conformance/property/integration plan for T3 | |
| **Documentation** — `spec/`, `GLOSSARY.md`, diagrams updated; no undefined capitalized term | |
| **Backward-Compatibility** — breaks any existing Resource/IR/API/Package? justified + versioned? | |
| **Migration** — path and (eventual) tooling for anything that changes shape | |

## 3. Impact tier

*T0 editorial / T1 clarification / T2 local design / T3 cross-cutting or public API. State which, and why. T3 pulls all gates.*

## 4. What I could not check alone

*Mandatory. What needs a real reviewer's eyes that a solo author cannot give — domain expertise you lack, a blind spot you suspect, a decision you are unsure of? This becomes the focus of the eventual ledger re-review.*

## 5. Independent adversarial pass

*Record the pass required by RFC-0012 §4.4: who/what performed it, the instruction given, and the raw findings. Link the PR comment holding the detail.*

- **Form:** solicited outside reader / AI adversarial pass
- **What it looked for:**
- **Findings and disposition:** (each finding → fixed in the change / dismissed because …)
