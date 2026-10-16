---
name: reason
description: Adversarial refinement for subjective decisions where no objective metric exists. A blind judge panel converges competing candidates through isolated multi-agent debate, with each judge unaware of the others. Use when the user says "reason", "help me decide", "should we", "is this the right call", "gut check this", "adversarial input", "blind judge", "I'm torn between", "which is better", "stress-test this decision", or faces a fork with no clear winner. For failure-mode analysis on a chosen path, use predict instead. Also see predict.
triggers:
  - "reason"
  - "help me decide"
  - "should we"
  - "is this the right call"
  - "gut check this"
  - "I'm torn between"
  - "which is better"
  - "stress-test this decision"
user_invocable: true
context: []
chains_to: [predict]
author: willmather95
degrees_of_freedom: low
delivers: converged-recommendation
---

# /reason - Adversarial Refinement

Converge on the best answer for subjective decisions where no objective metric exists. The blind judge panel IS the fitness function.

Inspired by: [Udit Goenka's /autoresearch:reason](https://github.com/uditgoenka/autoresearch), extended with cold-start isolation and dual-scoring.

## When to Use

- Architecture decisions ("event sourcing vs CRUD?")
- Pricing strategy ("$2,497 or $3,497?")
- Positioning ("lead with guarantee or lead with results?")
- Content quality ("which version is better?")
- Product direction ("build X or Y first?")
- Any decision where reasonable people disagree

## Invocation

```
/reason                                          # interactive setup
/reason --task "..." --domain software           # inline
/reason --iterations 8 --judges 3                # bounded
/reason --chain predict                          # chain to predict after convergence
```

## Setup

### Step 1: Capture Task
Ask: "What needs deciding?" Accept the question or decision in plain language.

### Step 2: Set Domain
Options: `software` | `product` | `business` | `security` | `research` | `content`
Domain shapes the expertise of generated candidates.

### Step 3: Set Parameters
- **Iterations:** default unlimited (converges on 3 consecutive wins)
- **Judges:** default 3 (must be odd - 3, 5, or 7)
- **Convergence threshold:** default 3 consecutive wins by same candidate
- **Max iterations:** default 10 (safety cap)

## Algorithm

### Generate Candidate A

Spawn a cold-start subagent:
- Agent tool, model: `sonnet`
- Prompt: "You are an expert in {domain}. Answer this question thoroughly: {task}. Be specific, concrete, and opinionated. Back up every claim with reasoning."
- Capture output as Candidate A

### Main Loop

Each iteration has 4 stages. **Every agent is a cold-start fresh invocation.** No shared session. No history bleed.

#### Stage 1: CRITIC

Spawn cold-start subagent (sonnet):
- Receives ONLY Candidate A (not the task history or previous rounds)
- Prompt:
  ```
  You are a critical reviewer. Your job is to find every weakness in this answer.
  
  ANSWER TO CRITIQUE:
  {candidate_a}
  
  Find:
  1. Logical gaps or unsupported claims
  2. Missing considerations
  3. Questionable assumptions
  4. Stronger alternatives not considered
  5. Practical problems with implementation
  
  Steelman first (acknowledge strengths), then strawman (attack weaknesses).
  Be ruthless but fair. Every criticism must be specific and actionable.
  ```

#### Stage 2: AUTHOR-B

Spawn cold-start subagent (sonnet):
- Receives: original task + Candidate A + critic's attacks
- Prompt:
  ```
  You are an expert responding to criticism of an answer.
  
  ORIGINAL QUESTION: {task}
  CURRENT ANSWER: {candidate_a}
  CRITICISMS: {critic_output}
  
  Write a BETTER answer that:
  1. Addresses every valid criticism
  2. Keeps the strengths of the original
  3. Adds considerations the original missed
  
  Don't just patch - write fresh. If the criticisms reveal a fundamentally
  better approach, take it.
  ```

#### Stage 3: SYNTHESIZER

Spawn cold-start subagent (sonnet):
- Receives: Candidate A + Candidate B + critic's attacks
- Prompt:
  ```
  You have two answers to the same question and a list of criticisms.
  
  ANSWER X: {candidate_a}
  ANSWER Y: {candidate_b}
  CRITICISMS: {critic_output}
  
  Merge the best of both into a single superior answer.
  Keep what's strong. Cut what's weak. Resolve contradictions.
  The result should be strictly better than either input.
  ```
- Output = Candidate C

#### Stage 4: BLIND JUDGE PANEL

Spawn {judge_count} subagents in PARALLEL (haiku):

**Critical invariant:** Labels are RANDOMIZED per judge. Some judges see Candidate A as "X" and C as "Y". Others see the reverse. This prevents position bias.

Each judge prompt:
```
You are a blind judge. Two answers to the same question are shown below.
You do NOT know which is the original and which is the revision.

QUESTION: {task}

ANSWER X:
{randomized_candidate_1}

ANSWER Y:
{randomized_candidate_2}

Pick the better answer. Consider:
- Completeness (covers all important aspects)
- Specificity (concrete, not vague)
- Reasoning quality (claims backed by logic)
- Practicality (actionable, not theoretical)

Output EXACTLY:
WINNER: X or Y
CONFIDENCE: high/medium/low
REASON: {2-3 sentences explaining why}
```

**Tally votes.** Majority wins. Map back from X/Y labels to actual candidates.

#### Decision

- If Candidate C wins: C becomes new A. Reset consecutive_A_wins to 0. Increment consecutive_C_wins.
- If Candidate A wins: A stays. Increment consecutive_A_wins. Reset consecutive_C_wins to 0.
- If consecutive_A_wins >= convergence_threshold: **CONVERGED.** Stop.
- If consecutive_C_wins >= 3 AND improvements are narrowing (critic attacks are execution-level, not strategic): **STABILIZED.** Stop. The core recommendation is stable even though refinements keep winning. Log the distinction.
- If iteration >= max_iterations: **SAFETY CAP.** Report best so far.

**STABILIZED vs CONVERGED:** CONVERGED means the answer stopped improving (A held). STABILIZED means the direction stopped changing but copy-level refinements keep winning. Both are valid stop reasons. STABILIZED saves 30+ subagent calls on diminishing returns.

### Log Each Round

```
Round {N}: {A_WINS | C_WINS} (votes: {tally})
Candidate A: {first 50 chars}...
Candidate C: {first 50 chars}...
Judge 1: {winner} ({confidence}) - {reason}
Judge 2: {winner} ({confidence}) - {reason}
Judge 3: {winner} ({confidence}) - {reason}
```

## Output

Create directory `./reason/{date}-{slug}/` (relative to the current working directory):

### converged-answer.md
The final winning recommendation. Clean, standalone, ready to act on.

### lineage.md
Evolution across rounds:
```
Round 1: A -> C (C won 2-1)
Round 2: C -> D (D won 3-0)
Round 3: D held (D won 2-1) [1/3 toward convergence]
Round 4: D held (D won 3-0) [2/3 toward convergence]
Round 5: D held (D won 2-1) [CONVERGED]
```

### judge-transcripts.md
All judge decisions with full justifications. Useful for understanding WHY one answer beat another.

### reason-results.tsv
```
round	winner	votes_for_a	votes_for_c	description
1	C	1	2	Added implementation timeline
2	C	0	3	Restructured around constraints
3	A	2	1	Held - refinement was marginal
```

## Final Report

```
REASON COMPLETE
Task: {task}
Domain: {domain}
Rounds: {N}
Stop reason: CONVERGED | STABILIZED | SAFETY CAP
Winner: converged-answer.md
Lineage: lineage.md ({N} rounds, {improvements} improvements)
```

## Chain Targets

- `/reason --chain predict` - converge then stress-test with 5 experts

## Key Rules

- **Cold-start isolation is non-negotiable.** Every agent spawns fresh. No shared context.
- **Judges are blind.** Randomized X/Y labels. Never A/B/C. Prevents anchoring.
- **Odd judge count.** No ties. 3, 5, or 7.
- **Convergence means stability.** 3 consecutive wins = the answer isn't improving anymore.
- **No em dashes in output.** Use `-` instead.
