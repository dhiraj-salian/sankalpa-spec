# Pattern Study: Microkernel / Plug-in Architecture

*Status: Accepted · Informs: Book 03 (Kernel) — the pattern ADR-0002 adopts; Books 11 §Ch10, 12.*

## 1. Pattern in one paragraph
The microkernel pattern splits a system into a **small trusted core** that owns only mechanism and invariants, and a set of **plugins** that provide all actual functionality, loaded through defined interfaces and unable to bypass the core. It began as an operating-system argument — Mach, L4, seL4 against the monolith — and lost that argument commercially (Linux won) while winning it everywhere else: LLVM's passes and targets, VS Code's extension host, Kubernetes' API server plus controllers, browsers' process-per-site. The reason is that the pattern's cost (IPC on every interaction) is fatal where the interactions are nanoseconds apart and negligible where they are milliseconds apart. ADR-0002 adopts it for exactly that reason.

## 2. Core ideas
1. **A minimal trusted core.** Only what *must* be trusted is in it — the TCB is a liability to be minimized, and every addition is a permanent one.
2. **Mechanism in the core, policy at the edges** (Mach's formulation; Linux honors it partially, seL4 rigorously).
3. **Plugins provide the function.** The core is useless alone, and that is the design working.
4. **Mediated communication.** Plugins talk through the core, never directly, so the core is the single place invariants can hold.
5. **Isolation and fault containment.** A plugin crash is a plugin's problem; a monolithic module's crash is everyone's.
6. **Stable interfaces as the contract.** The extension interface is the product; if it churns, the ecosystem does not form.
7. **Small enough to reason about.** seL4's formal verification was possible *because* the kernel is ~10k lines — minimality is what makes assurance affordable.

## 3. Design decisions & trade-offs
- **A small core** buys a small TCB, assurance, and replaceability, at the cost of IPC on every interaction. This is the whole argument, and it is empirical: whether the pattern is right depends entirely on the ratio of mediation cost to work done between mediations.
- **Mediation** buys a single enforcement and audit point, at the cost of latency and of the core becoming a throughput bottleneck.
- **Isolation** buys fault containment and safe untrusted code, at the cost of serialization, copying, and context switches.
- **Stable extension interfaces** buy an ecosystem, at the cost of freezing early design mistakes into permanent contracts.
- **The distributed-monolith failure mode** is the pattern's characteristic pathology: the core accretes plugin-specific knowledge one reasonable exception at a time, until you have paid all the indirection cost and kept none of the replaceability.
- **The historical verdict** deserves respect: Mach's performance made microkernels a punchline for a decade, and L4/seL4 recovered it only by being ruthless about minimality. "Microkernel" is not automatically a virtuous word.

## 4. Relevance to Sankalpa
It *is* the Kernel's architecture. ADR-0002 adopts it explicitly, citing OS microkernels, LLVM, VS Code, and Kubernetes as the proof set; Book 03 §Ch01 is the rationale chapter; P4 ("the Kernel mediates all communication") is the pattern's central rule promoted to a principle. Book 03 §Ch01 §4's in-core/never-in-core list and its litmus test are the minimality discipline written down, and §5 states the costs honestly rather than pretending mediation is free.

## 5. What we adopt
- **The core/plugin split with AEP-defined interfaces** (ADR-0002): Managers are the trusted core; planners, runtimes, compilers, channels, providers, and controllers are plugins.
- **Mediated communication as an invariant** (P4): the Kernel API (sync) or the Event Bus (async), never a direct call. Book 03 §Ch02 §3's uniform request pipeline — authenticate, authorize, admit, policy, execute, audit — is the payoff, and it is the honest answer to "why is the indirection worth it": the invariants hold *in exactly one place, by construction*.
- **Mechanism in the core, never "which tool"** (Book 03 §Ch01 §4) — Mach's rule, with a litmus test attached: could a third party replace this without changing the core? Then it is a plugin.
- **Isolation of plugins** (Book 11 §Ch10) — informed by VS Code's extension host and browser sandboxing, and going further because our plugins are untrusted by threat model (Book 11 §Ch01), not merely third-party.
- **Capability-passing instead of ambient authority** (P8, [study](capability-based-security.md)) — the seL4/KeyKOS lineage, which is where the microkernel tradition and the capability tradition are the same tradition.
- **The distributed-monolith risk named and structurally resisted** (ADR-0002 §risk, Book 03 §Ch01 §5): the core depends only on AEP interfaces, never on a concrete plugin. Naming the failure mode in the ADR is how it stays visible.
- **Minimality as a standing discipline** — the in-core list is finite and enumerated (Book 03 §Ch01 §4), so growth is a decision, not a drift.

## 6. How faithfully we apply it (and where we deviate)
- **The cost calculus is different, and it is why this works for us.** Linux rejected microkernels because a syscall's mediation cost dominates the work between syscalls. Our hot path — actual execution — runs *inside* a runtime, not across the Kernel per step (Book 03 §Ch01 §5), so mediation is amortized over network calls and model latency. This is the load-bearing argument, and it is worth stating as such: we are not adopting the microkernel because it is elegant, but because our cost ratio is the inverse of the OS's. If executions ever became fine-grained and Kernel-mediated per step, ADR-0002's reasoning would need re-examining.
- **Our core is *bigger* than a microkernel purist would allow, deliberately.** ADR-0002's Manager list is long (Resource, Capability, Scheduler, Runtime, Compiler, Plugin, Package, Security, Knowledge, Experience, Policy, Lifecycle, Controller Runtime, Observability, User, Session, Approval, Secret Broker). seL4 would call this a monolith with good manners. The justification is the minimality *criterion*, not the line count: each Manager is there because it enforces an invariant or defines the model (Book 03 §Ch01 §4), and none encodes knowledge of a specific plugin. But we should be honest that we have adopted the microkernel's *boundary discipline* rather than its *size* discipline — and that seL4's assurance benefit, which minimality buys, is therefore not available to us.
- **A compiler in the core.** No OS microkernel has an optimizing compiler inside the TCB. Ours does (Book 03 §Ch08), because the pipeline is where P1 and P9 are enforced — the policy pass cannot be a plugin (Book 05 §Ch01 §3). Passes and backends *are* plugins, run untrusted and re-verified, which is the compromise that keeps this defensible.
- **Not everything is pluggable.** The pattern taken to its limit would make policy, verification, and pipeline order extension points. We refuse (Book 05 §Ch01 §3): pluggable governance is not governance, and this is the line where extensibility (P11) yields to security (P9) per the precedence rule in Book 01 §Ch04 §3.
- **Plugins are hostile, not just foreign.** Mach's servers were written by the OS team; ours come from a marketplace (Book 12 §Ch06). So the pattern acquires obligations it never had: signing and provenance (Book 12 §Ch04), grants re-authorized on upgrade (RFC-0008), declared effects, and revocation (Book 11 §Ch03 §5).
- **Degradation is specified.** Book 03 §Ch13 asks what a microkernel does when the core is overloaded — backpressure and shedding (Book 03 §Ch02 §4) rather than unbounded queueing. The OS literature answers this poorly, and a mediating core that queues without bound converts a load spike into an outage.

## 7. Open questions
- **Core growth has a litmus test but no budget.** Book 03 §Ch01 §4's test — could a third party replace this without changing the core? — is applied per addition, and §5 names core-growth pressure as a standing risk. Neither is a *gate*: there is no stated ceiling on Manager count, no review requirement specific to core growth in [`review-gates.md`](../../process/review-gates.md), and no measurement that would reveal drift. The distributed monolith is constructed entirely from decisions that individually pass a litmus test. This is the study's central objection and it survives the audit.
- **Formal assurance is off the table, and that should be a stated decision.** Book 11 §Ch11 consolidates SI-1 through SI-13 as *"the checklist the security review gate enforces"* — a rigorous, well-organized, and thoroughly **human** instrument. seL4's argument is that minimality is what makes mechanized proof affordable, and §6 above concedes we have not taken the size discipline that would buy it. Whether some subset (SI-1's "only IR executes", SI-3's no-transitive-escalation) is a candidate for mechanized checking, or whether assurance rests permanently on reviewers, is nowhere on the record.

*Checked and answered, contrary to an earlier draft of this study: horizontal scaling of the Kernel (Book 03 §Ch13 specifies Kernel replication, with the single-writer guarantee surviving it via leader election plus a `resourceVersion` backstop, and a partitioned replica that cannot confirm leadership required to stop writing — fail-closed rather than risk dual-write). P4 survives replication intact.*

## 8. References
- Liedtke, "On µ-Kernel Construction" (1995) and "Toward Real µ-Kernels" (1996); Klein et al., "seL4: Formal Verification of an OS Kernel" (2009); Accetta et al. on Mach; the Tanenbaum–Torvalds debate (1992); ADR-0002; the [Linux](../prior-art/linux.md), [Kubernetes](../prior-art/kubernetes.md), and [capability-based security](capability-based-security.md) studies.
