# Book 10 · Chapter 01 — Experience as First-Class

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P2, P5, P10, P13. Companion to Book 05 §Ch06 (determinization), Book 09 (Knowledge), Book 11 §09 (audit).*

> Experience is what closes Sankalpa's loop. Every execution produces an **Experience** — a first-class record of what was intended, what ran, and how it went — and Experience feeds back into Knowledge and future planning, driving the system toward ever-greater determinism (Book 01 §05). This chapter establishes Experience as an architectural primitive, not an afterthought, and explains why the loop is the platform's compounding advantage.

## 1. The loop, restated

The core philosophy (Book 01 §02) is a loop, and Experience is its return path:

```
Intent → Goals → Planning → IR → Compilation → Execution → EXPERIENCE → Knowledge
   ▲                                                             │
   └──────────────── improves future planning ◀─────────────────┘
```

Without Experience, the system would be memoryless: it would solve the same problem, make the same reasoning, and pay the same cost every time, learning nothing. Experience is the mechanism by which *a solved problem makes the next one cheaper and more reliable* — the difference between a tool and a platform that improves (Book 01 §01).

## 2. Experience is a first-class Resource (P2)

Experience is an **`Experience` Resource** (Book 02 §Ch07), not a log line or a side effect. As a Resource it inherits the uniform treatment of everything in Sankalpa (P2): it is versioned, observable, reconciled, and governed. Concretely:

- Every **`Execution`** (Book 06 §Ch03 §5) **produces** an `Experience` it owns (`ownerRef`, Book 02 §Ch06), and the Experience typically **outlives** the Execution (Book 02 §Ch04 §5) — the run is ephemeral, the lesson is durable.
- Being first-class means Experience can be queried, related, retained, and fed into other subsystems through the ordinary Kernel API and Event mechanisms (Book 03) — no special back channel.

Treating Experience as a Resource is a deliberate architectural stance: the platform's *learning* is subject to the same governance, observability, and tenancy as its *doing*. There is no shadow memory.

## 3. Experience is not memory

A critical distinction, mirrored from Knowledge (Book 09 §01): **Experience is not a conversational memory buffer.** It is a *structured, first-class record of an execution's full lifecycle and outcome*, captured for analysis and feedback — not a rolling window of recent tokens stuffed back into a prompt. The difference matters:
- Memory-as-prompt-context is exactly the leak-prone reasoning surface P7 forbids for secrets (Book 11 §01) and is unstructured and unversioned.
- Experience is structured, queryable, secret-free (§Ch03), and consumed by *deterministic* machinery (the determinization engine, the cost model) as well as by planning — not merely re-injected as text.

Sankalpa learns by *structuring and analyzing* experience, not by *remembering conversations*.

## 4. What the loop enables

Experience feeds three distinct return paths, each its own chapter:

1. **Into Knowledge (§Ch04, Book 09).** Lessons, outcomes, and relationships are promoted to durable, curated Knowledge that improves how future Goals and plans are formed.
2. **Into planning and cost modeling (§Ch05).** Observed latencies, costs, failures, and successes condition future planning and the compiler's cost-based choices (Book 05 §Ch03 §3).
3. **Into determinization (§Ch06, Book 05 §Ch06).** Repeated reasoning with stable outcomes is *discovered* here and folded into deterministic Capabilities — the direct engine of P13.

The third is the most consequential: it is *how the system becomes more deterministic over time* (Book 01 §05). Every Experience is both a record and a potential lesson that shrinks the non-deterministic surface (Book 08 §05 §4).

## 5. Experience is secret-free and attributable (P7, P10)

Because Experience is assembled from the Event stream (§Ch03, Book 03 §Ch03), it inherits two guarantees:
- **Secret-free (P7).** The Event stream carries no secret values (Book 03 §Ch03 §8); Experience therefore contains none. Lessons reference secrets by class/reference, never by value (Book 11 §04 §6).
- **Attributable (P10).** Every Experience traces to its Intent, Goals, Compilation, and Execution by reference, and its constituent Events are attributed (Book 11 §09). The learning trail is as accountable as the audit trail — indeed it shares the same source (Book 11 §09 §7).

## 6. Experience vs. audit

Experience (this Book) and audit (Book 11 §09) both derive from the Event stream but answer different questions: audit asks *"who did what, provably"* (accountability); Experience asks *"how did it go, what should improve"* (learning). Keeping them distinct (Book 11 §09 §7) prevents learning concerns from diluting audit's integrity requirements, while letting both draw on one complete, secret-free, commit-consistent source of truth.

## 7. Invariants (normative summary)

1. Every Execution produces a first-class `Experience` Resource it owns; the Experience typically outlives the Execution (P2).
2. Experience is a structured, queryable, versioned record — not conversational memory re-injected as prompt context.
3. Experience feeds three return paths: Knowledge, planning/cost-modeling, and determinization — the last being the direct engine of the toward-determinism mission (P13).
4. Experience is assembled from the Event stream and is therefore secret-free (P7) and fully attributable (P10).
5. Experience (learning) and audit (accountability) are distinct systems over the same complete, secret-free, commit-consistent Event stream.
