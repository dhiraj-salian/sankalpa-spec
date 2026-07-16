# Pattern Study: Hexagonal Architecture (Ports & Adapters)

*Status: Accepted · Informs: Books 03 (Kernel, plugins), 06 (runtimes), 08 (planners), 13 (channels).*

## 1. Pattern in one paragraph
Hexagonal architecture (Alistair Cockburn's "ports and adapters") inverts the usual layered dependency: instead of the application depending on the database and the UI, the application defines **ports** — interfaces expressed in its own vocabulary — and every external technology attaches as an **adapter** implementing a port. The core depends on nothing outward; everything outward depends inward. The test is simple and merciless: *can you replace any external technology without touching the core?* Sankalpa's P11 is that question made a principle.

## 2. Core ideas
1. **The core defines the interface, not the periphery.** A port is written in the domain's language, not the technology's — the moment a port mentions SQL or Slack, the pattern has failed.
2. **Dependency inversion at the boundary.** The core never imports an adapter; adapters import the core.
3. **Driving vs. driven sides.** Driving adapters (UI, API, CLI) call in; driven adapters (databases, queues, services) are called out to. Same idea, opposite direction of control.
4. **Adapters are symmetric and swappable.** A real database and an in-memory fake are the same kind of thing to the core.
5. **Testability falls out.** With every external thing behind a port, the core is testable in isolation — which is the pattern's most-cited benefit and its best diagnostic.
6. **The technology is a detail, deliberately deferred.**

## 3. Design decisions & trade-offs
- **Ports in the domain's vocabulary** buy genuine replaceability, at the cost of an impedance mismatch in every adapter — someone must translate, and the translation is where the bugs live.
- **Dependency inversion** buys a core with no outward dependencies, at the cost of indirection: reading a flow means chasing interfaces, and the whole thing feels like ceremony until the day the technology changes.
- **Symmetric adapters** buy testability and portability, at the cost of a lowest-common-denominator port — a port that fits three databases exposes what all three share, which may be less than any one of them is good at. This is the pattern's central tax, and the one most often paid unknowingly.
- **Deferring technology** buys optionality, at the cost of never using a technology's best feature.

## 4. Relevance to Sankalpa
This is the pattern behind P11 ("everything is extensible and replaceable... no such component may be irreplaceable") and behind the microkernel's plugin boundary (ADR-0002, [microkernel study](microkernel-architecture.md)). Every one of Sankalpa's extension points is a port: the planner interface (Book 08 §Ch03), the runtime interface (Book 06 §Ch02), lowering backends (Book 05 §Ch05), channel adapters (Book 13 §Ch02), providers (Book 03 §Ch04), and the storage contract (Book 02 §Ch08 §1 — properties, not a product). Book 03 §Ch01 §4's litmus test — "could this be removed and replaced by a third party without changing the core?" — is Cockburn's question, restated as a rule for admitting code into the trusted core.

## 5. What we adopt
- **Ports as the extension mechanism, defined by AEP** (P11): planners, runtimes, compilers, channels, and providers are adapters behind interfaces the core owns and the periphery implements.
- **The core depends only on interfaces, never on a concrete plugin** (Book 03 §Ch01 §5's "core-growth pressure" is resisted structurally, not by good intentions).
- **Ports in *our* vocabulary.** A channel adapter speaks the channel model (Book 13 §Ch02: "channels are transport, not intelligence"), not Slack's; a runtime adapter consumes Low IR, not its own DSL. This is what stops a vendor's concepts colonizing the middle of the system — the same job the anti-corruption layer does in [DDD](domain-driven-design.md).
- **Driving/driven symmetry.** Channels and the API gateway (Book 13) drive in; runtimes, providers, and storage are driven out. Both sides are plugins; neither is privileged.
- **Adapter substitutability proven by conformance suites** (Books 06 §Ch07, 08 §Ch07). This is where we go past the pattern: Cockburn's replaceability is a design aspiration verified by unit tests, and ours is a *published, executable contract* a third party must pass. An interface without a conformance suite is a promise, not a port.
- **Storage as a port** (Book 02 §Ch08 §1) — the lesson learned from Kubernetes' etcd entanglement ([study](../prior-art/kubernetes.md)).
- **Testability as the diagnostic.** If the core cannot be exercised without a real planner or runtime, the boundary is wrong.

## 6. How faithfully we apply it (and where we deviate)
- **Faithful in shape, stricter in enforcement.** Classical hexagonal is a discipline sustained by developer taste, and it erodes quietly when someone imports a driver into the core. Ours is enforced by *process and runtime*: the microkernel boundary (P4 — components MUST NOT communicate directly), out-of-process plugin isolation (Book 11 §Ch10), capability grants (P8), and an architecture review gate. Cockburn's pattern has no answer to a hostile adapter; ours must, because our adapters come from a marketplace (Book 12 §Ch06).
- **Adapters are untrusted, not merely external.** This is the deviation with the most consequence. In hexagonal architecture the adapter is *your* code, written by your team, wrapping their technology. In Sankalpa an adapter may be a signed Package from a stranger (Book 12 §Ch04), so the port carries obligations the pattern never contemplated: declared effects (Book 04 §Ch06), explicit capability grants, isolation, and re-verification of its output (Book 05 §Ch01 §3). A port is a *trust boundary* (Book 11 §Ch02) as much as an interface.
- **The lowest-common-denominator tax, paid deliberately.** IR-P7 requires runtime-agnostic IR, which means the runtime port exposes what all conforming runtimes share — so a runtime's distinctive strength is reachable only via declared extension (a `Custom` effect, Book 04 §Ch06 §2) or not at all. We accept this consciously: portability of meaning (the same IR, the same semantics, on any conforming runtime) is worth more than any one runtime's best feature. Book 06 §Ch04's selection logic is where that tax gets negotiated.
- **Not every boundary is a port.** The policy checkpoint, the verify gates, and the pipeline order are fixed and non-pluggable (Book 05 §Ch01 §3) because they enforce invariants. Hexagonal, applied without judgment, would make them adapters too; Book 03 §Ch01 §4 explicitly refuses ("if no — because it enforces an invariant or defines the model — it is core"). Pluggable governance is not governance.
- **Ports at the transport level too.** The Kernel API is transport-independent (Book 03 §Ch02 §5), which is the pattern applied one level below where it is usually drawn.

## 7. Open questions
- The port-vocabulary rule is easy to state and hard to hold. Does the runtime interface (Book 06 §Ch02) leak *any* concept from a specific engine — and would we notice, given our reference runtime is likely to be the first implementer of it?
- If a `Custom` effect is how a runtime exposes what only it can do, is a plan that uses one still portable (IR-P7), or have we invented a portable IR with unportable programs — the fate of every "portable" abstraction that shipped an escape hatch?
- Conformance suites make ports real, and they are expensive. Does *every* port get one, or only planners and runtimes? Channels (Book 13 §Ch02) and providers (Book 03 §Ch04) currently have none.

## 8. References
- Cockburn, "Hexagonal Architecture" (2005); Martin, *Clean Architecture* (the dependency rule); Palermo, "Onion Architecture"; Freeman & Pryce, *Growing Object-Oriented Software, Guided by Tests* (ports and test doubles); the [microkernel](microkernel-architecture.md) and [DDD](domain-driven-design.md) studies.
