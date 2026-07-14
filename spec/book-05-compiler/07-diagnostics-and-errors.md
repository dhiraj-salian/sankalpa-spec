# Book 05 · Chapter 07 — Diagnostics and Errors

*Nature: **Normative**. · Reflects: RFC-0001; realizes principles P7, P10, P12.*

> A compiler is only as useful as its error messages. This chapter defines the diagnostic model: how the compiler reports what is wrong, at which stage, in a way that is structured, deterministic, secret-free, and actionable. Diagnostics are the feedback that teaches planners (and their authors) to emit valid, permitted, executable IR.

## 1. Diagnostics are first-class

Every non-success outcome of compilation is a **structured diagnostic**, not a bare string or a stack trace. A diagnostic is data the system can route, aggregate, and learn from (Book 10), and that a human or a planner can act on.

```
Diagnostic:
  code        string        # stable machine token, e.g. "E-POL-042", "E-TYP-003"
  severity    Error | Warning | Info
  stage       Verify | Optimize | PolicyValidate | Determinize | Lower | VerifyLow
  subject     Ref?          # offending node/edge/type by canonical id (Book 04 §Ch07)
  message     string        # human explanation, secret-free (§4)
  remediation string?       # what would make it compile/comply, when known
  policyRef   Ref?          # for policy diagnostics: the policy id + version (Ch 04)
  evidenceRef Ref?          # for determinization diagnostics: supporting evidence (Ch 06)
```

- **Errors** stop compilation (`Compilation` → `Failed`, Ch 08). **Warnings** do not stop it but are recorded (e.g. a deprecated IR construct, Book 04 §Ch09 §6). **Info** conveys notable but benign facts (e.g. "reasoning node determinized to Capability X").
- Codes are **stable and versioned**: a code's meaning does not silently change (P12), so tooling and docs can rely on it.

## 2. Staged, so the failure is unambiguous

Because the pipeline is ordered (Ch 01 §2), every diagnostic names the **stage** that produced it, which tells the reader *whose problem it is*:

| Stage | Typical failure | Whose problem |
|-------|-----------------|---------------|
| `Verify` (High) | ill-formed/ill-typed/under-annotated High IR | the **planner** (Book 08) emitted invalid IR |
| `Optimize` | (rare) a pass produced invalid IR | a **compiler pass** defect (caught by re-verify) |
| `PolicyValidate` | plan forbidden by policy | the **plan/user** — adjust or seek approval (Ch 04) |
| `Determinize` | substitution gate failed (info/warn) | benign; fell back to reasoning (Ch 06) |
| `Lower` | backend cannot honor an ExecPolicy | **runtime selection** — no suitable backend (Ch 05 §3) |
| `VerifyLow` | compiler produced invalid Low IR | an **internal compiler defect** (Book 03 §Ch08 §3) |

This staging is why the pipeline order matters for *diagnosis*, not just correctness: a `Verify`-stage error is the planner's to fix; a `VerifyLow`-stage error is the compiler's own bug and is surfaced as such.

## 3. Determinism of diagnostics

Diagnostics **MUST** be deterministic: the same input module under the same configuration produces the same diagnostics, in the same order (canonicalized by subject id). Rationale: reproducible failures are debuggable and cacheable; a compiler that reported different errors on re-run would be untrustworthy. Diagnostic production is itself a total function (a compilation always terminates with success or a diagnostic set, never a hang — IR-P8, Ch 02 §3).

## 4. Secret-freedom (P7)

Diagnostics are surfaced to users, logs, Experience, and sometimes back to planner authors — all high-fanout surfaces. Therefore a diagnostic **MUST NOT** contain secret material, even when reporting on a `SecretRef` node: it names the *reference* and the *violated rule*, never a value (consistent with Book 04 §Ch08 §4, Book 03 §Ch12 §3). A diagnostic that could leak a secret is a security defect (Book 11), not a mere usability issue.

## 5. Actionability

Wherever the compiler *can* say what would fix the problem, it **MUST** populate `remediation`. Examples:
- Type mismatch → *"insert a checked Convert from List<SalesRow> to Table at n2.data"* (Book 04 §Ch05).
- Undeclared effect → *"declare Network(write,'email') in node n3's effect set"* (Book 04 §Ch06).
- Policy deny → *"route through an approved internal relay Capability"* (Ch 04 §4).
- Retry on non-idempotent write → *"supply an idempotency key or remove the retry policy"* (Book 04 §Ch04 §4).

Actionable diagnostics close the loop with planners: a planner that receives a precise remediation can correct its High IR and re-submit, turning a rejection into a fast iteration rather than a dead end.

## 6. Diagnostics feed learning (P10, P13)

Every diagnostic is recorded on the `Compilation` Resource (Ch 08) and emitted as an Event (Book 03 §Ch03). The Experience engine (Book 10) aggregates them: recurring `Verify` failures from a planner reveal a systematic planning weakness; recurring `PolicyValidate` denials reveal a mismatch between what users want and what policy allows (a signal for humans to revisit either). Thus diagnostics are not just error reports — they are a data source that improves planning, policy, and the platform over time.

## 7. Invariants (normative summary)

1. Every non-success outcome is a structured, stage-tagged, stable-coded diagnostic — not a string or stack trace.
2. Errors fail the compilation; warnings/info are recorded without stopping it; codes are stable and versioned (P12).
3. The stage identifies responsibility: `Verify`→planner, `PolicyValidate`→plan/user, `Lower`→runtime selection, `VerifyLow`→internal compiler defect.
4. Diagnostics are deterministic and produced by a total process; the same input yields the same diagnostics.
5. Diagnostics never contain secret material, even about `SecretRef` nodes (P7).
6. Diagnostics carry remediation when known and are recorded/emitted so Experience can learn from recurring failures.
