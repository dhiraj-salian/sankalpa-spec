# Prior-Art Study: Linux

*Status: Accepted · Informs: Books 03 (Kernel), 00/01 (conformance, principles), and the stability rules in [`process/versioning-and-stability.md`](../../process/versioning-and-stability.md).*

## 1. System in one paragraph
Linux is a monolithic operating-system kernel that won the architecture argument it was supposed to lose. Its lasting lessons for Sankalpa are not about kernel internals but about **boundary discipline**: a syscall interface that has stayed compatible for three decades ("we do not break userspace"), a loadable-module system that admits extension without forking, and a monolith that survives precisely because its *external* contract is small and stable even though its interior is large. It is the strongest available counter-argument to our own microkernel choice (ADR-0002), which is why it is worth studying honestly.

## 2. Core ideas
1. **A stable syscall ABI as the product.** The kernel's public contract is the syscall surface; internal interfaces are explicitly *unstable* and refactored freely.
2. **"Don't break userspace"** as a non-negotiable, socially enforced rule — compatibility is a value judgment, not a technical accident.
3. **Loadable kernel modules** — extension without recompiling or forking the core, but loaded *into* the trusted address space.
4. **Everything is a file** — one uniform handle abstraction (open/read/write/close) over wildly different devices and services.
5. **Mechanism in the kernel, policy in userspace** — a deliberate (if imperfectly held) split.
6. **Distributed maintainership** — subsystem maintainers with a single integrator, and a mailing-list review culture that predates and outlives any tool.

## 3. Design decisions & trade-offs
- **Monolithic design** buys performance (no IPC on the hot path) and simple sharing of internal state, at the cost of a huge trusted computing base: any module defect is a whole-system defect.
- **Unstable internal interfaces** buy the freedom to refactor forever, at the cost of pushing an integration burden onto out-of-tree modules — a deliberate pressure toward upstreaming.
- **A stable ABI** buys an enormous ecosystem, at the cost of carrying mistakes permanently (old syscalls are never removed, only supplemented).
- **In-kernel modules** buy speed, at the cost of isolation — a lesson Sankalpa inverts (Book 11 §10).

## 4. Relevance to Sankalpa
Two places. First, Book 03: Linux is the honest alternative to our microkernel, and ADR-0002 must be argued *against* it, not around it. Second, everywhere we promise stability: the Kernel API (Book 03 §Ch02 §7) and the AEP interfaces make Linux's userspace promise to plugins, and Linux is the best evidence that such a promise is both keepable and expensive.

## 5. What we adopt
- **"Don't break userspace," restated as: don't break plugins.** The Kernel API, Event schemas, and AEP interfaces are Stable contracts governed like public APIs (Book 03 §Ch02 §7, [`versioning-and-stability`](../../process/versioning-and-stability.md)); P11 is worthless if the interfaces behind it churn.
- **Small, stable *external* contract; free interior.** Manager internals are refactorable; the Kernel API is not. This is what lets the core evolve without ecosystem breakage.
- **Additive-only evolution.** New capability arrives as new surface, not as changed meaning of existing surface — mirrored in event-schema evolution (Book 14 §Ch01 §4) and IR versioning (Book 04 §Ch09).
- **Mechanism/policy separation** — Book 03 §Ch01 §4's litmus test ("the core knows mechanism, never *which tool*") is Linux's split, sharpened.
- **A uniform handle abstraction.** "Everything is a file" is the ancestor of P2 ("everything is a Resource"): one anatomy, one set of verbs (Book 03 §Ch02 §2), many kinds.
- **Distributed maintainership with a single integrator** — reflected in [`MAINTAINERS.md`](../../MAINTAINERS.md) and the review gates.

## 6. What we reject / change
- **The monolith itself.** Linux's TCB includes every loaded module; Sankalpa's threat model (Book 11 §Ch01) assumes untrusted, third-party plugins — a malicious Package must not be a kernel-privilege event. We take the IPC cost knowingly (Book 03 §Ch01 §5) and isolate plugins instead (Book 11 §Ch10). Linux's performance argument does not apply to us: our hot path is *inside* a runtime, not across the Kernel per step, so mediation costs us far less than it would cost an OS.
- **Ambient authority.** The user/root model is exactly the ambient authority P8 rejects; capabilities replace it (Book 11 §Ch03).
- **In-tree extension pressure.** Linux nudges extensions to upstream into the core; we do the opposite — plugins stay *out* of the core, behind AEP interfaces, permanently (P11).
- **Never removing anything.** Linux's syscall table is a museum. Our versioning process allows deprecation on a stated schedule; we prefer a governed removal path to unbounded accretion.
- **Tacit conventions.** Much of Linux's contract lives in culture and lore; ADR-0001 requires ours to be written and reviewable (P12).

## 7. Open questions
- Is our Kernel API stability promise *as strong* as Linux's (never break, ever) or the weaker "deprecate on schedule"? The two imply different ecosystem economics and should be stated explicitly rather than assumed.
- Linux's internal-interface instability is what keeps it refactorable. Do we grant Managers the same freedom — and is "Manager-internal" a boundary we can actually hold once plugins start depending on observable behavior (Hyrum's Law)?
- eBPF is a live example of safely running untrusted code *in* a monolith via verification rather than isolation. Is there an analog for Sankalpa — verified plugin code that earns a faster path than full isolation?

## 8. References
- Linus Torvalds' "don't break userspace" mailing-list statements; the Tanenbaum–Torvalds debate (1992) on microkernels vs. monoliths; *Linux Kernel Development* (Love); `Documentation/process/stable-api-nonsense.rst`; Hyrum's Law.
