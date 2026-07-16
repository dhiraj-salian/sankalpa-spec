# Pattern Study: Capability-Based Security

*Status: Accepted · Informs: Book 11 (§Ch03 especially) — the pattern P8 adopts; Books 03 §Ch06, 12 §Ch05.*

## 1. Pattern in one paragraph
Capability-based security replaces *permission checked against identity* with *authority held as a reference*. A capability is an unforgeable token that simultaneously **designates** a specific object-and-operation and **conveys** the right to perform it: to act, you must hold the thing, and holding it is the whole of your authority. There is no lookup of "may this user do this?", because there is no ambient set of powers attached to a name. The tradition runs from Dennis & Van Horn (1966) through KeyKOS, EROS, and seL4 to Mark Miller's E and the object-capability model, and it is the basis of Sankalpa's P8.

## 2. Core ideas
1. **Designation and authority are the same act.** You cannot name what you may not use, and you cannot use what you cannot name — which is precisely what kills the confused-deputy attack.
2. **Unforgeable references.** A capability cannot be guessed or constructed; possession is the only path to authority.
3. **No ambient authority.** Nothing follows from identity, membership, or being "logged in as root." A subject with no capabilities can do nothing.
4. **Attenuation.** Delegation may only narrow: you can pass on less than you hold, never more.
5. **Revocation via indirection** — the Redell membrane/caretaker pattern: hand out a forwarder you can later sever, because a bare unforgeable reference has no natural off switch.
6. **POLA** (principle of least authority) becomes structural rather than aspirational: components naturally get only what they were handed.
7. **The confused deputy** (Hardy, 1988) is the pattern's motivating bug: a privileged program tricked into misusing its ambient authority on an attacker's behalf. With capabilities the request carries the authority, so there is nothing to confuse.

## 3. Design decisions & trade-offs
- **Authority-as-reference** buys least privilege by construction and immunity to the confused deputy, at the cost of every authority being explicit — there is no "just make it work" escape, which is why ACL systems keep winning on convenience.
- **No ambient authority** buys a tiny blast radius, at the cost of plumbing: capabilities must be threaded to wherever they are needed, and that plumbing *is* the design work.
- **Revocation is not free.** Because a capability is a reference, revoking it needs indirection designed in from the start; systems that add revocation later find it does not retrofit.
- **Auditing "who can do what" is hard.** ACLs answer it by inspection; capability systems must trace the graph of who was handed what, which is why enterprises find the model unfamiliar.
- **Historical adoption is the honest weak point.** KeyKOS and EROS were technically excellent and commercially marginal; the model's wins are recent and mostly in constrained niches (seL4, [Workers' bindings](../prior-art/cloudflare-workers.md), Fuchsia, WASI). The pattern is *right* and has been *losing* for fifty years, which is worth knowing when betting on it.

## 4. Relevance to Sankalpa
It is P8, and P8 is load-bearing under the whole security architecture. Book 11 §Ch03 is the pattern specified: §1 rejects ambient authority by name, §2 defines a capability with the object-capability properties, §3 states "no authority without a grant," §4 attenuation, §5 revocation. Book 03 §Ch06 is the Capability Manager that enforces it, and Book 03 §Ch02 §2.2 puts the choke point in the Kernel API: `Invoke` is the only way to cause an effect and is fully capability-gated. Under our threat model (Book 11 §Ch01) — untrusted Packages, prompt-injectable planners — this pattern is what bounds every adversary.

## 5. What we adopt
- **The object-capability model, cited to its source** (Book 11 §Ch03 §2 names KeyKOS and E): unforgeable, specific, delegable only by attenuation, revocable.
- **No ambient authority, stated absolutely** (Book 11 §Ch03 §3): no transitive escalation, no "admin" capability implying others, no inheritance from identity or membership; the default is powerlessness and power is added one capability at a time.
- **Designation = authority, at a single choke point.** `Invoke` a Capability by reference (Book 03 §Ch02 §2.2). There is no generic "run this" verb — which is exactly the property that makes the confused deputy unexpressible: a planner cannot ask the Kernel to "use your authority on this path," because the Kernel has no authority of its own to lend.
- **Attenuation for delegation** (Book 11 §Ch03 §4) — the only safe way to pass authority onward.
- **Revocation designed in, not retrofitted** (Book 11 §Ch03 §5), and taken further than the classical model: RFC-0005 requires revocation to pierce secret memoization, closing the gap where a revoked grant's *materialized* effects outlive the grant.
- **POLA at every boundary** — plugin isolation (Book 11 §Ch10), planner isolation (Book 08 §Ch04), package grants (Book 12 §Ch02 §4).
- **The pattern validated in production** — Cloudflare Workers' bindings ([study](../prior-art/cloudflare-workers.md)) are capabilities at scale, and evidence this is deployable rather than academic.

## 6. How faithfully we apply it (and where we deviate)
- **Faithful on the invariants, extended in two directions.** First, **capabilities are unified with behavior**: classically a capability is authority over an object; ours is *authority plus a typed function* — a `Capability` describes a typed behavior with a signature and declared effects (Book 11 §Ch03 §2, Book 15 §Ch04). This is a genuine extension, and its payoff is that authority becomes *statically analyzable*: because a Capability declares its effects (Book 04 §Ch06), the compiler can validate policy against what a plan may do (Book 05 §Ch04) *before* execution. Classical capability systems check authority at invocation; we can check it at compile time, because our capabilities carry types.
- **Second, grants bind to verified code identity, not to a name.** Book 11 §Ch03 §2 requires a grant to record the authorized-against identity — version, publisher identity, signing key (Book 12 §Ch04) — and RFC-0008 forces re-authorization on upgrade. The classical model has nothing to say here because it assumed the code holding a capability was the code you gave it to. In a marketplace ([Cargo's `build.rs`](../prior-art/rust-cargo.md), [Docker's registries](../prior-art/docker.md)), the upgrade path is the attack, and "explicit grant" without this binding is defeated by the ordinary act of shipping version 1.1.
- **Capabilities coexist with identity, and this is a real tension.** A pure ocap system needs no identity at all. We have users, sessions, and channel identity binding (Book 11 §Ch08, RFC-0009) because humans must be attributed (Book 11 §Ch09) and approvals must be traceable to a person (§Ch07). We keep the pattern honest by an ordering rule: identity may *inform who is granted what*, but authority still flows only from grants — identity never *confers* authority (§Ch03 §3). This is the seam where ambient authority would re-enter if anyone gets tired, and it deserves adversarial attention in Phase 2.
- **The audit weakness is answered by the event stream.** "Who can do what?" is hard to answer in a capability graph. Our answer is P5/P10: every grant and every `Invoke` is an event (Book 03 §Ch02 §3), so the authority graph is reconstructible from history ([event sourcing study](event-sourcing.md)) rather than inspected in place. That is a different answer than an ACL's, and it should be tested against a real auditor's question before we call it sufficient.
- **The plumbing cost is paid at the Kernel, not by every author.** Classical ocap makes each program thread its capabilities. Ours are granted to subjects and checked at one mediation point (P4), which is less pure — a true ocap purist wants no central checker — but is what makes the model usable by plugin authors who have never heard of Dennis and Van Horn.
- **Adoption history taken as a warning about ergonomics, not correctness.** KeyKOS and EROS did not fail on merit. If Sankalpa's capability model is unpleasant to *grant*, operators will grant broadly and the model degrades to ACLs with extra steps — so Book 12 §Ch05's install flow is where this pattern actually lives or dies.

## 7. Open questions
- Grant ergonomics are the adoption risk. If a Package declares fifteen capabilities (Book 12 §Ch02 §4), does the operator meaningfully review them, or click through — and is there a specified way to grant *attenuated* authority at install rather than all-or-nothing?
- RFC-0005 makes revocation pierce memoized secrets. Does revocation also reach an *in-flight* Execution that already holds materialized authority, and what are the semantics — abort, compensate ([Temporal study](../prior-art/temporal.md)), or run to completion?
- Identity informs grants but must never confer authority. Is there a mechanized check for that, or is it prose a future reviewer must uphold under pressure?
- Capabilities-as-typed-behavior is our extension. Does anything break when a *determinized* Capability (Book 10 §Ch06) is synthesized — who is it granted to, and does it inherit the reasoning step's authority or need a fresh grant?

## 8. References
- Dennis & Van Horn, "Programming Semantics for Multiprogrammed Computations" (1966); Hardy, "The Confused Deputy" (1988); Hardy et al. on KeyKOS; Shapiro et al., "EROS: A Fast Capability System" (1999); Miller, *Robust Composition* (2006) — the object-capability model, membranes, and POLA; Redell's revocation pattern (1974); Klein et al. on seL4; the [Cloudflare Workers](../prior-art/cloudflare-workers.md) and [microkernel](microkernel-architecture.md) studies.
