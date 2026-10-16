---
name: autoresearch
description: Autonomous goal-directed iteration loop - set a goal, metric, and verify command, the agent iterates until convergence. Inspired by Karpathy's autoresearch. Use when the user says "autoresearch", "iterate until it works", "loop on this", "goal-directed loop", "converge on a solution", "run autoresearch", "keep iterating", or "self-improving loop". For one-shot adversarial decisions, use reason.
triggers:
  - "autoresearch"
  - "iterate until it works"
  - "loop on this"
  - "goal-directed loop"
  - "converge on a solution"
  - "run autoresearch"
  - "keep iterating"
  - "self-improving loop"
trigger: "autoresearch, optimize, improve metric, autonomous iteration, run the loop"
user_invocable: true
context: []
degrees_of_freedom: low
delivers: false
---

# /autoresearch - Autonomous Goal-Directed Iteration

Set a GOAL. Define a METRIC. Run the LOOP. Keep what works, revert what doesn't. Never stop until convergence or interruption.

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) and [Udit Goenka's autoresearch skill](https://github.com/uditgoenka/autoresearch) (MIT). Also borrows from [pi-autoresearch](https://github.com/davebcn87/pi-autoresearch), [ATLAS](https://github.com/chrisworsey55/atlas-gic) (General Intelligence Capital), and Ole Lehmann (skill testing).

## Invocation

```
/autoresearch                                    # interactive setup wizard
/autoresearch --goal "..." --verify "..." --scope "..."   # inline config
/autoresearch --iterations N                     # bounded run
/autoresearch --validator strict                 # validator judges every KEEP (default: convergence gate only)
/autoresearch plan                               # goal -> config wizard
/autoresearch fix                                # auto-fix all errors
/autoresearch debug                              # scientific bug hunting
```

## Interactive Setup (no args or `plan`)

Walk through 6 steps. Every gate is mechanical.

### Step 1: Capture Goal
Ask: "What do you want to improve?" Accept plain language.
Examples: "increase test coverage", "reduce bundle size", "improve Lighthouse score"

### Step 2: Define Scope
Suggest files based on goal. User confirms.
- **Modifiable:** files the loop can edit (glob patterns)
- **Read-only:** context files (never touched)
- **Protected:** test files, eval harnesses, configs (never modified)

### Step 3: Define Metric
Suggest metric from goal. Must be a NUMBER extractable from command output.
- Name (e.g., "coverage %", "bundle KB", "p95 ms")
- Direction: `higher` or `lower` is better

### Step 4: Define Verify Command
Suggest shell command that outputs the metric. DRY RUN it NOW.
- Must be runnable via Bash tool
- Must output a parseable number
- If dry run fails: fix or ask user. Do not proceed with broken verify.

### Step 5: Optional Guard
Ask: "Any command that must always pass? (tests, types, lint)"
Guard = regression prevention. Verify measures the goal. Guard prevents breaking everything else.

### Step 6: Extract Metric Pattern
After dry run, show the raw output to the user. Ask: "Which number is the metric?"
Define an extraction strategy:
- **Regex:** e.g., `(\d+\.?\d*)%` for percentages
- **Line match:** "the number on the line containing 'coverage'"
- **Last number:** last numeric value in the output
- **jq path:** for JSON output, e.g., `.coverage.pct`

Store this pattern. Use it EVERY iteration to extract the same number consistently. Never guess which number to extract.

### Step 7: Confirm and Go
```
GOAL: {goal}
SCOPE: {modifiable_files}
METRIC: {name} ({direction})
VERIFY: {command}
EXTRACT: {pattern}
GUARD: {command or "none"}
ITERATIONS: {N or "unlimited"}
BASELINE: {dry_run_result}
```

**Permission gate:** "This loop will make git commits with 'experiment:' prefix and auto-revert failed experiments. It will NOT push to remote. Approve git operations for this run? (y/n)"

Once the user approves, the loop runs freely. Without approval, fall back to manual mode (present changes, ask to commit each time).

Confirm, then begin.

## Core Loop

```
SETUP:
  1. Read all in-scope files
  2. Run verify -> extract baseline metric -> log as iteration 0
  3. Git commit current state if uncommitted changes exist

LOOP:
  1. READ STATE
     - git log -n 10 (experiment history)
     - Results log (what worked, what didn't)
     - Current in-scope files

  2. ANALYZE
     - Biggest opportunity for improvement?
     - What hasn't been tried?
     - Near-misses from previous iterations that could be combined?

  3. CHANGE (ONE only)
     - Exactly ONE focused modification
     - Atomic: if it breaks, you know why

  4. COMMIT
     - git add changed files
     - git commit -m "experiment: {description}"

  5. VERIFY
     - Run verify command
     - Extract metric value
     - Calculate delta from best-so-far

  6. GUARD (if set)
     - Run guard command
     - If fails: attempt rework (max 2 tries)
     - If still fails: revert

  7. CONFIDENCE CHECK (after 3+ iterations)
     - Calculate noise floor from metric variance
     - If delta < noise floor: NOISE, revert even if "improved"
     - If delta > 2x noise floor: REAL, keep with confidence
     (Borrowed from pi-autoresearch)

  8. DECIDE
     - KEEP: metric improved AND guard passed AND confidence OK
       -> Update best. Reset no-improvement counter.
     - REVERT: metric worse OR guard failed OR within noise
       -> git revert HEAD. Increment no-improvement counter.
     - CRASH: verify failed to run
       -> Fix attempt (max 3). If still broken: revert + skip.

  9. LOG
     Append to autoresearch-results.tsv in output dir:
     iteration  commit  metric  delta  status  description

  10. STOP CONDITIONS
      - consecutive_at_target >= 3 -> VALIDATOR GATE (step 10b), then CONVERGED
      - consecutive_no_improvement >= 3 -> PLATEAUED
        (When stuck: re-read everything, combine near-misses, try radical change)
      - iteration > max_iterations -> SAFETY CAP

  10b. VALIDATOR GATE (separate model judges "goal met?")
      The worker grading its own metric is a conflict of interest - a loop can
      drive the number to target while missing the actual goal (gamed metric,
      Goodhart, narrow optimization). Before declaring CONVERGED, spawn a FRESH
      subagent with NO edit history and NO stake in the result. Give it:
        - the original GOAL statement (verbatim)
        - the cumulative diff of all KEPT experiments (git diff baseline..HEAD)
        - baseline metric -> final metric
        - the verify command + extraction pattern
      Ask it ONE question: "Is the stated goal genuinely met, or did the loop
      move the metric while missing the goal? Answer met=true/false + one line why."
      - met=true  -> CONVERGED, proceed to final report.
      - met=false -> DO NOT converge. Log the validator's reason as a new
        constraint, reset consecutive_at_target to 0, continue the loop with the
        reason fed into ANALYZE. The metric was a proxy; the goal is the target.
      This is the GOAP validator-model pattern (Claude Code or Codex /goal). It fires
      at the convergence checkpoint by default. `--validator strict` fires it on
      every KEEP decision instead (more expensive, catches drift earlier).

  11. PROGRESS (every 10 iterations)
      Print: baseline, current best, kept/reverted ratio

  12. REPEAT
```

## 8 Critical Rules

1. **Loop until done.** Unbounded: forever. Bounded: N times then summarize.
2. **Read before write.** Understand full context before modifying.
3. **One change per iteration.** Atomic. If it breaks, you know why.
4. **Mechanical verification only.** No subjective "looks good." Numbers only.
5. **Automatic rollback.** Failed changes revert instantly via git.
6. **Simplicity wins.** Equal results + less code = KEEP.
7. **Git is memory.** `experiment:` prefix commits. Read git log before each iteration.
8. **When stuck, think harder.** Re-read, combine near-misses, try radical changes.

## Crash Recovery

| Failure | Response |
|---|---|
| Syntax error | Fix immediately, don't count as iteration |
| Runtime error from verify | Fix attempt (max 3), then skip |
| Resource exhaustion | Revert, try smaller variant |
| Hang / infinite loop | Kill after 60s, revert |
| External dependency down | Skip, log, different approach |

## Subcommand: fix

Auto-detect broken state, fix one error at a time until zero remain.

```
1. DETECT: Run check commands (auto-detect from project files)
   - npm test / pytest / go test (tests)
   - tsc --noEmit / mypy (types)
   - eslint / ruff (lint)
   - npm run build / cargo build (build)

2. COUNT: Total errors = metric (lower is better)

3. LOOP: ONE fix per iteration
   - Pick highest-priority (build > type > test > lint)
   - Fix ONE error
   - Commit
   - Re-count errors
   - Count decreased AND no new errors: KEEP
   - Count same or increased: REVERT
   - Count = 0: STOP (even in unbounded mode)

4. Guard: existing passing tests must still pass
```

## Subcommand: debug

Scientific method bug hunting with hypothesis tracking.

```
1. GATHER symptoms (error messages, stack traces, repro steps)

2. RECON: Map error surface (files, functions, inputs)

3. LOOP (one hypothesis per iteration):
   a. HYPOTHESIZE: specific, testable, falsifiable
      "I believe {X} because {evidence}. This predicts {Y}."
   
   b. TEST: ONE experiment (add log, write test, change value)
   
   c. CLASSIFY:
      - CONFIRMED (bug found)
      - DISPROVEN (not the cause - equally valuable)
      - INCONCLUSIVE (need more evidence)
   
   d. LOG hypothesis + result + evidence
   
   e. 3+ disproven in same area: move elsewhere
   
   f. TECHNIQUES:
      - Binary search (narrow scope)
      - Differential debugging (what changed?)
      - Minimal reproduction (simplify)
      - Trace execution (follow data)
      - Pattern search (where else?)
      - Working backwards (error -> source)

4. OUTPUT: Confirmed bugs with evidence + repro steps
5. CHAIN: --fix flag auto-chains to fix subcommand
```

## Results Format

`autoresearch-results.tsv` in the output directory (`./autoresearch/{date}-{goal-slug}/`):

```
iteration	commit	metric	delta	status	description
0	a1b2c3d	85.2	0.0	baseline	initial state
1	b2c3d4e	87.1	+1.9	keep	add tests for auth edge cases
2	-	86.5	-0.6	discard	refactor helpers (broke 2 tests)
3	c3d4e5f	88.3	+1.2	keep	add error handling tests
```

## Final Report

```
AUTORESEARCH COMPLETE
Goal: {goal}
Baseline: {baseline} -> Final: {best} ({delta_pct}%)
Iterations: {total} ({kept} kept, {reverted} reverted, {skipped} skipped)
Stop reason: {CONVERGED | PLATEAUED | SAFETY_CAP | USER_INTERRUPT}
Results: autoresearch-results.tsv
```

## Integration

- After completion: run an independent code review in a fresh context
- All experiments committed with `experiment:` prefix for clean git history
