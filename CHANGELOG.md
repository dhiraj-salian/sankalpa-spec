# Changelog

All notable changes to the Sankalpa specification are recorded here. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this repository versions the **specification**, following [Semantic Versioning](https://semver.org/) applied to normative meaning (see [`process/versioning-and-stability.md`](process/versioning-and-stability.md)).

## [Unreleased]

### Added
- Initial repository scaffold: governance, contribution, and process framework.
- Templates for RFC, ADR, AEP, implementation plan, testing plan, security review, performance review, design doc.
- Specification skeleton (Books 00–15) with per-Book scope and chapter outlines.
- **Book 09 (Knowledge) authored to Draft-complete**: knowledge vs. memory (rejecting the memory-as-context anti-pattern), the knowledge taxonomy with mandatory provenance & trust, the Obsidian-compatible vault model, the graph model (replaceable backend), reconciled bidirectional vault⇄graph synchronization, knowledge in planning (non-secret, trust-weighted context), and knowledge packs. Completes the learning subsystem with Book 10.
- **Book 10 (Experience) authored to Draft-complete**: Experience as a first-class Resource (not memory), the Experience anatomy, the capture pipeline (deriving secret-free Experience from the Event stream), feedback into Knowledge, feedback into planning & cost models, the determinization *discovery* engine (pairing with Book 05's substitution passes to realize P13), and metrics & evaluation (ran ≠ succeeded). Closes the learning loop.
- **Book 11 (Security) authored to Draft-complete**: threat model (reasoning-context leakage as the defining threat), trust boundaries, capability-based security (P8), the Secret Broker and its separate protected store (P7), out-of-band credential acquisition, the Policy Engine and its three checkpoints (P9), the Approval Engine, identity/users/sessions, audit & attribution (P10 without exposing secrets), plugin isolation, and the consolidated SI-1…SI-12 security-invariant checklist.
- **Book 08 (Planners) authored to Draft-complete**: the role of planning as the compiler front-end and sole home of non-determinism, Intent→Goal derivation with human-in-the-loop clarification, the AEP-0001 planner interface, planner isolation & the no-secrets/no-execution-authority disciplines, the non-determinism boundary (typed Reasoning nodes that recede via determinization), reference-planner adapters, and the property-based planner conformance suite. Closes the intent→execution loop.
- **Book 06 (Runtimes) authored to Draft-complete**: runtime abstraction and the compiler/runtime boundary, the AEP-0002 runtime interface, execution semantics (determinism, idempotency/retries, saga compensation), runtime selection (fidelity/policy gates), reference-runtime mappings, secret materialization at execution only, and the runtime conformance suite. Completes the intent→execution arc.
- **Book 05 (The Compiler) authored to Draft-complete**: pipeline overview, pass framework (analysis/transform, purity, semantics preservation), optimization passes, the pre-execution policy-validation pass, the two-stage lowering framework, determinization passes (P13 substitution with safety gates and reversibility), the diagnostic model, and compilation-as-a-Resource.
- **Book 03 (The Kernel) authored to Draft-complete**: microkernel rationale, the Kernel API, the Event Bus, Managers overview, and full chapters for the Resource Manager, Capability Manager, Scheduler & Runtime Manager, Compiler Manager, Plugin & Package Managers, Lifecycle Manager & Controller Runtime, the security/knowledge/experience Manager integration contracts, the Observability Manager, and failure modes & degradation.
- **Book 02 (The Agent Resource Model) authored to Draft-complete**: uniform resource rationale, resource anatomy, desired-vs-actual reconciliation contract, lifecycle model, versioning & schema evolution, relationships & references (incl. secret references), core resource catalog, storage & consistency.
- **Book 04 (AOS IR) authored to Draft-complete**: IR rationale, design principles (IR-P1…P10), High IR, Low IR, type system, effect system, serialization & content addressing, verification, versioning & compatibility, worked examples.
- Seed decisions: ADR-0000, ADR-0001, ADR-0002; RFC-0000, RFC-0001; AEP-0000.
- Phase 0 research index for prior-art and pattern studies.
- Glossary, roadmap, security policy, code of conduct.

[Unreleased]: https://example.invalid/sankalpa-spec/tree/main
