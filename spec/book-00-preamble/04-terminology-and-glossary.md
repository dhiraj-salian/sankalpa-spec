# Book 00 · Chapter 04 — Terminology and the Glossary

*Nature: **Normative** (the single-source-of-truth policy binds).*

> This chapter establishes the [Glossary](../../GLOSSARY.md) as the single source of truth for terminology, and the policy that keeps it authoritative. Consistent terminology is a precondition for a coherent specification (Domain-Driven Design's "ubiquitous language"); this chapter is what enforces it.

## 1. The Glossary is the single source of truth

Every term capitalized as a proper noun in the specification (Intent, Goal, Capability, Runtime, Secret Broker, …) **MUST** be defined in the [Glossary](../../GLOSSARY.md). The Glossary is authoritative: where a Book's prose and the Glossary appear to differ on a term's meaning, the Glossary definition governs the *term*, and the owning Book governs the *full specification* of the concept (§3). There is exactly one place to learn what a word means.

## 2. New terms are defined in the same change (P12)

A change that introduces a new capitalized term **MUST** add its Glossary entry in the same change (`CONTRIBUTING` §quality bar, `process/review-gates` documentation gate). This is enforced by the documentation review gate and by CI (glossary coverage). Rationale: a term used but undefined is a coherence hole; requiring simultaneous definition keeps the ubiquitous language complete as the specification grows.

## 3. Term vs. concept: Glossary vs. owning Book

The Glossary gives the *authoritative short definition* and points to the *owning Book* that specifies the concept in full:
- The Glossary defines "Capability" in a sentence and points to Book 02 §Ch07 / Book 03 §Ch06.
- The owning Book specifies its lifecycle, contract, failure modes, and security — the full treatment.

So: use the Glossary to learn *what a term means*; use the owning Book to learn *how the concept works*. The [core resource catalog](../book-02-resource-model/07-core-resource-catalog.md) is the companion index for Resource kinds (which Book owns each).

## 4. Terminology discipline

- **One term, one meaning.** A term means the same thing everywhere; if two concepts need distinguishing, they get distinct terms (not one term with two senses). This is why, e.g., "Knowledge" and "memory" are sharply separated (Book 09 §Ch01) and "Experience" and "audit" are distinct (Book 10 §Ch01, Book 11 §Ch09).
- **Prefer the defined term.** Normative text uses the Glossary term, not a synonym, to avoid ambiguity.
- **Capitalization signals a defined term.** A capitalized proper noun is a promise that the Glossary defines it; lowercase usage is ordinary English.

## 5. Invariants (normative summary)

1. The Glossary is the single source of truth for terminology; every capitalized proper-noun term MUST be defined there.
2. A change introducing a new capitalized term MUST add its Glossary entry in the same change (enforced by the documentation gate and CI).
3. The Glossary gives the authoritative short definition and points to the owning Book, which specifies the concept in full.
4. One term means one thing everywhere; distinct concepts get distinct terms; normative text uses the defined term, and capitalization signals a Glossary-defined term.
