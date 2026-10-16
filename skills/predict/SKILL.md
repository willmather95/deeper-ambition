---
name: predict
description: Two-mode pre-mortem failure-mode analysis. Code targets get the 5-expert panel (Architect, Security, Performance, Reliability, Devil's Advocate). Plan/launch/hire/pricing/strategy targets get the Klein narrative premortem (assume failure, parallel agents per failure reason, synthesis with hidden assumption + revised plan). Use when the user says "predict", "pre-mortem", "premortem", "what could go wrong", "what could kill this", "stress-test this plan", "poke holes", "where will this break", "devil's advocate this", "am I missing something", "before we launch", "before we hire", "before we pivot", "before we change pricing", "find the blind spots", "risk analysis", or "future-proof this". Auto-routes by target type; force with --mode experts or --mode klein. For adversarial choice between candidates, use reason. Also see reason.
triggers:
  - "predict"
  - "pre-mortem"
  - "premortem"
  - "what could go wrong"
  - "what could kill this"
  - "stress-test"
  - "poke holes"
  - "where will this break"
  - "devil's advocate"
  - "am I missing something"
  - "before we launch"
  - "before we hire"
  - "before we pivot"
user_invocable: true
context: []
chains_to: [reason]
author: willmather95
degrees_of_freedom: medium
delivers: prediction-report
---

# /predict - Two-Mode Pre-Mortem

Stress-test a plan or system BEFORE acting. Two algorithms, picked by target type:

- **Algorithm A: 5-Expert Panel** for code/architecture/system targets. Cold-start specialists in parallel.
- **Algorithm B: Klein Narrative Premortem** for plan/launch/hire/pricing/strategy targets. Frame as already failed, fan out parallel agents per failure reason, synthesize.

Inspired by: [Udit Goenka's /autoresearch:predict](https://github.com/uditgoenka/autoresearch) (5-expert) + [Gary Klein's premortem method](https://hbr.org/2007/09/performing-a-project-premortem). Different psychological mechanisms: experts evaluate, Klein narrates failure backward.

## When to Use

- Before building a major feature
- Before an architecture decision
- Before shipping to production
- Before a pricing or positioning change
- Any time you want to know "what could go wrong"

## Invocation

```
/predict                                         # interactive - what to analyze?
/predict --scope src/api/                        # analyze specific code (auto -> Algorithm A)
/predict --target "our pricing strategy"         # analyze a decision/plan (auto -> Algorithm B)
/predict --mode experts                          # force Algorithm A (5-expert panel)
/predict --mode klein                            # force Algorithm B (Klein narrative)
/predict --mode hybrid                           # force both algorithms, merged synthesis
```

## Setup

### Step 1: Define Target

Ask: "What do you want analyzed?"

Three modes:
- **Code:** specify file paths or globs. Read all files in scope.
- **Plan:** describe the feature, architecture, or strategy in plain language.
- **Hybrid:** code context + a description of what's being changed.

### Step 2: Prepare Context

For code targets: read all in-scope files, summarize the architecture.
For plan targets: capture the description as-is. Also gather: who is it for, what does success look like, what's irreversible about it. The Klein method needs these three to ground failure scenarios.
For hybrid: both.

### Step 2.5: Route by Target Type

| Target type | Algorithm | Why |
|-------------|-----------|-----|
| Code (file paths, repo, system) | **A: 5-Expert Panel** | Technical lenses catch bugs, attack vectors, perf issues, reliability gaps |
| Plan / Launch / Pricing / Hire / Strategy / Decision | **B: Klein Narrative Premortem** | Failure-already-happened framing produces specific business/launch failure modes, not generic risk lists |
| Hybrid (code + business plan) | **Both A and B**, merged in synthesis | Full coverage |

Force a mode (overrides auto-routing):
- `/predict --mode experts` -> Algorithm A regardless of target
- `/predict --mode klein` -> Algorithm B regardless of target
- `/predict --mode hybrid` -> run both A and B, merge in synthesis
- `/predict` (default) -> auto-route by target type

## Algorithm A: 5-Expert Panel (Code Targets)

### Spawn 5 Experts (PARALLEL, cold-start)

Each expert is a separate subagent via the Agent tool:
- model: `sonnet`
- run_in_background: `true` (all 5 launch in parallel for speed)
- Wait for all 5 to complete before synthesizing
- Each gets the SAME context but a DIFFERENT analysis lens

#### Expert 1: ARCHITECT

```
You are a senior software architect reviewing this code/plan.

CONTEXT:
{target_context}

Analyze for:
1. Structural weaknesses and coupling issues
2. Abstraction leaks and wrong boundaries
3. Extensibility gaps (what's hard to change later?)
4. Patterns that will cause pain at 10x scale
5. Missing components or layers

Rules:
- Be specific. Cite file paths and line numbers where possible.
- No theoretical concerns. Only things that will actually cause problems.
- For each finding: what breaks, when, and how hard is it to fix later.
- Severity: critical / high / medium / low
```

#### Expert 2: SECURITY

```
You are a security analyst reviewing this code/plan.

CONTEXT:
{target_context}

Analyze for:
1. Attack surface and entry points
2. Authentication and authorization gaps
3. Injection vectors (SQL, XSS, command, template)
4. Data exposure risks (PII, secrets, tokens)
5. OWASP Top 10 violations

Rules:
- Every finding must include: what an attacker could do, how, and severity.
- No theoretical threats. Only exploitable issues.
- Severity: critical / high / medium / low
```

#### Expert 3: PERFORMANCE

```
You are a performance engineer reviewing this code/plan.

CONTEXT:
{target_context}

Analyze for:
1. N+1 queries and unbounded database operations
2. Missing pagination or limits on collections
3. Memory leaks and unbounded growth
4. Blocking I/O on hot paths
5. Missing caching opportunities
6. Cold start costs and initialization overhead

Rules:
- Quantify impact where possible (O(n) vs O(n^2), cache hit rates, etc.)
- No theoretical bottlenecks. Only things that will actually slow down.
- Severity: critical / high / medium / low
```

#### Expert 4: RELIABILITY

```
You are a reliability engineer reviewing this code/plan.

CONTEXT:
{target_context}

Analyze for:
1. Single points of failure
2. Missing or inadequate error handling
3. Silent failures (errors swallowed, data lost)
4. Missing retries, circuit breakers, timeouts
5. Monitoring and observability gaps
6. No graceful degradation path

Rules:
- For each issue: what fails, how you'd know (or wouldn't), blast radius.
- No theoretical failures. Only realistic failure modes.
- Severity: critical / high / medium / low
```

#### Expert 5: DEVIL'S ADVOCATE

```
You are a devil's advocate. Your job is to find what everyone else missed.

CONTEXT:
{target_context}

Your mandate:
1. Challenge fundamental assumptions
2. Question whether the approach is even correct
3. Find the uncomfortable truth nobody wants to hear
4. Identify hidden dependencies or assumptions
5. Ask: "What if we're solving the wrong problem?"

Rules:
- Be contrarian but grounded. Back up every challenge with reasoning.
- Don't just disagree - propose what's actually true instead.
- The best devil's advocate findings are the ones that change the plan.
- Severity: critical / high / medium / low
```

### Synthesize

After all 5 experts return, merge into a ranked report.

For each finding across all experts:

1. **Deduplicate:** If multiple experts found the same issue, merge into one finding with higher consensus.
2. **Score:** severity x consensus (5/5 consensus on a HIGH issue outranks 1/5 on a CRITICAL)
3. **Categorize:** risk (something bad) vs opportunity (something could be better)
4. **Action:** what to do about it (fix, investigate, debate, accept)

### Rank findings:

**Sort by:** severity * consensus score (highest first)

For consensus scoring:
- 5/5 experts: DEFINITE (consensus multiplier: 2.0)
- 4/5 experts: STRONG (1.5)
- 3/5 experts: MODERATE (1.0)
- 2/5 experts: SPLIT (0.7)
- 1/5 experts: SPECULATIVE (0.5)

For severity:
- CRITICAL: 4
- HIGH: 3
- MEDIUM: 2
- LOW: 1

Final score = severity_score * consensus_multiplier

## Algorithm B: Klein Narrative Premortem (Plan/Decision Targets)

Different psychological mechanism from Algorithm A. Instead of asking "what could go wrong?" (which produces hedged answers), assume the plan **already failed** and work backward (which produces specific narrative failures). This is Gary Klein's premortem method.

Use this for: product launches, pivots, hires, pricing changes, repositioning, autonomous system rollouts, big-spend commitments. Anything where the cost of being wrong is high and you can still change course.

### Step B1: Set the Premortem Frame

Anchor the frame explicitly before generating anything:

> "It is 6 months from now. [The plan/launch/decision] has failed. It's done. We're looking back and trying to understand what went wrong."

This shift from "evaluate this plan" to "explain why this died" is the core mechanism. Skip this step and the analysis collapses back to polite risk assessment.

### Step B2: Generate Failure Reasons (Single-Pass)

Run a comprehensive single-pass failure scan. No prescribed categories - just the core Klein method.

Self-prompt:
> "This plan failed 6 months from now. List every genuine reason it could have died. Be specific to THIS plan. Ground every reason in the actual details. Don't pad with weak reasons. Don't stop early if there are more."

Each reason must be:
- Specific to this plan (not generic risk advice that applies to anything)
- Grounded in actual details from the target context
- A genuine threat (not edge cases or minor inconveniences)

Output: 4-9 failure reasons, each in 1-2 sentences. The number is whatever is real for this plan - don't force a quota.

### Step B3: Deep-Dive Agents (PARALLEL, one per failure reason)

Spawn one Agent per failure reason via the Agent tool. All in parallel. Cold-start each (no shared context between agents). Cap at 7 parallel agents - if Step B2 produced more than 7 reasons, keep the 7 most specific (drop generic/overlapping ones) before spawning.

Sub-agent prompt template:

```
You are an investigator in a premortem analysis. You've been assigned ONE specific failure reason to analyze in depth.

THE PLAN:
---
{full target context: what it is, who it's for, what success looks like}
---

PREMORTEM FRAME: It is 6 months from now. This plan has failed.

YOUR ASSIGNED FAILURE REASON: {one failure reason from Step B2}

Write the story of how this specific failure played out. Use details from the plan. Make it feel like a case study of something that actually happened, not a hypothetical risk.

Output (under 300 words total):

1. THE FAILURE STORY (2-3 paragraphs): Specific narrative of how this played out. Name moments where things went wrong and why.

2. THE UNDERLYING ASSUMPTION (1 sentence): The thing the user was taking for granted that made this failure possible.

3. EARLY WARNING SIGNS (1-2 items): Concrete, observable signals that this failure mode is starting to play out. Things you can see or measure, not vague feelings.

Be direct. Don't hedge. Don't sugarcoat.
```

### Step B4: Klein Synthesis

After all agents return, produce the synthesis. The synthesis is the product - most users will read it and skim the cards.

```
PREMORTEM REPORT (Klein mode)
Target: {plan description}
Failure reasons analyzed: {N}

MOST LIKELY FAILURE: {which scenario is most probable given the plan, why - one paragraph}

MOST DANGEROUS FAILURE: {which scenario causes most damage if it hits, even if less likely - one paragraph}

THE HIDDEN ASSUMPTION: {single biggest unquestioned assumption across all analyses - one sentence. This is often where the real value lives: the thing so obvious to the user they forgot it was an assumption.}

REVISED PLAN: {3-5 concrete changes, each mapped to a specific failure mode}
1. {revision} -> addresses: {failure reason}
2. ...

PRE-LAUNCH CHECKLIST: {3-5 items to verify, test, or install before executing}
- [ ] {specific, observable check}
- [ ] ...

FAILURE CARDS:
[card 1] {failure reason} -> {failure story} -> {assumption} -> {early warning signs}
[card 2] ...
```

Each revision must be **concrete** ("run a $47 pilot with 20 people first") not **vague** ("consider testing your pricing"). Every revision should be something the user can do this week.

If MOST DANGEROUS FAILURE is unaddressed in the revised plan, **STOP** and check with the user before proceeding. That's a signal the plan needs to change, not just be tweaked.

## Output

Output format depends on which algorithm ran. Hybrid mode emits both, then a merged section.

### Algorithm A Output (5-Expert)

```
PREDICT REPORT (Algorithm A: 5-Expert)
Target: {target description}
Experts: 5 (Architect, Security, Performance, Reliability, Devil's Advocate)

TOP RISKS (ranked by severity x consensus):
1. [{severity}] [{N}/5 consensus] {finding}
   Action: {what to do}
   Flagged by: {expert names}
...

OPPORTUNITIES (things that could be better):
1. {opportunity} - {how to capture it}

CONSENSUS (all 5 agree):
- {finding}

DISSENTING VIEWS (1-2 experts only):
- {contrarian finding} - flagged by {expert}

CHAIN RECOMMENDATIONS:
- {finding} -> /reason (needs debate)
```

### Algorithm B Output (Klein)

```
PREMORTEM REPORT (Algorithm B: Klein)
Target: {plan description}
Failure reasons analyzed: {N}

MOST LIKELY FAILURE: {paragraph}
MOST DANGEROUS FAILURE: {paragraph}
THE HIDDEN ASSUMPTION: {one sentence}

REVISED PLAN:
1. {concrete revision} -> addresses: {failure reason}
...

PRE-LAUNCH CHECKLIST:
- [ ] {observable check}
...

FAILURE CARDS:
[1] {reason} -> {story} -> {assumption} -> {early warning}
...
```

### Hybrid Output

Run A and B in parallel. Output A's report, then B's report, then a brief MERGED section that surfaces overlap (failures both algorithms identified) and gaps (failures one caught and the other missed - dissent is signal).

## Chain Targets

**Algorithm A chains (post-output routing of findings):**
- Contentious findings (2-3/5 consensus) -> /reason (debate topic)

**Algorithm B chains:**
- Klein output -> revised plan -> execute (the revised plan IS the deliverable)
- If MOST DANGEROUS FAILURE unaddressed in revised plan -> STOP + check with the user
- If THE HIDDEN ASSUMPTION is contestable -> /reason (debate the assumption before committing)
- Klein after a go/no-go decision -> final stress test before launching

## Key Rules

- **Cold-start isolation.** Every spawned agent is fresh. No shared context.
- **Parallel execution.** Algorithm A's 5 experts run simultaneously. Algorithm B's per-failure agents also run simultaneously.
- **Auto-route by default; force with --mode.** Don't make the user think about which algorithm to use unless they want to.
- **No em dashes.** Use `-` instead.
- **Dissenting views are features, not bugs.** Algorithm A: 1/5 findings are often the most valuable. Algorithm B: a single agent that surfaces the hidden assumption can be worth more than the other four combined.
- **Klein revisions must be concrete.** Not "consider testing pricing" - "run a $47 pilot with 20 people first." If a revision can't be done this week, rewrite it.
- **Klein needs the frame.** Skip Step B1 (the "it has failed, look back" anchor) and the analysis collapses back to polite risk assessment. The frame IS the mechanism.
