# Prior-Art Study: Rust / Cargo

*Status: Accepted · Informs: [`process/`](../../process/README.md) (RFC/ADR/AEP), Book 12 (packages), [`process/versioning-and-stability.md`](../../process/versioning-and-stability.md).*

## 1. System in one paragraph
Rust is two prior arts wearing one name. As **governance**, it is the RFC process — a written, public, numbered record of every consequential decision, with a period for objection, an explicit accept/reject, and a norm that no significant change lands without one; it is the direct ancestor of [`process/rfc-process.md`](../../process/rfc-process.md). As an **ecosystem**, Cargo plus the editions/stability system is the demonstration that a language can evolve fast and hard for a decade without breaking its users, by making stability a mechanism (channels, editions, lock files) rather than a promise. Both halves are about the same thing: how a project keeps its shape while many strangers change it.

## 2. Core ideas
1. **The RFC process.** Substantial changes need a written RFC, public discussion, a Final Comment Period, and a team decision — and the *artifact persists* as the reasoning of record.
2. **Stability channels.** Stable/beta/nightly, with unstable features gated behind explicit opt-in and never leaking into stable — instability is a place, not a mood.
3. **Editions.** Opt-in, per-crate language epochs that allow breaking syntax changes while crates of different editions interoperate — evolution without a Python-3 event.
4. **SemVer with a lock file.** Declared ranges for humans, an exact resolved graph for reproducibility.
5. **Multiple versions of a dependency can coexist** in one graph, so the diamond problem is a build detail rather than an ecosystem deadlock.
6. **One canonical toolchain.** Build, test, docs, publish, and format behind one command — ecosystem-shaping ergonomics.
7. **Documentation and tests as build-system citizens** (`cargo doc`, doctests): the tooling makes the good behavior the easy one.

## 3. Design decisions & trade-offs
- **RFCs for everything substantial** buys durable, discoverable rationale and legitimacy, at the cost of throughput — RFC backlog and burnout are recurring, real problems.
- **Editions** buy syntactic evolution without an ecosystem split, at the cost of permanent multi-edition complexity in the compiler.
- **Never breaking stable** buys immense trust, at the cost of shipping mistakes forever (`std::mem::forget` and friends) — the same bill Linux pays ([study](linux.md)).
- **A permissive central registry** buys ecosystem velocity, at the cost of supply-chain exposure: crates.io is trust-by-default, name-squatted, and typo-squattable; `cargo install` runs `build.rs` with your full ambient privilege.
- **Coexisting dependency versions** buy resolution simplicity, at the cost of binary bloat and confusing type mismatches across versions.
- **A single blessed toolchain** buys coherence, at the cost of centralization.

## 4. Relevance to Sankalpa
Governance first: our RFC/ADR/AEP split ([`process/`](../../process/README.md)) is Rust's RFC process with an addition — spec-first (ADR-0001) means an accepted RFC must be *reflected into normative text* to be Final (Book 15 §Ch04), which Rust does not require because it has an implementation to be the truth and we deliberately do not, yet. Then Book 12: Cargo's manifest, SemVer-plus-lock resolution (§Ch03 §2–3), and compatibility contract (§Ch07) are the model, and crates.io's supply-chain posture is the anti-model.

## 5. What we adopt
- **The RFC as the unit of consequential change**, with public discussion, an FCP, and an explicit decision ([`rfc-process.md`](../../process/rfc-process.md)) — and, as Rust proved, a *numbered, permanent* artifact so the reasoning outlives the participants.
- **Lighter paths for lighter changes.** Rust does not RFC a typo fix; we do not RFC prose fixes or additive research (`rfc-process.md` §17). A process everyone routes around is worse than none.
- **Stability tiers with explicit opt-in.** Alpha/Beta/Stable for the Kernel API and AEP interfaces ([`versioning-and-stability.md`](../../process/versioning-and-stability.md), Book 03 §Ch02 §7); unstable surface must be reachable only deliberately.
- **SemVer for declaration, exact resolution for reproducibility** (Book 12 §Ch03 §2–3) — a lock file is what makes "deterministic resolution" true rather than aspirational.
- **Conflicts rejected, not papered over** (Book 12 §Ch03 §4) — Cargo's refusal to guess is why its resolutions are trustworthy.
- **Editions as the model for governed breaking change.** Book 04 §Ch09 (IR versioning) and Book 12 §Ch07 face exactly Rust's problem: evolve the language without splitting the ecosystem. Editions are the best known answer, and we should reach for them before reaching for a major version.
- **Tooling makes the discipline cheap.** Rust's ecosystem documents and tests because `cargo` made it trivial. Our conformance suites (Books 06 §Ch07, 08 §Ch07) and manifest `conformance` claims (Book 12 §Ch02 §6) must be similarly frictionless, or P12 is a wish.

## 6. What we reject / change
- **Publish-implies-trust.** `cargo install` executes `build.rs` with the developer's full ambient authority — the sharpest ambient-authority failure in a modern ecosystem, and precisely what P8 exists to prevent. In Sankalpa, install grants *nothing*: capabilities are declared in the manifest (Book 12 §Ch02 §4), granted explicitly per capability, and **re-authorized on upgrade** (RFC-0008), because inheriting authority across a publisher change is how this attack actually lands.
- **Unauthenticated identity in the registry.** Grants bind to a verified publisher identity and signing key (Book 11 §Ch03 §2, Book 12 §Ch04), not to a name — names get transferred, sold, and squatted.
- **Implementation as the source of truth.** Rust's RFC describes what will be built; the compiler ends up as the real spec. ADR-0001 inverts this: the spec is authoritative, and an RFC is not Final until the normative text reflects it (`rfc-process.md`).
- **The "never break stable, ever" absolute.** Rust and Linux both pay this bill permanently. We keep a governed deprecation path (`versioning-and-stability.md`), and should be explicit that our promise is "deprecate on a stated schedule," not "carry mistakes forever."
- **Coexisting versions of the same dependency.** Reasonable for linked code; wrong for us. Two versions of a Capability contributing to one plan would make effects, policy, and grants ambiguous — Book 12 §Ch03 §4 rejects conflicts instead.
- **Unreconciled install.** `cargo install` is imperative; ours is a reconciled, transactional plan (Book 12 §Ch03 §5) — P3 has no exceptions for packages.
- **RFC-process throughput risk, taken seriously.** Rust's backlog is a warning, not a footnote: [`review-gates.md`](../../process/review-gates.md) must keep the heavy path narrow, or the process becomes the bottleneck it was meant to prevent.

## 7. Open questions
- Rust's RFC volume nearly stalled it. With Sankalpa's much smaller maintainer set, is the RFC path narrow enough — and what is the tripwire that says it has become the bottleneck?
- Editions require a compiler that can hold several language versions at once. Is the analogous cost acceptable for AOS IR (Book 04 §Ch09) — or is our content-addressed, replayable IR *forced* into edition-like behavior anyway, since an in-flight Execution must replay against its original IR ([Temporal study](temporal.md))?
- crates.io's quality floor is `cargo test` plus community norms. What is the marketplace's floor (Book 12 §Ch06) — mandatory conformance suites, signing, curation, or all three — and who pays for enforcing it?

## 8. References
- The Rust RFC repository and process documentation; Rust's edition guide and stability/channel documentation (`stability-as-a-deliverable`); the Cargo book (SemVer, resolution, lock files); crates.io policies; RustSec advisories on `build.rs` supply-chain exposure.
