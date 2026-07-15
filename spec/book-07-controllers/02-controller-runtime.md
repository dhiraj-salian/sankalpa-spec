# Book 07 · Chapter 02 — The Controller Runtime

*Nature: **Normative**. · Reflects: ADR-0002; realizes principles P3, P4, P8. Companion to Book 03 §Ch10 (Lifecycle Manager & Controller Runtime), Book 11 §10 (isolation).*

> The Controller Runtime is the Kernel machinery that *hosts and schedules* controllers. Book 03 §Ch10 specified it as a Manager (its place in the Kernel); this chapter specifies its *operational contract* — the guarantees it provides to the controllers it hosts, and the discipline (work queues, rate limiting, leader election) that keeps reconciliation efficient and safe. It is why individual controllers can be simple: the hard parts are solved once, here.

## 1. Why a shared runtime

Reconciliation machinery — watching, triggering, queueing, rate-limiting, leader-electing, crash-recovering — is *mechanism shared by every kind* (Book 03 §Ch10 §5). Reimplementing it per controller would be wasteful and error-prone. The Controller Runtime provides it once, so a controller author writes only the *domain logic* (the `reconcile` for their kind, §Ch03) and inherits correct scheduling and safety. This is what makes P3 uniform and cheap across dozens of kinds.

## 2. Guarantees to a hosted controller

The Controller Runtime guarantees each controller (Book 03 §Ch10 §3.2):

1. **At-least-once, coalesced triggers.** It subscribes to the Event Bus on the controller's behalf (§Ch01, Book 03 §Ch03) and delivers reconcile triggers for the controller's subjects. Duplicate/bursty triggers are **coalesced** per subject id into a single pending reconcile — so a storm of Events collapses into reconciling current state once (level-triggered, §Ch01 §2).
2. **Single active ownership.** At most one active reconciler writes a given Resource's status at a time (§Ch04, Book 03 §Ch10 §3.1) — even across replicated Kernel instances (via leader election, §5).
3. **Bounded execution.** Rate limits and exponential backoff-with-jitter bound reconcile frequency, protecting the Kernel and external systems from hot loops (§4).
4. **Isolation.** A hosted controller runs at its trust level (§6); a plugin controller is isolated and contained (Book 11 §10) so its fault does not harm the runtime or other controllers.

These guarantees are what let §Ch01's simple loop be *correct in production*: the controller author does not implement coalescing, leader election, or backoff — they are provided.

## 3. Work queues

The runtime maintains, per controller, a **work queue keyed by subject id** (Book 03 §Ch10 §3.1):
- A trigger for subject S enqueues S (if not already queued) — this is the coalescing (§2.1): many triggers for S become one queued item.
- A worker dequeues S, reconciles current state, and — on failure — re-enqueues S with backoff (§4).
- Keying by id (not by event) is what makes coalescing and level-triggering work: the queue holds *subjects to reconcile*, not *events to process*.

## 4. Rate limiting and backoff

- **Rate limiting** bounds how often any subject (and the controller overall) reconciles, preventing a controller from overwhelming the Kernel API or an external system.
- **Exponential backoff with jitter** governs retries after a failed/incomplete reconcile: repeated failure backs off (so a persistently-failing Resource does not spin), and jitter prevents synchronized retry storms across many Resources.
- A slow or failing reconcile is thus *retried with backoff*, not spun — the difference between graceful degradation and a resource-exhausting hot loop (Book 03 §Ch13 §2).

## 5. Leader election and single-writer across replicas

The single-writer guarantee (§Ch04, Book 02 §Ch03 §4) must survive Kernel replication. The runtime uses **leader election** so that, even during a network partition, at most one replica actively reconciles a given Resource (Book 03 §Ch13 §5):
- A replica reconciles a Resource only while it holds leadership for it.
- A replica that cannot confirm leadership **stops writing** (fail-closed, Book 03 §Ch13 §1) rather than risk dual-write.
- **Optimistic concurrency** (`resourceVersion`, Book 02 §Ch08 §2) is the backstop: even a brief leadership overlap cannot corrupt state, because a stale writer's commit is rejected. Leader election reduces contention; `resourceVersion` guarantees correctness.

## 6. Trust and isolation of hosted controllers

- **Core controllers** (for core kinds) are part of the trusted Kernel.
- **Plugin controllers** (shipped by Packages for their own kinds, Book 12) are **untrusted** (Book 11 §10): isolated, resource-limited, granted only least-privilege capabilities (P8) for the kinds they own, and unable to write Resources they do not own (§Ch04). A plugin controller's fault is contained; its subjects' reconciliation stalls with a surfaced condition rather than crashing the runtime (§Ch05, Book 03 §Ch13).
- Either way, a controller acts through the Kernel API (P4) as an attributed, capability-scoped identity (Book 11 §08 §6) — never with ambient power.

## 7. Invariants (normative summary)

1. The Controller Runtime provides reconciliation mechanism once (watch, queue, coalesce, rate-limit, leader-elect, recover) so controllers implement only domain logic.
2. It guarantees hosted controllers at-least-once coalesced triggers, single active ownership, bounded execution, and isolation.
3. Per-controller work queues are keyed by subject id, coalescing bursts into one reconcile of current state and re-enqueuing with backoff on failure.
4. Rate limiting and exponential backoff-with-jitter bound reconcile frequency; failing reconciles back off rather than spin.
5. Leader election ensures at most one replica reconciles a Resource; a replica without confirmed leadership stops writing (fail-closed), with `resourceVersion` as the correctness backstop.
6. Core controllers are trusted; plugin controllers are isolated, least-privilege, contained, and act as attributed capability-scoped identities through the Kernel API (P4, P8).
