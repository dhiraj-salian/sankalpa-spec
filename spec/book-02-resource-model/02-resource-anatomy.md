# Book 02 · Chapter 02 — Resource Anatomy

*Nature: **Normative**. · Reflects: ADR-0002, RFC-0001; realizes principles P2, P5, P6, P8, P10.*

> This chapter defines the exact envelope every ARM Resource MUST have. Concrete kinds (Ch 07) specialize `spec` and `status`; they MUST NOT alter the envelope.

## 1. The envelope

Every Resource **MUST** be expressible as a document with exactly these top-level fields:

```
apiVersion   string    # "<group>/<version>", e.g. "core.sankalpa.dev/v1alpha1"
kind         string    # PascalCase kind name, e.g. "Intent"
metadata     object    # identity + system-managed bookkeeping (§2)
spec         object    # DESIRED state — the intent for this Resource (§3)
status       object    # OBSERVED state — reality as last reconciled (§4)
```

`apiVersion` and `kind` together identify the **schema** that governs `spec` and `status` (Ch 05). A Resource whose `spec`/`status` do not validate against the schema for its `apiVersion`/`kind` **MUST** be rejected by the Resource Manager at admission (Book 03 §05).

The canonical serialization of this envelope is defined in Ch 08; examples in this Book use YAML for readability, but YAML is not privileged.

## 2. `metadata`

`metadata` carries identity and system bookkeeping. Fields marked *system-managed* **MUST NOT** be set or mutated by clients; the Resource Manager owns them.

| Field | Type | Managed by | Meaning |
|-------|------|-----------|---------|
| `id` | string (ULID) | system | Globally unique, **immutable** identity assigned at creation. Never reused. |
| `name` | string | client/system | Human-meaningful name, unique within `workspace` + `kind`. |
| `workspace` | ref | client/system | The Workspace (tenancy boundary) this Resource belongs to. Root Resources may be workspace-less. |
| `labels` | map<string,string> | client/system | Indexed key/values for selection and grouping. |
| `annotations` | map<string,string> | client/system | Non-indexed metadata for tools; not for selection. |
| `generation` | int64 | system | Incremented **only** when `spec` changes. Identifies a desired-state version. |
| `resourceVersion` | string (opaque) | system | Storage-level optimistic-concurrency token (Ch 08). Changes on any write. |
| `createdAt` | timestamp | system | RFC 3339 creation time. Immutable. |
| `updatedAt` | timestamp | system | RFC 3339 time of the last write. |
| `ownerRefs` | list<OwnerRef> | client/system | Ownership edges for cascading deletion (Ch 06). |
| `finalizers` | list<string> | controller | Named cleanup hooks that block deletion until cleared (Ch 04). |
| `deletionTimestamp` | timestamp? | system | Set when deletion is requested; presence means "terminating" (Ch 04). |

Rules:
- `id` **MUST** be assigned by the system and **MUST NOT** change for the life of the Resource. `name` **MAY** be mutable if the kind permits, but references (Ch 06) resolve by `id`.
- `generation` **MUST** increment if and only if `spec` changes; a `status`-only write **MUST NOT** change `generation`. This is the hinge of reconciliation (Ch 03).
- `labels` are for selection and **MUST** be indexable; `annotations` are opaque and **MUST NOT** be required for correctness.

## 3. `spec` — desired state

`spec` is the **desired state**: what this Resource should be or do. Its shape is defined per kind by the kind's schema.

- `spec` is authored by whoever holds the capability to create/update the kind — usually the system (a planner, compiler, or controller), sometimes a human (Ch 01 §3).
- `spec` **MUST** be self-contained desired intent, not observations. Anything the system *discovers* belongs in `status`, never in `spec`.
- Secret material **MUST NOT** appear in `spec`; only secret *references* may (P7, Ch 06 §Secret references).
- A change to `spec` is the signal that triggers reconciliation; it **MUST** bump `generation`.

## 4. `status` — observed state

`status` is the **observed state**: reality as the owning Controller last reconciled it. Its shape is defined per kind, but every `status` **MUST** include the common substructure below.

| Field | Type | Meaning |
|-------|------|---------|
| `observedGeneration` | int64 | The `metadata.generation` the Controller most recently acted on. If `observedGeneration < generation`, reconciliation is pending. |
| `phase` | string (enum) | The current lifecycle phase (Ch 04). A coarse, kind-independent state. |
| `conditions` | list<Condition> | Fine-grained, orthogonal state assertions (§5). |
| `actualState` | object | The kind-specific observed reality that `spec` is trying to achieve. |

- Only the owning **Controller** (P3) may write `status`. Clients **MUST NOT** write `status` directly; the Resource Manager **MUST** reject such writes.
- `status` writes **MUST NOT** change `generation` and **SHOULD** be performed through a dedicated status sub-operation (Book 03 §05) so that spec and status authority are separable and independently capability-gated (P8).

### 4.1 Desired vs. actual, located precisely

The desired/actual distinction (P3) is realized structurally: **desired state = `spec`; actual state = `status.actualState`.** A Resource is *converged* when `status.actualState` satisfies `spec` and `observedGeneration == generation`. Chapter 03 defines convergence and drift formally.

## 5. `conditions`

`conditions` express orthogonal, machine-readable assertions about a Resource. The model follows the Kubernetes condition convention because it has proven durable and tool-friendly.

```
Condition:
  type               string     # e.g. "Ready", "PolicyValidated", "Compiled"
  status             enum        # "True" | "False" | "Unknown"
  reason             string     # machine token, e.g. "SecretUnavailable"
  message            string     # human explanation
  observedGeneration int64       # the generation this condition reflects
  lastTransitionTime timestamp   # when status last changed
```

Rules:
- `type` values are defined per kind and **MUST** be documented in the kind's schema. Every kind **SHOULD** define a `Ready` condition summarizing fitness for use.
- Conditions are **orthogonal**: multiple may be `True` simultaneously; they are not a state machine (that role belongs to `phase`, Ch 04).
- Controllers **MUST** set `observedGeneration` on conditions so consumers can tell whether a condition reflects the current desired state.

## 6. Events

Beyond the persisted envelope, every Resource participates in the Event Bus (P5). On every admitted create/update/delete and on significant `status`/`condition` transitions, the responsible component **MUST** emit an Event (Book 03 §03, Book 14 §01). Events are **not** stored inside the Resource; the Resource is the current state, the Event stream is the history. Event payloads **MUST NOT** contain secret values (P7).

## 7. Worked example

An `Intent` Resource shortly after creation, before planning has run:

```yaml
apiVersion: core.sankalpa.dev/v1alpha1
kind: Intent
metadata:
  id: 01J9Z3K7Q8ABCDEF0123456789   # system-assigned ULID
  name: weekly-report
  workspace: ws-acme
  generation: 1
  resourceVersion: "4821"
  createdAt: "2026-07-14T09:00:00Z"
  updatedAt: "2026-07-14T09:00:00Z"
spec:
  utterance: "Every Monday, summarize last week's sales and email it to the team."
  channel: slack://acme/#sales
  submittedBy: user/01J9...
status:
  observedGeneration: 0        # controller has not yet acted
  phase: Pending
  conditions:
    - type: Ready
      status: "False"
      reason: AwaitingGoalDerivation
      message: "Goals have not yet been derived from this Intent."
      observedGeneration: 0
      lastTransitionTime: "2026-07-14T09:00:00Z"
  actualState: {}
```

Here `observedGeneration (0) < generation (1)`: reconciliation is pending. The Intent controller (Book 08) will derive `Goal` Resources, link them (Ch 06), and advance `phase`/`conditions`.

## 8. Invariants (normative summary)

An implementation of ARM **MUST** guarantee:

1. Every Resource has exactly the five envelope fields; `spec`/`status` validate against the schema for `apiVersion`+`kind`.
2. `metadata.id` is system-assigned, globally unique, immutable, and never reused.
3. `generation` increments iff `spec` changes; `status` writes never change `generation`.
4. Only the owning Controller writes `status`; clients cannot.
5. No secret value appears in `metadata`, `spec`, `status`, or any emitted Event (P7).
6. Every admitted mutation emits an Event (P5).
7. Desired state is `spec`; actual state is `status.actualState`; convergence is defined in Ch 03.
