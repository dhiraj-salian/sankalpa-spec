# Book 07 · Chapter 06 — Writing a Controller

*Nature: **Informative** (a worked walkthrough; the normative rules are §Ch01–§Ch05). · Illustrates P3, P5, P8.*

> This chapter walks through writing a controller for a core kind, to make the abstract contract (§Ch03) concrete. It is informative: where it and a normative chapter differ, the chapter wins. The example is the **Intent controller** (Book 08 §Ch02), which derives Goals from an Intent — a representative controller that reads one kind, invokes a plugin, and produces owned Resources.

## 1. The example: the Intent controller

**Job:** reconcile an `Intent` (Book 02 §Ch07) toward its desired state — Goals derived from the intent's utterance (Book 08 §Ch02). It owns the `Intent` kind's status and produces `Goal` Resources the Intent owns.

**Descriptor** (`describe`, §Ch03 §1): owns `Intent`; requires the capability to invoke a Planner (Book 08 §Ch03) and to create `Goal` Resources; supports the current `Intent` apiVersion.

## 2. The reconcile, step by step

Following the model (§Ch01 §1) — observe, diff, act, emit:

```
reconcile(intentRef):
  intent = Get(intentRef)                       # OBSERVE current state (given id, not event — §Ch01 §2)
  if intent.deletionTimestamp: return           # (deletion handled in finalize, §4)

  # DIFF: does actual satisfy desired for this generation?
  if intent.status.observedGeneration == intent.generation
     and goalsExistFor(intent): return done      # converged — idempotent no-op (§Ch01 §3)

  # ACT: derive goals via the Planner (a granted capability — P8)
  result = Invoke(planner.deriveGoals, {intent})   # only through a held grant (§Ch03 §2 obl.6)
  if result is ClarificationRequest:
     setCondition(intent, Ready=False, reason=AwaitingClarification)   # PROGRESS-OR-EXPLAIN (§Ch01 §4)
     surfaceToHuman(result)                        # Book 13
     return requeue(after=...)
  if result is error:
     setCondition(intent, Ready=False, reason=PlanningFailed); return error   # backoff (§Ch02 §4)

  # ACT: create the derived Goals, owned by the Intent (idempotent — existence-checked, §Ch05 §3)
  for g in result.goals:
     ensureCreated(Goal, ownerRef=intent, spec=g)  # existence check avoids double-create

  # EMIT: write own status only (§Ch03 §2 obl.3); Events follow from the status write (P5)
  setStatus(intent, observedGeneration=intent.generation,
                    actualState={goals: ...}, conditions=[Ready=True])
  return done
```

Every normative obligation appears here: level-triggered observe (§Ch01 §2), idempotent act via existence checks (§Ch01 §3, §Ch05 §3), progress-or-explain conditions (§Ch01 §4), own-status-only writes (§Ch03 §2), capability-gated invocation (P8), and Event emission via status (P5).

## 3. What the author does NOT write

Notice everything *absent* from §2 — because the Controller Runtime provides it (§Ch02 §1):
- No watch setup, no event handling, no queue, no coalescing.
- No leader election or single-writer logic (§Ch02 §5).
- No rate limiting or backoff loop (the author just returns `error`/`requeue`; the runtime backs off, §Ch02 §4).
- No crash-recovery code (re-list + re-reconcile is automatic, §Ch05 §2).

The author writes *domain logic* — "an Intent should have Goals derived by a planner" — and inherits correctness. This is the payoff of the shared runtime (§Ch02 §1).

## 4. Handling deletion

```
finalize(intentRef):                              # invoked on deletion, finalizer present (§Ch03 §4)
  intent = Get(intentRef)
  # cleanup external effects THIS controller created; owned Goals cascade via ownerRef GC
  releaseAnyExternalRefs(intent)                   # idempotent (§Ch05 §3)
  removeOwnFinalizer(intent)                       # remove ONLY our finalizer (§Ch03 §4)
  return done
```

Owned `Goal`s are reclaimed by owner-based cascading GC (Book 02 §Ch04 §5) through the normal deletion path — the controller need not delete them explicitly; it cleans up only what *it* created outside ARM.

## 5. Common mistakes (and which rule catches them)

| Mistake | Rule violated | Consequence |
|---------|---------------|-------------|
| Acting on the trigger event's payload instead of re-reading state | level-triggered (§Ch01 §2) | wrong/lost updates under duplicate/reordered delivery |
| Creating Goals without an existence check | idempotent (§Ch01 §3, §Ch05 §3) | duplicate Goals on re-reconcile |
| Writing another kind's status | single-writer (§Ch03 §2, §Ch04 §2) | races, contradictory state |
| Stalling with no condition when blocked | progress-or-explain (§Ch01 §4) | invisible hang, undebuggable |
| Invoking a capability it wasn't granted | no ambient authority (P8, §Ch03 §2) | denied — fail-closed (Book 11) |
| Assuming its Goals and some other Resource change atomically | no cross-Resource txn (§Ch04 §4) | broken invariant under partial failure |

Each mistake is caught by a normative rule — which is why the rules exist. A controller that honors §Ch01–§Ch05 is correct by construction; one that shortcuts them fails in exactly these ways.

## 6. Summary (informative)

Writing a controller is writing a `reconcile` (and `finalize`) that observes current state, drives actual toward desired idempotently, writes only its own status, progresses-or-explains, and acts within its grants — while the Controller Runtime handles watching, queueing, leader election, backoff, and recovery. The Intent controller shows the shape; every controller in Sankalpa — sync (Book 09), compilation (Book 05), execution (Book 06), package install (Book 12) — follows it.
