# Security Policy

*Status: Draft · Normative security architecture lives in [Book 11 — Security](spec/book-11-security/README.md).*

## Scope

This repository is a specification. "Security issues" here are primarily **design flaws** — a proposed mechanism that leaks secrets, escalates privilege, or violates the trust boundaries defined in Book 11. Once implementation begins, this policy will be extended to cover code vulnerabilities and a coordinated-disclosure process for released artifacts.

## Reporting a design vulnerability

Do **not** open a public issue for a suspected security-relevant design flaw. Instead email **security@sankalpa.dev** (placeholder) with:

- The document / RFC / mechanism affected.
- The trust boundary or invariant you believe is violated (reference Book 11).
- A concrete attack scenario.

You will receive an acknowledgment within **5 working days**. The Security Domain Lead coordinates triage and, where warranted, a corrective ADR/RFC.

## Non-negotiable security invariants

These invariants are load-bearing across the entire specification. Any proposal that weakens one is a security issue by definition:

1. **Natural language is never executable.** Only AOS IR executes.
2. **Secrets never enter** planner context, prompts, logs, memory, vector stores, or IR. They flow **by reference** through the Secret Broker only.
3. **Capability-based authority.** A component can do only what its granted capabilities permit — no ambient authority.
4. **Everything is governed by policy.** The Policy Engine validates plans *before* compilation and actions *before* execution.
5. **Every action is attributable and auditable** via the Event Bus and Experience record.

## Safe harbor

Good-faith research that respects user privacy and these invariants will not be pursued legally. Details will be finalized alongside the first released implementation.
