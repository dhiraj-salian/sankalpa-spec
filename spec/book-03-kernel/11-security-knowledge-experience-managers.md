# Book 03 · Chapter 11 — Security, Knowledge, and Experience Managers

*Nature: **Normative** (their Kernel-integration contract) with pointers to their full Books. · Reflects: ADR-0002; realizes principles P7, P8, P9, P13.*

> Several Managers are *core* (part of the trusted Kernel) but are specified in depth in their own Books, because their subject matter is large. This chapter fixes their **Kernel-integration contract** — how they plug into the Kernel API (§Ch02), the Event Bus (§Ch03), and the other Managers — and points to the authoritative Book for the rest. It exists so Book 03 remains the complete *map* of the core without duplicating Books 09/10/11.

## 1. Security-plane Managers → Book 11

These Managers collectively enforce the security invariants (P7, P8, P9) at the single front door (§Ch02 §3). Their full specification is Book 11; their Kernel contract:

### 1.1 Security Manager
- The enforcement point invoked by the Kernel API request pipeline (§Ch02 §3) for **authentication** and **authorization/capability checks** (P8). No Kernel API request proceeds without passing it.
- Orchestrates, but does not replace, the Capability Manager (§Ch06) — it decides *whether this caller may* and delegates *what a capability permits* to the Capability Manager. Full spec: Book 11 §§03, 08.

### 1.2 Policy Engine
- Provides two enforcement hooks: **control-plane policy** at the Kernel API (§Ch02 §3 step 4) and the **compiler policy-validation pass** on High IR (§Ch08 §3, Book 05 §04) — the two P9 checkpoints, complemented by the runtime checkpoint (Book 14 §06).
- Policies are Resources (Book 02 §Ch07) and may be shipped as Packages (Book 12). Full spec: Book 11 §06.

### 1.3 Secret Broker
- The **sole** custodian of secret values (P7). It owns a **protected store separate from the ARM store** (Book 02 §Ch08 §6) — deliberately *not* a normal Resource-backed store, so a full ARM-store compromise exposes no secret.
- Kernel contract: it issues `SecretRef`s (Book 02 §Ch06 §4), and materializes values **only** at execution, **only** to the selected runtime, **only** under a capability grant (§Ch06 §5, §Ch07 §3.1). No other Manager, plugin, log, Event, or IR ever holds a value. Full spec: Book 11 §04.

### 1.4 Approval, User, Session Managers
- **Approval Engine** — gates actions that require human authorization (materialized, e.g., as an approval instruction in Low IR, Book 04 §Ch10 A.4); it renders through channels/web runtime (Book 13 §06) and records non-repudiable decisions.
- **User / Session Managers** — identities, roles, and authenticated interaction contexts feeding the Security Manager. `Session` is a Resource (Book 02 §Ch07). Full spec: Book 11 §§07–08.

## 2. Knowledge Manager → Book 09

- **Kernel contract:** the Knowledge Manager owns the vault ⇄ graph knowledge system and exposes knowledge to planners (Book 08) as retrievable, provenance-tagged, **non-secret** context. It MUST NOT surface any secret material into planning context (P7, Book 08 §04).
- It is **core**, but its backends (the graph database, the vault store) are **provider plugins** (P11) — the Manager knows the provider interface, not a specific database (§Ch04 §1). Knowledge units are Resources (Book 02 §Ch07). Full spec: Book 09.

## 3. Experience Manager → Book 10

- **Kernel contract:** the Experience Manager subscribes to the Event Bus (§Ch03) to assemble an `Experience` Resource per Execution (Book 02 §Ch07), redacting secrets by construction (the stream is already secret-free, §Ch03 §8). It feeds two loops:
  - **Into Knowledge** (Book 09) — promoting lessons with provenance.
  - **Into planning and the compiler** — surfacing cost/health signals to runtime selection (§Ch07) and **determinization candidates** to the Compiler Manager (§Ch08 §2, P13).
- The determinization pathway is the platform's compounding advantage (Book 01 §05): the Experience Manager is where "we keep doing this reasoning" becomes a signal, and the Capability Manager (§Ch06 §6) is where it becomes a deterministic Capability. Full spec: Book 10.

## 4. Why these are core, not plugins

Each of these Managers either **enforces an invariant** (Security, Policy, Secret Broker, Approval) or **owns a cross-cutting system of record** (Knowledge, Experience) that the whole platform depends on. By the §Ch01 §4 litmus test, they cannot be third-party-replaceable — a plugin that could replace the Secret Broker or the Policy Engine could nullify P7/P9. Their *backends and providers*, however, are pluggable (P11), which is where extensibility safely lives.

## 5. Invariants (normative summary)

1. Security Manager and Policy Engine enforce authn/authz (P8) and policy (P9) at the Kernel API front door and, for policy, at the compiler and runtime checkpoints.
2. The Secret Broker is the sole secret custodian, with a protected store separate from ARM; it materializes values only at execution, only to the selected runtime, only under a capability grant (P7).
3. The Knowledge Manager exposes only non-secret, provenance-tagged context to planners; its backends are provider plugins.
4. The Experience Manager assembles secret-free Experience from the Event stream and feeds Knowledge, planning, and determinization (P13).
5. These Managers are core because they enforce invariants or own systems of record; only their backends/providers are replaceable.
