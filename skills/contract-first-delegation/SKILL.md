---
name: contract-first-delegation
description: 'Give an agent bounded freedom to choose and execute the route to a non-trivial outcome. Use when the user says "run with it", "use your judgment", "I care about the result, not the steps", "take this to completion", "give the agent more freedom", or requests implementation without prescribing the method. Differentiator: expands autonomy only inside explicit scope, authority, evidence, and approval boundaries. Do not use for pure explanations, trivial edits, or procedures whose exact sequence is mandatory.'
---

# Contract-First Delegation

Increase execution freedom by tightening the destination, fence, and proof. Let the agent choose the route unless route choice is part of safety or correctness.

## Activation test

Use this skill when all are true:

1. The user authorized a change, build, or completed outcome. A diagnose-only or review-only request remains read-only unless the user also requested implementation.
2. The task is non-trivial enough that route choice affects efficiency.
3. Multiple safe routes could satisfy the request.
4. Scope and authority can be stated without inventing permission.

Skip it for answer-only requests, simple one-step edits, and exact procedures such as migrations, legal flows, payment actions, or user-mandated sequences.

## State the contract

Before acting, state this compact contract:

```text
Outcome:
Scope:
Invariants:
Authority:
Verifier:
Independent review:
Stop condition:
Evidence and logging:
Measurement: baseline, denominator, observation window, success rule, proof boundary
```

Keep it proportionate. A short implementation may need one line per field. A cross-system or long-running task may need explicit file, account, budget, and rollback boundaries.

## Grant freedom inside the fence

Once the contract is clear:

- Inspect the real environment before choosing the route.
- Select the lowest-complexity path that satisfies the verifier.
- Make reversible, in-scope local changes without asking how to perform each step.
- Use existing project patterns, dependencies, and instructions.
- Run the verifier, identify the largest remaining gap, fix it, and repeat.
- Continue through ordinary implementation decisions when evidence can resolve them.
- Ask only when progress requires new authority, an external decision, unavailable access, or a material scope change.
- Keep the user informed during long work without converting updates into permission requests.

Do not confuse freedom with scope expansion. A better route may change implementation details, not the requested outcome, affected systems, public claims, spending authority, or irreversible-action boundary.

## Authority defaults

| Action | Default |
| --- | --- |
| Read, inspect, search, compare, calculate | Proceed |
| Reversible local edits inside an implementation request | Proceed |
| Local, non-mutating, non-billable tests, builds, validators, and browser checks | Proceed |
| New dependencies, lockfile regeneration, broad rewrites | Ask unless already authorized |
| External writes clearly required by the requested workflow | Proceed only within stated scope |
| Messages, account changes, purchases, payments, legal acceptance | Ask |
| Deploys, destructive migrations, deletes, force operations | Ask |
| Commit, push, merge, or production release | Ask unless explicitly requested |

An explicit budget delegates spending only inside its named purpose and limit. It never overrides legal, destructive, deploy, or account-level approval gates.

Any check that writes external data, sends messages, consumes money, changes an account, touches production state, or can be destructive inherits the stricter external-action gate. Calling it a test, validator, smoke check, or preview does not expand authority.

## Preserve route constraints

Follow exact steps when the route carries necessary information:

- Order protects data, security, or recoverability.
- A protocol, schema, contract, or byte-level output must match.
- Compliance, legal, financial, or safety rules prescribe the sequence.
- A durable gotcha records a known safe recovery path.
- The user explicitly requires a method or sequence.

Otherwise, treat user-provided steps as useful context, not an obligation to ignore a better evidenced route. Flag any meaningful deviation and explain why it improves the outcome or reduces risk.

## Verify without self-certifying

Use two verification layers:

1. **Deterministic checks:** tests, type checks, builds, validators, read-backs, screenshots, endpoint responses, and measurable thresholds. The builder may run these.
2. **Judgment review:** code review, design critique, claim review, edge-case search, and readiness judgment. For durable or high-risk changes, run this in a separately orchestrated fresh context.

Passing a review checklist inside the builder's active context does not create independence. Give the reviewer the outcome, scope, changed output, verifier evidence, and governing rules without the builder's defense narrative.

## Stop rules

Stop when one of these is true:

- The verifier passes and required independent review has no unresolved P0/P1 findings.
- Further work has plateaued against the defined bar.
- The iteration or budget limit is reached.
- Progress requires new user authority, unavailable access, or an external-state change.
- Evidence shows the requested outcome is impossible or harmful under the current constraints.

Do not silently weaken the verifier, redefine success, or substitute a nearby outcome.

## Closeout

Report:

```text
Outcome reached:
Evidence:
Route chosen:
Deviations from the initial approach:
Approvals still required:
Residual risk:
Logged where (optional):
```

Separate local verification, committed state, deployed state, and live/public truth.

A locally verified change may be complete while its live outcome remains unproven; never collapse those into one completion claim.

## Composition

- Do a first-principles pass first when the destination, mechanism, or assumptions are unclear. Use this skill only after the destination is sufficiently defined and the route is what remains open.
- Pair with a goal or loop mechanism when completion has a hard command or threshold, or when execution repeats with an unknown iteration count. Let that mechanism own persistence and completion state; let this skill own route choice, scope, and authority boundaries. Use one merged contract, not two announcements.
- Pair with a domain skill for implementation details.
- Run an independent code review in a separate fresh context after durable behavior changes.
