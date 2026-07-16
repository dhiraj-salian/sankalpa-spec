# Prior-Art Study: Kubernetes

*Status: Accepted (exemplar) · Informs: Books 02 (ARM), 03 (Kernel), 07 (Controllers), and the API conventions throughout.*

## 1. System in one paragraph
Kubernetes is a container orchestration platform whose real contribution is a **declarative, API-centric control plane**: users declare desired state as versioned API objects, and independent controllers continuously reconcile actual state toward it. Its architecture — a small set of core API machinery plus many controllers and pluggable components — has proven extraordinarily extensible and durable.

## 2. Core ideas
1. **Everything is an API object** with `apiVersion`, `kind`, `metadata`, `spec`, `status`.
2. **Declarative desired state + reconciliation.** Controllers watch objects and drive reality toward `spec`, reporting in `status`.
3. **Level-triggered, not edge-triggered.** Controllers reconcile from observed state, so they self-heal after missed events.
4. **The API server is the single front door.** All components talk to it; none talk directly to each other. etcd is the state of record.
5. **Extensibility via CRDs + custom controllers** and admission webhooks — new kinds without forking the core.
6. **Strong API conventions & versioning** (alpha/beta/stable) with conversion between versions.

## 3. Design decisions & trade-offs
- **API-server-mediated communication** buys uniformity and a single enforcement/audit point, at the cost of indirection and API-server load.
- **Level-triggered reconciliation** buys robustness under failure, at the cost of some latency and the need for idempotent controllers.
- **etcd as the single consistent store** buys correctness, at the cost of a scaling bottleneck.
- **Eventual consistency** across controllers is accepted in exchange for availability and simplicity.

## 4. Relevance to Sankalpa
Kubernetes is the closest existing analog to Sankalpa's substrate: it is essentially a microkernel (API server + core) with reconciling controllers over a uniform resource model — exactly the shape of Books 02/03/07.

## 5. What we adopt
- The **Metadata/Spec/Status + Desired/Actual state** anatomy for ARM (Book 02).
- **Kernel-mediated communication** (our P4) mirrors the API-server-as-front-door rule (ADR-0002).
- **Level-triggered, idempotent reconciliation** as the controller model (Book 07).
- **Alpha/Beta/Stable API versioning** conventions, adapted for our spec and AEP stability levels.
- **Extend-via-new-kinds** (CRD analog) as the default over modifying core kinds.

## 6. What we reject / change
- **Container-centric assumptions** — Sankalpa's "workload" is an AOS IR execution graph, not a pod; runtimes are pluggable and not container-bound.
- **etcd-specific storage** — we specify storage *properties* (Book 02 §08), not a mandated store.
- **YAML-as-primary authoring** — humans express *Intent* in natural language; Resources are largely system-derived, not hand-written manifests.
- **Ambient cluster trust** — Kubernetes' historically weak default authority model is explicitly replaced by capability-based security (P8, Book 11).

## 7. Open questions
- Do we need Kubernetes-style admission webhooks, or does the compiler's policy-validation pass (Book 05 §04) subsume that role?
- How much of the reconciliation machinery is needed for *ephemeral* Resources like a single Execution vs. long-lived ones like a Service?

## 8. References
- Kubernetes API Conventions; "Kubernetes: Up and Running"; Borg/Omega/Kubernetes (Burns et al., 2016); the controller/reconciliation literature.
