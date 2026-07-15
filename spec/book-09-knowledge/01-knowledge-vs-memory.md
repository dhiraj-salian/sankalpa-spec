# Book 09 · Chapter 01 — Knowledge vs. Memory

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P2, P6, P7, P10. Companion to Book 10 (Experience), Book 08 §04 (planner isolation).*

> Knowledge is durable, curated understanding that improves how the system plans. It is emphatically **not** conversational memory. This chapter draws that distinction precisely — because conflating them is the single most common design error in AI systems, and because the distinction is what keeps the platform's most useful surface (planning context) both effective and safe.

## 1. The claim: Knowledge is first-class, structured, and durable

**`Knowledge`** is an ARM Resource (Book 02 §Ch07): a versioned (P6), observable (P10), governed unit of curated understanding — a fact, a runbook, a relationship, an organizational convention — that the system retains because it improves future planning (Book 08 §02–§03). Being a first-class Resource means Knowledge is queried, related, retained, and governed through the ordinary Kernel mechanisms (Book 03), never through a shadow store.

Knowledge is *understanding*, distilled and maintained. It answers "what does the system durably know that helps it plan well here?" — not "what were the last N things said?"

## 2. What Knowledge is NOT: the memory anti-pattern

The prevailing pattern in agent systems is **memory-as-context**: a rolling buffer of recent conversation turns (and retrieved snippets) stuffed back into a model's prompt. Sankalpa rejects this as the primary knowledge mechanism for three reasons, each grounded in a foundational principle:

1. **It is the leak surface P7 forbids.** Reasoning context is exactly the surface through which secrets leak (Book 11 §01). A memory buffer that accumulates conversation is precisely where a pasted credential, a returned token, or sensitive data ends up in a prompt, a log, or a vector store. Knowledge is *secret-free by construction* (§6); a memory buffer is not.
2. **It is unstructured and unversioned.** A prompt-context buffer has no schema, no provenance, no version, no lifecycle — it cannot be audited (P10), corrected (P6), or governed (P9). Knowledge is structured, provenanced, versioned, and revisable (§Ch06, Book 10 §Ch04 §5).
3. **It conflates recency with relevance.** A buffer surfaces what was *recent*; Knowledge surfaces what is *relevant and trusted*, weighted by provenance (§Ch06). Recency is a poor proxy for relevance and a worse one for correctness.

Sankalpa learns by **structuring and curating understanding**, not by remembering conversations. Where short-term conversational state is genuinely needed, it lives in the `Conversation`/`Session` Resources (Book 13 §05) with their own lifecycle — it is *not* silently promoted into durable Knowledge or into planning context.

## 3. Two representations of one Knowledge

Knowledge has two synchronized representations, each serving a different consumer (specified in §Ch03–§Ch05):

- **A human-readable vault** — an **Obsidian-compatible** Markdown vault (§Ch03), so humans can read, author, and curate Knowledge directly, with links and structure they already understand.
- **A machine-readable graph** — a **graph database** (§Ch04), so planners and the system can query relationships and retrieve relevant Knowledge efficiently.

The two are kept **synchronized** (§Ch05): the vault is the human face, the graph is the machine face, and they represent the same underlying Knowledge. Neither is a cache of the other; both are authoritative views kept consistent.

## 4. Where Knowledge comes from

Knowledge accrues from two sources, both governed:
- **Human curation** — people write and edit the vault directly (§Ch03), asserting facts, runbooks, and conventions.
- **Experience promotion** — the learning loop distills recurring, evaluated lessons into Knowledge, with provenance (Book 10 §Ch04). This is the closed loop: Experience → Knowledge → better planning → better Experience.

Both sources produce the *same kind* of first-class, provenanced, revisable Knowledge. Neither can inject secrets (§6), and consequential machine-promoted Knowledge is human-confirmable (Book 10 §Ch04 §4).

## 5. Knowledge improves planning, deterministically bounded

Knowledge conditions **non-deterministic planning** (Book 08): it informs Goal derivation and plan construction as *retrieved, non-secret, provenanced context* (§Ch06). But — critically — Knowledge influences *what the planner considers*, and the planner still emits **verified** High IR (Book 08 §01). Knowledge makes the planner better-informed; it never lets the planner bypass verification, and it never enters the deterministic machine (compiler/runtime) as anything but what a planner chose to encode in IR. Learning improves the plan; the invariants around the plan are unchanged.

## 6. Knowledge is secret-free (P7)

An absolute, structural guarantee that flows from §2: **Knowledge never contains secret values.** It is surfaced into planning context (§Ch06, Book 08 §04), which is the surface P7 protects most fiercely — so Knowledge that held a secret would breach P7 directly. Knowledge may *reference* a secret by class/reference ("this integration uses a `payments`-class credential") but never the value (Book 11 §04 §5). This is why Knowledge is safe to feed to planners and why it is emphatically not a memory buffer: a buffer accumulates whatever was said (including secrets); Knowledge is curated to hold understanding, never credentials.

## 7. Invariants (normative summary)

1. Knowledge is first-class, structured, versioned, provenanced, durable understanding — not conversational memory re-injected as prompt context.
2. Memory-as-context is rejected as the knowledge mechanism because it is the P7 leak surface, is unstructured/unversioned, and conflates recency with relevance; short-term conversational state lives in Conversation/Session Resources, not in durable Knowledge.
3. Knowledge has two synchronized authoritative representations: a human-readable Obsidian-compatible vault and a machine-readable graph.
4. Knowledge accrues from human curation and provenanced Experience promotion; both produce the same first-class, revisable Knowledge.
5. Knowledge conditions non-deterministic planning as non-secret provenanced context; the planner still emits verified High IR and Knowledge never enters the deterministic machine except via chosen IR.
6. Knowledge never contains secret values; it references secrets only by class/reference (P7), which is what makes it safe as planning context.
