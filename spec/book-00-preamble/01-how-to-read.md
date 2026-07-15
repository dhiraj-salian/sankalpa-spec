# Book 00 · Chapter 01 — How to Read This Specification

*Nature: **Informative**.*

> This chapter explains how the specification is organized and how to read it efficiently, whether you are a newcomer seeking understanding or an implementer seeking precision.

## 1. Books and Chapters

The specification is organized as **Books** (00–15), each covering one concern, divided into numbered **Chapters** (`NN-slug.md`). Each Book has a `README.md` stating its **scope** and **chapter outline** and its **status** (`Draft skeleton` → `Draft-complete` → toward `Final`). The master index is [`../README.md`](../README.md).

Books are ordered so each builds on the last (spec/README §Reading order). A newcomer should read Books 00–01 (the *why* and vocabulary), then 02–06 (the *what*: model, kernel, IR, compiler, runtimes), then the rest by interest. An implementer jumps to the Book owning their subsystem and follows its cross-references.

## 2. Normative vs. Informative

Every chapter declares its **nature** at the top:
- **Normative** — binding requirements. Uses RFC 2119 keywords (§Ch02) in its normative statements, and ends with an "Invariants (normative summary)" section restating its binding rules.
- **Informative** — explanation, motivation, examples, walkthroughs. Helps understanding but binds nothing. Where an informative chapter and a normative one appear to conflict, **the normative chapter wins** (and the conflict is a defect to report).

A chapter may be mostly one with a clearly-marked section of the other (e.g. a Normative chapter's illustrative example is informative).

## 3. Each chapter declares what it reflects

A chapter's header names the **principles it realizes** (P1–P13, Book 01 §Ch04) and the **RFCs/ADRs it reflects**. This is the traceability chain (`process/versioning-and-stability` §change traceability): normative text traces to an Accepted RFC/ADR, which traces to a CHANGELOG entry and a version. To understand *why* a rule exists, follow its header back to its RFC/ADR.

## 4. Cross-references

The specification is densely cross-referenced (`Book NN §ChNN §S`). This is deliberate: the system is an integrated whole, and a mechanism is rarely understandable in isolation. Follow the references — a claim in one Book (e.g. "materialized only at execution") is *specified* in another (Book 11 §04). The cross-reference is a promise that the detail exists and is consistent.

## 5. How to find a term

Every capitalized term is defined once in the [Glossary](../../GLOSSARY.md) (§Ch04). If a Book uses a term you do not know, the Glossary is authoritative; the Book that *owns* the concept (named in the Glossary and the [core resource catalog](../book-02-resource-model/07-core-resource-catalog.md) for Resources) specifies it in full.

## 6. What to read for a given question

| You want to… | Read |
|--------------|------|
| Understand the *why* | Book 01 (Foundations) |
| Understand the vocabulary | [Glossary](../../GLOSSARY.md), Book 01 §Ch04 (principles) |
| See it all work end-to-end | Book 15 §Ch01 (the walkthrough) |
| Implement a subsystem | that subsystem's Book, following cross-refs |
| Extend Sankalpa | the relevant AEP (planner 08, runtime 06, package 12) + Book 12 |
| Check a change is safe | Book 11 §Ch11 (security invariants), `process/review-gates` |
| Contribute a design | `CONTRIBUTING`, `process/`, the templates |

## 7. Summary (informative)

Read top-down for understanding, jump-and-follow-refs for implementation. Trust the "Nature" header (Normative binds; Informative explains); trust the Glossary for terms; trust the cross-references to lead you to where each detail is specified. The specification is designed to be understood by a newcomer in an afternoon and by an expert in depth — this chapter is the key to both.
