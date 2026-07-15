# Book 00 · Chapter 02 — Conformance Language

*Nature: **Normative**.*

> This chapter defines the requirement keywords used throughout the specification and what it means for an implementation or extension to *conform*. Precision here is what makes every other normative statement enforceable.

## 1. Requirement keywords (RFC 2119)

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119). In this specification:
- **MUST / MUST NOT / REQUIRED / SHALL** — an absolute requirement or prohibition. A conforming implementation cannot deviate.
- **SHOULD / SHOULD NOT / RECOMMENDED** — a strong recommendation; deviation requires a considered reason and does not break conformance, but should be documented.
- **MAY / OPTIONAL** — a genuinely permitted choice; conformance does not depend on it.

These keywords appear **only in Normative chapters** (Book 00 §Ch01 §2) and only in normative statements. Their appearance in an Informative chapter (e.g. quoting a rule) is descriptive, not a new requirement.

## 2. What conformance means

An **implementation** or **extension** *conforms to Sankalpa specification version X* if it satisfies every **MUST/MUST NOT** applicable to it in version X's normative text, and its deviations from **SHOULD** are documented. Conformance is always **to a specific version** (`process/versioning-and-stability`), never to "latest" — because the specification evolves and a conformance claim must be precise and durable.

## 3. Conformance is scoped

Not every requirement applies to every artifact. Conformance is **scoped** to what an implementation provides:
- A **runtime** conforms to the runtime requirements (Book 06) and passes the runtime conformance suite (Book 06 §Ch07).
- A **planner** conforms to Book 08 and its suite (Book 08 §Ch07).
- A **package** conforms to AEP-0003 (Book 12).
- A **core implementation** conforms to the Kernel/ARM/IR/Compiler requirements (Books 02–06) and their suites (verifier Book 04 §Ch08, Kernel API Book 03 §Ch02).

An artifact claims conformance only for the scope it implements, at a stated **stability level** where the interface is AEP-graded (Experimental/Beta/Stable, the AEP process).

## 4. Conformance suites make it testable

Wherever the specification defines a **conformance suite** (verifier, planner, runtime, package, Kernel API), conformance to that scope is **verifiable by passing the suite** — not merely asserted. This is the "verify, don't trust" posture (Book 11 §10 §4) applied to conformance itself: a claim is checkable. Two artifacts conforming to the same scope at the same version behave equivalently where the specification requires equivalence (Book 06 §Ch07 §5, Book 08 §Ch07 §5) — which is what makes them interchangeable (P11).

## 5. Normative summaries are authoritative restatements

Each Normative chapter ends with an **"Invariants (normative summary)"** section. These restate the chapter's binding requirements for quick reference and review; they are authoritative (a summary and its chapter body do not conflict — if they appear to, it is a defect to report). Reviewers use these summaries as checklists (e.g. Book 11 §Ch11 is the consolidated security checklist for the security gate).

## 6. Invariants (normative summary)

1. RFC 2119 keywords carry their standard meanings; MUST/MUST NOT are absolute, SHOULD is a documented-deviation recommendation, MAY is a free choice; they appear only in normative statements.
2. Conformance is satisfying every applicable MUST/MUST NOT (with documented SHOULD deviations) for a specific specification version — never "latest."
3. Conformance is scoped to what an artifact provides (runtime, planner, package, core), at a stated stability level for AEP-graded interfaces.
4. Where a conformance suite is defined, conformance to that scope is verifiable by passing it; equal-scope equal-version conformers are interchangeable.
5. Each Normative chapter's "Invariants" summary is an authoritative restatement used as a review checklist.
