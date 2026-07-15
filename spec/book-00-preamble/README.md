# Book 00 — Preamble

*Status: **Draft-complete** (all chapters authored) · Nature: Informative (except §Conformance, which is Normative).*

## Scope
How to read, cite, and conform to the Sankalpa specification. This Book contains no system design; it defines the meta-rules that make the other Books unambiguous.

## Chapters
1. **`01-how-to-read.md`** — The Book/Chapter structure; the Normative vs. Informative distinction; how each chapter declares the RFCs/ADRs it reflects.
2. **`02-conformance-language.md`** *(Normative)* — RFC 2119 keyword usage; what it means for an implementation or extension to *conform* to a specific spec version; the role of conformance suites.
3. **`03-notation.md`** — Notation used throughout: schema notation for Resources, grammar notation for IR, sequence-diagram conventions, and the Event naming scheme.
4. **`04-terminology-and-glossary.md`** — Pointer to the [Glossary](../../GLOSSARY.md) as the single source of truth; policy that new terms are defined there in the same change that introduces them.
5. **`05-relationship-to-implementation.md`** — Restates ADR-0001: this spec is authoritative; implementations realize it; discrepancies are spec-or-implementation bugs, resolved via the process.
