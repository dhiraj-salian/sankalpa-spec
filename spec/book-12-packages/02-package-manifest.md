# Book 12 · Chapter 02 — The Package Manifest

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P6, P8, P11. Part of the specification body of **AEP-0003**.*

> The manifest is a Package's declaration of identity: what it is, what it provides, what it depends on, and what authority it requires. This chapter specifies the manifest schema. It is the document the Package Manager reads to resolve dependencies (§Ch03), verify provenance (§Ch04), surface capability requirements (P8), and register contributions (Book 03 §Ch09 §3). A precise manifest is what makes the ecosystem analyzable and governable.

## 1. The manifest schema

Every Package MUST carry a manifest with, at minimum:

```
PackageManifest:
  identity:
    name           string          # globally unique package name
    version        SemVer          # the package's own version (P6, §Ch07)
    publisher      Identity         # the publishing identity (for provenance, §Ch04)
  provides:                         # what this Package contributes (§Ch01 §2)
    kinds          list<KindSchema>        # new ARM Resource kinds (Book 02 §Ch05)
    capabilities   list<CapabilityDecl>    # typed signatures + declared effects (Book 04 §Ch06)
    planners / runtimes / backends / channels / providers / controllers / policies /
    knowledgePacks / workflows / ui / docs  # each with the interface + version it implements
  requires:
    capabilities   list<CapabilityRequirement>   # authority its extensions need (§4, P8)
    dependencies   list<PackageDependency>        # other Packages + version constraints (§Ch03)
    compatibility:                                 # version-space compatibility (§Ch07)
      specVersion  range          # Sankalpa spec versions supported
      irVersions   range          # AOS IR versions (Book 04 §Ch09)
      aepVersions  map<AEP, range> # e.g. AEP-0002 range for a runtime
  conformance:
    claims         list<ConformanceClaim>   # suites passed + level (§Ch07, Book 06 §Ch07)
  integrity:
    signature      Signature       # publisher signature (§Ch04)
    provenance     Provenance       # supply-chain attestation (§Ch04)
```

The manifest is itself part of the signed artifact (§Ch04): its claims are only as trustworthy as the signature over them.

## 2. `provides` — declaring contributions

`provides` enumerates what the Package contributes, each entry naming the **interface and version** it implements (planner AEP-0001, runtime AEP-0002, package AEP-0003, controller SDK Book 07 §Ch03, etc.). On install, the Package Manager registers each contribution with the owning Manager (Book 03 §Ch09 §3) — a runtime with the Runtime Manager, a policy with the Policy Engine, a kind with the Resource Manager. A Package cannot contribute something it does not declare; declaration is the basis for registration, discovery, and governance.

## 3. `requires.dependencies` — depending on other Packages

A Package may depend on others (a runtime pack depending on a capability pack), declared as `PackageDependency` with **version constraints** (§Ch03). Dependencies are resolved deterministically at install (§Ch03); an unsatisfiable or conflicting dependency graph is rejected (§Ch03 §4). This is how the ecosystem composes: Packages build on Packages, with declared, resolvable version relationships.

## 4. `requires.capabilities` — declaring needed authority (P8)

Critically, a Package **declares the capabilities its extensions require** — the authority they need to function (a runtime needs secret-materialization + effect capabilities; a channel needs egress). This declaration is what installation **surfaces for explicit authorization** (Book 03 §Ch09 §5, Book 11 §03): a human/policy sees exactly what authority the Package is asking for and grants it (least-privilege) or refuses. Consequences:
- **Transparency.** The manifest makes a Package's authority requests legible *before* install — no hidden privilege escalation.
- **Least-privilege enforcement.** Grants are scoped to what is declared and needed; an extension cannot exceed its granted, attenuated capabilities (Book 11 §03 §4).
- **install ≠ privilege** (§Ch01 §4): declaration is a *request*, not a grant; nothing is authorized until a human/policy approves.

A Package that performs an effect requiring a capability it did not declare is denied at runtime (fail-closed, P8) — declaration is mandatory for authority.

## 5. `compatibility` — the version-space contract (§Ch07)

The manifest declares compatibility across Sankalpa's **independent version spaces** (Book 04 §Ch09 §1): the spec version, the IR version(s), and the relevant AEP version(s). The Package Manager enforces **joint consistency** (Book 03 §Ch09 §6): a Package requiring `irVersion ≥ X` or `AEP-0002 v2` is not installed where those are unavailable. This is what prevents a Package from being installed into an incompatible platform and failing mysteriously — incompatibility is caught at install, declared in the manifest, not discovered at runtime.

## 6. `conformance` — provable claims

A Package may claim conformance to interface suites (runtime Book 06 §Ch07, planner Book 08 §Ch07, controller Book 07 §Ch03) at a stated stability level. These claims are **verifiable** (the suites are executable) and surface in the Marketplace as trust signals (§Ch06). A conformance claim is a promise the ecosystem can check, not merely assert — consistent with the "verify, don't trust" posture toward all extensions (Book 11 §10 §4).

## 7. Invariants (normative summary)

1. Every Package carries a signed manifest declaring identity, provides, requires (capabilities + dependencies + compatibility), and conformance claims.
2. `provides` names each contribution's implemented interface and version; contributions are registered with their owning Managers on install; a Package cannot contribute what it does not declare.
3. Dependencies are declared with version constraints and resolved deterministically; unsatisfiable/conflicting graphs are rejected.
4. `requires.capabilities` declares needed authority, which installation surfaces for explicit least-privilege authorization; declaration is a request, not a grant (install ≠ privilege, P8).
5. `compatibility` declares spec/IR/AEP version support; the Package Manager enforces joint consistency, catching incompatibility at install, not runtime.
6. Conformance claims are verifiable via executable suites and surface as Marketplace trust signals.
