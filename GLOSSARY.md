# Glossary

*Status: Living document. Every term capitalized as a proper noun in the specification MUST be defined here. Add terms in the same PR that introduces them.*

Terms are grouped for reading but should be kept alphabetized within each group.

## Core loop

- **Intent** — A human-expressed desired outcome, in natural language. Non-executable. The top of the transformation chain.
- **Goal** — A structured, machine-derived objective refined from Intent. Still declarative, not yet a plan.
- **Plan** — An ordered/branching structure of steps that, if executed, achieves the Goals. Produced by a Planner.
- **AOS IR** — *Agent Operating System Intermediate Representation.* The **only** executable representation in Sankalpa. Has a High level (planner-facing, runtime-agnostic) and a Low level (compiler-produced, closer to execution).
- **Execution** — The runtime-specific realization of Low IR; produces Events.
- **Experience** — The first-class record of one Execution: its Intent, Goals, IR, runtime, metrics, outcome, feedback, and lessons. Feeds Knowledge.
- **Knowledge** — Curated, durable understanding (facts, architecture, policies, runbooks, relationships) that improves future Planning. *Not* short-term memory.
- **Determinization** — The process of converting repeated non-deterministic reasoning into a reusable, deterministic Capability.

## Substrate

- **Kernel** — The microkernel that owns orchestration. Nothing bypasses it; components communicate only via the Kernel API or the Event Bus.
- **Kernel API** — The synchronous request/response contract for interacting with the Kernel and its managers.
- **Event Bus** — The asynchronous publish/subscribe backbone. Every state change emits an Event.
- **Controller** — A reconciliation loop that drives a Resource's Actual State toward its Desired State.
- **Controller Runtime** — The Kernel subsystem that hosts and schedules Controllers.
- **Manager** — A Kernel-owned service for a domain (Resource, Capability, Scheduler, Runtime, Compiler, Plugin, Package, Security, Knowledge, Experience, Policy, Lifecycle, User, Session, Approval, Secret Broker, Observability).

## Resource model

- **ARM** — *Agent Resource Model.* The uniform model in which *everything is a Resource*.
- **Resource** — A versioned, observable, controller-backed entity with Metadata, Spec, Status, Desired State, Actual State, Lifecycle, and Events.
- **Desired State / Actual State** — The declared target vs. the observed reality of a Resource; the gap the Controller closes.
- **Spec / Status / Metadata** — The declared intent, the observed condition, and the identifying/annotating data of a Resource (Kubernetes-derived convention).

## Compiler & extension

- **Planner** — A plugin that transforms Goals into High AOS IR. Never knows about specific runtimes. (e.g., LangGraph adapter.)
- **Compiler** — Optimizes High IR, validates against Policy, and lowers it to Low IR and then to runtime-specific graphs.
- **Runtime** — A plugin that executes runtime-specific graphs (e.g., n8n, Temporal, Kubernetes, Bash).
- **Capability** — A reusable, deterministic unit of behavior a Plan can invoke; the target of Determinization.
- **Policy** — A machine-checkable rule set enforced by the Policy Engine before compilation and before execution.
- **Package** — A distributable, versioned bundle (capabilities, compilers, policies, knowledge packs, templates, workflows, providers, UI, docs, tests, controllers).
- **Plugin** — A runtime-loaded extension conforming to an AEP-defined interface (planners, runtimes, compilers, channels are plugins).

## Security

- **Secret Broker** — The sole custodian of secrets; delivers them to Execution **by reference**, never by value into any reasoning context.
- **Capability-based security** — Authority granted as unforgeable references to specific capabilities; no ambient authority.
- **Approval Engine** — The subsystem that gates actions requiring human authorization.

## Interfaces

- **Channel** — A transport adapter (Telegram, Slack, Email, REST, CLI, Web UI, Voice, MCP, …). Channels carry, they do not decide.
- **Web Runtime** — The integrated web layer for dashboards, approvals, rendered artifacts, file browsing, reverse proxy, and the API gateway.

## Process

- **RFC / ADR / AEP** — Request for Comments / Architecture Decision Record / Architecture Extension Proposal. See [`process/`](process/README.md).
- **FCP** — Final Comment Period; the fixed window before a proposal is accepted or rejected.
