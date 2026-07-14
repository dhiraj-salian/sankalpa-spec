# Research (Phase 0)

*Status: Draft · Nature: Informative — research is **input** to the specification, never normative on its own.*

Phase 0 of the [roadmap](../ROADMAP.md) is deliberate, disciplined study before design. We do not invent what has been solved; we learn from the systems worth learning from, and we are explicit about **what we adopt, what we reject, and why**. Research documents feed RFCs and ADRs; they do not themselves change the spec.

## Structure

- [`prior-art/`](prior-art/README.md) — studies of specific systems (Linux, LLVM, Kubernetes, Temporal, Git, Terraform, Docker, Cloudflare Workers, PostgreSQL, Rust).
- [`patterns/`](patterns/README.md) — studies of architectural patterns (compiler design, DDD, CQRS, hexagonal, microkernel, event sourcing, capability-based security).

## Exit criterion for Phase 0

Every study listed in the two indexes below reaches an accepted state with a completed **"What we adopt / What we reject / Why"** section. Only then does Phase 1 (authoring the spec) formally begin. Studies may be revised later as understanding deepens.

## Study template

Each study MUST contain:

1. **System/pattern in one paragraph** — what it is and the problem it solves.
2. **Core ideas** — the 3–7 ideas that make it work.
3. **Design decisions & trade-offs** — the consequential choices its designers made.
4. **Relevance to Sankalpa** — which layer(s)/Book(s) it informs.
5. **What we adopt** — concrete ideas we take, and where.
6. **What we reject** — ideas that do not fit, and why.
7. **Open questions** — what it raises for our design.
8. **References** — primary sources.
