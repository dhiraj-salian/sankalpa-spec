# Pattern Study: Domain-Driven Design

*Status: Accepted · Informs: Books 01 (foundations), 02 (resource model), the [Glossary](../../GLOSSARY.md).*

## 1. Pattern in one paragraph
Domain-Driven Design is the claim that a system's hardest problem is usually not technical but **linguistic**: teams fail because "customer" means three different things in three different rooms and the code faithfully encodes the confusion. DDD's remedy is a **ubiquitous language** — one vocabulary shared by domain experts, specification, and code, with no translation layer — bounded by explicit **bounded contexts**, within which a term has exactly one meaning, and between which the mapping is deliberate and written down. For a specification-first project (ADR-0001), this is not a modeling technique; it is the operating discipline.

## 2. Core ideas
1. **Ubiquitous language.** One term, one meaning, used identically in conversation, spec, and code. If the code says `Job` and the team says `Execution`, the model has already drifted.
2. **Bounded contexts.** A model is valid only within a boundary; the same word may legitimately mean different things in different contexts, provided the boundary is explicit.
3. **Context mapping.** The relationships between contexts (shared kernel, customer/supplier, anti-corruption layer) are designed, not emergent.
4. **The anti-corruption layer.** When integrating a foreign model, translate at the boundary rather than letting its concepts leak in and corrode yours.
5. **Entities vs. value objects.** Identity-bearing things vs. things defined wholly by their attributes.
6. **Aggregates.** A consistency boundary with a root: invariants hold within an aggregate, and *across* aggregates you get eventual consistency — the transaction boundary is a modeling decision.
7. **Domain events** as first-class facts about what happened.
8. **The model is not the database.** Persistence is an implementation concern behind a repository.

## 3. Design decisions & trade-offs
- **A ubiquitous language** buys precision that compounds across every artifact, at the cost of continuous, tiring enforcement — one unchallenged synonym in a review starts the rot.
- **Bounded contexts** buy team and model autonomy, at the cost of translation at every seam and duplicated concepts by design (which looks like waste to anyone who has not lived the alternative).
- **Aggregates as consistency boundaries** buy scalability and clear invariants, at the cost of forcing eventual consistency into the domain model where developers expected transactions.
- **The full DDD apparatus** buys a durable model for genuinely complex domains, at the cost of ceremony that is pure overhead for simple ones — the pattern is routinely cargo-culted into systems whose domain is CRUD.

## 4. Relevance to Sankalpa
Sankalpa is inventing a domain vocabulary — Intent, Goal, Mission, Capability, Execution, Experience, Determinization — where none existed. If those words are imprecise, every Book, RFC, and eventual implementation inherits the imprecision, and there is no compiler to catch it because the spec *is* the artifact. The [Glossary](../../GLOSSARY.md) is a ubiquitous language enforced by process ("every term capitalized as a proper noun MUST be defined here; add terms in the same PR"), Book 00 §Ch04 makes terminology normative, and Book 02's uniform Resource model is the model made mechanical.

## 5. What we adopt
- **The ubiquitous language, enforced by review.** The Glossary is normative and maintained in-PR; Book 00 §Ch04 fixes terminology before Book 01 says anything with it. In a spec-first project this is the *primary* DDD contribution and the reason the pattern is here.
- **Bounded contexts as the Books.** Each Book owns its concepts, and cross-Book references are explicit citations rather than assumed shared meaning. The Books are our context map.
- **Anti-corruption layers at every plugin boundary.** This is the DDD idea with the most bite for us: a planner's concepts (agents, chains, tools) MUST NOT leak into ours (Book 08 §Ch03), a runtime's concepts stay behind its interface (Book 06 §Ch02), and a channel's stay behind the channel model (Book 13 §Ch02). Book 03 §Ch01 §4's rule — "the core knows mechanism, never *which tool*" — is an anti-corruption layer promoted to an architectural invariant. The whole IR exists so that a foreign model's vocabulary can never reach the middle of the system.
- **Domain events as first-class facts** (P5, Book 14 §Ch01) with a specified naming scheme (§Ch03).
- **Aggregates as consistency boundaries.** Book 02 §Ch08 §4's "no cross-Resource transactions" is the aggregate rule: the Resource is the consistency boundary, and across Resources you reconcile (P3) rather than transact.
- **Entity/value distinction** — Resources have identity and lifecycle (Book 02 §Ch02, §Ch04); IR modules, effects, and signatures are value objects, defined entirely by content (Book 04 §Ch07's content addressing is the value-object idea taken to its conclusion).
- **The model is not the storage.** Book 02 §Ch08 §1 specifies properties, not a product.

## 6. How faithfully we apply it (and where we deviate)
- **Faithful on language, deliberately partial on machinery.** We take ubiquitous language, bounded contexts, anti-corruption layers, aggregates-as-consistency-boundaries, and domain events. We do not adopt repositories, factories, application/domain-service layering, or the tactical patterns as prescribed vocabulary — those are implementation-shaped guidance for an OO codebase, and Book 00 §Ch05 keeps the spec above implementation structure.
- **One context, not many, for the core model.** DDD expects the same word to mean different things in different contexts. We reject that *within* Sankalpa: P2's uniform Resource model exists precisely so that "Resource" means one thing everywhere, and the Glossary permits no per-Book redefinition. Our context boundaries are with the *outside* (planners, runtimes, channels), not internal. This is a real deviation, and it is affordable only because we are designing the domain rather than mapping an existing organization's — DDD's multiple contexts are usually a concession to Conway's Law, and we do not have that org yet.
- **The domain experts are absent.** DDD assumes a domain expert to extract language from. Sankalpa's domain does not exist yet, so our language is *stipulated* by ADR/RFC rather than discovered from practice. That is a weaker epistemic position than DDD assumes, and it means terms should be expected to move — the Glossary is a living document (its own status line says so), and P12 makes the reasoning behind each term recoverable.
- **The model is under-tested against reality.** DDD's feedback loop is a working system correcting the model. We do not have one until Phase 3, so Phase 2's adversarial review (ROADMAP) is standing in for it — an inferior substitute, and worth naming as a known risk.
- **Aggregate ≠ Resource, exactly.** DDD aggregates can span several entities under one root; our consistency boundary is a single Resource. Where a real invariant spans Resources, we reconcile toward it rather than enforce it atomically (Book 07 §Ch04) — a weaker guarantee, chosen for the reasons Book 02 §Ch08 §4 gives.

## 7. Open questions
- If "Resource" must mean one thing in every Book, is Book 09's Knowledge (an Obsidian-shaped vault, Book 09 §Ch03) genuinely the same concept as an Execution, or are we forcing one word over two models because P2 is attractive?
- Our language is stipulated rather than elicited. What is the mechanism for a term that Phase 3 reveals to be wrong — an RFC per rename, and who bears the reflection cost across 16 Books?
- Is there a bounded context *inside* Sankalpa we are refusing to admit? Planning (Book 08) and Execution (Book 06) reason about "a plan" quite differently — High IR vs. Low IR is arguably that seam, admitted in the IR rather than in the language.

## 8. References
- Evans, *Domain-Driven Design* (2003), esp. the ubiquitous language, bounded context, and anti-corruption layer chapters; Vernon, *Implementing Domain-Driven Design* (2013); Brandolini on context mapping; Conway, "How Do Committees Invent?" (1968).
