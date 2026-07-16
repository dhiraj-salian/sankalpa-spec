# Roadmap

*Status: Draft · This roadmap sequences the **specification**, not the implementation. Implementation phases (Phase 3+) begin only after `spec/` reaches a stable v1.0.*

The project's north star: **transform human intent into deterministic execution**, and continuously convert repeated reasoning into reusable deterministic capabilities.

## Phase gates

Each phase ends at a **gate** with explicit exit criteria. We do not advance a phase because time passed; we advance because criteria are met.

### Phase 0 — Research (complete, out of order)
Study the systems worth learning from and the patterns worth reusing. **No implementation, no normative spec.** Output: [`research/`](research/README.md) documents.
- **Prior art:** Linux, LLVM, MLIR, Kubernetes, Temporal, Git, Terraform, Docker, Cloudflare Workers, PostgreSQL, Rust.
- **Patterns:** compiler design, DDD, CQRS, hexagonal architecture, microkernel architecture, event sourcing, capability-based security.
- **Exit criteria:** every prior-art and pattern study has an accepted summary with explicit "what we adopt / what we reject / why" sections.
- **Status:** all 18 studies `Accepted`, each carrying the required sections. **The criterion is met; the gate's purpose is not.** These were authored *after* Phases 1–2, so they did not inform the design they describe, and they were accepted without independent review (every Domain Lead is vacant). [`research/README.md`](research/README.md#provenance-and-its-limits) records both limits. This is the one gate the project advanced through by time passing rather than by criteria being met in sequence — noted here rather than quietly closed, since §"Phase gates" above claims we do not do that.

### Phase 1 — Author the specification
Produce the complete normative spec in [`spec/`](spec/README.md): all 16 Books, seeded RFCs/ADRs/AEPs, resource model, kernel/compiler/controller/event/package/knowledge/security specs, API specs, diagrams.
- **Exit criteria:** every Book reaches `Draft`-complete (no empty chapters); the end-to-end "intent → execution" narrative is internally consistent; the Resource Model and AOS IR are fully specified with examples.

### Phase 2 — Harden the architecture
Adversarial review. Find inconsistencies, missing failure modes, unspecified lifecycles. Iterate until stable.
- **Exit criteria:** `spec/` tagged **v1.0**; two independent reviewers sign off per Book; no open blocking objections.

### Phase 3 — `sankalpa-core`
Implement **only** the substrate: Kernel, Event Bus, Resource Manager, Kernel API, Lifecycle Manager, Controller Runtime, Plugin Manager, Package Manager. **No planners, no runtimes, no AI.**

### Phase 4 — ARM (Agent Resource Model)
Implement the resource model, storage, versioning, and reconciliation.

### Phase 5 — AOS IR
Implement the High and Low IR data structures, validation, and serialization.

### Phase 6 — Compiler pipeline
Passes, optimization, policy validation, lowering framework.

### Phase 7 — First runtime compiler: n8n
Prove IR → real execution graph end-to-end against one runtime.

### Phase 8 — Knowledge system
Obsidian-compatible vault ⇄ graph database synchronization.

### Phase 9 — Secret Broker
Reference-based secret flow; one-time HTTPS credential pages; OAuth-preferred acquisition.

### Phase 10 — Planner interface
The stable planner plugin contract (AEP-defined).

### Phase 11 — First planner adapter: LangGraph
Prove intent → IR end-to-end against one planner.

### Phase 12 — Experience engine
Capture, store, and feed Experience back into planning; detect determinization opportunities.

### Phase 13 — Marketplace
Package publishing, discovery, signing, and installation across capabilities, compilers, policies, knowledge packs, runtimes, planners, and UI.

## Sequencing rationale

The order is a **topological sort of dependency, not of excitement.** The kernel and resource model come first because everything is a Resource with a controller. IR precedes any compiler; the compiler precedes any runtime; the planner interface precedes any planner. AI arrives *last among the core primitives*, deliberately — Sankalpa's value is the deterministic machine around the model, and building the model in first would tempt us to leak non-determinism into layers that must stay deterministic.
