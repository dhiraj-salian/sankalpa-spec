# Prior-Art Study: Cloudflare Workers

*Status: Accepted · Informs: Book 06 (runtimes, execution), Book 11 §Ch10 (plugin isolation), Book 03 §Ch07 (scheduling).*

## 1. System in one paragraph
Cloudflare Workers runs untrusted, multi-tenant code with sub-millisecond startup by rejecting the container model entirely: instead of a process per tenant, it uses **V8 isolates** — many tenants inside one process, separated by the language runtime's own memory safety — and pays for that choice with a strict capability-shaped API surface, a hard prohibition on ambient authority, and a deliberately constrained execution model. It is the best available evidence that **capability-based, no-ambient-authority design is a practical performance strategy**, not merely a security posture.

## 2. Core ideas
1. **Isolates, not containers.** Isolation comes from V8's memory-safe sandbox; startup is microseconds and memory per tenant is kilobytes, so density is orders of magnitude better.
2. **No ambient authority in the API.** There is no filesystem, no arbitrary sockets, no `process`. Capability comes from *bindings* — object references injected into the environment — and code holds exactly what it was given.
3. **Deny-by-default surface.** The runtime exposes web-standard APIs only; anything absent is absent on purpose, and every added surface is an audited decision.
4. **Constrained execution.** CPU-time limits, no long blocking, I/O-driven concurrency — the constraints are what make hostile multi-tenancy tractable.
5. **Determinism enforced by API design.** Timers and randomness are shaped (e.g. clock advancement tied to I/O) to close timing side channels — non-determinism is *removed from the API*, not discouraged in docs.
6. **Standards as the interface** (Service Workers, WinterCG) — portability by conforming to an existing contract rather than inventing one.
7. **Durable Objects** — single-threaded, addressable, stateful actors giving strong consistency per key without distributed transactions.

## 3. Design decisions & trade-offs
- **Isolates** buy density and instant start, at the cost of a *shared process*: a V8 escape crosses every tenant boundary at once. The bet is that V8's sandbox is the most-attacked and best-hardened one in existence — a bet only a few organizations can honestly make.
- **A restricted API** buys a small attack surface and portability, at the cost of "you cannot run arbitrary existing code here."
- **CPU limits** buy fair multi-tenancy, at the cost of ruling out long, compute-heavy work.
- **Bindings-as-capabilities** buys structural least privilege, at the cost of everything being explicit at deploy time — no ambient escape hatch when you need one.
- **Spectre-class defenses via API shaping** buy real mitigation, at the cost of an execution model that surprises developers who expect an ordinary clock.

## 4. Relevance to Sankalpa
Two places, one of them unexpected. Obviously Book 06: Workers is a lowering target and a model for a low-latency runtime. Less obviously **Book 11 §Ch03 and §Ch10**: Workers is the production existence proof for P8 — bindings *are* capabilities, and the platform demonstrates at scale that no-ambient-authority is deployable, fast, and commercially viable rather than an academic preference. It is also the strongest counterexample to "isolation must mean a container."

## 5. What we adopt
- **Bindings-as-capabilities as the validation of P8.** A subject holds object references and can do exactly what they designate — Book 11 §Ch03 §2's unforgeable, specific, delegable-only-by-attenuation properties, running in production today.
- **Deny-by-default surface.** Nothing is reachable unless granted (Book 11 §Ch03 §3: a subject with no grants can do nothing effectful). Workers shows this is a *design that ships*, not a design that stalls.
- **Non-determinism removed by API shape, not by convention.** This is Workers' deepest lesson and it agrees with our sharpest disagreement with Temporal ([study](temporal.md)): if determinism matters, the *representation or the API* must make violation impossible. Compare IR-P1 with `Time`/`Random` as declared effects (Book 04 §Ch06 §4).
- **Isolation specified as properties, with mechanism left open** (Book 11 §Ch10) — Workers proves the mechanism space is wider than containers, so mandating one would be a mistake.
- **Standards as the portability play** — conform to an existing contract where one exists (Book 13 §Ch07's MCP integration is the same instinct).
- **Constrained execution as a feature.** Book 03 §Ch07's scheduling and RFC-0007's admission, liveness, priority, and deadlines encode the same conviction: bounded work is governable work.
- **Single-writer state as the alternative to distributed transactions.** Durable Objects and Book 02 §Ch08 §4 ("no cross-Resource transactions") reach the same conclusion from opposite directions.

## 6. What we reject / change
- **Shared-process isolation as our default.** Workers can bet the company on V8's sandbox because it is Cloudflare; our threat model (Book 11 §Ch01) must hold for operators who cannot maintain a hardened isolate fleet. Book 11 §Ch10 therefore specifies properties and permits stronger mechanisms — an isolate is one acceptable implementation, not the specified one.
- **A JavaScript-shaped world.** Our executable artifact is verified IR (P1), not JS. Workers is a *lowering target* for it (Book 06 §Ch04), and its constraints — CPU limits especially — make it appropriate for some plans and not others; runtime selection must reason about that rather than assume it.
- **CPU/wall-clock limits as universal.** Sankalpa's executions include multi-day human approvals (Book 11 §Ch07) and durable waits ([Temporal study](temporal.md)); the Workers model excludes exactly that shape.
- **Deploy-time-only capability binding.** Bindings are fixed at deploy; our grants are revocable at runtime (Book 11 §Ch03 §5), and revocation must pierce even memoized secret material (RFC-0005). A static binding model cannot express that.
- **The platform's opacity.** Workers is a proprietary managed service; our runtimes are plugins behind an AEP interface (P11) with a conformance suite (Book 06 §Ch07) — no runtime may be irreplaceable, including a good one.
- **Edge-locality as an architectural assumption.** Placement is a runtime-selection concern (Book 06 §Ch04), not a property of the model.

## 7. Open questions
- Workers' timing defenses assume an adversary *inside* the sandbox. Our planners handle untrusted input (prompt injection, Book 11 §Ch01 §3) — is there an analogous "shape the API so the attack is unexpressible" move for planner isolation (Book 08 §Ch04), or is confinement the only tool?
- If a runtime cannot honor a plan's `ExecPolicy` (a long durable wait on a CPU-limited runtime), does runtime selection (Book 06 §Ch04) reject it statically from the IR's declared effects, or does it fail at execution? The former is the whole point of declared effects; §Ch04 should say so.
- Does Book 11 §Ch10 state a minimum isolation strength, and can an isolate-based runtime host plugins from unknown publishers under it?

## 8. References
- Cloudflare's "Cloud Computing without Containers" and isolate-architecture posts; the Workers runtime (`workerd`) source and security model documentation; Cloudflare's Spectre-mitigation writeups; Durable Objects documentation; the WinterCG minimum common API.
