# Prior-Art Study: Terraform

*Status: Accepted · Informs: Books 07 (reconciliation), 02 (desired vs. actual), 03 §Ch04 (providers), 12 (packages).*

## 1. System in one paragraph
Terraform is declarative infrastructure-as-code whose two durable contributions are the **plan/apply split** — compute and *show the human* the diff between desired and actual state before touching anything — and the **provider model**, a plugin interface that let a small core acquire an enormous ecosystem of resource types it knows nothing about. Where Kubernetes reconciles continuously and invisibly, Terraform reconciles on demand and *shows its work first*. That difference is the interesting part.

## 2. Core ideas
1. **Desired state in code, actual state discovered** by refreshing from the real provider APIs.
2. **Plan as a first-class artifact.** The diff is computed, rendered, reviewable, and can be saved and applied later — an approval gate that is a data structure, not a meeting.
3. **Providers as plugins** behind a stable RPC interface: the core knows CRUD and dependency ordering, providers know AWS or Cloudflare or anything else.
4. **A dependency graph** derived from references between resources, driving parallelism and ordering without the author declaring it.
5. **State as an explicit, persisted artifact** mapping declared resources to real-world IDs — the source of both its power and its worst failure modes.
6. **Drift detection** — refresh reveals when reality has diverged from the last-known state.
7. **Version-pinned providers and modules** with a lock file, so a plan is reproducible.

## 3. Design decisions & trade-offs
- **Explicit state file** buys a clear mapping to real-world objects and fast planning, at the cost of state being *corruptible, lockable, and a secret-bearing artifact* — Terraform state famously stores secrets in plaintext, a design accident we must not repeat.
- **Plan/apply** buys human review of consequential change, at the cost of TOCTOU: the world can move between plan and apply, so a plan is a prediction, not a promise.
- **Reconcile-on-demand, not continuously** buys predictability and human control, at the cost of drift persisting silently until someone runs it.
- **Provider RPC isolation** buys a huge ecosystem and crash isolation, at the cost of a rigid CRUD shape that fits some resources badly.
- **HCL** buys readable declarations, at the cost of accreting a programming language by degrees (count, for_each, dynamic) — the standard fate of a config language.

## 4. Relevance to Sankalpa
Book 07's reconciliation model is Kubernetes-shaped ([study](kubernetes.md)), but Terraform contributes the parts Kubernetes lacks: a **plan the human approves before it happens** (Book 11 §Ch07, Book 13 §Ch06) and a **provider plugin model** (Book 03 §Ch04) proven at ecosystem scale. Book 12 §Ch03's deterministic dependency resolution and lock-file discipline are Terraform's (and Cargo's) lesson. Its state file is the clearest cautionary tale in the prior art for P7.

## 5. What we adopt
- **Plan/apply as a shape, not just a feature.** Our compiler *is* the plan step: it produces a reviewable artifact (Low IR + a Compilation record, Book 05 §Ch01 §5) before anything executes, and policy validates it (Book 05 §Ch04) before it is committed to execution. Where a human gate is required, the Approval Engine (Book 11 §Ch07) approves a *specific, content-addressed plan* — Terraform's saved plan, made unforgeable.
- **Show the diff.** Desired vs. actual (Book 02 §Ch03) is only useful if the delta is renderable to a human; Terraform is the evidence that this is the feature operators actually trust.
- **Providers as plugins behind a stable interface** (Book 03 §Ch04, P11): the core knows mechanism, the provider knows the system.
- **A derived dependency graph** driving ordering and parallelism (Book 04 §Ch03's explicit dataflow, IR-P9), rather than author-declared sequencing.
- **Drift detection as a named concern** — Terraform's refresh is the ancestor of RFC-0003's shadow sampling, which asks the same question of a determinized Capability: *is reality still what we assumed?*
- **Deterministic, locked resolution** (Book 12 §Ch03 §2) with version pinning, so the same declaration yields the same plan.
- **Out-of-process plugins** — provider crash isolation is the modest ancestor of Book 11 §Ch10.

## 6. What we reject / change
- **The state file.** Two things go wrong here and we reject both. First, **secrets in state**: Terraform's state holds materialized values in plaintext, which is exactly the P7 violation the Secret Broker exists to prevent — our state of record never stores secret values (Book 02 §Ch08 §6), only references. Second, **state as a separate, corruptible artifact** the operator must lock and back up: ARM *is* the state of record (Book 02 §Ch08), continuously reconciled, not a file to be `terraform import`-ed back into agreement with reality.
- **Reconcile only when a human runs it.** P3 requires continuous, level-triggered reconciliation (Book 07 §Ch01). Terraform's on-demand model means drift is normal between runs; ours means drift is a reconcilable condition the system notices itself.
- **Hand-written declarations.** HCL is authored by humans; ARM Resources are derived from Intent (Book 01 §Ch06 §2). We do not need — and must not grow — a configuration language, and HCL's slow accretion of loops and conditionals is the reason to say so now.
- **CRUD as the universal resource shape.** Our Resources have lifecycles (Book 02 §Ch04), not create/read/update/delete; an Execution or a Conversation does not fit CRUD.
- **Plan/apply TOCTOU accepted as-is.** Terraform tolerates the gap; we bind approval to a content-addressed artifact and re-validate at execution (Book 14 §Ch06), because for us the gap can span a human's approval delay.
- **Ambient provider credentials.** Providers authenticate from environment variables and ambient cloud credentials — the confused deputy waiting to happen. Capabilities are granted explicitly (P8) and secrets flow by reference (P7).

## 7. Open questions
- Terraform's provider interface and our Capability model both answer "how does the core act on a system it knows nothing about" — is a provider a Package that contributes Capabilities (Book 12 §Ch02 §2), or a distinct kind of plugin? Book 03 §Ch04 should be explicit rather than leaving the analogy loose.
- If a human approves a plan and the world drifts before execution, what precisely re-validates — the whole plan, or the effect set? RFC-0003 and Book 14 §Ch06 touch this from different sides.
- Terraform proves a rich ecosystem needs a *rigid, boring* plugin interface. Is our Capability signature (Book 04 §Ch05) rigid enough to be implementable by strangers, and boring enough to be stable for a decade?

## 8. References
- Terraform documentation on plan/apply, state, and the provider plugin protocol; *Terraform: Up & Running* (Brikman); HashiCorp's provider-SDK and state-encryption discussions; the "sensitive values in state" documentation.
